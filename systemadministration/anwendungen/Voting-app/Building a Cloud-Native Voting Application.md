# Introduction

In today’s digital landscape, building scalable and efficient applications is paramount. This article explores the implementation of a cloud-native voting application using a combination of modern technologies on Google Cloud. We’ll leverage GitOps practices with Argo CD and Helm for continuous deployment, utilize Kubernetes for orchestration, and monitor application performance with Prometheus and Grafana. Our application stack includes Python, .NET, Redis, PostgreSQL, and Node.js, providing a robust foundation for real-time observability and user interaction.
# Architecture Overview

The cloud-native voting application comprises several components, each deployed as Kubernetes services. The architecture is designed for scalability and resilience, enabling seamless user experiences even during high traffic.
# Core Components

1. **PostgreSQL**: The primary relational database for storing voting data and user information.
2. **Redis**: An in-memory data structure store used for caching and managing real-time data.
3. **Voting Application**: Consists of various microservices including:

- **Vote Service**: Handles user votes and interactions.
- **Result Service**: Displays voting results to users.
- **Worker Service**: Processes votes and performs background tasks.
# Tech-Stacks:

Our cloud-native voting application is built on a robust and dynamic tech stack that leverages the strengths of the following technologies:

- **Python**: Known for its simplicity and versatility, Python is used for developing backend services that handle data processing and business logic, enabling rapid development and easy integration with other components.
- **.NET**: Harnessing the power of the .NET framework, we create high-performance web services that provide a seamless user experience. This allows us to leverage existing enterprise solutions while ensuring scalability and security.
- **Redis**: As an in-memory data structure store, Redis offers fast data retrieval and caching capabilities. It enhances application performance by efficiently managing real-time data, such as user sessions and voting results.
- **PostgreSQL**: This powerful open-source relational database serves as our primary data store, providing strong data integrity and complex query capabilities. PostgreSQL is ideal for handling the relational data needs of our voting application, ensuring robust data management.
- **Node.js**: Utilizing Node.js allows us to build fast and scalable network applications. Its event-driven architecture makes it perfect for handling asynchronous events, enabling real-time updates and interactions within the voting platform.
# Deployment Configuration

The deployment configuration is defined using Kubernetes manifests (YAML files) that specify the application’s state, including deployment and service specifications. Below is a breakdown of the configurations for the voting application:

![](ATzce1jtcCd6_S44p59cdQ.webp)

Sample Diagram of the Project

 PostgreSQL Deployment:

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    app: db  
  name: db  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: db  
  template:  
    metadata:  
      labels:  
        app: db  
    spec:  
      containers:  
      - image: postgres:15-alpine  
        name: postgres  
        env:  
        - name: POSTGRES_USER  
          value: postgres  
        - name: POSTGRES_PASSWORD  
          value: postgres  
        ports:  
        - containerPort: 5432  
          name: postgres  
        volumeMounts:  
        - mountPath: /var/lib/postgresql/data  
          name: db-data  
      volumes:  
      - name: db-data  
        emptyDir: {}
```

 **Redis Deployment:**

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    app: redis  
  name: redis  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: redis  
  template:  
    metadata:  
      labels:  
        app: redis  
    spec:  
      containers:  
      - image: redis:alpine  
        name: redis  
        ports:  
        - containerPort: 6379  
          name: redis  
        volumeMounts:  
        - mountPath: /data  
          name: redis-data  
      volumes:  
      - name: redis-data  
        emptyDir: {}
```

> **Voting and Result Services Deployment**

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    app: result  
  name: result  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: result  
  template:  
    metadata:  
      labels:  
        app: result  
    spec:  
      containers:  
      - image: dockersamples/examplevotingapp_result  
        name: result  
        ports:  
        - containerPort: 80  
          name: result  
  
---  
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    app: vote  
  name: vote  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: vote  
  template:  
    metadata:  
      labels:  
        app: vote  
    spec:  
      containers:  
      - image: dockersamples/examplevotingapp_vote  
        name: vote  
        ports:  
        - containerPort: 80  
          name: vote
