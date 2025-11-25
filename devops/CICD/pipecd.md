---
tags:
  - devops
  - cicd
---
The One CD for All {applications, platforms, operations}

A GitOps style continuous delivery platform that provides  
consistent deployment and operations experience for any applications

[pipecd](https://pipecd.dev/)


# pipecd Install

```sh
kubectl create namespace pipecd
kubectl apply -n pipecd -f https://raw.githubusercontent.com/pipe-cd/pipecd/master/quickstart/manifests/control-plane.yaml
```

`kubectl port-forward -n pipecd svc/pipecd 8080`

For example, in case of using `MySQL` as datastore and `MINIO` as filestore, the ControlPlane configuration will be as follow:

```yaml
apiVersion: "pipecd.dev/v1beta1"
kind: ControlPlane
spec:
  stateKey: {RANDOM_STRING}
  datastore:
    type: MYSQL
    config:
      url: {YOUR_MYSQL_ADDRESS}
      database: {YOUR_DATABASE_NAME}
  filestore:
    type: MINIO
    config:
      endpoint: {YOUR_MINIO_ADDRESS}
      bucket: {YOUR_BUCKET_NAME}
      accessKeyFile: /etc/pipecd-secret/minio-access-key
      secretKeyFile: /etc/pipecd-secret/minio-secret-key
      autoCreateBucket: true
```

Accessing the PipeCD web

```console
kubectl port-forward svc/pipecd 8080 --namespace={NAMESPACE}
```

Initialize a new project

```console
kubectl port-forward service/pipecd-ops 9082 --namespace={NAMESPACE}
```

