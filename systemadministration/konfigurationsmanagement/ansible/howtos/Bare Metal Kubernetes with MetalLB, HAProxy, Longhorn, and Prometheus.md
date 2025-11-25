A few years back I created a [blog post](https://fdeantoni.medium.com/running-rook-with-k3s-5e2c79159eaf) about creating a Kubernetes cluster with Rook/Ceph. My main goal at that time was to see if we could use Kubernetes as a clustering solution for our software stack on the edge. Although k3s worked great, everything else still had some rough edges.

For one, ARM support was still lacking for many dependencies, and [MetalLB](https://github.com/metallb/metallb) was still in its early days. HAProxy was not available for Kubernetes so the only choice was to use Traefik with [k3s](https://k3s.io) (which does not mean that Traefik is not a good option, it is just that we use HAProxy extensively and know it works well with our stack).

Things have moved on fast since then though! Installing MetalLB is dead simple, and HAProxy now also has [support for Kubernetes](https://www.haproxy.com/documentation/kubernetes/latest/). Moreover, [t](https://rancher.com/docs/k3s/latest/en/)he people at Rancher have developed [Longhorn](https://longhorn.io) which is an excellent alternative to Rook/Ceph.

So let’s give everything a spin and see how it all works out. We will deploy k3s with MetalLB, HAProxy, Prometheus, and a test echo server on a 7 node ARM based cluster.
# Prepare Cluster Nodes

We will create a 7 node cluster having 3 nodes as controllers and 4 nodes as workers. On each node we will install Ubuntu Server 20.04 LTS, and to help us install everything else we will be using Ansible.

We start with the following:

```sh
+----------+--------------+------------+  
| Hostname |      IP      |    Role    |  
+----------+--------------+------------+  
| node1    | 10.211.55.12 | controller |  
| node2    | 10.211.55.13 | controller |  
| node3    | 10.211.55.14 | controller |  
| node4    | 10.211.55.15 | worker     |  
| node5    | 10.211.55.16 | worker     |  
| node6    | 10.211.55.17 | worker     |  
| node7    | 10.211.55.18 | worker     |  
+----------+--------------+------------+
```

For convenience create a user `admin` on each node:

r convenience create a user `admin` on each node:

```sh
sudo groupadd -g 1000 admin  useradd -u 1000 -d /home/admin -s /bin/bash -m -g admin admin
echo my-secret-pw | sudo chpasswd 
```
For password I used `my-secret-pw` but you should use whatever works for you. To make your life easy do keep it the same for each host. After running through this guide you can change it or disable it completely if you want.

# Node1 Cluster Admin

We will use `node1` to bootstrap our cluster. Log into `node1` as user `admin` and install Ansible on it:

```sh
sudo apt-add-repository ppa:ansible/ansible  
sudo apt-get update  
sudo apt-get install ansible
```

Next, if none exists yet, create a key-pair for user `admin` on `node1` that you can use to send to all the other nodes for login:

`ssh-keygen`

Now use `ssh-copy-id` to copy over the keys of user `admin` to all the other nodes:

```sh
ssh-copy-id admin@node2  
ssh-copy-id admin@node3  
ssh-copy-id admin@node4  
ssh-copy-id admin@node5  
ssh-copy-id admin@node6  
ssh-copy-id admin@node7
```

Now create a file called `hosts` with the following content:

```ini
[control]
node1  ansible_connection=local
node2
node3[workers]
node4
node5
node6
node7[nodes:children]
control  
workers
```

Let’s test everything out:

`ansible -i hosts nodes -m ping`

Ansible on `node1` should be able to ping all the other nodes without problems. With Ansible working we can now complete our initial preparation of the nodes for k3s installation.

First remove some software we will not need to save some resources:

```sh
$ ansible -i hosts nodes \
    -b -K -m shell \
    -a "snap remove lxd && snap remove core18 && snap remove snapd"
```

For good measure update the software on each node to the latest:

```
$ ansible -i hosts nodes \
     -b -K -m apt \
     -a "upgrade=yes update_cache=yes"
```
# Install k3s

Our nodes are now ready for installation of k3s. On `node1` install k3s with the following:

```sh
curl -sfL [https://get.k3s.io](https://get.k3s.io) | K3S_TOKEN=my_super_secret \  
     sh -s - server --cluster-init \
                    --disable servicelb \
                    --disable traefik
```

This will install a kubernetes controller without the k3s provided `servicelb` and `traefik`. For load-balancer we will instead use MetalLB and for Ingress we will use HAProxy instead of Traefik. For `K3S_TOKEN` I used a not so secret `my_super_secret`. This token will be used later to connect the other nodes to the Kubernetes cluster, so change the token as you see fit.

With `node1` now running the first controller, we can copy over the Kubnernetes configuration file over to `admin`'s home directory so that we do not need to use `sudo` to run `kubectl`:

```sh
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/k3s-config
sudo chown $USER: ~/.kube/k3s-config
export KUBECONFIG=~/.kube/k3s-config
```

You can add `export KUBECONFIG=~/.kube/k3s-config` to `~/.profile` so that the environment variable is set with every new shell.

With that out of the way we can install the remaining controllers on `node2` and `node3`. We can do so using Ansible:

```sh
ansible -i hosts node2,node3 -b -K \
     -m shell \
     -a "curl -sfL [https://get.k3s.io](https://get.k3s.io) | K3S_TOKEN=my_super_secret sh -s - server --server [https://10.211.55.12:6443](https://10.211.55.12:6443) --disable servicelb --disable traefik"
```

As you can see we use the same `K3S_TOKEN` as we defined earlier on `node1`. For `server` we now use a URL that points to the controller on `node1`. As. with `node1` we also disable `servicelb` and `traefik`.

We can now install the workers:

```sh
$ ansible -i hosts workers -b -K \  
    -m shell \  
    -a "curl -sfL [https://get.k3s.io](https://get.k3s.io) | K3S_URL=[https://10.211.55.12:6443](https://10.211.55.12:6443) K3S_TOKEN=my_super_secret sh -"
```

Check if everything is running as expected:

```
$ kubectl get nodes -A
```

You should see all the controllers and all the workers. From the output you will see that the controllers have a role but the workers do not. Let’s fix that by giving the workers a the appropriate role:

```sh
kubectl label nodes node4 kubernetes.io/role=worker  
kubectl label nodes node5 kubernetes.io/role=worker  
kubectl label nodes node6 kubernetes.io/role=worker  
kubectl label nodes node7 kubernetes.io/role=worker
```

o control on what nodes we can deploy what, we will also add another label called `node-type` that we can use in deployment specs:

```sh
kubectl label nodes node4 node-type=worker  
kubectl label nodes node5 node-type=worker  
$ kubectl label nodes node6 node-type=worker  
$ kubectl label nodes node7 node-type=worker
```

You can check all the labels given to all the nodes as follows:

`kubectl get nodes --show-labels`
# Install Helm

With Kubernetes now running on all our nodes, follow the instructions from here: [https://helm.sh/docs/intro/install](https://helm.sh/docs/intro/install/)

I would recommend using the script install method.

# Install MetalLB

With Helm now installed we can use Helm to install MetalLB:

```sh
helm repo add metallb [https://metallb.github.io/metallb](https://metallb.github.io/metallb)
```

With the MetalLB repo added we can install MetalLB, but before we do, create a configurations value file called `metallb-values.yaml` with the following content:

```c
configInline:  
  address-pools:  
   - name: default  
     protocol: layer2  
     addresses:  
     - 10.211.55.240-10.211.55.250
```

The above configuration will instruct MetalLB to use IP range `10.211.55.240` to `250` to services that are marked as `LoadBalancer` types. You should change this of course to values that make sense for your environment.

## Let’s now install MetalLB:

`helm install metallb metallb/metallb -f metallb-values.yaml`

Excellent! MetalLB is now installed. We can now install HAProxy.
# Install HAProxy

Like with MetalLB, we can use Helm to install HAProxy. First add the repo:

```sh
helm repo add haproxytech [https://haproxytech.github.io/helm-charts](https://haproxytech.github.io/helm-charts)
```
Before we install HAProxy however, there is an ongoing issue (`[#222](https://github.com/haproxytech/kubernetes-ingress/issues/222)`) when installing on ARM hardware, you need to use the following command:

```sh
$ helm install haproxy haproxytech/kubernetes-ingress --set defaultBackend.image.repository=gcr.io/google_containers/defaultbackend-arm64
```

If you are using AMD hardware, you can use the following instead:

`helm install haproxy haproxytech/kubernetes-ingress`

After installing HAProxy, you can change the service type from `NodePort` to `LoadBalancer` instead. You can do that by editing the service spec as follows:

`kubectl edit service/haproxy-kubernetes-ingress`

This will open up the HAProxy service spec. Look for the `type` that currently is set to `NodePort`. Change this to `LoadBalancer`. Once changed, save your changes and exit the editor.

# Install Longhorn

We can now install the shared storage service called Longhorn. Before we can install it we need to make sure our cluster has all the [necessary requirements](https://longhorn.io/docs/1.2.2/deploy/install) installed. First install `iscsi` on all the nodes:

```sh
ansible -i hosts nodes -b -K \  
    -m apt \  
    -a "name=open-iscsi state=present"
```

Next install `nfs-common` on all nodes:

```sh
ansible -i hosts nodes -b -K \  
    -m apt \  
    -a "name=nfs-common state=present"
```

Now check if our cluster is ready:

```sh
sudo apt install jq  
curl -sSfL [https://raw.githubusercontent.com/longhorn/longhorn/v1.2.2/scripts/environment_check.sh](https://raw.githubusercontent.com/longhorn/longhorn/v1.2.2/scripts/environment_check.sh) | bash
```

The script will check our cluster if all requirements are satisfied. If something went wrong, check the [Longhorn Install Guide](https://longhorn.io/docs/1.2.2/deploy/install) on how to fix it. If you followed this guide then the script *should* have completed successfully.

Now we are ready to install Longhorn:

```sh
helm repo add longhorn [https://charts.longhorn.io](https://charts.longhorn.io)  
helm repo update  
kubectl create namespace longhorn-system  
helm install longhorn longhorn/longhorn --n longhorn-system
```

You can check the [Longhorn Helm Install](https://longhorn.io/docs/1.2.2/deploy/install/install-with-helm) page for more info.

Check if Longhorn is running properly:

`kubectl -n longhorn-system get pod`

If Longhorn is running properly, go change the `type` of the `longhorn-frontend` from `ClusterIP` to `NodePort`:

`kubectl edit service longhorn-frontend -n longhorn-system`

Check the `longhorn-system` services to see what port number has been assigned for the longhorn-frontend app:

`kubectl get service longhorn-frontend -n longhorn-system`

From the output we can see that `longhorn-frontend` is running on port `32057` on each cluster node. This means we can use any node IP to access it, for example: `[http://10.211.55.12:32057](http://10.211.55.12:32057)`
# Install Prometheus

Now that we have a cluster block device service available via Longhorn, we can deploy Prometheus. Why? Because we will have Prometheus place its database on a the cluster block device so that if Prometheus goes down on one node, it can safely restart on another node without (serious) data loss.

Let’s first create a namespace `monitoring` where we will deploy Prometheus:

`kubectl create namespace monitoring`

Next we need to create a cluster role for Prometheus. Create a file called `prometheus-role.yaml` with the following content:

```
apiVersion: rbac.authorization.k8s.io/v1  
kind: ClusterRole  
metadata:  
  name: prometheus  
rules:  
- apiGroups: [""]  
  resources:  
  - nodes  
  - nodes/proxy  
  - services  
  - endpoints  
  - pods  
  verbs: ["get", "list", "watch"]  
- apiGroups:  
  - extensions  
  resources:  
  - ingresses  
  verbs: ["get", "list", "watch"]  
- nonResourceURLs: ["/metrics"]  
  verbs: ["get"]  
---  
apiVersion: rbac.authorization.k8s.io/v1  
kind: ClusterRoleBinding  
metadata:  
  name: prometheus  
roleRef:  
  apiGroup: rbac.authorization.k8s.io  
  kind: ClusterRole  
  name: prometheus  
subjects:  
- kind: ServiceAccount  
  name: default  
  namespace: monitoring
```

**Create the RBAC role:**

`kubectl create -f prometheus-role.yaml`

Now create a file called `prometheus-config-map.yaml` with the following content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-server-conf
  labels:
    name: prometheus-server-conf
  namespace: monitoring
data:
  prometheus.yml: |-
    global:
      scrape_interval: 5s
      evaluation_interval: 5s
    rule_files:
      - /etc/prometheus/prometheus.rules
    alerting:
      alertmanagers:
      - scheme: http
        static_configs:
        - targets:
          - "alertmanager.monitoring.svc:9093"
    scrape_configs:
      - job_name: 'node-exporter'
        kubernetes_sd_configs:
          - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_endpoints_name]
          regex: 'node-exporter'
          action: keep
      
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
      - job_name: 'kubernetes-nodes'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics     
      
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
        - role: pod
        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
          action: replace
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
          target_label: __address__
        - action: labelmap
          regex: __meta_kubernetes_pod_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_pod_name]
          action: replace
          target_label: kubernetes_pod_name
      
      - job_name: 'kubernetes-cadvisor'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
      
      - job_name: 'kubernetes-service-endpoints'
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
          action: replace
          target_label: __scheme__
          regex: (https?)
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
          action: replace
          target_label: __address__
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          action: replace
          target_label: kubernetes_name
```

This will configure Prometheus to scrape metrics from deployments, services, etc. See the [Prometheus Kubernetes Example](https://github.com/prometheus/prometheus/blob/release-2.30/documentation/examples/prometheus-kubernetes.yml) for more information about what the above does.

Create the configuration map for prometheus:

`kubectl create -f prometheus-config-map.yaml`

Now create a Storage Claim for Prometheus. Create a file called `prometheus-pvc.yaml` with the following content:

```yaml
apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:  
  name: prometheus-pvc  
  namespace: monitoring  
spec:  
  accessModes:  
    - ReadWriteOnce  
  storageClassName: longhorn  
  resources:  
    requests:  
      storage: 1Gi
```

We will only assign 1G of data as this is just a test cluster so that should be more than enough. In production you definitely would want more.

Create the storage claim:

`kubectl create -f prometheus-pvc.yaml`

Now we are ready to create the deployment spec for Prometheus. Create a file called `prometheus-app.yaml` with the following:

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: prometheus-deployment  
  namespace: monitoring  
  labels:  
    app: prometheus-server  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: prometheus-server  
  template:  
    metadata:  
      labels:  
        app: prometheus-server  
    spec:  
      securityContext:  
        runAsUser: 65534  
        runAsGroup: 65534  
        fsGroup: 65534        
      containers:  
        - name: prometheus  
          image: prom/prometheus  
          args:  
            - "--storage.tsdb.retention.time=12h"  
            - "--storage.tsdb.retention.size=500MB"  
            - "--config.file=/etc/prometheus/prometheus.yml"  
            - "--storage.tsdb.path=/prometheus/"  
            - "--web.external-url=[http://10.211.55.240/prometheus](http://10.211.55.241/prometheus)"  
            - "--web.route-prefix=/"  
          ports:  
            - containerPort: 9090  
          resources:  
            requests:  
              cpu: 500m  
              memory: 250M  
            limits:  
              cpu: 1  
              memory: 500M  
          volumeMounts:  
            - name: prometheus-config-volume  
              mountPath: /etc/prometheus/  
            - name: prometheus-storage-volume  
              mountPath: /prometheus/  
      volumes:  
        - name: prometheus-config-volume  
          configMap:  
            defaultMode: 420  
            name: prometheus-server-conf  
    
        - name: prometheus-storage-volume  
          persistentVolumeClaim:  
            claimName: prometheus-pvc
```

You should note a couple of things here. We are using a persistent storage volume which we will mount in the container under `/prometheus`. To ensure this mount gets the correct permissions, we set the `securityContext` (prometheus runs as user `nobody` which is UID 65534 with GID 65534). We also configure Prometheus to only retain 12h worth of data, and never more than 500MB. In production you should change this (be sure to have it match the size you gave for the persistent volume claim!). Lastly, we define the `web.external-url` to match the `LoadBalancer` IP of HAProxy as we want to have HAProxy front it (we will define this further in the ingress spec of prometheus). In production you would probably use a DNS name instead.

Create the prometheus deployment on the cluster:

`kubectl create -f prometheus-app.yaml`

Next create the service to expose the prometheus port, and tell it to scrape itself too. Create a file called `prometheus-service.yaml` with the following:

```yaml
apiVersion: v1  
kind: Service  
metadata:  
  name: prometheus-service  
  namespace: monitoring  
  annotations:  
      prometheus.io/scrape: 'true'  
      prometheus.io/port:   '9090'  
spec:  
  selector:   
    app: prometheus-server  
  ports:  
    - port: 8080  
      targetPort: 9090
```

Create the service:

`kubectl create -f prometheus-service.yaml`

Now that we have a service, we can create an ingress. Create a file called `prometheus-ingress.yaml` with the following:

```yaml
apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
    name: prometheus-ingress  
    namespace: monitoring  
    annotations:  
        haproxy.org/path-rewrite: /prometheus/?(.*) /\1  
        kubernetes.io/ingress.class: haproxy  
spec:  
    rules:  
    - http:  
        paths:  
        - path: /prometheus  
          pathType: Prefix  
          backend:  
            service:  
              name: prometheus-service  
              port:  
                number: 8080
```

Here we instruct HAProxy to handle the ingress for Prometheus. Any request received by HAProxy that starts with path `/prometheus` will be forwarded to the prometheus service which we exposed on port 8080. The service will in turn forward it to port 9090 running inside the container.

Deploy the ingress configuration:

`kubectl create -f prometheus-ingress.yaml`

Now you should be able to access prometheus at the following URL: [http://10.211.55.240/prometheus](http://10.211.55.240/prometheus.).

Excellent! Now we have a running prometheus server that uses a clustered block storage. Although it already collects information about our cluster, it does not yet for any metrics generated by the nodes themselves (e.g. CPU utilization etc). For this we will need to install Prometheus Node Exporter.

# Prometheus Node Exporter

Create a file called `node-exporter-daemon.yaml` with the following:

```yaml
apiVersion: apps/v1  
kind: DaemonSet  
metadata:  
  labels:  
    app.kubernetes.io/component: exporter  
    app.kubernetes.io/name: node-exporter  
  name: node-exporter  
  namespace: monitoring  
spec:  
  selector:  
    matchLabels:  
      app.kubernetes.io/component: exporter  
      app.kubernetes.io/name: node-exporter  
  template:  
    metadata:  
      labels:  
        app.kubernetes.io/component: exporter  
        app.kubernetes.io/name: node-exporter  
    spec:  
      containers:  
      - args:  
        - --path.sysfs=/host/sys  
        - --path.rootfs=/host/root  
        - --no-collector.wifi  
        - --no-collector.hwmon  
        - --collector.filesystem.ignored-mount-points=^/(dev|proc|sys|var/lib/docker/.+|var/lib/kubelet/pods/.+)($|/)  
        - --collector.netclass.ignored-devices=^(veth.*)$  
        name: node-exporter  
        image: prom/node-exporter  
        ports:  
          - containerPort: 9100  
            protocol: TCP  
        resources:  
          limits:  
            cpu: 250m  
            memory: 180Mi  
          requests:  
            cpu: 102m  
            memory: 180Mi  
        volumeMounts:  
        - mountPath: /host/sys  
          mountPropagation: HostToContainer  
          name: sys  
          readOnly: true  
        - mountPath: /host/root  
          mountPropagation: HostToContainer  
          name: root  
          readOnly: true  
      volumes:  
      - hostPath:  
          path: /sys  
        name: sys  
      - hostPath:  
          path: /  
        name: root
```

Where before the `kind` of spec was usually `Deployment` for others, here we will use `[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)`. This will ensure the node exporter will run on all nodes, even if we add new nodes to the cluster.

Create the daemon set:

`kubectl create -f node-exporter-daemon.yaml`

Now create a service so Prometheus can scrape from the node exporters. Create a file called `node-exporter-service.yaml` with the following:

```yaml
apiVersion: v1  
kind: Service  
apiVersion: v1  
metadata:  
  name: node-exporter  
  namespace: monitoring  
  annotations:  
      prometheus.io/scrape: 'true'  
      prometheus.io/port:   '9100'  
spec:  
  selector:  
      app.kubernetes.io/component: exporter  
      app.kubernetes.io/name: node-exporter  
  ports:  
  - name: node-exporter  
    protocol: TCP  
    port: 9100  
    targetPort: 9100
```

Deploy it to the cluster:

`kubectl create -f node-exporter-service.yaml`

Good! Now Prometheus will also give us all the metrics about each node in the cluster. Next up, let’s deploy a test application to our newly created cluster.
# Test Echo Server

To test our cluster we will deploy a simple test [echo server](https://github.com/fdeantoni/echo-server). We will deploy it in its own namespace called `test`:

```sh
kubectl create namespace test
```

Now create a file called `echo-server-app.yaml` with the following content:

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    run: echo  
  name: echo  
  namespace: test  
spec:  
  replicas: 3  
  selector:  
    matchLabels:  
      run: echo  
  template:  
    metadata:  
      labels:  
        run: echo  
    spec:  
      nodeSelector:  
        node-type: worker      
      containers:  
      - name: echo  
        image: fdeantoni/echo-server  
        ports:  
        - containerPort: 9000  
        readinessProbe:  
          httpGet:  
            path: /  
            port: 9000  
          initialDelaySeconds: 5  
          periodSeconds: 5  
          successThreshold: 1
```

Couple of things to note here:

- The property `replicas` is set to 3 so we will create 3 instances of the echo server
- A `selector` is defined to ensure the echo server only gets deployed on `worker` nodes.
- We added a `readinessProbe` which will test each instance at the root path to see if the service is up.

Let’s deploy the app to the cluster:

`kubectl apply -f echo-server-app.yaml`

Now we need to create a service that will expose the echo server service to the cluster. Create a file called `echo-server-service.yaml` with the following:

```yaml
apiVersion: v1  
kind: Service  
metadata:  
    name: echo-service  
    namespace: test  
    annotations:  
        prometheus.io/scrape: 'true'  
        prometheus.io/port:   '9000'  
        prometheus.io/path:   '/metrics'      
spec:  
    selector:  
      run: echo  
    ports:  
    - name: http  
      protocol: TCP  
      port: 9000  
      targetPort: 9000
```

Some things to note here too:

- We instruct Prometheus to also scrape the metrics from the echo server at path `/metrics` on port `9000`.
- We open up port `9000` on the cluster to expose the service inside the container also running on port `9000`.

Create the service:

`kubectl apply -f echo-server-service.yaml`

Now define an ingress configuration that will allow HAProxy to proxy requests for our echo server. Create a file called `echo-server-ingress.yaml` with the following:

```yaml
apiVersion: networking.k8s.io/v1  
kind: Ingress  
metadata:  
    name: echo-ingress  
    namespace: test  
    annotations:  
        haproxy.org/path-rewrite: /test/?(.*) /\1  
        kubernetes.io/ingress.class: haproxy  
spec:  
    rules:  
    - http:  
        paths:  
        - path: /test  
          pathType: Prefix  
          backend:  
            service:  
              name: echo-service  
              port:  
                number: 9000
```

Deploy it to the cluster:

`kubectl apply -f echo-server-ingress.yaml`

Now test it! As our HAProxy runs at `10.211.55.240` we should be able to access the echo server at `[http://10.211.55.240/test](http://10.211.55.240/test)`:
# Useful Sources

To help me along creating this guide possible I used the following sources you should check out too:

- [Raspberry Pi Cluster](https://rpi4cluster.com) — a really good guide for installing kubernetes on a Raspberry Pi cluster.
- [How to Setup Prometheus Monitoring on Kubernetes Cluster](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes)
- [How to Setup Prometheus Node Exporter on Kubernetes](https://devopscube.com/node-exporter-kubernetes)

Thanks for reading!