```

> **Service Definitions**

For internal communication and external access, Kubernetes services are defined to expose the application:

```yaml
apiVersion: v1  
kind: Service  
metadata:  
  labels:  
    app: db  
  name: db  
spec:  
  type: ClusterIP  
  ports:  
  - name: "db-service"  
    port: 5432  
    targetPort: 5432  
  selector:  
    app: db  
  
---  
apiVersion: v1  
kind: Service  
metadata:  
  labels:  
    app: redis  
  name: redis  
spec:  
  type: ClusterIP  
  ports:  
  - name: "redis-service"  
    port: 6379  
    targetPort: 6379  
  selector:  
    app: redis  
  
---  
apiVersion: v1  
kind: Service  
metadata:  
  labels:  
    app: result  
  name: result  
spec:  
  type: NodePort  
  ports:  
  - name: "result-service"  
    port: 5001  
    targetPort: 80  
    nodePort: 31001  
  selector:  
    app: result  
  
---  
apiVersion: v1  
kind: Service  
metadata:  
  labels:  
    app: vote  
  name: vote  
spec:  
  type: NodePort  
  ports:  
  - name: "vote-service"  
    port: 5000  
    targetPort: 80  
    nodePort: 31002  
  selector:  
    app: vote
```

# GitOps with Argo CD and Helm

# Argo CD

Argo CD is a powerful tool for continuous delivery in Kubernetes. By leveraging GitOps principles, we can synchronize our application state in the Git repository with the Kubernetes cluster. Argo CD continuously monitors the repository for changes and applies them to the cluster, ensuring that the deployed state matches the desired state.
# Helm

Helm serves as the package manager for Kubernetes, simplifying the deployment and management of applications. Using Helm charts, we can define our application’s configuration, dependencies, and deployment instructions, making it easier to manage complex applications.
# Monitoring with Prometheus and Grafana

# Prometheus

Prometheus is an open-source monitoring system that collects metrics from configured targets. In our application, we deploy Prometheus to gather metrics from our services, allowing us to monitor their performance and health.
# Grafana

Grafana provides a powerful dashboarding solution for visualizing metrics collected by Prometheus. By integrating Grafana with Prometheus, we can create interactive dashboards that display real-time data about our application’s performance, enabling quick insights and troubleshooting.
# Real-Time Observability

To achieve real-time observability, we integrate various technologies that provide insights into application performance, user interactions, and system health.

- **Logging**: Implement logging in our microservices to capture and analyze events.
- **Tracing**: Use distributed tracing to understand the flow of requests across services and identify bottlenecks.
- **Monitoring**: Regularly monitor key performance indicators (KPIs) to assess application health and make informed decisions.
# Steps to Perform:

**Step:1** Create a VM instance

![](6mDyGNvFHruX7RiICxzHwQ.webp)

Compute Engine →VM Instances

![](fr3xRx9u73T9RzybEC167w.webp)

Creating the instance

![](7KX8LRwVJILKqxlQ_IWkIg.webp)

Choose Ubuntu Image

![](aoNZ5zE2c6d0cpNmlCFdOQ.webp)

Use the network tag as ‘cicd’

![](TtW_2NwwPwWUbxyYgnlLAQ.webp)

Network Firewall Rule with tag of ‘cicd’

![](Yn0pd32Y69OAqFa0u-rAow.webp)

VM is Ready

**Step2:** Open the cloudshell and ssh to VM with gcloud command.

![](3lTJxfLrSLJ_-wnIuQumsw.webp)

Open Cloud Shell and login to VM

![](5ll6nwyZW5q9t9uY5Sm_tw.webp)

Login to VM Successful

**Step3:** Update the System

`sudo apt-get update` 

![](WsMzMAl14GYHgZ_6_7zDDA.webp)

**Step4**: Install the docker

```sh
sudo apt install docker.io  
sudo usermod -aG docker $USER && newgrp docker
```

![](TDZxjmEMTTNlffcZZYO_7A.webp)

**Step5:** Clone the Repo to the VM

`git clone https://github.com/anshumaan-10/k8s-voting-app-demo.git`

