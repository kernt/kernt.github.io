---
tags:
  - kubespray
  - kubernetes
  - ansible
  - backup
---
Kubespray uses both Ansible and Kubeadm to deploy Kubernetes cluster in Virtual Machines or dedicated servers. Being composable, it allows you to choose from a wide range of options during deployment such as Linux distribution, network plugins, container runtimes, e.t.c. With Kubespray you can perform installation in cloud platforms such as Amazon EC2 (AWS), Azure, Google Cloud, and private cloud platforms like OpenStack.

This brief article is created to help you with upgrading of your Kubernetes cluster deployed using Kubespray. For new installation check out our article in the link below.

- [Deploy Kubernetes Cluster with Ansible & Kubespray](https://computingforgeeks.com/deploy-production-kubernetes-cluster-with-ansible/)

And for only adding a new node into the cluster we have: [Adding a New Node into Kubernetes Cluster using Kubespray](https://computingforgeeks.com/adding-new-node-into-kubernetes-cluster-using-kubespray/)

**Our Kubernetes version before upgrade.**

```sh
kubectl get node
NAME        STATUS   ROLES           AGE   VERSION
k8smas01    Ready    control-plane   39d   v1.30.4
k8smas02    Ready    control-plane   39d   v1.30.4
k8smas03    Ready    control-plane   39d   v1.30.4
k8snode01   Ready    <none>          39d   v1.30.4
```
## Backup current configurations

Our inventory directory for Kubespray is `kubespray/inventory/k8scluster/`. We’ll copy the contents of three main files for later reference when updating parameters.

```sh
mkdir ~/kubespray-backups
cp kubespray/inventory/k8scluster/group_vars/k8s_cluster/k8s-cluster.yml ~/kubespray-backups
cp kubespray/inventory/k8scluster/group_vars/all/all.yml ~/kubespray-backups
cp kubespray/inventory/k8scluster/inventory.ini ~/kubespray-backups
```

In our previous deployment below values were use. Check if you had customizations and save for later use.

```sh
$ vim kubespray/inventory/k8scluster/group_vars/k8s_cluster/k8s-cluster.yml
cluster_name: k8s.example.com
kube_network_plugin: calico
container_manager: crio

$ vim kubespray/inventory/k8scluster/group_vars/all/all.yml
bin_dir: /opt/bin #because the OS is Flatcar container Linux
apiserver_loadbalancer_domain_name: api.k8s.example.com
loadbalancer_apiserver:
  address: 192.168.1.8
  port: 6443
```
## Clone Kubespray source if does’t exist

**If you don’t have kubespray source locally with latest source, clone it.**

```sh
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
```

From my cluster status nodes, we can confirm the version of Kubernetes is **_v1.24.6_**. This was deployed from tag **_v2.20.0_**.

You can check official Kubespray git [releases](https://github.com/kubernetes-sigs/kubespray/releases) or [tags](https://github.com/kubernetes-sigs/kubespray/tags) page. To list available tags in the git repository run:

```sh
git tag --list --sort=version:refname
...
v2.22.2
v2.23.0
v2.23.1
v2.23.2
v2.23.3
v2.24.0
v2.24.1
v2.24.2
v2.25.0
v2.26.0
```

**Or sort from latest tag.**

```sh
git tag -l --sort=-version:refname
v2.26.0
v2.25.0
v2.24.3
v2.24.2
v2.24.1
v2.24.0
v2.23.3
v2.23.2
v2.23.1
v2.23.0
v2.22.2
v2.22.1
v2.22.0
....

git tag -l | sort -V --reverse
...
```

From the output we can confirm the next available release tag. Our initial upgrade will be from its source.

**For git branches use:**

```sh
git branch --list --remotes --sort=-version:refname
  origin/release-2.25
  origin/release-2.24
  origin/release-2.23
  origin/release-2.22
  origin/release-2.21
  origin/release-2.20
.....
```
## Upgrade Kubernetes cluster using Kubespray

Let’s update files in the working tree to match the release we are upgrading to. For me this is tag **2.21.0.**

```sh
git checkout v2.26.0
D	inventory/sample/group_vars/all/aws.yml
D	inventory/sample/group_vars/all/azure.yml
D	inventory/sample/group_vars/all/containerd.yml
D	inventory/sample/group_vars/all/coreos.yml
D	inventory/sample/group_vars/all/cri-o.yml
D	inventory/sample/group_vars/all/docker.yml
D	inventory/sample/group_vars/all/gcp.yml
D	inventory/sample/group_vars/all/hcloud.yml
D	inventory/sample/group_vars/all/oci.yml
D	inventory/sample/group_vars/all/vsphere.yml
D	inventory/sample/group_vars/etcd.yml
D	inventory/sample/group_vars/k8s_cluster/k8s-net-calico.yml
D	inventory/sample/group_vars/k8s_cluster/k8s-net-flannel.yml
D	inventory/sample/group_vars/k8s_cluster/k8s-net-kube-ovn.yml
D	inventory/sample/group_vars/k8s_cluster/k8s-net-kube-router.yml
D	inventory/sample/group_vars/k8s_cluster/k8s-net-macvlan.yml
D	inventory/sample/group_vars/k8s_cluster/k8s-net-weave.yml
D	inventory/sample/inventory.ini
HEAD is now at 2cf23e310 Don't search filesystem mounts in docker build step (#10131) (#10194)

$ git describe --tags
v2.26.0
```

**We can also use release number with `git checkout` command.**

```sh
git checkout release-2.26
```

**Let’s copy inventory sample to**

```sh
cp -rfp inventory/sample inventory/k8scluster
```

Review the inventory files and variables before performing an upgrade. Set them to match your current installation to avoid any issue after upgrading the cluster.

```sh
inventory/k8scluster/inventory.ini
inventory/k8scluster/group_vars/all/all.yml
inventory/k8scluster/group_vars/k8s_cluster/k8s-cluster.yml
```

**Contents of my `inventory.ini` file.**

```ini
[all]
master01 ansible_host=192.168.1.10 etcd_member_name=etcd1   ansible_user=core
master02 ansible_host=192.168.1.11 etcd_member_name=etcd2   ansible_user=core
master03 ansible_host=192.168.1.12 etcd_member_name=etcd3   ansible_user=core
node01   ansible_host=192.168.1.13 etcd_member_name=        ansible_user=core
node02   ansible_host=192.168.1.14 etcd_member_name=        ansible_user=core
node03   ansible_host=192.168.1.15 etcd_member_name=        ansible_user=core
node04   ansible_host=192.168.1.16 etcd_member_name=        ansible_user=core
node05   ansible_host=192.168.1.17 etcd_member_name=        ansible_user=core

# ## configure a bastion host if your nodes are not directly reachable
# [bastion]
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube_control_plane]
master01
master02
master03

[etcd]
master01
master02
master03

[kube_node]
node01
node02
node03
node04
node05

[new_nodes]
node04
node05

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

Once done initiate an upgrade of your cluster by running the following commands.

```sh
ansible-playbook -i inventory/k8scluster/inventory.ini -b upgrade-cluster.yml
```

Extra options to use if not set permanently in inventory file:

- `-e ansible_user===rocky==`
- `--become-user ==root==`

To limit upgrade to one node use `--limit===nodename==`

If everything goes well, you’ll get output similar to below after successful upgrade.

![kubespray upgrade cluster](https://computingforgeeks.com/wp-content/uploads/2023/08/kubespray-upgrade-cluster-1024x353.png?ezimgfmt=rs:696x240/rscb23/ng:webp/ngcb23 "Upgrading Kubespray Kubernetes Cluster to newer release 1")

**List your nodes after upgrade to see runtime version & Kubernetes version numbers.**

```sh
$ kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                                             KERNEL-VERSION     CONTAINER-RUNTIME
master01   Ready    control-plane   10d   v1.28.6   192.168.1.10   <none>        Flatcar Container Linux by Kinvolk 3510.2.5 (Oklo)   5.15.119-flatcar   containerd://1.7.15
master02   Ready    control-plane   10d   v1.28.6   192.168.1.11   <none>        Flatcar Container Linux by Kinvolk 3510.2.5 (Oklo)   5.15.119-flatcar   containerd://1.7.15
master03   Ready    control-plane   10d   v1.28.6   192.168.1.12   <none>        Flatcar Container Linux by Kinvolk 3510.2.5 (Oklo)   5.15.119-flatcar   containerd://1.7.15
node01     Ready    <none>          10d   v1.28.6   192.168.1.13   <none>        Flatcar Container Linux by Kinvolk 3510.2.5 (Oklo)   5.15.119-flatcar   containerd://1.7.15
node02     Ready    <none>          10d   v1.28.6   192.168.1.14   <none>        Flatcar Container Linux by Kinvolk 3510.2.5 (Oklo)   5.15.119-flatcar   containerd://1.7.15
node03     Ready    <none>          10d   v1.28.6   192.168.1.15   <none>        Flatcar Container Linux by Kinvolk 3510.2.5 (Oklo)   5.15.119-flatcar   containerd://1.7.15
node04     Ready    <none>          10d   v1.28.6   192.168.1.16   <none>        Flatcar Container Linux by Kinvolk 3510.2.5 (Oklo)   5.15.119-flatcar   containerd://1.7.15
node05     Ready    <none>          10d   v1.28.6   192.168.1.17   <none>        Flatcar Container Linux by Kinvolk 3510.2.5 (Oklo)   5.15.119-flatcar   containerd://1.7.15
node06     Ready    <none>          10d   v1.28.6   192.168.1.18   <none>        Flatcar Container Linux by Kinvolk 3510.2.5 (Oklo)   5.15.119-flatcar   containerd://1.7.15
```

**Running `kubectl version` will display the Server version after update.**

```sh
kubectl version
```

## Performing multiple upgrades (after first success)

**Checkout to next release / tag.**

```sh
git checkout v2.26.0
Previous HEAD position was c4346e590 kubeadm/etcd: use config to download certificate (#9609)
HEAD is now at 4014a1ccc fix multus include (#10105)

$ git branch
* (HEAD detached at v2.26.0)
  master
```

**Update inventory directory that contains all settings for the deployment.**

```sh
mv  inventory/k8scluster{,.bak}
cp -rfp inventory/sample inventory/k8scluster
```

**The perform cluster upgrade**

```sh
ansible-playbook -i inventory/k8scluster/inventory.ini -b upgrade-cluster.yml
```

**Check next Kubernetes version after the upgrade.**

```sh
kubectl get nodes -o wide
```
## Conclusion

In a matter of minutes, and with few commands, we have been able to upgrade our Kubernetes cluster using Kubespray. Note that to use this guide you need a Kubernetes cluster that is fully functional, and was deployed using Kubespray. Kubespray is a powerful automation tool that is highly adaptable, configurable, and extensible. Kubespray incorporates operations and security practices and enables you to focus cluster administration and focus more on building your applications.