---
tags:
  - helm
  - redis
  - kubernetes
---
# What is Redis Cluster

Redis Cluster is a distributed implementation of the Redis in-memory data store that provides high availability, automatic partitioning, and fault tolerance. It achieves these goals by sharding data across multiple redis nodes and replicating each shard to a fixed number of replica nodes. This structure allows Redis Cluster to continue operating, even in the face of node failures or network partitions.

When a node fails, Redis Cluster will automatically promote a replica to be the new master and redistribute hash slots accordingly. This ensures data redundancy, load balancing, and continued operation under various failure scenarios, making Redis Cluster suitable for mission-critical applications that require high performance and reliability.

For more information about Redis cluster, checkout the official Redis website: [https://redis.io/docs/management/scaling/](https://redis.io/docs/management/scaling/)

# Why Deploy Redis Cluster in K8s?

Deploying a Redis cluster in K8s offers several benefits, mainly derived from the combination of Redis’s data store capabilities and Kubernetes’ container orchestration features. Some of the key benefits include:

- **Scalability**: K8s simplifies the process of scaling the Redis cluster by managing the deployment of additional nodes.
- **High Availability**: Running Redis cluster in K8s ensures high availability by facilitating the replication of data across multiple nodes. In case of a node failure, K8s automatically reschedules the affected pods on other available nodes, maintaining data redundancy and sustained operations.
- **Automatic Recovery**: K8s continuously monitors the health of Redis cluster nodes and, if a failure is detected, automatically restarts the affected containers or reschedules them on other nodes, ensuring continuous data availability.
- **Easy Management**: Deploying and managing a Redis cluster in K8s is simplified using manifest files, Helm charts, or K8s operators like the Redis Operator. These tools make it easier to configure, deploy, and manage Redis clusters, streamlining the overall process.
- **Rolling Updates:** K8s allows zero-downtime rolling updates for Redis cluster nodes, ensuring smooth and uninterrupted operations during software upgrades or configuration changes.
- **Resource Isolation**: By running a Redis cluster in a K8s environment, you can leverage the benefits of containerization, such as resource isolation, which ensures that the Redis cluster does not interfere with other applications running on the same infrastructure.

# Why Helm?

Helm is a powerful package manager and deployment tool for K8s, designed to streamline the process of managing, deploying, and configuring applications on K8s clusters. It is often referred to as the “package manager for K8s.”

Using Helm to deploy a Redis cluster to K8s offers several benefits that simplify the management, deployment, and configuration of Redis instances in a Kubernetes environment. Here are some key advantages:

- **Simplified Deployment**: Helm charts provide pre-configured templates for deploying a Redis cluster, which significantly reduces the complexity and effort required to set up and manage Redis instances compared to using raw Kubernetes manifests.
- **Versioning and Reusability**: Helm charts are versioned, allowing you to easily maintain, upgrade, or rollback different versions of your Redis cluster configuration.
- **Customization**: Helm charts are highly customizable through the use of variables and templates.
- **Release Management**: Helm provides built-in release management for your Redis cluster deployments, allowing you to track, manage, and rollback deployments as needed. This is particularly useful when dealing with complex applications where manual management can be error-prone.

# Deployment Steps

## Download and install Helm

```sh
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash  
$ helm version  
version.BuildInfo{Version:"v3.12.2", GitCommit:"1e210a2c8cc5117d1055bfaa5d40f51bbc2e345e", GitTreeState:"clean", GoVersion:"go1.20.5"}
```

## Clone the bitnami redis charts to local

I choose to clone the charts to local, so I can make customizations if necessary:

```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami  
"bitnami" has been added to your repositories  
  
$ helm repo update  
Hang tight while we grab the latest from your chart repositories...  
...Successfully got an update from the "bitnami" chart repository  
Update Complete. ⎈Happy Helming!⎈
```

## List available versions

```sh
$ helm search repo bitnami/redis-cluster --versions  
NAME                    CHART VERSION   APP VERSION     DESCRIPTION  
bitnami/redis-cluster   8.6.10          7.0.12          Redis(R) is an open source, scalable, distribut...  
bitnami/redis-cluster   8.6.9           7.0.12          Redis(R) is an open source, scalable, distribut...  
bitnami/redis-cluster   8.6.8           7.0.12          Redis(R) is an open source, scalable, distribut...  

# Download to local  
$ helm pull bitnami/redis-cluster --version 8.6.10 --untar  
  
$ ls redis-cluster/  
Chart.lock  charts  Chart.yaml  img  README.md  templates  values.yaml
```
## Deploy to K8s

```sh
$ helm install 8.6.10 bitnami/redis-cluster -n redis-cluster --set global.imageRegistry=“<registry.com>/<repo_name>”  
–set global.imagePullSecrets[0]=“regcred”  
–set global.storageClass=“<storageclass_name>”  
–set global.redis.password=“test1234”
```
## Check Deployment

```sh
$ helm list -n redis-cluster  
NAME            NAMESPACE               REVISION        UPDATED                                 STATUS          CHART                   APP VERSION  
redis-server    redis-cluster           1               2023-07-27 10:40:59.95805905 -0400 EDT  deployed        redis-cluster-8.6.10    7.0.12  
```
  
```sh
$ kubectl get po -n redis-cluster  
NAME                                          READY   STATUS    RESTARTS   AGE  
redis-server-redis-cluster-0                  1/1     Running   0          3d23h  
redis-server-redis-cluster-1                  1/1     Running   0          3d23h  
redis-server-redis-cluster-2                  1/1     Running   0          3d23h  
redis-server-redis-cluster-3                  1/1     Running   0          3d23h  
redis-server-redis-cluster-4                  1/1     Running   0          3d23h  
redis-server-redis-cluster-5                  1/1     Running   0          3d23h
```
## Check service

```sh
$ kubectl get svc -n redis-cluster  
NAME                                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE  
redis-server-redis-cluster            ClusterIP   172.20.72.100   <none>        6379/TCP             4d  
redis-server-redis-cluster-headless   ClusterIP   None            <none>
```
## Test service

Open one terminal, and port-forward the redis

```sh
$ kubectl port-forward -n redis-cluster service/redis-server-redis-cluster --address 0.0.0.0 8080:6379  
Forwarding from 0.0.0.0:8080 -> 6379
```

Open another terminal, run the following command:

```sh
$ redis-cli -c -h 127.0.0.1 -p 8080 -a 'xxxx'  
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.  
127.0.0.1:8080> set a 'hello'  
OK  
127.0.0.1:8080> exit
```