**Step6: Installing the kind cluster**

Go to kind-cluster folder

Provide the necessary permission with kind_install.sh

```sh
cd k8s-voting-app-demo/
ls
cd kind-cluster/
chmod +x install_kind.sh
./install_kind.sh
```

```sh
#!/bin/bash
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64  
chmod +x ./kind  
sudo cp ./kind /usr/local/bin/kind  
rm -rf kind
```

![](6GHN3bp2iCNpjv5fTkznNg.webp)

**Step6:** Installing the kubectl Binary:

```sh
cd k8s-voting-app-demo/  
ls  
cd kind-cluster/  
chmod +x install_kubectl.sh   
./install_kubectl.sh
```

![](OQLwDZBRdzn6gq8DcPcjTA.webp)

kubectl Installation
## Creating and Managing Kubernetes Cluster with Kind

```
config.yml  
  
kind: Cluster  
apiVersion: kind.x-k8s.io/v1alpha4  
  
nodes:  
- role: control-plane  
  image: kindest/node:v1.30.0  
- role: worker  
  image: kindest/node:v1.30.0  
- role: worker  
  image: kindest/node:v1.30.0
```

**Step7:** Create a 3-node Kubernetes cluster using Kind:

`kind create cluster --config=config.yml --name=k8s-voting-cluster`

![](kgeQV1cgtXwqgaN7iBog.webp)

**Step8:** Check cluster information

kubectl cluster-info --context kind-k8s-voting-cluster  
kubectl get nodes  
kind get clusters  
kubectl get pods -A

![](VZdF2vj1xIqzUE9evWSLuQ.webp)

Verify the installation of cluster
## Step9: Installing ArgoCD

Create a namespace for Argo CD:

`kubectl create namespace argocd`

Apply the Argo CD manifest:

`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

![](Hk63zcyde6w3kKjAjiKP1Q.webp)

Check services in Argo CD namespace:

`kubectl get svc -n argocd`

![](tX_OPvfeswsiSLlVeUaHRA.webp)

**Step10:** Expose Argo CD server using NodePort:

`kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'`

Forward ports to access Argo CD server:

```sh
kubectl get svc -n argocd  
kubectl port-forward -n argocd service/argocd-server 8444:443 --address=0.0.0.0 &
```

![](y1yV3nU-GPSUXRhUIsvuww.webp)

**Step11:** ArgoCD is Up and Running

**tep12:** To retrieve ArgoCD username and password run the following command

