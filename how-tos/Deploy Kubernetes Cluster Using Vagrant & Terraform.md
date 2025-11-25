---
tags:
  - vagrant
  - kubernetes
  - terraform
  - kops
  - ansible
---
With the increasing complexity of modern applications and infrastructure, manual setup and configuration are no longer efficient or scalable. Automation tools like Kubernetes, Vagrant, and Terraform provide a way to define infrastructure and application configurations as code, allowing for version control, reproducibility, and scalability. These tools enable organizations to achieve consistent and repeatable deployments, reduce human errors, and accelerate the time to market for their applications.

**Kubernetes**, as a container orchestration platform, automates the deployment, scaling, and management of containerized applications. It abstracts away the underlying infrastructure, allowing developers and operators to focus on defining the desired state of their applications and leaving Kubernetes to handle the execution and management of those applications.

**Vagrant**, on the other hand, automates the setup and provisioning of development environments by defining virtual machines as code. It eliminates the need for manual environment configuration, allowing developers to quickly create and share reproducible development environments. This automation saves time and ensures consistency across development teams.

**Terraform** takes automation a step further by providing infrastructure-as-code capabilities. It allows organizations to define and provision infrastructure resources across various cloud providers and infrastructure platforms. With Terraform, infrastructure configurations can be written as code, enabling automation, collaboration, and version control. This approach removes manual intervention in the infrastructure provisioning process and provides a scalable and efficient way to manage infrastructure resources.

When used together, Kubernetes, Vagrant, and Terraform can greatly simplify the process of deploying a Kubernetes cluster. Vagrant can be used to set up the necessary virtual machines for the cluster, while Terraform can provision the required infrastructure resources. Once the infrastructure is in place, Kubernetes can be deployed and managed using its own tools like [[kubernetes kubeadm]] or [[kops]].

The significant advantages associated when using Kubernetes, Vagrant, and Terraform together when deploying a Kubernetes cluster are:

- **Scalability and Flexibility**: Kubernetes provides built-in scalability features, allowing applications to scale horizontally and handle increased traffic and workloads. Combined with Terraform, which can provision and manage infrastructure resources on various cloud providers, organizations can easily scale their Kubernetes clusters based on demand. Vagrant also offers flexibility by allowing developers to create and replicate development environments with ease.
- **Simplified Deployment**: Kubernetes abstracts away the complexities of managing containerized applications, while Vagrant and Terraform automate the setup and provisioning of the required infrastructure resources. This combination allows for streamlined and simplified deployment processes, reducing the time and effort required to get a Kubernetes cluster up and running.
- **Collaboration and Team Productivity**: The use of these automation tools promotes collaboration and enhances team productivity. Infrastructure configurations and deployment scripts can be stored in version control systems, enabling multiple team members to contribute, review, and collaborate on the infrastructure setup. This improves transparency, knowledge sharing, and overall efficiency within the team.
- **Consistency and Reproducibility**: By using infrastructure-as-code practices with Terraform and Vagrant, organizations can ensure consistent and reproducible deployments. Infrastructure and development environments can be defined in code and shared across teams, reducing the risk of configuration errors and inconsistencies.
- **Time and Cost Savings**: Automation reduces manual intervention and eliminates repetitive tasks, saving time and effort. It minimizes human errors and accelerates the deployment process. With faster and more efficient deployments, organizations can bring their applications to market quicker, resulting in potential cost savings and competitive advantages.
- **Portability and Multi-Cloud Support**: Kubernetes, Vagrant, and Terraform support multiple cloud providers and infrastructure platforms. This allows organizations to deploy their Kubernetes clusters in various environments, including on-premises data centres, public clouds, or hybrid cloud setups. The flexibility and portability offered by these tools enable organizations to adopt a multi-cloud strategy and avoid vendor lock-in.

Join me as we learn how to deploy a Kubernetes Cluster Using Vagrant & Terraform.
## Install the Required Packages

For this guide, you need to have:

**_Install VirtualBox:_**

