---
tags:
  - kubernetes
  - helm
  - charts
  - dashboard
---
Welcome to this guide on how to manage your Helm Charts using Helm Dashboard. In our previous guide, we walked through how to [install and use Helm 3 on Kubernetes Cluster](https://computingforgeeks.com/install-and-use-helm-3-on-kubernetes-cluster/).

In Linux, there are quite a number of package managers. They include APT and Synaptic Package Manager for DEB packages, DNF and YUM for RPM, Pacman Package Manager for Arch Linux, Zypper Package Manager for OpenSUSE, Portage Package Manager for Gentoo, Snap a universal package manager e.t.c.

_Kubernetes_ is a container orchestration tool that has been highly adopted for application deployment in recent years. Similar to Linux systems, Kubernetes also has a package manager known as **Helm**. This is a Kubernetes deployment tool used for automation, packaging, configuration and deployment of applications and services to Kubernetes clusters. It makes application update and rollbacks more efficient while improving collaboration.

Helm works using helm charts that contain pre-configured application resources along with all the versions into one easily manageable package. Helm charts contain YAML files and templates that can be converted into manifest files for Kubernetes. This mainly streamlines the installation, upgrading fetching dependencies, and configuration process on Kubernetes apps.

Helm offers a lot of benefits in the Kubernetes ecosystem. Some of these features are:

- **Reduces the complexity of deploying Microservices**: once the Helm chart is created, it can be used over and over again by anyone. This can help reduce the complexities involved when deploying applications over time.
- **Enhances deployment speed**: application deployment can be done by running a single **Helm Install** command to create and prepare all the resources required.
- **Boosts productivity**: It allows the software to deploy its test environments at the click of a button

Once the helm charts have been added, especially in a large environment, it becomes so hard to manage them. This is where the **Helm Dashboard** comes in! It provides a UI-driven way to view the installed Helm charts. This makes it easier to view the revision history, and corresponding k8s resources and perform other simple activities like rollback or upgrades to newer versions.

The Helm Dashboard is part of **Komodor’s** vision of helping system admins and developers navigate and troubleshoot their Kubernetes clusters although not an official project by the helm team. With Helm Dashboard, users are able to:

- Switch between multiple clusters
- Easy rollback or upgrade version with a clear and easy manifest diff
- See manifest diff of the past revisions
- See all installed charts and their revision history
- Integration with popular problem scanners
- Browse k8s resources resulting from the chart

Let’s dive in and enjoy the awesomeness of this tool!
## 1. Setup Pre-requisites

Begin by installing all the required packages:

```sh
## On RHEL/CentOS/RockyLinux 8
sudo yum update
sudo yum install curl git vim

## On Debian/Ubuntu
sudo apt update && sudo apt upgrade
sudo apt install curl git vim

## On Fedora
sudo dnf update
sudo dnf -y install curl git vim
```

In this guide, I assume that you already have a Kubernetes cluster up and running. To easily spin up a Kubernetes cluster, use any of the guides below:

- [Deploy HA Kubernetes Cluster on Rocky Linux 8 using RKE2](https://computingforgeeks.com/deploy-kubernetes-on-rocky-using-rke2/)
- [Run Kubernetes on Debian with Minikube](https://computingforgeeks.com/run-kubernetes-on-debian-11-with-minikube/)
- [Deploy Kubernetes Cluster on Linux With k0s](https://computingforgeeks.com/deploy-kubernetes-cluster-on-linux-with-k0s/)
- [Install Kubernetes Cluster on Ubuntu using K3s](https://computingforgeeks.com/install-kubernetes-on-ubuntu-using-k3s/)
- [Install Kubernetes Cluster on Rocky Linux 8 with Kubeadm & CRI-O](https://computingforgeeks.com/install-kubernetes-cluster-on-rocky-linux-with-kubeadm-crio/)
- [Deploy k0s Kubernetes on Rocky Linux 9 using k0sctl](https://computingforgeeks.com/deploy-k0s-kubernetes-on-rocky-linux-9-using-k0sctl/)
- [Install Minikube Rocky Linux 9 and Create Kubernetes Cluster](https://computingforgeeks.com/install-minikube-kubernetes-rocky-linux-9/)

**For easier management of the cluster, you might need `kubectl`. Install it with the commands:**

```sh
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin
```

**Now export the admin config:**

```sh
##For RKE2
export PATH=$PATH:/var/lib/rancher/rke2/bin export KUBECONFIG=/etc/rancher/rke2/rke2.yaml

##For K0s
export KUBECONFIG=/var/lib/k0s/pki/admin.conf
```

You also need to have Helm installed. We have dedicated guides on this page to help you.

- [Install and Use Helm 3 on Kubernetes Cluster](https://computingforgeeks.com/install-and-use-helm-3-on-kubernetes-cluster/)

**Verify the installation:**

```sh
helm version
version.BuildInfo{Version:"v3.10.3", GitCommit:"835b7334cfe2e5e27870ab3ed4135f136eecc704", GitTreeState:"clean", GoVersion:"go1.18.9"}
```
## 2. Installing the Helm Dashboard

In this guide, I will provide two ways how to get the Helm Dashboard installed on your system. These methods are:

- Install it as a plugin
- Install Helm Dashboardon Kubernetes
### Method 1 – Install Helm Dashboard as a Helm plugin

**Once the prerequisites have been done, we can easily install the helm plugin manager on your system with the command:**

```sh
helm plugin install https://github.com/komodorio/helm-dashboard.git
```

**Sample output:**

```sh
https://github.com/komodorio/helm-dashboard/releases/download/v1.3.3/helm-dashboard_1.3.3_Linux_x86_64.tar.gz

Helm Dashboard is installed, to start it, run in your terminal:
    helm dashboard

Installed plugin: dashboard
```

**To update your plugin to the latest version, issue the command:**

```sh
helm plugin update dashboard
```
#### Start and Access the Helm Dashboard

Now we can easily start the Helm Dashboard by executing the command:

```sh
helm dashboard
```

This will start the dashboard in the local Web server and will automatically open the UI in a new browser tab. This will remain to hang, waiting for you to terminate it either from the command line or web UI.

**You can start the dashboard and set it to bind on your IP address or all interfaces using the `--bind`flag. For example:**

```sh
helm dashboard --bind 0.0.0.0
```

**You can also specify a port if the default port 8080 is busy.**

```sh
helm dashboard --bind 0.0.0.0 --port 8090
```

You can now access the dashboard using the set port and IP as shown:

![Manage your Helm Charts using Helm Dashboard](https://computingforgeeks.com/wp-content/uploads/2022/12/Manage-your-Helm-Charts-using-Helm-Dashboard-1024x465.png?ezimgfmt=rs:696x316/rscb23/ng:webp/ngcb23 "How To Manage your Helm Charts using Helm Dashboard 1")

### Method 2 – Install Helm Dashboard on Kubernetes

For this method, you need the following:

- Helm 3+
- Kubernetes 1.16+

**Begin by adding the chart:**

```sh
helm repo add komodorio https://helm-charts.komodor.io
```

**Now update the Helm charts:**

```sh
helm repo update
```

The Helm Dashboard by default requires a persistent volume to store its files and configurations. You can still disable this using the variables explained later in this guide.

**Create a storage account:**

```sh
kubectl apply -f - <<EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: hem-dashboard-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF
```

**Set it as the default storage class:**

```sh
kubectl patch storageclass hem-dashboard-sc -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

**Verify the creation:**

```sh
kubectl get sc
NAME                         PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
hem-dashboard-sc (default)   kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  25s
```

**Create a persistent volume:**

```YAML
vim helm-dahboard-pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name:  helm-dahboard-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: hem-dashboard-sc
  local:
    path: /mnt/disk/vol1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - worker1
```

Now on the specified worker name, create the path:

```sh
sudo mkdir -p /mnt/disk/vol1
sudo chmod 777 /mnt/disk/vol1

##Also run this On Rhel-based systems#####
sudo chcon -Rt svirt_sandbox_file_t /mnt/disk/vol1
```

**Now apply the manifest:**

```sh
kubectl create -f  helm-dahboard-pv.yml
```
#### a. Deploy the Helm Dashboard with Defaults

**To install the dashboard with defaults, use:**

```sh
helm upgrade --install my-release komodorio/helm-dashboard
```

**To remove/delete the installation:**

```sh
helm uninstall my-release
```
#### b. Customize the deployment

When installing, there are quite a number of variables you can set, these include:

|Parameter|Description|Default value|
|---|---|---|
|`image.repository`|Image registry/name|`docker.io/komodorio/helm-dashboard`|
|`image.tag`|Image tag||
|`image.pullPolicy`|Image pull policy|`IfNotPresent`|
|`replicaCount`|Number of dashboard Pods to run|`1`|
|`dashboard.allowWriteActions`|Enables write actions. Allow modifying, deleting and creating charts and Kubernetes resources.|`true`|
|`resources.requests.cpu`|CPU resource requests|`200m`|
|`resources.limits.cpu`|CPU resource limits|`1`|
|`resources.requests.memory`|Memory resource requests|`256Mi`|
|`resources.limits.memory`|Memory resource limits|`1Gi`|
|`service.type`|Kubernetes service type|`ClusterIP`|
|`service.port`|Kubernetes service port|`8080`|
|`serviceAccount.create`|Creates a service account|`true`|
|`serviceAccount.name`|This is an optional name for the service account|`{RELEASE_FULLNAME}`|
|`nodeSelector`|Node labels for pod assignment||
|`affinity`|Affinity settings for pod assignment||
|`tolerations`|Tolerations for pod assignment||
|`dashboard.persistence.enabled`|Enable helm data persistence using PVC|`true`|
|`dashboard.persistence.accessModes`|Persistent Volume access modes|`["ReadWriteOnce"]`|
|`dashboard.persistence.storageClass`|Persistent Volume storage class|`""`|
|`dashboard.persistence.size`|Persistent Volume size|`100M`|
|`dashboard.persistence.hostPath`|Set path in case you want to use local host path volumes (not recommended in production)|`""`|

**Remember, each parameter is used with the syntax below in addition to the helm install command:**

```
--set key=value[,key=value]
```

**For example:**

```sh
helm upgrade --install my-release komodorio/helm-dashboard --set dashboard.allowWriteActions=true --set service.port=8090 --set service.type=NodePort 
```

**Sample Output:**

```sh
Release "my-release" does not exist. Installing it now.
NAME: my-release
LAST DEPLOYED: Fri Dec 23 12:16:30 2022
NAMESPACE: trow
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing Helm Dashboard.
Helm Dashboard can be accessed:
  * Within your cluster, at the following DNS name at port 8090:

    my-release-helm-dashboard.trow.svc.cluster.local

  * From outside the cluster, run these commands in the same shell:

    export POD_NAME=$(kubectl get pods --namespace trow -l "app.kubernetes.io/name=helm-dashboard,app.kubernetes.io/instance=my-release" -o jsonpath="{.items[0].metadata.name}")
    export CONTAINER_PORT=$(kubectl get pod --namespace trow $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
    echo "Visit http://127.0.0.1:8080 to use your application"
    kubectl --namespace trow port-forward $POD_NAME 8080:$CONTAINER_PORT

Visit our repo at:
https://github.com/komodorio/helm-dashboard
```

**Verify if the container has been created:**

```sh
$ kubectl get pods
NAME                                         READY   STATUS    RESTARTS   AGE
my-release-helm-dashboard-577b7f99bc-qfqtw   1/1     Running   0          2m22s
```

**View the service:**

```sh
$ kubectl get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
my-release-helm-dashboard   NodePort    10.100.230.110   <none>        8090:31080/TCP   3m13s
```
## 3. Access Helm Dashboard

Now you can access the dashboard using the provided NodePort. For this case, it is **31080**

![Manage your Helm Charts using Helm Dashboard 1](https://computingforgeeks.com/wp-content/uploads/2022/12/Manage-your-Helm-Charts-using-Helm-Dashboard-1-1024x405.png?ezimgfmt=rs:696x275/rscb23/ng:webp/ngcb23 "How To Manage your Helm Charts using Helm Dashboard 2")

Now you are set the manage your Helm charts using this dashboard. To see any **resources** associated with a chart, click on it.

![Manage your Helm Charts using Helm Dashboard 2](https://computingforgeeks.com/wp-content/uploads/2022/12/Manage-your-Helm-Charts-using-Helm-Dashboard-2-1024x514.png?ezimgfmt=rs:696x349/rscb23/ng:webp/ngcb23 "How To Manage your Helm Charts using Helm Dashboard 3")

You can also see the **manifests** associated with the chart.

![Manage your Helm Charts using Helm Dashboard 3](https://computingforgeeks.com/wp-content/uploads/2022/12/Manage-your-Helm-Charts-using-Helm-Dashboard-3-1024x515.png?ezimgfmt=rs:696x350/rscb23/ng:webp/ngcb23 "How To Manage your Helm Charts using Helm Dashboard 4")

You can also add any other repository and use it as desired:

![Manage your Helm Charts using Helm Dashboard 4](https://computingforgeeks.com/wp-content/uploads/2022/12/Manage-your-Helm-Charts-using-Helm-Dashboard-4-1024x711.png?ezimgfmt=rs:696x483/rscb23/ng:webp/ngcb23 "How To Manage your Helm Charts using Helm Dashboard 5")

## Verdict

We can now agree that we have successfully learned how to deploy the Helm Dashboard and use it to manage Helm Charts. I hope this was significant to you.

See more:

- [Deploy Metrics Server in Kubernetes using Helm Chart](https://computingforgeeks.com/deploy-metrics-server-in-kubernetes-using-helm/)
- [Perform security checks on Kubernetes manifests and Helm charts using Datree](https://computingforgeeks.com/security-checks-on-kubernetes-using-datree/)
- [Deploy Nginx Ingress Controller on Kubernetes using Helm Chart](https://computingforgeeks.com/deploy-nginx-ingress-controller-on-kubernetes-using-helm-chart/)
- [Install Vault Cluster in GKE via Helm, Terraform and BitBucket Pipelines](https://computingforgeeks.com/install-vault-cluster-gke-via-helm-terraform-bitbucket-pipelines/)