```sh
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

![](JV74QVIRdk4aWuHpN-1dqw.webp)

Password of ArgoCD

Username and Password

![](M3e9pNUM7JDGbeGJPO2mtw.webp)

Successfully logged in

**Step13:** Click on New App and fill the details as mentioned in the screenshot

![](A4KbWwF-WTyqjfG7nAU7oA.webp)

![](o-iUL8ZVb4in54Z29gG59g.webp)

**Step14**: Click on create.

![](r2tfN0Id9n2yau9xMCQSHg.webp)

![](A5yzhlPtNvb6RKiUcf82w.webp)

All the Services are healthy

![](bj4xyFmkFPjN95B_fL_tKA.webp)

Worker Node Status

![](zfKu01PE9D2SvIvUQDbocg.webp)

![](TrEtVnbUGKOhB6tIxbYiuQ.webp)

check the service

**Step15:** Forward local ports for accessing the voting and result apps:

```sh
kubectl port-forward service/vote 5000:5000 --address=0.0.0.0 &  
kubectl port-forward service/result 5001:5001 --address=0.0.0.0 &
```

![](qNLhPem3yHzdWd0bUV0N6w.webp)

Voting App

![](zLt9nhdHjhqxqsnLnrVbWg.webp)

Result App

## Step16: Installing Kubernetes Dashboard:

Deploy Kubernetes dashboard:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

![](MMGw0JdlnG1_733BYkVmYQ.webp)

Step17: Create a token for dashboard access:

```sh
cd k8s-voting-app-demo/kind-cluster  
ls   
kubectl apply -f dashboard-adminuser.yml   
kubectl -n kubernetes-dashboard create token admin-user
```

![](skwY2NZYVaVG8r2StvNBw.webp)
Token Generated

**Step18**: Do the port forwarding to access

```sh
kubectl get svc -n kubernetes-dashboard  
kubectl port-forward svc/kubernetes-dashboard -n kubernetes-dashboard 8081:443 --address=0.0.0.0 &
```

![](kD7E6G-j2N3v8V5sheMYKw.webp)
Paste the token

![](eEIrjeARbvS7OB9oHJW8kA.webp)

Kubernetes Dashboard is Up and Running

![](14fxTHxu7uUzjUhV2YMzUQ.webp)
Deployments

![](u64CzF0_h9ewJom6F2A_Yg.webp)
Pods

![](NclP4TumuuN2aGpFNCbHEA.webp)
Replica Sets

![](gTiEkOV-wWc8dUN0rLcUQA.webp)
Services

![](GIjVVW60BNGP9C_XS5PUg.webp)
ConfigMaps

![](I9PZfYNK4aIvCoELphWHw.webp)
Secrets

**Step19:** Now let’s Setup the Monitoring Part using Helm Chart.

**Install Helm**

```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3  
chmod 700 get_helm.sh  
./get_helm.sh
```

![](ioDMZPNAoGyAqG3ysO2mNw.webp)
Helm

## Step20: Install Kube Prometheus Stack

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts  
helm repo add stable https://charts.helm.sh/stable  
helm repo update  
kubectl create namespace monitoring  
helm install kind-prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --set prometheus.service.nodePort=30000 --set prometheus.service.type=NodePort --set grafana.service.nodePort=31000 --set grafana.service.type=NodePort --set alertmanager.service.nodePort=32000 --set alertmanager.service.type=NodePort --set prometheus-node-exporter.service.nodePort=32001 --set prometheus-node-exporter.service.type=NodePort  
kubectl get svc -n monitoring  
kubectl get namespace
```

![](GqjCtUnYgDm_GTPLFvV9Pw.webp)
Prometheus Repo Added

![](YZFaTd3n5-R2Fmkv3j3NZg.webp)
Checking the Status

**Step21**: To access the prometheus UI do the port forwarding

```sh
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
```

![](nZ7cw38W5KBCOpQoo8UV2Q.webp)
Promethus is Running

**Step22:** Goto Status and click on target to see the metrics

![](a7wyffMGoGAYXzh2zghRmw.webp)

Up and Running

