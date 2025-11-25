---
tags:
  - virtualisierung
  - kvm
  - kubernetes
  - flatcar
  - container
---
A Container is a standard way of packaging an application and all its dependencies for running across multiple or different environments seamlessly. Containers will contain the application code, system tools, run time, libraries, and all the settings required to run the app. Having multiple containers without a management platform will not give you the desired capabilities required for running microservices in an aggressive production environment. For this reason Kubernetes came into existence.

On the flip side, Kubernetes is a container management solution with the goal of automating deployment of containers, its scaling, descaling, load balancing among many other operations. The two major server components of Kubernetes is the Control Plane and Worker nodes. In this article we discuss in detail the process of deploying both control plane nodes and worker nodes using Flatcar Container Linux, on a KVM powered virtualized infrastructure.

Our Lab setup will be based on the following systems and hostnames

|**_Server Role_**|**_Hostname_**|**_IP Address_**|**_OS_**|
|---|---|---|---|
|Control Plane / Etcd|master01.k8s.cloudlabske.io|192.168.1.10|Flatcar Container Linux|
|Worker Node|node01.k8s.cloudlabske.io|192.168.1.13|Flatcar Container Linux|
|Worker Node|node02.k8s.cloudlabske.io|192.168.1.14|Flatcar Container Linux|
|Worker Node|node03.k8s.cloudlabske.io|192.168.1.15|Flatcar Container Linux|

For the purpose of this article, we’ve created a DNS zone **_k8s.cloudlabske.io_** in our DNS server (FreeIPA). We shall configure **A** records in the zone. If you have a different type of DNS server, configure it as per its official documentation.
## Step 1 – Prepare KVM Infrastructure

KVM is an alimentation in the delivery of this article. If you’re doing this setup from scratch, refer to our articles for KVM installation and configurations.

