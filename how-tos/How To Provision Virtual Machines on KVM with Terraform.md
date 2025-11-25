---
tags:
  - kvm
  - virtualisierung
  - terraform
---
If you’re a fan of terraform and KVM, I’m assured you’ve been looking for a way to provision Virtual Machines on KVM in automated manner with Terraform. In this blog post, I’ll walk you through installation of Terraform KVM provider and using it to manage instances running on KVM hypervisor.

Terraform is an open-source infrastructure as code software tool created by HashiCorp. It allows you to safely and predictably create, change, and improve infrastructure. All your infrastructure code can be saved in a Git repository and versioned.

A provider in Terraform is responsible for the lifecycle of a resource: create, read, update, delete. Hashicorp has a number of officially [supported providers](https://www.terraform.io/docs/providers/index.html) available for use. Unfortunately, KVM is not in the list.
## Installing KVM hypervisor

The major pre-requisite for this setup is KVM hypervisor. Install KVM in your Linux system by referring to a relevant article from the list below.

- [How to install KVM on RHEL / CentOS 8](https://computingforgeeks.com/how-to-install-kvm-on-rhel-8/)
- [How to install KVM on Fedora](https://computingforgeeks.com/how-to-install-kvm-on-fedora/)
- [Install KVM Hypervisor on Ubuntu](https://computingforgeeks.com/install-kvm-hypervisor-on-ubuntu-linux/)
- [How To Install KVM on Debian](https://computingforgeeks.com/how-to-install-kvm-virtualization-on-debian/)
- [Install KVM on CentOS 7 / Ubuntu / Debian / SLES](https://computingforgeeks.com/install-kvm-centos-rhel-ubuntu-debian-sles-arch/)
- [Install KVM on Arch Linux / Manjaro](https://computingforgeeks.com/install-kvm-qemu-virt-manager-arch-manjar/)

**The KVM service (libvird) should be running and enabled to start at boot.**

```sh
sudo systemctl start libvirtdsudo systemctl enable libvirtd
```

**Enable `vhost-net` kernel module on Ubuntu/Debian.**

```sh
sudo modprobe vhost_netecho vhost_net | sudo tee -a /etc/modules
```

If you want to generate KVM VM templates, refer to:

- [Create CentOS / Fedora / RHEL VM Templates on KVM](https://computingforgeeks.com/how-to-create-centos-fedora-rhel-vm-templates-on-kvm/)

## Install Terraform IaC tool

After installing and starting KVM, do Terraform installation.

- [How To Install Terraform on Linux Systems](https://computingforgeeks.com/how-to-install-terraform-on-linux/)

Terraform installation is much easier. You just need to downloaded a binary archive, extract and place the binary file in a directory in your `$PATH`.

## Install Terraform KVM provider

The Terraform KVM provider will provision infrastructure with Linux’s KVM using libvirt. It is maintained by [Duncan Mac-Vicar P](https://github.com/dmacvicar/terraform-provider-libvirt) with other contributors.

The provider is available for auto-installation from the [Terraf](https://registry.terraform.io/providers/dmacvicar/libvirt/latest)[orm Registry](https://registry.terraform.io/providers/dmacvicar/libvirt/latest). In your `main.tf` file, just specify the version you want to use:

```sh
terraform {
  required_providers {
    libvirt = {
      source = "dmacvicar/libvirt"
    }
  }
}

provider "libvirt" {
  # Configuration options
}
```
### Manual installation of Terraform KVM provider (Not necessary)

But if you wish to install it manually, follow the steps provided in this section.

**Install wget, curl and unzip tools**

```sh
# Ubuntu / Debian
sudo apt update
sudo apt install wget curl unzip vim

# RHEL Based systems
sudo yum -y install wget curl unzip vim
```

**Initialize Terraform working directory.**

```sh
$ cd ~
$ terraform init
Terraform initialized in an empty directory!
```

**Create directory for storing Terraform Plugins.**

```sh
cd ~/.terraform.d
mkdir plugins
```

Check the [Github releases](https://github.com/dmacvicar/terraform-provider-libvirt/releases) page for available downloads.

#### Install Terraform KVM provider on Linux

**_Linux 64-bit system:_**

```sh
curl -s https://api.github.com/repos/dmacvicar/terraform-provider-libvirt/releases/latest \
  | grep browser_download_url \
  | grep linux_amd64.zip \
  | cut -d '"' -f 4 \
  | wget -i -
```

**_Linux 32-bit system:_**

```sh
curl -s https://api.github.com/repos/dmacvicar/terraform-provider-libvirt/releases/latest \
  | grep browser_download_url \
  | grep linux_386.zip \
  | cut -d '"' -f 4 \
  | wget -i -
```

**Extract the file downloaded:**

```sh
# 64-bit Linux
unzip terraform-provider-libvirt_*_linux_amd64.zip
rm -f terraform-provider-libvirt_*_linux_amd64.zip

# 32-bit Linux
unzip terraform-provider-libvirt_*_linux_386.zip
rm -f terraform-provider-libvirt_*_linux_386.zip
```

**Move `terraform-provider-libvirt` binary file to the `~/.terraform.d/plugins` directory.**

```sh
mkdir -p ~/.terraform.d/plugins/
mv terraform-provider-libvirt_* ~/.terraform.d/plugins/terraform-provider-libvirt
```
## Using Terraform To Provision VMs on KVM

Once you have provider inside plugins directory. Create your Terraform projects folder.

```sh
mkdir -p ~/projects/terraform
cd ~/projects/terraform
```

**For automatic installation of KVM Provider, define like below:**

```sh
vim main.tf
terraform {
  required_providers {
    libvirt = {
      source = "dmacvicar/libvirt"
    }
  }
}

provider "libvirt" {
  ## Configuration options
  uri = "qemu:///system"
  #alias = "server2"
  #uri   = "qemu+ssh://root@192.168.100.10/system"
}
```

**Thereafter, run terraform init command to initialize the environment:**

```sh
terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of dmacvicar/libvirt...
- Installing dmacvicar/libvirt v0.7.1...
- Installed dmacvicar/libvirt v0.7.1 (self-signed, key ID 96B1FE1A8D4E1EAB)

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

**We can now create `libvirt.tf` file for your VM deployment on KVM.**

```sh
vim libvirt.tf
```

**Here are the file contents we’ll use in our example:**

```sh
# Defining VM Volume
resource "libvirt_volume" "centos7-qcow2" {
  name = "centos7.qcow2"
  pool = "default" # List storage pools using virsh pool-list
  source = "https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2"
  #source = "./CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}

# Define KVM domain to create
resource "libvirt_domain" "centos7" {
  name   = "centos7"
  memory = "2048"
  vcpu   = 2

  network_interface {
    network_name = "default" # List networks with virsh net-list
  }

  disk {
    volume_id = "${libvirt_volume.centos7-qcow2.id}"
  }

  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }
}

# Output Server IP
output "ip" {
  value = "${libvirt_domain.centos7.network_interface.0.addresses.0}"
}
```

**Generate and show Terraform execution plan**

```sh
terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # libvirt_domain.centos7 will be created
  + resource "libvirt_domain" "centos7" {
      + arch        = (known after apply)
      + disk        = [
          + {
              + block_device = null
              + file         = null
              + scsi         = null
              + url          = null
              + volume_id    = (known after apply)
              + wwn          = null
            },
        ]
      + emulator    = (known after apply)
      + fw_cfg_name = "opt/com.coreos/config"
      + id          = (known after apply)
      + machine     = (known after apply)
      + memory      = 2048
      + name        = "centos7"
      + qemu_agent  = false
      + running     = true
      + vcpu        = 2

      + console {
          + source_host    = "127.0.0.1"
          + source_service = "0"
          + target_port    = "0"
          + target_type    = "serial"
          + type           = "pty"
        }

      + graphics {
          + autoport       = true
          + listen_address = "127.0.0.1"
          + listen_type    = "address"
          + type           = "spice"
        }

      + network_interface {
          + addresses    = (known after apply)
          + hostname     = (known after apply)
          + mac          = (known after apply)
          + network_id   = (known after apply)
          + network_name = "default"
        }
    }

  # libvirt_volume.centos7-qcow2 will be created
  + resource "libvirt_volume" "centos7-qcow2" {
      + format = "qcow2"
      + id     = (known after apply)
      + name   = "centos7.qcow2"
      + pool   = "default"
      + size   = (known after apply)
      + source = "https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────
```

Then build your Terraform infrastructure if desired state is confirmed to be correct.

```sh
$ terraform apply
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # libvirt_domain.centos7 will be created
  + resource "libvirt_domain" "centos7" {
      + arch        = (known after apply)
      + disk        = [
          + {
              + block_device = null
              + file         = null
              + scsi         = null
              + url          = null
              + volume_id    = (known after apply)
              + wwn          = null
            },
        ]
      + emulator    = (known after apply)
      + fw_cfg_name = "opt/com.coreos/config"
      + id          = (known after apply)
      + machine     = (known after apply)
      + memory      = 2048
      + name        = "centos7"
      + qemu_agent  = false
      + running     = true
      + vcpu        = 2

      + console {
          + source_host    = "127.0.0.1"
          + source_service = "0"
          + target_port    = "0"
          + target_type    = "serial"
          + type           = "pty"
        }

      + graphics {
          + autoport       = true
          + listen_address = "127.0.0.1"
          + listen_type    = "address"
          + type           = "spice"
        }

      + network_interface {
          + addresses    = (known after apply)
          + hostname     = (known after apply)
          + mac          = (known after apply)
          + network_id   = (known after apply)
          + network_name = "default"
        }
    }

  # libvirt_volume.centos7-qcow2 will be created
  + resource "libvirt_volume" "centos7-qcow2" {
      + format = "qcow2"
      + id     = (known after apply)
      + name   = "centos7.qcow2"
      + pool   = "default"
      + size   = (known after apply)
      + source = "https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

**Press “yes” to confirm execution. Below is my terraform execution output.**

```sh
libvirt_volume.centos7-qcow2: Creating...
  format: "" => "qcow2"
  name:   "" => "db.qcow2"
  pool:   "" => "default"
  size:   "" => "<computed>"
  source: "" => "./CentOS-7-x86_64-GenericCloud.qcow2"
libvirt_volume.centos7-qcow2: Creation complete after 8s (ID: /var/lib/libvirt/images/db.qcow2)
libvirt_domain.centos7: Creating...
  arch:                             "" => "<computed>"
  console.#:                        "" => "1"
  console.0.target_port:            "" => "0"
  console.0.target_type:            "" => "serial"
  console.0.type:                   "" => "pty"
  disk.#:                           "" => "1"
  disk.0.scsi:                      "" => "false"
  disk.0.volume_id:                 "" => "/var/lib/libvirt/images/db.qcow2"
  emulator:                         "" => "<computed>"
  graphics.#:                       "" => "1"
  graphics.0.autoport:              "" => "true"
  graphics.0.listen_address:        "" => "127.0.0.1"
  graphics.0.listen_type:           "" => "address"
  graphics.0.type:                  "" => "spice"
  machine:                          "" => "<computed>"
  memory:                           "" => "1024"
  name:                             "" => "centos7"
  network_interface.#:              "" => "1"
  network_interface.0.addresses.#:  "" => "<computed>"
  network_interface.0.hostname:     "" => "<computed>"
  network_interface.0.mac:          "" => "<computed>"
  network_interface.0.network_id:   "" => "<computed>"
  network_interface.0.network_name: "" => "default"
  qemu_agent:                       "" => "false"
  running:                          "" => "true"
  vcpu:                             "" => "1"
libvirt_domain.centos7: Creation complete after 0s (ID: e5ee28b9-e1da-4945-9eb0-0cda95255937)

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

**Confirm VM creation with `virsh` command.**

```sh
$ sudo virsh  list
 Id   Name       State
--------------------------
 7    centos7    running
```

**Get Instance IP address.**

```sh
$ sudo virsh net-dhcp-leases default 
 Expiry Time           MAC address         Protocol   IP address           Hostname   Client ID or DUID
------------------------------------------------------------------------------------------------------------------------------------------------
 2019-03-24 16:11:18   52:54:00:3e:15:9e   ipv4       192.168.122.61/24    -          -
 2019-03-24 15:30:18   52:54:00:8f:8c:86   ipv4       192.168.122.198/24   rhel8      ff:61:69:21:bd:00:02:00:00:ab:11:0e:9c:c6:63:ee:7d:c8:d1
```

**My instance IP is `192.168.122.61`. I can ping the instance.**

```sh
$  ping -c 1 192.168.122.61 
PING 192.168.122.61 (192.168.122.61) 56(84) bytes of data.
64 bytes from 192.168.122.61: icmp_seq=1 ttl=64 time=0.517 ms

--- 192.168.122.61 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.517/0.517/0.517/0.000 ms
```

**To destroy your infrastructure, run:**

```sh
$ terraform destroy
.....
Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes
```
## Using cloud-init with Terraform Libvirt provider

The instance resource we used didn’t have an option for passing user password. So if you’re using cloud template which doesn’t support password authentication, you won’t be able to . Luckily, we can use [libvirt_cloudinit_disk](https://github.com/dmacvicar/terraform-provider-libvirt/blob/master/website/docs/r/cloudinit.html.markdown) resource to pass user data to the instance.

**Create Cloud init configuration file.**

```sh
$ vim cloud_init.cfg
#cloud-config
# vim: syntax=yaml
#
# ***********************
# 	---- for more examples look at: ------
# ---> https://cloudinit.readthedocs.io/en/latest/topics/examples.html
# ******************************
#
# This is the configuration syntax that the write_files module
# will know how to understand. encoding can be given b64 or gzip or (gz+b64).
# The content will be decoded accordingly and then written to the path that is
# provided.
#
# Note: Content strings here are truncated for example purposes.
ssh_pwauth: True
chpasswd:
  list: |
     root: StrongRootPassword
  expire: False

users:
  - name: jmutai # Change me
    ssh_authorized_keys:
      - ssh-rsa AAAAXX #Chageme
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    shell: /bin/bash
    groups: wheel
```

- This will set root password to `StrongRootPassword`
- Add user named `jmutai` with specified Public SSH keys
- The user will be added to wheel group and be allowed to run sudo commands without password.

**Edit `libvirt.tf` to use Cloud init configuration file.**

```sh
# Defining VM Volume
resource "libvirt_volume" "centos7-qcow2" {
  name = "centos7.qcow2"
  pool = "default"
  source = "https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2"
  #source = "./CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}

# get user data info
data "template_file" "user_data" {
  template = "${file("${path.module}/cloud_init.cfg")}"
}

# Use CloudInit to add the instance
resource "libvirt_cloudinit_disk" "commoninit" {
  name = "commoninit.iso"
  pool = "default" # List storage pools using virsh pool-list
  user_data      = "${data.template_file.user_data.rendered}"
}

# Define KVM domain to create
resource "libvirt_domain" "centos7" {
  name   = "centos7"
  memory = "2048"
  vcpu   = 2

  network_interface {
    network_name = "default"
  }

  disk {
    volume_id = "${libvirt_volume.centos7-qcow2.id}"
  }

  cloudinit = "${libvirt_cloudinit_disk.commoninit.id}"

  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }
}

# Output Server IP
output "ip" {
  value = "${libvirt_domain.centos7.network_interface.0.addresses.0}"
}
```

**Re-initialize Terraform working directory.**

```sh
terraform init
Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/template...
- Reusing previous version of dmacvicar/libvirt from the dependency lock file
- Installing hashicorp/template v2.2.0...
- Installed hashicorp/template v2.2.0 (signed by HashiCorp)
- Using previously-installed dmacvicar/libvirt v0.7.1

Terraform has made some changes to the provider dependency selections recorded
in the .terraform.lock.hcl file. Review those changes and commit them to your
version control system if they represent changes you intended to make.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

**Then create the Virtual Machine and its resources using apply command:**

```sh
terraform planterraform apply
```

**Execution output:**

![terraform apply kvm provider](https://computingforgeeks.com/wp-content/uploads/2019/03/terraform-apply-kvm-provider-1024x576.png?ezimgfmt=rs:696x392/rscb23/ng:webp/ngcb23 "How To Provision Virtual Machines on KVM with Terraform 1")

**Note the Server IP printed on the screen.**

![terraform kvm outputs](https://computingforgeeks.com/wp-content/uploads/2019/03/terraform-kvm-outputs-1024x165.png?ezimgfmt=rs:696x112/rscb23/ng:webp/ngcb23 "How To Provision Virtual Machines on KVM with Terraform 2")

**Or use `virsh` command to get the server IP address.**

```sh
$ sudo virsh net-dhcp-leases default
 Expiry Time           MAC address         Protocol   IP address           Hostname   Client ID or DUID
---------------------------------------------------------------------------------------------------------
 2019-03-24 16:41:32   52:54:00:22:45:57   ipv4       192.168.122.219/24   -          -
```

Try login to the instance as root user and password set.

![terraform kvm login](https://computingforgeeks.com/wp-content/uploads/2019/03/terraform-kvm-login.png?ezimgfmt=rs:695x237/rscb23/ng:webp/ngcb23 "How To Provision Virtual Machines on KVM with Terraform 3")

**Check if ssh user created can login with SSH key and run sudo without password.**

```sh
$ ssh jmutai@192.168.122.219
The authenticity of host '192.168.122.219 (192.168.122.219)' can't be established.
ECDSA key fingerprint is SHA256:G8ByhT4+FXBzh/MabB67rcS6JpTUn1TcrusXhiy8ke0.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.122.219' (ECDSA) to the list of known hosts.

[jmutai@localhost ~]$ sudo su -
Last login: Sun Mar 24 13:16:46 UTC 2019 on tty1
[root@localhost ~]# 
```

Check Terraform [KVM provider documentation](https://registry.terraform.io/providers/dmacvicar/libvirt/latest/docs/) for resources usage and provided [examples](https://github.com/dmacvicar/terraform-provider-libvirt/tree/master/examples) on how to use the provider.

Refer to [Documentation](https://cloudinit.readthedocs.io/en/latest) for its usage.

- [Deploy VM Instances on Hetzner Cloud with Terraform](https://computingforgeeks.com/deploy-vm-instances-on-hetzner-cloud-with-terraform/)
- [Install Terraform on Windows 10 / Windows Server 2019](https://computingforgeeks.com/install-and-use-terraform-on-windows/)
- [Install and Configure Hashicorp Vault Server on Ubuntu / CentOS / Debian](https://computingforgeeks.com/install-and-configure-vault-server-linux/)
- [How to run Minikube on KVM](https://computingforgeeks.com/how-to-run-minikube-on-kvm/)
- [virsh commands cheatsheet to manage KVM guest virtual machines](https://computingforgeeks.com/virsh-commands-cheatsheet/)