- [How To Install VirtualBox](https://computingforgeeks.com/?s=install+virtualbox)

**_Install **Vagrant**:_**

- [How To Install Vagrant](https://computingforgeeks.com/?s=install+vagrant)

Verify your Vagrant installation:

```sh
vagrant --version
```

**_Install Ansible_**

- [How To Install Ansible](https://computingforgeeks.com/?s=install+ansible)

You can install Ansible on Linux using the below commands:

```sh
##On Ubuntu / Debian
sudo apt update
sudo apt install ansible

##On RHEL/Rocky Linux/Alma Linux
sudo yum install epel-release
sudo yum install ansible ansible-core
```

Verify the installation:

```sh
$ ansible --version
ansible [core 2.13.6]
  config file = None
  configured module search path = ['/Users/jkmutai/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/Cellar/ansible/6.6.0/libexec/lib/python3.10/site-packages/ansible
  ansible collection location = /Users/jkmutai/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.8 (main, Oct 13 2022, 10:17:43) [Clang 14.0.0 (clang-1400.0.29.102)]
  jinja version = 3.1.2
  libyaml = True
```
## Create Kubernetes cluster using Vagrant

Once the required packages have been installed, you can proceed and spin a Kubernetes cluster using Vagrant.

First[, clone the repository that provides the required configs](https://github.com/cloudspinx/virtualbox-kubernetes):

```sh
git clone https://github.com/techviewleo/virtualbox-kubernetes.git
cd virtualbox-kubernetes
```

In the directory, there is a **Vagrantfile** and several other files like Ansible playbooks etc. You can open the Vagrantfile for editing:

```sh
vim Vagrantfile
```

The file has default variables declared, you can modify them to suit your environment. For example;

- **IMAGE_NAME** = “bento/ubuntu-16.04” This can be changed to any other version. (Try and report issues if any)
- **NODES** = 2 defines the number of worker nodes in the cluster. Based on this value, Vagrant will create multiple nodes automatically.
- **CLUSTER_NAME** = “_demo_” can be changed if you need another cluster in the same workstation (cluster name will be prefixed with VM name to identify)
- **APISERVER_ADVERTISE_ADDRESS** = “192.168.56.102” and NODE_IP_ADDRESS_RANGE = “192.168.56” are used based on **vboxnet0** Host-only adaptor. If you using a different network, then modify the IP addresses accordingly.

To use the host-only network on Virtualbox, create it with the command:

```sh
VBoxManage hostonlyif create
```

List the created interface:

```sh
$ VBoxManage list dhcpservers
NetworkName:    HostInterfaceNetworking-vboxnet0
Dhcpd IP:       192.168.56.100
LowerIPAddress: 192.168.56.101
UpperIPAddress: 192.168.56.254
NetworkMask:    255.255.255.0
Enabled:        Yes
Global Configuration:
    minLeaseTime:     default
    defaultLeaseTime: default
    maxLeaseTime:     default
    Forced options:   None
    Suppressed opts.: None
        1/legacy: 255.255.255.0
Groups:               None
Individual Configs:   None
```

If any hostname changes are made in the Vagrant file, you also need to modify them in the Playbook.

```sh
vim kubernetes-setup/controlplane-playbook.yml 
# Step 2.3: Initialize the Kubernetes cluster with kubeadm using the below code (applicable only on controlplane node).
  - name: Initialize the Kubernetes cluster using kubeadm
    command: sudo kubeadm init --apiserver-advertise-address={{ apiserver_advertise_address }} --apiserver-cert-extra-sans={{ apiserver_advertise_address }} --node-name demo-k8s-controlplane --pod-network-cidr=192.168.0.0/16
```

Save the changes and apply the file:

```sh
vagrant up
```

If all goes well, you will see Ansible scripts executed as well.

![Deploy Kubernetes Cluster Using Vagrant Terraform](https://computingforgeeks.com/wp-content/uploads/2023/05/Deploy-Kubernetes-Cluster-Using-Vagrant-Terraform.png?ezimgfmt=rs:696x667/rscb23/ng:webp/ngcb23 "Deploy Kubernetes Cluster Using Vagrant & Terraform 1")

View the running VMs on VirtualBox:

![Deploy Kubernetes Cluster Using Vagrant Terraform 1](https://computingforgeeks.com/wp-content/uploads/2023/05/Deploy-Kubernetes-Cluster-Using-Vagrant-Terraform-1.png?ezimgfmt=rs:696x205/rscb23/ng:webp/ngcb23 "Deploy Kubernetes Cluster Using Vagrant & Terraform 2")

Now we need to access the Kubernetes cluster provisioned form or workstation. To achieve that, you need to obtain the **KUBECONFIG** file from the control plane.

SSH into the control plane with the command:

```sh
vagrant ssh controlplane
```

Once there, copy the config to your workstation:

```sh
scp ~/.kube/config  username@IP_Address:~/.kube/config
```

Replace the **username** and **IP_Address** with the exact details for your workstation. Once copied, exit SSH:

```sh
exit
```

To access the cluster, install kubectl on the workstation:

```sh
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin
```
## Deploy Kubernetes Networking

At this point, the cluster is up but the nodes are not ready. To verify that, use:

```sh
$ kubectl get nodes -o wide
NAME                    STATUS     ROLES           AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
demo-compute-1          NotReady   <none>          6m7s    v1.27.2   <none>           <none>        Ubuntu 20.04.6 LTS   5.4.0-144-generic   containerd://1.6.21
demo-compute-2          NotReady   <none>          3m22s   v1.27.2   <none>           <none>        Ubuntu 20.04.6 LTS   5.4.0-144-generic   containerd://1.6.21
demo-k8s-controlplane   NotReady   control-plane   8m55s   v1.27.2   192.168.56.102   <none>        Ubuntu 20.04.6 LTS   5.4.0-144-generic   containerd://1.6.21
```

For the nodes to get ready, we need to deploy the network. Begin by installing the operator:

```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```

Now pull and apply the manifests:

```sh
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml -O
kubectl create -f custom-resources.yaml
```

View the pods after some time:

```sh
$ kubectl get pods --all-namespaces
NAMESPACE         NAME                                            READY   STATUS    RESTARTS      AGE
calico-system     calico-kube-controllers-789dc4c76b-rr47f        0/1     Running   2 (28s ago)   2m39s
...
```

View the nodes:

```sh
$ kubectl get nodes
NAME                    STATUS   ROLES           AGE   VERSION
demo-compute-1          Ready    <none>          15m   v1.27.2
demo-compute-2          Ready    <none>          12m   v1.27.2
demo-k8s-controlplane   Ready    control-plane   18m   v1.27.2
```
## Automate Deployments on Kubernetes using Terraform

We will be using Terraform to automate resource deployment in the Kubernetes cluster. First, ensure that Terraform is installed on your system. This can be done by following any of the below guides:

- [Install Terraform on CentOS 8 / Rocky Linux 8](https://computingforgeeks.com/how-to-install-terraform-on-centos-linux/)
- [How To Install Terraform on Linux Systems](https://computingforgeeks.com/how-to-install-terraform-on-linux/)
- [Install Terraform on Windows / Windows Server 2019](https://computingforgeeks.com/install-and-use-terraform-on-windows/)

Verify the installation:

```sh
terraform  version
Terraform v1.4.6
on linux_amd64
```

When connecting to your cluster with Terraform, you can use the below authentication methods. The most recommended is first and the least recommended last

- Use cloud-specific auth plugins (for example, `eks get-token`, `az get-token`, `gcloud config`)
- Use [oauth2 token](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/guides/getting-started#provider-setup)
- Use [TLS certificate credentials](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs#statically-defined-credentials)
- Use `kubeconfig` file by setting **both** [`config_path`](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs#config_path) and [`config_context`](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs#config_context)
- Use [username and password (HTTP Basic Authorization)](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs#statically-defined-credentials)

in this guide, we will use **kubeconfig** authentication. For that, you need the config_path and config_context.

View your current context with the command:

```sh
kubectl config current-context
kubernetes-admin@kubernetes
```

After obtaining your **context name**, proceed to create a Teraform script for the deployment.

For this guide, we will deploy a simple Nginx app on the Kubernetes cluster.

```sh
mkdir ~/nginx && cd  ~/nginx
vim kubernetes-nginx.tf
```

In the file add the lines below:

```sh
provider "kubernetes" {
  config_path    = "~/.kube/config"
  config_context = "kubernetes-admin@kubernetes"
}

resource "kubernetes_deployment" "nginx" {
  metadata {
    name = "nginx-deployment"
    labels = {
      app = "nginx"
    }
  }

  spec {
    replicas = 3

    selector {
      match_labels = {
        app = "nginx"
      }
    }

    template {
      metadata {
        labels = {
          app = "nginx"
        }
      }

      spec {
        container {
          name  = "nginx"
          image = "nginx:latest"
          port {
            container_port = 80
          }
        }
      }
    }
  }
}

resource "kubernetes_service" "nginx" {
  metadata {
    name = "nginx-service"
  }

  spec {
    selector = {
      app = "nginx"
    }

    port {
      node_port   = 30201
      port        = 80
      target_port = 80
    }
   type = "NodePort"
  }
}
```

Replace the variable in bold as desired. Then run the Terraform script.

```sh
terraform init
terraform plan
terraform apply
```

Proceed as shown when applying the changes:

```sh
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

The deployment should then happen as shown:

![Deploy Kubernetes Cluster Using Vagrant Terraform 2](https://computingforgeeks.com/wp-content/uploads/2023/05/Deploy-Kubernetes-Cluster-Using-Vagrant-Terraform-2.png?ezimgfmt=rs:696x249/rscb23/ng:webp/ngcb23 "Deploy Kubernetes Cluster Using Vagrant & Terraform 3")

View if the pods are running:

```sh
kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77cd647b9c-8tttn   1/1     Running   0          2m59s
nginx-deployment-77cd647b9c-gnpqp   1/1     Running   0          2m59s
nginx-deployment-77cd647b9c-l5rdd   1/1     Running   0          2m59s

kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        73m
nginx-service   NodePort    10.97.92.142   <none>        80:30201/TCP   7m45s
```

You can also view the status with the command:

```sh
terraform show
```

You can now access the service using your NodeIP address which is **192.168.56.51** and **192.168.56.52**.

For example:

```html
$ curl http://192.168.56.51:30201
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

That marks the end of this guide on how to deploy Kubernetes Cluster Using Vagrant & Terraform. You can now agree with me that these 3 tools can make it so easy t deploy and manage resources in a Kubernetes cluster. I hope this was informative.

See more:

- [Install and Use KubeSphere on existing Kubernetes cluster](https://computingforgeeks.com/install-use-kubesphere-on-existing-kubernetes-cluster/)
- [Install and Use Trow Container Image Registry With Kubernetes](https://computingforgeeks.com/trow-container-image-registry-with-kubernetes/)
- [Install and Configure Traefik Ingress Controller on Kubernetes](https://computingforgeeks.com/install-configure-traefik-ingress-controller-on-kubernetes/)