[http://104.198.141.111:9090/metrics](http://104.198.141.111:9090/metrics) to check the metrics are getting exported.

![](wVjCSz63B9ZbccljOKNodQ.webp)
Metrics

## Step23: Prometheus Queries(PromQL expressions):

1. **CPU Usage Percentage Across All Containers in** `**default**` **Namespace**

```
sum (rate (container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum (machine_cpu_cores) * 100
```

![](EC_xIphB2J1rAxs0IZS5g.webp)

**Explanation**:

**Explanation**:

- `container_cpu_usage_seconds_total`: Tracks the total CPU time each container has used.
- `rate(...[1m])`: Calculates the rate of change over a 1-minute interval, showing how much CPU time containers in the `default` namespace are using per second.
- `sum(...)`: Adds up the CPU usage rate for all containers in the `default` namespace.
- `sum(machine_cpu_cores)`: Adds up the total number of CPU cores available.
- The division by `machine_cpu_cores` and multiplication by 100 gives the CPU usage as a percentage of total CPU capacity.

**Use**: This query provides the overall CPU usage percentage of containers within the `default` namespace.

1. **Memory Usage of Each Pod in the** `**default**` **Namespace**

```sh
sum (container_memory_usage_bytes{namespace="default"}) by (pod)
```

![](crvzslW7DPUn6WbXu3ozMg.webp)

**Explanation**:

- `container_memory_usage_bytes`: Shows the memory currently used by each container.
- `sum(...{namespace="default"}) by (pod)`: Aggregates memory usage at the pod level in the `default`namespace.

**Use**: This query helps monitor memory usage on a per-pod basis, identifying which pods are consuming the most memory.

**3. Network Traffic for Each Pod in the** `**default**` **Namespace**

**Inbound Traffic (Received)**:

```sh
sum by (pod) (rate(container_network_receive_bytes_total{namespace="default"}[5m]))
```

![](ESgUeCFWTMYdvoGgGM2bvA.webp)

**Explanation**:

- `container_network_receive_bytes_total`: Measures the total inbound network traffic received by each container.
- `rate(...[5m])`: Computes the rate over the last 5 minutes, giving an approximate inbound traffic rate.
- `sum(...) by (pod)`: Aggregates the network receive rate at the pod level.

**Outbound Traffic (Transmitted)**:

```sh
sum(rate(container_network_transmit_bytes_total{namespace="default"}[5m])) by (pod)
```

![](W5CA248qYiDHj5LkYqYktw.webp)

**Explanation**:

- `container_network_transmit_bytes_total`: Measures the total outbound network traffic sent by each container.
- The rest is similar to the inbound traffic query.

**Use**: These queries help track network traffic per pod, useful for monitoring network usage and identifying high-traffic pods.

# Additional Useful PromQL Queries

## 4. Disk I/O (Read and Write) by Pod

**Disk Read**:

```sh
sum(rate(container_fs_reads_bytes_total{namespace="default"}[5m])) by (pod)
```

![](mueYYUgdqDnZb9iPFSHCYg.webp)

**Disk Write**:

```sh
sum(rate(container_fs_writes_bytes_total{namespace="default"}[5m])) by (pod)
```

**Explanation**:

- `container_fs_reads_bytes_total` and `container_fs_writes_bytes_total`: Track total disk read and write operations.
- `rate(...[5m])`: Calculates the rate over 5 minutes, showing average read/write throughput.
- `sum(...) by (pod)`: Aggregates read and write rates for each pod.

**Use**: Monitor disk I/O activity per pod, useful for tracking storage performance and identifying storage-heavy applications.

## 5. CPU Usage by Pod (Average Usage)

```sh
avg(rate(container_cpu_usage_seconds_total{namespace="default"}[5m])) by (pod)
```

![](kFkBP_rqU2oVKbcirPfKig.webp)

**Explanation**:

- `avg(...) by (pod)`: Calculates the average CPU usage rate per pod, giving a general CPU usage level for each pod over the specified period.

**Use**: This query gives insights into average CPU utilization across pods, useful for monitoring steady-state resource usage.

## 6. Memory Requests vs. Memory Limits in a Namespace

**Memory Requests**:

```sh
sum(kube_pod_container_resource_requests_memory_bytes{namespace="default"})
```

**Memory Limits**:

```sh
sum(kube_pod_container_resource_limits_memory_bytes{namespace="default"})
```

**Explanation**:

- `kube_pod_container_resource_requests_memory_bytes`: Shows the requested memory for containers.
- `kube_pod_container_resource_limits_memory_bytes`: Shows the memory limits set for containers.

**Use**: This query pair helps check if memory limits are being set appropriately, which is useful for capacity planning and preventing out-of-memory issues.

**Step24:** Now to visualize the promethus Results let’s setup the Grafana Dashboard.

```sh
kubectl get svc -n monitoring  
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &
```

**Default username: admin**
**Default password: prom-operator**

Grafana UI

![](n-Ei78vw-oFO9d-puMPudA.webp)

Login Successfully

**Step25**: Goto Home →Connections →Data Sources

![](UCVoltOOOpHecIY7bQV0ww.webp)

**Step26**: Click on Dashboard →New →Import →Use id 15661

![](ogv52dbF0mYQOrw9DRH9IA.webp)

![](aG2d7kcvWZqMudhGzO3Ezw.webp)
Choose Prometheus

![](kVktLsFQi0YSErePGHh4rg.webp)
Kubernetes Node Consumption

![](TdJ5_KeV_TQTm1TSYXYg4A.webp)

![](NvLeF6952VrFYmbJC3aS1w.webp)
CPU Usage

![](n308Gu1pvO1ELJhWvfHGgw.webp)
Network Bandwidth and Packet Rate

![](PpNizGhPBHJDGfwxC1gVbg.webp)

Kubernetes Cluster Status

![](fCdEJgYZ06IMhoJi_qsQA.webp)

> **Grafana with Kubernetes that you can implement or import into your Grafana setup:**

# 1. Kubernetes Cluster Monitoring Dashboard

- **Dashboard ID**: `893`
- **Description**: Provides an overview of cluster health, node status, and resource usage.

# 2. Kubernetes Node Exporter Dashboard

- **Dashboard ID**: `1860`
- **Description**: Displays metrics collected from Kubernetes nodes, including CPU and memory usage.

# 3. Kubernetes Pod Monitoring Dashboard

- **Dashboard ID**: `1538`
- **Description**: Visualizes the performance and status of Kubernetes pods, including resource usage.

# 4. Kubernetes Dashboard

- **Dashboard ID**: `5555`
- **Description**: A comprehensive overview of your Kubernetes cluster, showing pod counts, deployments, and other vital metrics.

# 5. Kube-state-metrics Dashboard

- **Dashboard ID**: `7440`
- **Description**: Displays various Kubernetes metrics from `kube-state-metrics`, such as deployments, replicas, and pod statuses.
- **Source**: Grafana Dashboards

# 6. Kubernetes Resources Overview

- **Dashboard ID**: `868`
- **Description**: Offers insights into resources used across the cluster, including memory and CPU allocation.
- **Source**: Grafana Dashboards

# 7. Prometheus Kubernetes Monitoring

- **Dashboard ID**: `315`
- **Description**: Designed for monitoring Kubernetes clusters using Prometheus as a data source.
- **Source**: Grafana Dashboards

# 8. Kubernetes Resource Utilization

- **Dashboard ID**: `633`
- **Description**: Visualizes resource consumption metrics like CPU and memory for each namespace.

# 9. Kubernetes Health Dashboard

- **Dashboard ID**: `8889`
- **Description**: Monitors the health of your Kubernetes cluster, including pod and node status.

# 10. Kubernetes Events Dashboard

- **Dashboard ID**: `1560`
- **Description**: Visualizes Kubernetes events to help track changes and issues in the cluster.

# How to Import Dashboards

To import these dashboards into your Grafana instance:

1. **Open Grafana** and navigate to the “+” icon in the left sidebar.
2. Click on **“Import”**.
3. Enter the Dashboard ID (e.g., `893`) in the **Grafana.com Dashboard** field or upload a JSON file if you have one.
4. Click **Load**, adjust any data source settings if necessary, and then click **Import**.

# Conclusion

Building a cloud-native voting application using Kubernetes on Google Cloud provides a scalable and robust solution. By integrating GitOps with Argo CD, leveraging Helm for deployments, and utilizing Prometheus and Grafana for monitoring, we achieve a streamlined development process with real-time observability. This approach enables teams to respond quickly to issues and continuously improve application performance, ultimately enhancing user experiences.

By following best practices in application architecture and deployment, organizations can effectively manage and scale their applications in the cloud, ensuring they meet the demands of users and maintain high availability.