- [Install KVM on CentOS / RHEL / Ubuntu / Debian / SLES / Arch](https://computingforgeeks.com/install-kvm-centos-rhel-ubuntu-debian-sles-arch/)

After the installation check to confirm if virtualization extensions are enabled in BIOS for your CPU.

```sh
egrep --color 'vmx|svm' /proc/cpuinfo
```

See if you see below lines in the output;

- **_vmx_** – Intel VT-x, means virtualization support enabled in BIOS.
- **svm** – AMD SVM, means virtualization enabled in BIOS.

Other commands that can be used to validate:

```sh
### Debian based systems ###
sudo kvm-ok

### RHEL Based systems ###
sudo virt-host-validate
```
### Create Libvirt network

We’ll consider two options on network creation for this setup. One will be a local NAT network, and another being a bridge on an actual network interface.
#### Option 1: Using NAT network

When you install and start libvirtd service on a virtualization host, an initial virtual network configuration is created in network address translation (NAT) mode. By default, all VMs on the host are connected to the same libvirt virtual network, named **_default_**.

```sh
$ sudo  virsh  net-list
 Name         State    Autostart   Persistent
-----------------------------------------------
 default      active   yes         yes
```

In NAT mode network, the Virtual Machines on the network can connect to locations outside the host but are not visible to them. Outbound traffic is affected by the NAT rules, as well as the host system’s firewall. You can use the **default** network or create another NATed network on KVM.
##### Creating new NAT Libvirt network

Create a new files called **k8s-network.xml**

```sh
vim k8s-network.xml
```

Paste and edits given network configuration content. This uses **192.168.120.0/24** network.

```xml
  <network>
    <name>k8s-network</name>
    <forward mode='nat'>
      <nat>
        <port start='1024' end='65535'/>
      </nat>
    </forward>
    <bridge name='k8s' stp='on' delay='0' />
    <ip address='192.168.120.1' netmask='255.255.255.0'>
      <dhcp>
        <range start='192.168.120.2' end='192.168.120.254' />
      </dhcp>
    </ip>
  </network>
```

Create libvirt network and start it.

```sh
sudo virsh net-define k8s-network.xml
sudo virsh net-start  k8s-network
sudo virsh net-autostart k8s-network
```

Confirm this was successful.

```sh
ip address show dev k8s
50: k8s: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:1f:e8:22 brd ff:ff:ff:ff:ff:ff
    inet 192.168.120.1/24 brd 192.168.120.255 scope global k8s
       valid_lft forever preferred_lft forever

sudo virsh net-list
 Name          State    Autostart   Persistent
------------------------------------------------
 default       active   yes         yes
 k8s-network   active   yes         yes
```
#### Option 2: Using Bridged network

For the Virtual Machines to appear on the same external network as the hypervisor, you must use bridged mode instead. We assume you already have a bridge created on your host system. Below are host configurations to be used for reference.

```sh
### NIC and Bridge configurations on RHEL based system ###
sudo vim /etc/sysconfig/network-scripts/ifcfg-enp89s0
TYPE=Ethernet
NAME=enp89s0
DEVICE=enp89s0
ONBOOT=yes
BRIDGE=bridge0

sudo vim /etc/sysconfig/network-scripts/ifcfg-bridge0
STP=no
TYPE=Bridge
NAME=bridge0
DEVICE=bridge0
ONBOOT=yes
AUTOCONNECT_SLAVES=yes
DEFROUTE=yes
IPADDR=172.20.30.7
PREFIX=24
GATEWAY=172.20.30.1
DNS1=172.20.30.252
```

With the bridge on host active we can define libvirt network that uses this bridge.

```sh
$ vim k8s-bridge.xml
<network>
  <name>k8s-bridge</name>
  <forward mode="bridge"/>
  <bridge name="bridge0"/>
</network>
```

Now we can define the network:

```sh
$ virsh net-define k8s-bridge.xml
Network k8s-bridge defined from k8s-bridge.xml

$ virsh net-start k8s-bridge
Network k8s-bridge started

$ virsh net-autostart k8s-bridge
Network k8s-bridge marked as autostarted
```

Checking network state.

```sh
$ sudo virsh net-list --all
 Name         State    Autostart   Persistent
-----------------------------------------------
 default      active   yes         yes
 k8s-bridge   active   yes         yes

$ sudo virsh net-info --network k8s-bridge
Name:           k8s-bridge
UUID:           6aa0e02b-fb37-4513-bc05-8e20004ae1e2
Active:         yes
Persistent:     yes
Autostart:      yes
Bridge:         bridge0
```

If interested in using Open vSwitch refer to guide below:

- [How To Use Open vSwitch Bridge on KVM Virtual Machines](https://computingforgeeks.com/use-open-vswitch-bridge-on-kvm-virtual-machines/)
## Step 2 – Create required DNS records for Kubernetes

Based on the network design in the table shared, we’ll create required DNS settings.
- `master01: 192.168.1.10`
- `node01: 192.168.1.13`
- `node02: 192.168.1.14`
- `node03: 192.168.1.15`

On my FreeIPA server, which is used as DNS server, the following commands were executed to add A records.
```sh
[root@ipa02 ~]# ipa dnsrecord-add k8s.cloudlabske.io --a-create-reverse --a-rec 192.168.1.10 master01
  Record name: master01
  A record: 192.168.1.10

[root@ipa02 ~]# ipa dnsrecord-add k8s.cloudlabske.io --a-create-reverse --a-rec 192.168.1.13 node01
  Record name: node01
  A record: 192.168.1.13

[root@ipa02 ~]# ipa dnsrecord-add k8s.cloudlabske.io --a-create-reverse --a-rec 192.168.1.14 node02
  Record name: node02
  A record: 192.168.1.14

[root@ipa02 ~]# ipa dnsrecord-add k8s.cloudlabske.io --a-create-reverse --a-rec 192.168.1.15 node03
  Record name: node03
  A record: 192.168.1.15
```
Once the records for all the nodes are added we can proceed to VMs deployment.
## Step 3 – Create Flatcar Container Linux VMs

[Flatcar Container Linux](https://flatcar-linux.org/) is designed from the ground up for running container workloads. I chose this operating system to power Kubernetes because of its immutable nature and automated atomic updates.

Terraform is our solution of choice used in provisioning Virtual Machines. Terraform is an open-source and cloud-agnostic IAC tool created by HashiCorp. This powerful provisioning tool is written in the Go language for SysAdmins and DevOps teams to automate different kinds of infrastructure tasks. In this guide we’ll use Terraform to provision Virtual Machine instances on KVM for running Kubernetes.

### 1. Install Terraform

We start by installing **terraform** on KVM node for ease of interaction with the hypervisor.

```sh
### Install Terraform on RHEL based systems ###
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install terraform

### Install Terraform on Fedora ###
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo dnf -y install terraform

### Install Terraform on Ubuntu / Debian ###
sudo apt update
sudo apt install -y gnupg software-properties-common
# Import GPG key
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

#Add repo
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
    https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
    sudo tee /etc/apt/sources.list.d/hashicorp.list

# Install terraform
sudo apt update
sudo apt install terraform
```

Confirm terraform is installed and working by checking its version.

```sh
terraform --version
```

### 2. Download Flatcar KVM base image

In this guide, all Virtual Machine’s files are stored in `/var/lib/libvirt/images/` directory. This is not a mandatory requirement, so feel free to substitute that path if you have another storage pool configured.

Flatcar Container Linux is designed to be updated automatically with different schedules per channel. There are three main channels available:

- Stable channel
- Beta channel
- Alpha channel

For production clusters the Stable channel should be used. We start by downloading the most recent _stable_ disk image:

```sh
wget https://stable.release.flatcar-linux.net/amd64-usr/current/flatcar_production_qemu_image.img.bz2{,.sig}
gpg --verify flatcar_production_qemu_image.img.bz2.sig
bunzip2 flatcar_production_qemu_image.img.bz2
```

Move image file to `/var/lib/libvirt/images`

```sh
sudo mv flatcar_production_qemu_image.img /var/lib/libvirt/images/flatcar_qemu_image.img
```

When you create an instance from the image it’s assigned **5GB** disk, Let’s increase this default size by **30GB**

```
sudo qemu-img resize /var/lib/libvirt/images/flatcar_qemu_image.img +30G
```

### 3. Create Terraform code for VM resources

Create directory that will contain all terraform code.

```sh
mkdir ~/flatcar_tf_libvirt
cd ~/flatcar_tf_libvirt
```

Create variables file with below content. Most don’t have default values as they’ll be set in a separate file.

**variables.tf**

```sh
variable "machines" {
  type        = list(string)
  description = "Machine names, corresponding to machine-NAME.yaml.tmpl files"
}

variable "cluster_name" {
  type        = string
  description = "Cluster name used as prefix for the machine names"
}

variable "cluster_domain" {
  type        = string
  description = "Cluster base domain name, e.g k8scluster.com"
}

variable "ssh_keys" {
  type        = list(string)
  description = "SSH public keys for user 'core'"
}

variable "base_image" {
  type        = string
  description = "Path to unpacked Flatcar Container Linux image flatcar_qemu_image.img (probably after a qemu-img resize IMG +20G)"
}

variable "virtual_memory" {
  type        = number
  default     = 2048
  description = "Virtual RAM in MB"
}

variable "virtual_cpus" {
  type        = number
  default     = 1
  description = "Number of virtual CPUs"
}

variable "storage_pool" {
  type        = string
  description = "Storage pool for VM images"
}

variable "network_name" {
  type        = string
  description = "Libvirt network used by Flatcar instances"
}
```

Create another terraform file that will handle actual resources provisioning.

**kvm-machines.tf**

```sh
terraform {
  required_version = ">= 0.13"
  required_providers {
    libvirt = {
      source  = "dmacvicar/libvirt"
    }
    ct = {
      source  = "poseidon/ct"
    }
    template = {
      source  = "hashicorp/template"
    }
  }
}

# Define libvirt URI
provider "libvirt" {
  uri = "qemu:///system"
  #uri = "qemu+ssh://root@192.168.1.4/system"
}

#resource "libvirt_pool" "flatcar_storage_pool" {
#  name = "${var.cluster_name}-pool"
#  type = "dir"
#  path = "/data/libvirt/${var.cluster_name}-pool"
#}

resource "libvirt_volume" "base" {
  name   = "flatcar-base"
  source = var.base_image
  pool   = var.storage_pool
  format = "qcow2"
}

resource "libvirt_volume" "vm-disk" {
  for_each = toset(var.machines)
  # workaround: depend on libvirt_ignition.ignition[each.key], otherwise the VM will use the old disk when the user-data changes
  name           = "${var.cluster_name}-${each.key}-${md5(libvirt_ignition.ignition[each.key].id)}.qcow2"
  base_volume_id = libvirt_volume.base.id
  pool           = var.storage_pool
  format         = "qcow2"
}

resource "libvirt_ignition" "ignition" {
  for_each = toset(var.machines)
  name     = "${var.cluster_name}-${each.key}-ignition"
  pool     = var.storage_pool
  content  = data.ct_config.vm-ignitions[each.key].rendered
}

resource "libvirt_domain" "machine" {
  for_each = toset(var.machines)
  name     = "${var.cluster_name}-${each.key}"
  vcpu     = var.virtual_cpus
  memory   = var.virtual_memory

  fw_cfg_name     = "opt/org.flatcar-linux/config"
  coreos_ignition = libvirt_ignition.ignition[each.key].id

  disk {
    volume_id = libvirt_volume.vm-disk[each.key].id
  }

  graphics {
    listen_type = "address"
  }

# dynamic IP assignment on the bridge, NAT for Internet access
  network_interface {
    network_name   = var.network_name
    wait_for_lease = true
  }
}

data "ct_config" "vm-ignitions" {
  for_each = toset(var.machines)
  content  = data.template_file.vm-configs[each.key].rendered
}

data "template_file" "vm-configs" {
  for_each = toset(var.machines)
  template = file("${path.module}/machine-${each.key}.yaml.tmpl")

  vars = {
    ssh_keys = jsonencode(var.ssh_keys)
    name     = each.key
    host_name = "${each.key}.${var.cluster_name}.${var.cluster_domain}"
  }
}
```

Define configurations for outputting the IP addresses of created virtual machines.

**outputs.tf**

```sh
output "ip-addresses" {
  value = {
    for key in var.machines :
    "${var.cluster_name}-${key}" => libvirt_domain.machine[key].network_interface.0.addresses.*
  }
}
```

#### Define actual variables for this environment

Create a file called `terraform.tfvars` with the values for the variables defined in `variables.tf`

```sh
$ terraform.tfvars
storage_pool   = "default"
network_name   = "k8s-bridge"
base_image     = "/var/lib/libvirt/images/flatcar_qemu_image.img"
cluster_name   = "k8s"
cluster_domain = "example.com"
machines       = ["master01","node01","node02","node03"]
virtual_memory = 4096
virtual_cpus   = 1
ssh_keys       = ["REPLACE-WITH-YOUR-SSH-PUBKEY-HERE"]
```
#### Create each Virtual Machine configuration file.

The format used is _machine-machine-name.yaml.tmpl_

- For Master node – **_master01_**

**machine-master01.yaml.tmpl**

```yaml
---
passwd:
  users:
    - name: core
      ssh_authorized_keys: ${ssh_keys}

storage:
  files:
    - path: /etc/hostname
      filesystem: "root"
      contents:
        inline: ${host_name}
    - path: /home/core/works
      filesystem: root
      mode: 0755
      contents:
        inline: |
          #!/bin/bash
          set -euo pipefail
          echo My name is ${name} and the hostname is ${host_name}
```

- For Worker node 01 – **_**_node01_**_**
**machine-node01.yaml.tmpl**

```yaml
---
passwd:
  users:
    - name: core
      ssh_authorized_keys: ${ssh_keys}

storage:
  files:
    - path: /etc/hostname
      filesystem: "root"
      contents:
        inline: ${host_name}
    - path: /home/core/works
      filesystem: root
      mode: 0755
      contents:
        inline: |
          #!/bin/bash
          set -euo pipefail
          echo My name is ${name} and the hostname is ${host_name}
```

- Create other files as dictated by the desired number of nodes in your cluster.

#### Using Static IP address on VM instances

If you don’t want to use DHCP for IP address assignment on your Flatcar instances, there is an option to write your own networkd units to replace or override the units created automatically. Any network unit injected via a Container Linux Config is written to the system before networkd is started.

See machine configuration example of a master node which manually sets IP address on the interface **eth0.**

**machine-master01.yaml.tmpl**

```sh
---
passwd:
  users:
    - name: core
      ssh_authorized_keys: ${ssh_keys}

networkd:
  units:
  - name: 00-eth0.network
    contents: |
      [Match]
      Name=eth0

      [Network]
      Address=192.168.1.10
      Gateway=192.168.1.1
      DNS=172.20.30.252

storage:
  files:
    - path: /etc/hostname
      filesystem: "root"
      contents:
        inline: ${host_name}
    - path: /home/core/works
      filesystem: root
      mode: 0755
      contents:
        inline: |
          #!/bin/bash
          set -euo pipefail
          echo My name is ${name} and the hostname is ${host_name}
```

In the network unit be sure to modify the `[Match]` section with the name of your desired interface, and replace the IPs accordingly.

### 4. Create Flatcar VM instances using Terraform

Let’s initialize Terraform by running the following commands:

_terraform init_

```sh
terraform init
Initializing the backend...

Initializing provider plugins...
- Finding latest version of dmacvicar/libvirt...
- Finding latest version of poseidon/ct...
- Finding latest version of hashicorp/template...
- Installing hashicorp/template v2.2.0...
- Installed hashicorp/template v2.2.0 (signed by HashiCorp)
- Installing dmacvicar/libvirt v0.7.0...
- Installed dmacvicar/libvirt v0.7.0 (self-signed, key ID 96B1FE1A8D4E1EAB)
- Installing poseidon/ct v0.11.0...
- Installed poseidon/ct v0.11.0 (self-signed, key ID 8F515AD1602065C8)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Show changes required by the current configuration without applying:

```
terraform plan
```

To create infrastructure resources use `apply` command option.

```sh
$ terraform apply
data.template_file.vm-configs["node03"]: Reading...
data.template_file.vm-configs["node01"]: Reading...
data.template_file.vm-configs["node02"]: Reading...
data.template_file.vm-configs["master01"]: Reading...
data.template_file.vm-configs["node02"]: Read complete after 0s [id=347f4516615bce79b0de8b99eba7668078de8872c8670432fa5f85e4dffcfa45]
data.template_file.vm-configs["node01"]: Read complete after 0s [id=13d3b7c55d7ab2addc152e1d0e1d8669362e79156b5ceadfa52ec13599a14c8d]
data.template_file.vm-configs["node03"]: Read complete after 0s [id=12edb1bbf2a77ce62dbfff63007dad5409829bd3130ef83f2cd2853f275ad77c]
data.template_file.vm-configs["master01"]: Read complete after 0s [id=7b0e651619d66b319a3a8835e861e9d2c56d11cff9c7c888e7ec58bfda822c3f]
data.ct_config.vm-ignitions["node03"]: Reading...
data.ct_config.vm-ignitions["master01"]: Reading...
data.ct_config.vm-ignitions["node02"]: Reading...
data.ct_config.vm-ignitions["node01"]: Reading...
data.ct_config.vm-ignitions["node02"]: Read complete after 0s [id=2011962698]
data.ct_config.vm-ignitions["node01"]: Read complete after 0s [id=2145228696]
data.ct_config.vm-ignitions["master01"]: Read complete after 0s [id=303298983]
data.ct_config.vm-ignitions["node03"]: Read complete after 0s [id=3335756091]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
.....
```

Approve resources creation as requested using `yes`

```sh
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

If you’re using network with DHCP on the VMs, IP addresses will be printed after execution as configured in the `outputs.tf` file.

```sh
Apply complete! Resources: 13 added, 0 changed, 0 destroyed.

Outputs:

ip-addresses = {
  "k8s-master01" = tolist([
    "192.168.120.240",
  ])
  "k8s-node01" = tolist([
    "192.168.120.73",
  ])
  "k8s-node02" = tolist([
    "192.168.120.138",
  ])
  "k8s-node03" = tolist([
    "192.168.120.151",
  ])
}
```

On KVM Node, make the virtual machines to start up on system boot.

```sh
root@spinx01:~/k8s_flatcar# virsh list
 Id   Name           State
------------------------------
 2    k8s-node01     running
 3    k8s-master01   running
 4    k8s-node03     running
 5    k8s-node02     running

[root@kvm02 ~]# virsh autostart k8s-master01
Domain 'k8s-master01' marked as autostarted

[root@kvm02 ~]# virsh autostart k8s-node01
Domain 'k8s-node01' marked as autostarted

[root@kvm02 ~]# virsh autostart k8s-node02
Domain 'k8s-node02' marked as autostarted

[root@kvm02 ~]# virsh autostart k8s-node03
Domain 'k8s-node03' marked as autostarted
```

We can test SSH access to the instance with **core** user account and SSH key used while provisioning.

```sh
$ ssh -i ~/.ssh/id_rsa core@192.168.120.240
The authenticity of host '192.168.120.240 (192.168.120.240)' can't be established.
ED25519 key fingerprint is SHA256:VEciL+1+4H1CWvTL+fyYFUohHsP/K0H3D2GhLqaXxn4.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.120.240' (ED25519) to the list of known hosts.
Last login: Tue Nov 29 17:30:03 UTC 2022 on tty1
Flatcar Container Linux by Kinvolk stable 3227.2.4 for QEMU

core@master01 ~ $ cat /etc/os-release
NAME="Flatcar Container Linux by Kinvolk"
ID=flatcar
ID_LIKE=coreos
VERSION=3227.2.4
VERSION_ID=3227.2.4
BUILD_ID=2022-10-27-1321
SYSEXT_LEVEL=1.0
PRETTY_NAME="Flatcar Container Linux by Kinvolk 3227.2.4 (Oklo)"
ANSI_COLOR="38;5;75"
HOME_URL="https://flatcar.org/"
BUG_REPORT_URL="https://issues.flatcar.org"
FLATCAR_BOARD="amd64-usr"
CPE_NAME="cpe:2.3:o:flatcar-linux:flatcar_linux:3227.2.4:*:*:*:*:*:*:*"
```

## Step 4 – Deploy Kubernetes Cluster using Kubespray

Kubespray is a composition of Ansible playbooks, inventory, and provisioning tools for Kubernetes clusters. It automates the whole process of installing and configuring Kubernetes pieces. Once the instances have been provisioned, you only need to configure inventory and you’re good to deploy Kubernetes using kubespray.

Ansible has to be installed prior to using Kubespray playbooks to deploy Kubernetes cluster. Please note that the ansible must be between **_2.11.0 and 2.13.0_** versions.

### Install Ansible on Debian based systems

Install Python3 and Pip3 package:

```sh
sudo apt update
sudo apt install python3 python3-pip
```

Install Ansible using pip Python package manager.

```sh
python3 -m pip install --user ansible-core
pip3 install netaddr --user
echo 'export PATH=$PATH:~/.local/bin/' | tee -a ~/.bashrc
source ~/.bashrc
```

### Install Ansible on RHEL based systems

Install Python3 pip package.

```sh
sudo yum -y install epel-release 
sudo yum -y install python39 python39-pip
```

Install ansible automation tool on the system.

```sh
python3.9 -m pip install --user ansible-core
echo 'export PATH=$PATH:~/.local/bin/' | tee -a ~/.bashrc
source ~/.bashrc
```

Use `pip3` to install netaddr Python module.

```sh
$ pip3.9 install netaddr --user
Collecting netaddr
  Downloading https://files.pythonhosted.org/packages/ff/cd/9cdfea8fc45c56680b798db6a55fa60a22e2d3d3ccf54fc729d083b50ce4/netaddr-0.8.0-py2.py3-none-any.whl (1.9MB)
     |████████████████████████████████| 1.9MB 18kB/s
Installing collected packages: netaddr
Successfully installed netaddr-0.8.0
```

Check installation of Ansible if it was successful.

```sh
$ ansible --version
ansible [core 2.13.0]
  config file = None
  configured module search path = ['/home/jkmutai/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/jkmutai/.local/lib/python3.9/site-packages/ansible
  ansible collection location = /home/jkmutai/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/jkmutai/.local/bin/ansible
  python version = 3.9.13 (main, Nov 16 2022, 15:31:39) [GCC 8.5.0 20210514 (Red Hat 8.5.0-15)]
  jinja version = 3.1.2
  libyaml = True
```

### Download latest kubespray from github

Download the latest release kubespray project from Github.

```sh
VER=$(curl -s https://api.github.com/repos/kubernetes-sigs/kubespray/releases/latest|grep tag_name|cut -d '"' -f 4|sed 's/v//')
wget https://github.com/kubernetes-sigs/kubespray/archive/refs/tags/v$VER.tar.gz
```

Extract download archive file.

```sh
tar xvf v$VER.tar.gz
cd kubespray-$VER/
```

Copy sample inventory to your desired file that will contain server ips / hostnames.

```sh
cp -rfp inventory/sample inventory/k8scluster
```

Edit default inventory in the folder and provide the IP addresses of your servers, and ssh user.

```sh
$ vim inventory/k8scluster/inventory.ini
# ## Configure 'ip' variable to bind kubernetes services on a
# ## different ip than the default iface
# ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.
[all]
master01 ansible_host=192.168.1.10 etcd_member_name=etcd1   ansible_user=core
node01   ansible_host=192.168.1.13 etcd_member_name=            ansible_user=core
node02   ansible_host=192.168.1.14 etcd_member_name=            ansible_user=core
node03   ansible_host=192.168.1.15 etcd_member_name=            ansible_user=core

# ## configure a bastion host if your nodes are not directly reachable
# [bastion]
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube_control_plane]
master01

[etcd]
master01

[kube_node]
node01
node02
node03

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
```

For **Flatcar Linux** we need to set binaries directory to `/opt/bin`

```sh
$ vim inventory/k8scluster/group_vars/all/all.yml
bin_dir: /opt/bin
```

Finally install Kubernetes in your machines by running the commands given below.

```sh
ansible-playbook -i inventory/k8scluster/inventory.ini  --become --become-user=root cluster.yml
```

Execution output example;

```sh
....
TASK [network_plugin/calico : Check calico network backend defined correctly] ************************************************************************************************************************
ok: [master01] => {
    "changed": false,
    "msg": "All assertions passed"
}
Monday 14 November 2022  18:35:06 +0300 (0:00:00.025)       1:56:09.512 *******

TASK [network_plugin/calico : Check ipip and vxlan mode defined correctly] ***************************************************************************************************************************
ok: [master01] => {
    "changed": false,
    "msg": "All assertions passed"
}
Monday 14 November 2022  18:35:06 +0300 (0:00:00.026)       1:56:09.539 *******
Monday 14 November 2022  18:35:06 +0300 (0:00:00.020)       1:56:09.560 *******

TASK [network_plugin/calico : Check ipip and vxlan mode if simultaneously enabled] *******************************************************************************************************************
ok: [master01] => {
    "changed": false,
    "msg": "All assertions passed"
}
Monday 14 November 2022  18:35:06 +0300 (0:00:00.027)       1:56:09.587 *******

TASK [network_plugin/calico : Get Calico default-pool configuration] *********************************************************************************************************************************
ok: [master01]
Monday 14 November 2022  18:35:07 +0300 (0:00:00.323)       1:56:09.910 *******

TASK [network_plugin/calico : Set calico_pool_conf] **************************************************************************************************************************************************
ok: [master01]
Monday 14 November 2022  18:35:07 +0300 (0:00:00.026)       1:56:09.937 *******

TASK [network_plugin/calico : Check if inventory match current cluster configuration] ****************************************************************************************************************
ok: [master01] => {
    "changed": false,
    "msg": "All assertions passed"
}
Monday 14 November 2022  18:35:07 +0300 (0:00:00.030)       1:56:09.967 *******
Monday 14 November 2022  18:35:07 +0300 (0:00:00.020)       1:56:09.988 *******
Monday 14 November 2022  18:35:07 +0300 (0:00:00.019)       1:56:10.007 *******

PLAY RECAP *******************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
master01                   : ok=684  changed=127  unreachable=0    failed=0    skipped=1261 rescued=0    ignored=10
node01                     : ok=461  changed=79   unreachable=0    failed=0    skipped=751  rescued=0    ignored=3

Monday 14 November 2022  18:35:07 +0300 (0:00:00.035)       1:56:10.043 *******
===============================================================================
container-engine/containerd : download_file | Download item -------------------------------------------------------------------------------------------------------------------------------- 1198.28s
download : download_file | Download item --------------------------------------------------------------------------------------------------------------------------------------------------- 1082.30s
bootstrap-os : Run bootstrap.sh ------------------------------------------------------------------------------------------------------------------------------------------------------------- 906.86s
download : download_file | Download item ---------------------------------------------------------------------------------------------------------------------------------------------------- 740.89s
container-engine/crictl : download_file | Download item ------------------------------------------------------------------------------------------------------------------------------------- 325.12s
download : download_file | Download item ---------------------------------------------------------------------------------------------------------------------------------------------------- 276.23s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------- 261.87s
container-engine/runc : download_file | Download item --------------------------------------------------------------------------------------------------------------------------------------- 222.89s
container-engine/nerdctl : download_file | Download item ------------------------------------------------------------------------------------------------------------------------------------ 216.89s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------- 179.04s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------- 146.23s
download : download_file | Download item ---------------------------------------------------------------------------------------------------------------------------------------------------- 142.42s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------- 134.31s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------- 131.73s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------- 119.94s
download : download_file | Download item ----------------------------------------------------------------------------------------------------------------------------------------------------- 87.45s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------- 79.16s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------- 70.60s
download : download_file | Download item ----------------------------------------------------------------------------------------------------------------------------------------------------- 55.45s
download : download_file | Download item ----------------------------------------------------------------------------------------------------------------------------------------------------- 54.73s
```

Login to one of the cluster machines

```sh
[root@kvm02 ~]# ssh core@192.168.1.10
Warning: Permanently added '192.168.1.10' (ECDSA) to the list of known hosts.
Last login: Mon Nov 14 15:34:55 UTC 2022 from 172.20.30.7 on pts/1
Flatcar Container Linux by Kinvolk stable 3227.2.4 for QEMU
```

If you check VM network configurations you should see extras created specifically for Kubernetes.

```sh
core@master01 ~ $ ip ad
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:f2:1e:80 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    altname ens3
    inet 192.168.1.10/24 brd 192.168.1.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fef2:1e80/64 scope link
       valid_lft forever preferred_lft forever
3: kube-ipvs0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
    link/ether 8e:54:46:c6:8f:4e brd ff:ff:ff:ff:ff:ff
    inet 10.233.0.1/32 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet 10.233.0.3/32 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet6 fe80::8c54:46ff:fec6:8f4e/64 scope link
       valid_lft forever preferred_lft forever
6: vxlan.calico: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default
    link/ether 66:58:bc:42:2d:24 brd ff:ff:ff:ff:ff:ff
    inet 10.233.106.128/32 scope global vxlan.calico
       valid_lft forever preferred_lft forever
    inet6 fe80::6458:bcff:fe42:2d24/64 scope link
       valid_lft forever preferred_lft forever
7: calibd7efcc9118@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1430 qdisc noqueue state UP group default
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-92169ecf-2b7a-6e15-2206-106b881c511d
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link
       valid_lft forever preferred_lft forever
```

On the master / control plane node(s) the API server should be bound on port **6443.**

```sh
core@master01 ~ $ ss -tunelp | grep 6443
tcp   LISTEN 0      4096               *:6443             *:*    ino:68521 sk:b cgroup:/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pode485194365f749742f6561a2a30dd22f.slice/cri-containerd-5fd344a173e0f685ec1e6e4c1913a553e82ce7796123e2544963bfe41dbcab83.scope v6only:0 <->
```

Listing containers with `crictl` command.

```sh
core@master01 ~ $ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
22baa69307e69       5bae806f8f123       41 minutes ago      Running             node-cache                0                   5375710eddbb6
6298463e02dfd       5f5175f39b19e       41 minutes ago      Running             calico-node               2                   912839af5ff1b
85a63da250bd3       a4ca41631cc7a       41 minutes ago      Running             coredns                   2                   d8b09bccff2fd
06a4f642ed30c       0bb39497ab33b       41 minutes ago      Running             kube-proxy                2                   2f4aba2bcaead
96acf475f613d       c6c20157a4233       41 minutes ago      Running             kube-controller-manager   3                   30b1c434a88b1
ce487082e132c       c786c777a4e1c       41 minutes ago      Running             kube-scheduler            3                   82af8ac4cb38d
1158f5dea2e7b       860f263331c95       41 minutes ago      Running             kube-apiserver            2                   c5fbc56dda322
```

### Copy kubeconfig file

The cluster access details and credentials are stored in `/etc/kubernetes/admin.conf`

```sh
core@master01 ~ $ ls /etc/kubernetes/admin.conf
```

Create `~/.kube` directory and copy the file.

```sh
core@master01 ~ $ mkdir -p ~/.kube
core@master01 ~ $ sudo cp /etc/kubernetes/admin.conf ~/.kube/config
core@master01 ~ $ sudo chown core:core ~/.kube/config
```

You should be able to query your cluster using `kubectl` command line tool.

```sh
core@master01 ~ $ kubectl  get nodes
NAME       STATUS   ROLES           AGE     VERSION
master01   Ready    control-plane   6h2m    v1.24.6
node01     Ready    <none>          6h1m    v1.24.6
node02     Ready    <none>          5m49s   v1.24.6
node03     Ready    <none>          5m49s   v1.24.6
```

If you need to get extra OS details pass the `-o wide` into the command.

```sh
core@master01 ~ $ kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                                             KERNEL-VERSION    CONTAINER-RUNTIME
master01   Ready    control-plane   27h   v1.24.6   192.168.1.10   <none>        Flatcar Container Linux by Kinvolk 3227.2.4 (Oklo)   5.15.70-flatcar   containerd://1.6.8
node01     Ready    <none>          27h   v1.24.6   192.168.1.13   <none>        Flatcar Container Linux by Kinvolk 3227.2.4 (Oklo)   5.15.70-flatcar   containerd://1.6.8
node02     Ready    <none>          21h   v1.24.6   192.168.1.14   <none>        Flatcar Container Linux by Kinvolk 3227.2.4 (Oklo)   5.15.70-flatcar   containerd://1.6.8
node03     Ready    <none>          21h   v1.24.6   192.168.1.15   <none>        Flatcar Container Linux by Kinvolk 3227.2.4 (Oklo)   5.15.70-flatcar   containerd://1.6.8
```

## Step 5 – Install Metrics Server

_Metrics Server_ is a cluster-wide aggregator of resource usage data. It collects metrics from the _Summary API_, exposed by **Kubelet** on each node. Use our guide below to deploy it:

- [How To Deploy Metrics Server to Kubernetes Cluster](https://computingforgeeks.com/how-to-deploy-metrics-server-to-kubernetes-cluster/)

## Step 6 – Deploy Prometheus / Grafana Monitoring

Prometheus is a full fledged solution that enables you to access advanced metrics capabilities in a Kubernetes cluster. Grafana is used for analytics and interactive visualization of metrics that’s collected and stored in Prometheus database. We have a complete guide on how to setup complete monitoring stack on Kubernetes Cluster:

- [Setup Prometheus and Grafana on Kubernetes using prometheus-operator](https://computingforgeeks.com/setup-prometheus-and-grafana-on-kubernetes/)

## Step 7 – Configure Persistent Storage (Optional)

If you need a quick to setup persistent storage on Kubernetes check out our NFS article.

- [Configure NFS as Kubernetes Persistent Volume Storage](https://computingforgeeks.com/configure-nfs-as-kubernetes-persistent-volume-storage/)

Fore more production level solution check rook:

- [How To Deploy Rook Ceph Storage on Kubernetes Cluster](https://computingforgeeks.com/how-to-deploy-rook-ceph-storage-on-kubernetes-cluster/)

## Step 8 – Install Kubernetes Dashboard (optional)

Kubernetes dashboard can be used to deploy containerized applications to a Kubernetes cluster, troubleshoot your containerized application, and manage the cluster resources.

Refer to our guide for installation: [How To Install Kubernetes Dashboard with NodePort](https://computingforgeeks.com/how-to-install-kubernetes-dashboard-with-nodeport/)

## Step 9 – Deploy MetalLB on Kubernetes (optional)

Follow the guide below to install and configure MetalLB on Kubernetes:

- [How To Deploy MetalLB Load Balancer on Kubernetes Cluster](https://computingforgeeks.com/deploy-metallb-load-balancer-on-kubernetes/)

## Step 10 – Install Ingress Controller (optional)

If you need an Ingress controller for Kubernetes workloads, you can use our guide in the following link for the installation process:

- [Deploy Nginx Ingress Controller on Kubernetes using Helm Chart](https://computingforgeeks.com/deploy-nginx-ingress-controller-on-kubernetes-using-helm-chart/)
- [Install and Configure Traefik Ingress Controller on Kubernetes Cluster](https://computingforgeeks.com/install-configure-traefik-ingress-controller-on-kubernetes/)

**_Books For Learning Kubernetes Administration:_**

- [Best Kubernetes Study books](https://computingforgeeks.com/best-kubernetes-study-books/)
