Use the following to create a Mino S3 backed by rook-ceph storage

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  # This name uniquely identifies the Deployment
  name: minio
  namespace: minio
spec:
  selector:
    matchLabels:
      app: minio # has to match .spec.template.metadata.labels
  strategy:
    # Specifies the strategy used to replace old Pods by new ones
    # Refer: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy
    type: Recreate
  template:
    metadata:
      labels:
        # This label is used as a selector in Service definition
        app: minio
    spec:
      # Volumes used by this deployment
      volumes:
      - name: data
        # This volume is based on PVC
        persistentVolumeClaim:
          # Name of the PVC created earlier
          claimName: minio-pv-claim
      containers:
      - name: minio
        # Volume mounts for this container
        volumeMounts:
        # Volume 'data' is mounted to path '/data'
        - name: data 
          mountPath: "/data"
        # Pulls the lastest Minio image from Docker Hub
        image: minio/minio:RELEASE.2020-05-01T22-19-14Z
        args:
        - server
        - /data
        env:
        # MinIO access key and secret key
        - name: MINIO_ACCESS_KEY
          value: "minio"
        - name: MINIO_SECRET_KEY
          value: "minio123"
        ports:
        - containerPort: 9000
        # Readiness probe detects situations when MinIO server instance
        # is not ready to accept traffic. Kubernetes doesn't forward
        # traffic to the pod while readiness checks fail.
        readinessProbe:
          httpGet:
            path: /minio/health/ready
            port: 9000
          initialDelaySeconds: 120
          periodSeconds: 20
        # Liveness probe detects situations where MinIO server instance
        # is not working properly and needs restart. Kubernetes automatically
        # restarts the pods if liveness checks fail.
        livenessProbe:
          httpGet:
            path: /minio/health/live
            port: 9000
          initialDelaySeconds: 120
          periodSeconds: 20
--------
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  finalizers:
  - kubernetes.io/pvc-protection
  labels:
    app: minio
  name: minio-pv-claim
  namespace: minio
spec:
  storageClassName: rook-ceph-block
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi 
----
apiVersion: v1
kind: Service
metadata:
  # This name uniquely identifies the service
  name: minio-service
  namespace: minio
spec:
  type: LoadBalancer
  ports:
    - port: 9000
      targetPort: 9000
      protocol: TCP
  selector:
    # Looks for labels `app:minio` in the namespace and applies the spec
    app: minio
---
 kubectl port-forward minio-64b7c649f9-9xf5x --address 0.0.0.0 7000:9000 --namespace minio
 
 ---
 Thats it 
 http://10.131.232.223:7000/minio/
```

After this port-forward so that you can access Mino externally

`kubectl port-forward minio-64b7c649f9-9xf5x --address 0.0.0.0 7000:9000 --namespace minio`

Alternately if you have HAProxy IngressController installed in your K8s cluster you can create an Ingress and expose it

```
HOST=minio.10.131.232.223.nip.io  
kubectl create -f - <<EOF  
apiVersion: extensions/v1beta1  
kind: Ingress  
metadata:  
  name: minio  
  namespace: minio  
spec:  
  rules:  
  - host: $HOST  
    http:  
      paths:  
      - backend:  
          serviceName: minio-service  
          servicePort: 9000  
        path: /  
EOF
```

Note:I edited the minio service from type LB to CluserIP and removed the NodePort. This may not be really needed

```sh
[root@green--1 ~]# kubectl  -n minio describe service minio-service  
Name:                     minio-service  
Namespace:                minio  
Labels:                   <none>  
Annotations:              kubectl.kubernetes.io/last-applied-configuration:  
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"minio-service","namespace":"minio"},"spec":{"ports":[{"port":9000...  
Selector:                 app=minio  
Type:                     LoadBalancer --> changed this to ClusterIP  
IP:                       10.97.9.200  
Port:                     <unset>  9000/TCP  
TargetPort:               9000/TCP  
NodePort:                 <unset>  31675/TCP --> removed this  
Endpoints:                10.244.2.21:9000  
Session Affinity:         None  
External Traffic Policy:  Cluster  
Events:                   <none>
```

f you are using an Ingress Controller you can create an Ingress like below

```sh
# kubectl -n minio describe service minio-serviceName: minio-service Namespace: minio Labels: Annotations: kubectl.kubernetes.io/last-applied-configuration: {“apiVersion”:”v1",”kind”:”Service”,”metadata”:{“annotations”:{},”name”:”minio-service”,”namespace”:”minio”},”spec”:{“ports”:[{“port”:9000… Selector: app=minio Type: LoadBalancer → changed this to ClusterIP IP: 10.97.9.200 Port: 9000/TCP TargetPort: 9000/TCP NodePort: 31675/TCP → removed this Endpoints: 10.244.2.21:9000 Session Affinity: None External Traffic Policy: Cluster Events:  
HOST=minio.10.131.232.223.nip.io kubectl create -f — << EOF   
apiVersion: extensions/v1beta1  
kind: Ingress   
metadata:   
  name: minio  
  namespace: minio  
spec:  
 rules:  
 - host: $HOST  
   http:  
      paths:  
      - backend:   
         serviceName: minio-service  
         servicePort: 9000   
status:  
 loadBalancer:  
  ingress:  
  - {}  
EOF
```