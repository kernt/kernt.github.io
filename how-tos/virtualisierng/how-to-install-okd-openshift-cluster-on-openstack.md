---
tags:
  - openstack
  - okd
  - openshift
---
In this guide we will be performing an installation of OKD / OpenShift 4.x Cluster on OpenStack Cloud Platform. OpenShift is a powerful, enterprise grade containerization software solution developed by Red Hat. The solution is built around Docker containers orchestrated and managed by Kubernetes on a foundation of Red Hat Enterprise Linux .

The OpenShift platform offers automated installation, upgrades, and lifecycle management throughout the container stack – from the operating system, Kubernetes and cluster services, to deployed applications. Operating system that will be used on both the Control plan and Worker machines is **Fedora CoreOS (FCOS)** for OKD deployment, and **Red Hat CoreOS (RHCOS)** for OpenShift deployment. This OS includes the kubelet, which is the Kubernetes node agent, and the CRI-O container runtime, which is optimized for Kubernetes.

Fedora CoreOS / Red Hat Enterprise Linux CoreOS also includes a critical first-boot provisioning tool called Ignition which enables the cluster to configure the machines. With all the machines in the cluster running on RHCOS/FCOS, the cluster will manage all aspects of its components and machines, including the operating system.

Below is a diagram showing subset of the installation targets and dependencies for OpenShift / OKD Cluster.
![openshift okd installation steps](https://computingforgeeks.com/wp-content/uploads/2021/05/openshift-okd-installation-steps-1024x412.png?ezimgfmt=rs:696x280/rscb23/ng:webp/ngcb23 "How To Install OKD OpenShift 4.13 Cluster on OpenStack 1")
The latest release of OpenShift as of this article writing is version _4._13. Follow the steps outlined in this article to have a working installation of OpenShift / OKD Cluster on OpenStack. There is a requirement of having a running installation of OpenStack Cloud – on-premise, co-located infrastructure or Cloud IaaS setup.
## Step 1: Download Installation Program / Client Tools

Download the installation program(openshift-install) and cluster management tools from:

- [OKD releases tools](https://github.com/openshift/okd/releases)
- For OpenShift Cluster Setup: [OpenShift releases page](https://cloud.redhat.com/openshift/releases)
### OKD Installation program and Client tools

Install Libvirt to avoid error “./openshift-install: error while loading shared libraries: libvirt-lxc.so.0: cannot open shared object file: No such file or directory“

```sh
# CentOS / Fedora / RHEL / Rocky
sudo yum -y install libvirt

# Ubuntu / Debian
sudo apt update
sudo apt -y install  libvirt-daemon-system libvirt-daemon 
```

**Downloading OKD 4.x installer:**

```sh
mkdir -p ~/okd/tools
cd ~/okd/tools
# Linux
wget https://github.com/okd-project/okd/releases/download/4.13.0-0.okd-2023-06-04-080300/openshift-install-linux-4.13.0-0.okd-2023-06-04-080300.tar.gz
# macOS
wget https://github.com/okd-project/okd/releases/download/4.13.0-0.okd-2023-06-04-080300/openshift-install-mac-4.13.0-0.okd-2023-06-04-080300.tar.gz
```

**Extract the file after downloading:**

```sh
# Linux
tar xvf openshift-install-linux-*.tar.gz

# macOS
tar xvf openshift-install-mac-*.tar.gz
```

**Move resulting binary file to _/usr/local/bin_ directory:**

```sh
sudo mv openshift-install /usr/local/bin
```

**Download Client tools:**

```sh
# Linux
wget https://github.com/okd-project/okd/releases/download/4.13.0-0.okd-2023-06-04-080300/openshift-client-linux-4.13.0-0.okd-2023-06-04-080300.tar.gz
tar xvf openshift-client-linux-*.tar.gz
sudo mv kubectl oc /usr/local/bin
# macOS
wget https://github.com/okd-project/okd/releases/download/4.13.0-0.okd-2023-06-04-080300/openshift-client-mac-4.13.0-0.okd-2023-06-04-080300.tar.gz
tar xvf openshift-client-mac-*.tar.gz
sudo mv kubectl oc /usr/local/bin
```

**Check versions of both _oc_ and _openshift-install_ to confirm successful installation:**

```sh
oc version
Client Version: 4.13.0-0.okd-2023-06-04-080300
Kustomize Version: v4.5.7

openshift-install version
openshift-install 4.13.0-0.okd-2023-06-04-080300
built from commit 90bb61f38881d07ce94368f0b34089d152ffa4ef
release image quay.io/openshift/okd@sha256:d696b82dbdac77fa98518ef1e8891104eb937d29c6461539407b301c79ea2177
release architecture amd64
```

### OpenShift 4.x Installation program and client tools (==Only for RedHat OpenShift installation==)

Before you install OpenShift Container Platform, download the installation file on a local computer.

- Access the [Infrastructure Provider](https://cloud.redhat.com/openshift/install) page on the Red Hat OpenShift Cluster Manager site
- Select your infrastructure provider – (Red Hat OpenStack)
- Download the installation program for your operating system

```sh
# Linux
mkdir -p ~/ocp/tools
cd ~/ocp/tools
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/openshift-install-linux.tar.gz
tar xvf openshift-install-linux.tar.gz
sudo mv openshift-install /usr/local/bin/

# macOS
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/openshift-install-mac.tar.gz
tar xvf openshift-install-mac.tar.gz
sudo mv openshift-install /usr/local/bin/
```

**Installation of Cluster Management tools:**

```sh
# Linux
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/openshift-client-linux.tar.gz
tar xvf openshift-client-linux.tar.gz
sudo mv oc kubectl  /usr/local/bin/

# macOS
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/openshift-client-mac.tar.gz
tar xvf openshift-client-mac.tar.gz
sudo mv oc kubectl  /usr/local/bin/
```

**Confirm installation:**

```sh
openshift-install version
openshift-install 4.13.1
built from commit 2f227cc47f4e7ca6ed8edf0caf7c48283a82fc03
release image quay.io/openshift-release-dev/ocp-release@sha256:9c92b5ec203ee7f81626cc4e9f02086484056a76548961e5895916f136302b1f
release architecture amd64

kubectl version --client --short
Client Version: v1.26.1
Kustomize Version: v4.5.7

oc version
Client Version: 4.13.1
Kustomize Version: v4.5.7
```
## Step 2: Configure OpenStack Clouds in clouds.yaml file

In OpenStack, `clouds.yaml` is a configuration file that contains everything needed to connect to one or more clouds. It may contain private information and is generally considered private to a user.

OpenStack Client will look for the  `clouds.yaml` file in the following locations:

- current working directory
- `~/.config/openstack`
- `/etc/openstack`

We will place our Clouds configuration file in the _~/.config/openstack_ directory:

```sh
mkdir -p ~/.config/openstack/
```

Create a new file:

```sh
vim ~/.config/openstack/clouds.yaml
```

Sample configuration contents for two clouds. Change accordinly:

```sh
clouds:
  osp1:
    auth:
      auth_url: http://192.168.200.2:5000/v3
      project_name: admin
      username: admin
      password: 'AdminPassword'
      user_domain_name: Default
      project_domain_name: Default
    identity_api_version: 3
    region_name: RegionOne
  osp2:
    auth:
      auth_url: http://192.168.100.3:5000/v3
      project_name: admin
      username: admin
      password: 'AdminPassword'
      user_domain_name: Default
      project_domain_name: Default
    identity_api_version: 3
    region_name: RegionOne
```

A cloud can be selected on the command line:

```sh
openstack --os-cloud osp1 network list --format json
[
  {
    "ID": "44b32734-4798-403c-85e3-fbed9f0d51f2",
    "Name": "private",
    "Subnets": [
      "1d1f6a6d-9dd4-480e-b2e9-fb51766ded0b"
    ]
  },
  {
    "ID": "70ea2e21-79fd-481b-a8c1-182b224168f6",
    "Name": "public",
    "Subnets": [
      "8244731c-c119-4615-b134-cfad768a27d4"
    ]
  }
]
```

Reference: [OpenStack Clouds configuration guide](https://docs.openstack.org/python-openstackclient/latest/configuration/index.html)
## Step 3: Create Compute Flavors for OpenShift Cluster Nodes

A flavor with **at least 16 GB** memory, **4 vCPUs**, and **25 GB** storage space is required for cluster creation.

Let’s create the Compute flavor:

```sh
$ openstack flavor create  --ram 16384    --vcpus 4 --disk 30 m1.openshift
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| description                | None                                 |
| disk                       | 30                                   |
| id                         | 90234d29-e059-48ac-b02d-e72ce3f6d771 |
| name                       | m1.openshift                         |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 16384                                |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 4                                    |
+----------------------------+--------------------------------------+
```

If you have more compute resources you can add more CPU, Memory and Storage to the flavor being created.
## Step 4: Create Floating IP Addresses

You’ll two Floating IP addresses for:

- A floating IP address to associate with the **Ingress port**
- A floating IP Address to associate with the **API load balancer**.

Create API Load balancer floating IP Address:

```sh
openstack floating ip create --description "API <cluster_name>.<base_domain>" <external_network>
```

Create Ingress Floating IP:

```sh
openstack floating ip create --description "Ingress <cluster_name>.<base_domain>" <external_network>
```

You can list your networks using the command:

```sh
$ openstack network list
+--------------------------------------+---------------------+--------------------------------------+
| ID                                   | Name                | Subnets                              |
+--------------------------------------+---------------------+--------------------------------------+
| 155ef402-bf39-494c-b2f7-59509828fcc2 | public              | 9d0e8119-c091-4a20-b03a-80922f7d43dd |
| af7b4f7c-9095-4643-a470-fefb47777ae4 | private             | 90805451-e2cd-4203-b9ac-a95dc7d92957 |
+--------------------------------------+---------------------+--------------------------------------+
```

My Floating IP Addresses will be created from the public subnet. An external network should be configured in advance.

```sh
openstack floating ip create --description "API  ocp.mycluster.com"  public
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2021-05-29T19:48:23Z                 |
| description         | API  ocp.mycluster.com               |
| dns_domain          | None                                 |
| dns_name            | None                                 |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.21.200.20                        |
| floating_network_id | 155ef402-bf39-494c-b2f7-59509828fcc2 |
| id                  | a0f41edb-c90b-417d-beff-9c03f180c71b |
| name                | 172.21.200.20                        |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | d0515ffa23c24e54a3b987b491f17acb     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2021-05-29T19:48:23Z                 |
+---------------------+--------------------------------------+

$ openstack floating ip create --description "Ingress ocp.mycluster.com"  public
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2021-05-29T19:42:02Z                 |
| description         | Ingress ocp.mycluster.com            |
| dns_domain          | None                                 |
| dns_name            | None                                 |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.21.200.22                        |
| floating_network_id | 155ef402-bf39-494c-b2f7-59509828fcc2 |
| id                  | 7035ff39-2903-464c-9ffc-c07a3245448d |
| name                | 172.21.200.22                        |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | d0515ffa23c24e54a3b987b491f17acb     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2021-05-29T19:42:02Z                 |
+---------------------+--------------------------------------+
```
## Step 5: Create required DNS Entries

Access your DNS server management portal or console and create required DNS entries:

```sh
api.<cluster_name>.<base_domain>.  IN  A  <API_FIP>
*.apps.<cluster_name>.<base_domain>. IN  A <apps_FIP>
```

Where:

- **<cluster_name>** is the base domain – e.g **computingforgeeks.com**
- **<cluster_name>** is the name that will be given to your cluster – e.g **ocp**
- **<API_FIP>** is the floating IP address created in Step 4 for API load balancer
- **<apps_FIP>** is the floating IP address created in Step 4 for Ingress (Access to apps deployed)

Example of API DNS entry:

![openshift api dns record](https://computingforgeeks.com/wp-content/uploads/2021/05/openshift-api-dns-record-1024x293.png?ezimgfmt=rs:696x199/rscb23/ng:webp/ngcb23 "How To Install OKD OpenShift 4.13 Cluster on OpenStack 2")

Example of Ingress DNS entry:

![openshift ingress dns record](https://computingforgeeks.com/wp-content/uploads/2021/05/openshift-ingress-dns-record-1024x164.png?ezimgfmt=rs:696x111/rscb23/ng:webp/ngcb23 "How To Install OKD OpenShift 4.13 Cluster on OpenStack 3")

## Step 6: Generate OpenShift install-config.yaml file

From the [Pull Secret](https://cloud.redhat.com/openshift/install/pull-secret) page on the Red Hat OpenShift Cluster Manager site, download your installation pull secret as a .txt file.

![openshift install pull secret](https://computingforgeeks.com/wp-content/uploads/2021/05/openshift-install-pull-secret-1024x207.png?ezimgfmt=rs:696x141/rscb23/ng:webp/ngcb23 "How To Install OKD OpenShift 4.13 Cluster on OpenStack 4")

Run the following command to generate _install-config.yaml_ file:

```
cd ~/
openshift-install create install-config --dir=<installation_directory> 
```

For `<installation_directory>`, specify the directory name to store the files that the installation program creates. The installation directory specified must be empty.

Example:

```sh
$ openshift-install create install-config --dir=ocp
```

At the prompts, provide the configuration details for your cloud:

```txt
? Platform openstack # Select openstack as the platform to target.
? Cloud osp1 # Choose cloud configured in clouds.yml
? ExternalNetwork public # Specify OpenStack external network name to use for installing the cluster.
? APIFloatingIPAddress  [Use arrows to move, enter to select, type to filter, ? for more help]
> 172.21.200.20 # Specify the floating IP address to use for external access to the OpenShift API
  172.21.200.22 
? FlavorName  [Use arrows to move, enter to select, type to filter, ? for more help]
  m1.large
  m1.magnum
  m1.medium
> m1.openshift # Specify a RHOSP flavor with at least 16 GB RAM to use for control plane and compute nodes.
  m1.small
  m1.tiny
  m1.xlarge
? Base Domain [? for help] mycluster.com # Select the base domain to deploy the cluster to
? Cluster Name ocp # Enter a name for your cluster. The name must be 14 or fewer characters long.
? Pull Secret [? for help] <paste-pull-secret>
INFO Install-Config created in: ocp
```

File creation

```sh
ls ocp/
install-config.yaml
```

You can edit to customize further:

```sh
$ vim ocp/install-config.yaml
```

- Confirm that Floating IPs are added to the _install-config.yaml_ file as the values of the following parameters:

```sh
platform.openstack.ingressFloatingIP
platform.openstack.apiFloatingIP
```

Example:

```sh
...
platform:
  openstack:
    apiFloatingIP: 172.21.200.20
    ingressFloatingIP: 172.21.200.22
    apiVIP: 10.0.0.5
    cloud: osp1
```

Also add ssh public key

```sh
vim ocp/install-config.yaml
...
sshKey: replace-me-with-ssh-pub-key-contents
```

If you do not have an SSH key that is configured for password-less authentication on your computer, create one:

```sh
$ ssh-keygen -t ed25519 -N '' -f <path>/<file_name>
```
## Step 7: Deploy OKD / OpenShift Cluster on OpenStack

Change to the directory that contains the installation program and backup up the _install-config.yaml_ file:

```sh
cp install-config.yaml install-config.yaml.bak
```

Initialize the cluster deployment:

```sh
openshift-install create cluster --dir=ocp --log-level=info 
INFO Credentials loaded from file "/root/.config/openstack/clouds.yaml"
INFO Consuming Install Config from target directory
INFO Obtaining RHCOS image file from 'https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/33.20210217.3.0/x86_64/fedora-coreos-33.20210217.3.0-openstack.x86_64.qcow2.xz?sha256=ae088d752a52859ad38c53c29090efd5930453229ef6d1204645916aab856fb1'
INFO The file was found in cache: /root/.cache/openshift-installer/image_cache/41b2fca6062b458e4d5157ca9e4666f2. Reusing...
INFO Creating infrastructure resources...
INFO Waiting up to 20m0s for the Kubernetes API at https://api.ocp.mycluster.com:6443...
INFO API v1.26.5-1073+df9c8387b2dc23-dirty up
INFO Waiting up to 30m0s for bootstrapping to complete...
INFO Destroying the bootstrap resources...
INFO Waiting up to 40m0s for the cluster at https://api.ocp.mycluster.com:6443 to initialize...
INFO Waiting up to 10m0s for the openshift-console route to be created...
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/root/okd/ocp/auth/kubeconfig'
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.ocp.mycluster.com
INFO Login to the console with user: "kubeadmin", and password: "33yzG-Ogiup-huGI9"
INFO Time elapsed: 42m39s
```

Listing created servers on OpenStack:

```sh
openstack server list --column  Name --column Networks --column Status
+--------------------------------------+--------+---------------------------------------+
| Name                                 | Status | Networks                              |
+--------------------------------------+--------+---------------------------------------+
| ocp-nlrnw-worker-0-nz2ch             | ACTIVE | ocp-nlrnw-openshift=10.0.1.197        |
| ocp-nlrnw-worker-0-kts42             | ACTIVE | ocp-nlrnw-openshift=10.0.0.201        |
| ocp-nlrnw-worker-0-92kvf             | ACTIVE | ocp-nlrnw-openshift=10.0.2.197        |
| ocp-nlrnw-master-2                   | ACTIVE | ocp-nlrnw-openshift=10.0.3.167        |
| ocp-nlrnw-master-1                   | ACTIVE | ocp-nlrnw-openshift=10.0.1.83         |
| ocp-nlrnw-master-0                   | ACTIVE | ocp-nlrnw-openshift=10.0.0.139        |
+--------------------------------------+--------+---------------------------------------+
```

Export the cluster access config file:

```sh
export KUBECONFIG=ocp/auth/kubeconfig
```

You can as well make it default kubeconfig:

```sh
cp ocp/auth/kubeconfig ~/.kube/config
```

List available nodes in the cluster

```sh
oc get nodes
NAME                       STATUS   ROLES    AGE     VERSION
ocp-nlrnw-master-0         Ready    master   3h48m   v1.26.5
ocp-nlrnw-master-1         Ready    master   3h48m   v1.26.5
ocp-nlrnw-master-2         Ready    master   3h48m   v1.26.5
ocp-nlrnw-worker-0-92kvf   Ready    worker   3h33m  v1.26.5
ocp-nlrnw-worker-0-kts42   Ready    worker   3h33m  v1.26.5
ocp-nlrnw-worker-0-nz2ch   Ready    worker   3h33m  v1.26.5
```

View your cluster cluster’s version:

```sh
oc get clusterversion
NAME      VERSION                         AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.13.0-0.okd-2023-06-04-080300   True        False         3h16m   Cluster version is 4.13.0-0.okd-2023-06-04-080300
```

Confirm that all cluster operators are available and none is degraded:

```sh
oc get clusteroperator
NAME                                       VERSION                          AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.13.0-0.okd-2023-06-04-080300   True        False         False      3h24m
baremetal                                  4.13.0-0.okd-2023-06-04-080300   True        False         False      3h52m
cloud-credential                           4.13.0-0.okd-2023-06-04-080300   True        False         False      3h57m
cluster-autoscaler                         4.13.0-0.okd-2023-06-04-080300   True        False         False      3h51m
config-operator                            4.13.0-0.okd-2023-06-04-080300   True        False         False      3h52m
console                                    4.13.0-0.okd-2023-06-04-080300   True        False         False      3h31m
csi-snapshot-controller                    4.13.0-0.okd-2023-06-04-080300   True        False         False      3h52m
dns                                        4.13.0-0.okd-2023-06-04-080300   True        False         False      3h51m
etcd                                       4.13.0-0.okd-2023-06-04-080300   True        False         False      3h51m
image-registry                             4.13.0-0.okd-2023-06-04-080300   True        False         False      3h37m
ingress                                    4.13.0-0.okd-2023-06-04-080300   True        False         False      3h38m
insights                                   4.13.0-0.okd-2023-06-04-080300   True        False         False      3h45m
kube-apiserver                             4.13.0-0.okd-2023-06-04-080300   True        False         False      3h49m
kube-controller-manager                    4.13.0-0.okd-2023-06-04-080300   True        False         False      3h50m
kube-scheduler                             4.13.0-0.okd-2023-06-04-080300   True        False         False      3h49m
kube-storage-version-migrator              4.13.0-0.okd-2023-06-04-080300   True        False         False      3h37m
machine-api                                4.13.0-0.okd-2023-06-04-080300   True        False         False      3h46m
machine-approver                           4.13.0-0.okd-2023-06-04-080300   True        False         False      3h51m
machine-config                             4.13.0-0.okd-2023-06-04-080300   True        False         False      3h50m
marketplace                                4.13.0-0.okd-2023-06-04-080300   True        False         False      3h50m
monitoring                                 4.13.0-0.okd-2023-06-04-080300   True        False         False      3h37m
network                                    4.13.0-0.okd-2023-06-04-080300   True        False         False      3h52m
node-tuning                                4.13.0-0.okd-2023-06-04-080300   True        False         False      3h50m
openshift-apiserver                        4.13.0-0.okd-2023-06-04-080300   True        False         False      3h45m
openshift-controller-manager               4.13.0-0.okd-2023-06-04-080300   True        False         False      3h44m
openshift-samples                          4.13.0-0.okd-2023-06-04-080300   True        False         False      3h43m
operator-lifecycle-manager                 4.13.0-0.okd-2023-06-04-080300   True        False         False      3h52m
operator-lifecycle-manager-catalog         4.13.0-0.okd-2023-06-04-080300   True        False         False      3h52m
operator-lifecycle-manager-packageserver   4.13.0-0.okd-2023-06-04-080300   True        False         False      3h46m
service-ca                                 4.13.0-0.okd-2023-06-04-080300   True        False         False      3h52m
storage                                    4.13.0-0.okd-2023-06-04-080300   True        False         False      3h50m
```

You can always print OpenShift Login Console using the command:

```sh
oc whoami --show-console
https://console-openshift-console.apps.ocp.mycluster.com
```

You can then login using URL printed out:

![okd console login](https://computingforgeeks.com/wp-content/uploads/2021/05/okd-console-login-1024x562.png?ezimgfmt=rs:696x382/rscb23/ng:webp/ngcb23 "How To Install OKD OpenShift 4.13 Cluster on OpenStack 5")
## Step 8: Configure HTPasswd Identity Provider

By default you’ll login in as a temporary administrative user and you need to update the cluster OAuth configuration to allow others to log in. Refer to guide in the link below:

- [Manage OpenShift / OKD Users with HTPasswd Identity Provider](https://computingforgeeks.com/manage-openshift-okd-cluster-users-using-htpasswd-identity-provider/)
## Uninstalling OKD / OpenShift Cluster

For you to destroy Cluster created on OpenStack you’ll need to have:

- A copy of the installation program that you used to deploy the cluster.
- Files that the installation program generated when you created your cluster.

A cluster can then be destroyed using the command below:

```sh
openshift-install destroy cluster --dir=<installation_directory> --log-level=info 
```

You can optionally delete the directory and the OpenShift Container Platform installation program.

**_Books For Learning Kubernetes Administration:_**

- [Best Kubernetes Study books](https://computingforgeeks.com/best-kubernetes-study-books/)

More OpenStack related guides:

- [Create Kubernetes Cluster on OpenStack Magnum with Fedora CoreOS](https://computingforgeeks.com/create-kubernetes-cluster-on-openstack-magnum-with-fedora-coreos/)
- [Upgrade Kubernetes Cluster on OpenStack Magnum](https://computingforgeeks.com/upgrade-kubernetes-cluster-on-openstack-magnum/)
- [Scale up Worker Nodes in OpenStack Magnum Kubernetes Cluster](https://computingforgeeks.com/scale-up-worker-nodes-in-openstack-magnum-kubernetes/)