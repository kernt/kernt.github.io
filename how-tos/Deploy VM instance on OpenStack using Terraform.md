---
tags:
  - openstack
  - terraform
---
Terraform is a powerful open source tool for building, changing, and versioning your infrastructure deployments in a safely and efficient manner. Terraform can be used to manage existing infrastructures, popular service providers as well as custom in-house solutions and implementations. Terraform uses configuration files which describes the components needed to run a single application or your entire datacenter, and desired state will be effected.

In this short guide we will be discussing how you can use terraform to automate instance creation on OpenStack Cloud platform. The OpenStack can be on self-hosted infrastructure or a public managed service offering. All that is required is OpenStack API endpoint that can be accessed from the machine where terraform is installed.

I already have a working OpenStack setup that was installed using the guide below.

[Install OpenStack Victoria on CentOS 8 With Packstack](https://computingforgeeks.com/install-openstack-victoria-on-centos/)
## Step 1: OpenStack preparation

To ease user learning experience we will do an end-to-end deployment automation of a virtual machine provisioning on OpenStack using Terraform.
### Install and configure OpenStack Client

If you’ve not configured OpenStack client already use the following guide:

- [Install and Configure OpenStack Client on Linux](https://computingforgeeks.com/install-and-configure-openstack-client-on-linux/)

**Skip Step 3** in the guide as we’ll be using _clouds.yaml_ configs.
### Upload image to Glance

We will download Debian 10 qcow2 image and upload it to Glance image service, upload SSH key and create _clouds.yml_ configuration file.

Download OS image to glance:

```sh
wget http://cdimage.debian.org/cdimage/openstack/current-10/debian-10-openstack-amd64.qcow2
```

Upload image to glance

```sh
openstack image create \
    --container-format bare \
    --disk-format qcow2 \
    --file debian-10-openstack-amd64.qcow2 \
    Debian-10
```

Confirm image is uploaded and active

```sh
$ openstack image list
+--------------------------------------+--------------+--------+
| ID                                   | Name         | Status |
+--------------------------------------+--------------+--------+
| 211daeef-eee7-4b13-a778-72c06b8d2c27 | Cirros-0.5.1 | active |
| 5ceccfb1-2389-4ffd-a82f-348d6cd6b3b2 | Debian-10    | active |
+--------------------------------------+--------------+--------+
```
### Upload SSH key

Generate new SSH key if you don’t have one already.

```sh
$ ssh-keygen # if you don't have ssh keys already
```

Upload the public key to Openstack. This key will be used during instance creation for passwordless authentication.

```sh
$ openstack keypair create --public-key ~/.ssh/id_rsa.pub admin
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | 19:7b:5c:14:a2:21:7a:a3:dd:56:c6:e4:3a:22:e8:3f |
| name        | admin                                           |
| user_id     | 513f0abd6eba4b0fab2754166f38e0f2                |
+-------------+-------------------------------------------------+
```

Confirm keypair is available on OpenStack:

```sh
$ openstack keypair list
+-------+-------------------------------------------------+
| Name  | Fingerprint                                     |
+-------+-------------------------------------------------+
| admin | 19:7b:5c:14:a2:21:7a:a3:dd:56:c6:e4:3a:22:e8:3f |
+-------+-------------------------------------------------+
```
### Create clouds.yml file

**Create new directory where _clouds.yml_ file will be stored.**

```sh
mkdir ~/.config/openstack
```

**Create clouds.yml file**

```sh
vim ~/.config/openstack/clouds.yaml
```

Add and modify the following contents to [match your OpenStack setup](https://docs.openstack.org/python-openstackclient/pike/configuration/index.html).

```yaml
clouds:
  osp_admin:                                 # Name of this cloud
    auth:
      auth_url: http://192.168.10.10:5000/v3 # API endpoint
      project_name: admin                    # OpenStack Project name
      username: admin                        # Authentication user
      password: 'AdminPssword'             # User password
    identity_api_version: 3
    region_name: RegionOne                   # OpenStack region
    operation_log:                           # Client Logging configurations
      logging: TRUE
      file: ~/openstackclient_admin.log
      level: info
```

You can also download and modify the OpenStack environment file credentials from the OpenStack dashboard.

- Log in to the OpenStack dashboard > **Project** > “**Access & Security**“
- Click “**Download OpenStack RC File**” and save the file.

**Create log file**

```sh
touch ~/openstackclient_admin.log
```

**Test if your configurations are working.**

```sh
openstack service list  --os-cloud osp_admin
+----------------------------------+------------+----------------+
| ID                               | Name       | Type           |
+----------------------------------+------------+----------------+
| 016e1a0f299e4188a4ff2f0951041890 | swift      | object-store   |
| 02b03ebfe32a48a8ba1b4eb886fea509 | cinderv2   | volumev2       |
| 0ee374b1619e44dd8c3f1f8c8792b08b | nova       | compute        |
| 4eddc25d9c6c42c29ed4aaf3a690e073 | aodh       | alarming       |
| 51ec76355583449aac07c7570750bfda | heat       | orchestration  |
| 75797c5e394f419f9de85e8f424914fa | neutron    | network        |
| 75e2d698d2114d028769621995232a35 | glance     | image          |
| 84da19176cb84382a7a87d9461ab926e | placement  | placement      |
| 8d228baf96b24d97934d1f722337f0ee | heat-cfn   | cloudformation |
| 9e944a5b9a3d474ebc60fd85f0c080bd | cinderv3   | volumev3       |
| 9e9507529ec4454daebeb30183a06d16 | gnocchi    | metric         |
| bf915960baff410db3583cc66ee55daa | keystone   | identity       |
| fbb3e1eb3d6b489386648476e1c55877 | ceilometer | metering       |
+----------------------------------+------------+----------------+
```
## Step 2: Get required information from OpenStack

**List all available images in Glance:**

```sh
openstack image list
+--------------------------------------+--------------+--------+
| ID                                   | Name         | Status |
+--------------------------------------+--------------+--------+
| 3d042db5-2a0e-4b1d-8a71-12bb43bc638c | CentOS-7     | active |
| cbed9a33-e854-4894-bf9b-1b4596c49782 | CentOS-8     | active |
| 211daeef-eee7-4b13-a778-72c06b8d2c27 | Cirros-0.5.1 | active |
| 5ceccfb1-2389-4ffd-a82f-348d6cd6b3b2 | Debian-10    | active |
| 636f3209-2c08-49f8-92c9-3a3b50d8850e | Ubuntu-20.04 | active |
| 4aee3927-da23-4f83-b887-a9b7eafd7192 | fcos         | active |
+--------------------------------------+--------------+--------+
```

**List available compute flavors:**

```sh
openstack flavor list
+----+-----------+-------+------+-----------+-------+-----------+
| ID | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+-----------+-------+------+-----------+-------+-----------+
| 1  | m1.tiny   |   512 |    1 |         0 |     1 | True      |
| 2  | m1.small  |  2048 |   20 |         0 |     1 | True      |
| 3  | m1.medium |  4096 |   40 |         0 |     2 | True      |
| 4  | m1.large  |  8192 |   80 |         0 |     4 | True      |
| 5  | m1.xlarge | 16384 |  160 |         0 |     8 | True      |
+----+-----------+-------+------+-----------+-------+-----------+
```

**List configured networks:**

```sh
$ openstack network list
+---------------+---------+--------------------------------------+
| ID            | Name    | Subnets                              |
+---------------+---------+--------------------------------------+
| 03eff42c-0b21-43e6-bbb6-164552279961 | private | bd52f697-7e61-4f70-a416-78dde193b0c2 |
| 95cbb9bc-ddcc-412f-9496-3f77dff3f030 | public  | 0063aaf9-9e3d-4634-a4c7-ddf0e66c2b75 |
+---------------+---------+--------------------------------------+
```

## Step 3: Install Terraform and create main config

We’ll be using the OpenStack [Terraform provider](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest) to connect to OpenStack cloud platform and provision resources. Create a new terraform main configuration file which defines the provider to be used as openstack.

```sh
$ vim main.tf
# Configure the OpenStack Provider
terraform {
  required_providers {
    openstack = {
      source = "terraform-provider-openstack/openstack"
    }
  }
}
```

Source addresses consist of three parts delimited by slashes (`/`), as follows:

```sh
[<HOSTNAME>/]<NAMESPACE>/<TYPE>
```

- **Hostname** (optional): The hostname of the Terraform registry that distributes the provider. If omitted, this defaults to `registry.terraform.io`, the hostname of [the public Terraform Registry](https://registry.terraform.io/).
- **Namespace:** An organizational namespace within the specified registry. For the public Terraform Registry and for Terraform Cloud’s private registry, this represents the organization that publishes the provider. This field may have other meanings for other registry hosts.
- **Type:** A short name for the platform or system the provider manages. Must be unique within a particular namespace on a particular registry host.

If terraform client tool is not installed in your current machine refer to our guide below.

- [Install Terraform on Linux](https://computingforgeeks.com/how-to-install-terraform-on-linux/)

**Initialize to download provider and create required files.**

```sh
$ terraform init
Initializing the backend...

Initializing provider plugins...
- Finding latest version of terraform-provider-openstack/openstack...
- Installing terraform-provider-openstack/openstack v1.49.0...
- Installed terraform-provider-openstack/openstack v1.49.0 (self-signed, key ID 4F80527A391BEFD2)

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
## Step 4: Deploy VM instance on OpenStack With Terraform

**Let’s update our terraform configuration to add VM creation resource definitions.**

```sh
# Configure the OpenStack Provider
terraform {
  required_providers {
    openstack = {
      source = "terraform-provider-openstack/openstack"
    }
  }
}

provider "openstack" {
  cloud  = "osp_admin" # cloud defined in cloud.yml file
}

# Variables
variable "keypair" {
  type    = string
  default = "admin"   # name of keypair created 
}

variable "network" {
  type    = string
  default = "private" # default network to be used
}

variable "security_groups" {
  type    = list(string)
  default = ["default"]  # Name of default security group
}

# Data sources
## Get Image ID
data "openstack_images_image_v2" "image" {
  name        = "Debian-10" # Name of image to be used
  most_recent = true
}

## Get flavor id
data "openstack_compute_flavor_v2" "flavor" {
  name = "m1.small" # flavor to be used
}

# Create an instance
resource "openstack_compute_instance_v2" "server" {
  name            = "Debian10"  #Instance name
  image_id        = data.openstack_images_image_v2.image.id
  flavor_id       = data.openstack_compute_flavor_v2.flavor.id
  key_pair        = var.keypair
  security_groups = var.security_groups

  network {
    name = var.network
  }
}

# Output VM IP Address
output "serverip" {
 value = openstack_compute_instance_v2.server.access_ip_v4
}
```

**Plan for VM creation to see what will be created and validate config.**

```sh
terraform plan

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # openstack_compute_instance_v2.server will be created
  + resource "openstack_compute_instance_v2" "server" {
      + access_ip_v4        = (known after apply)
      + access_ip_v6        = (known after apply)
      + all_metadata        = (known after apply)
      + all_tags            = (known after apply)
      + availability_zone   = (known after apply)
      + flavor_id           = "2"
      + flavor_name         = (known after apply)
      + force_delete        = false
      + id                  = (known after apply)
      + image_id            = "5ceccfb1-2389-4ffd-a82f-348d6cd6b3b2"
      + image_name          = (known after apply)
      + key_pair            = "admin"
      + name                = "ubuntu20.04"
      + power_state         = "active"
      + region              = (known after apply)
      + security_groups     = [
          + "default",
        ]
      + stop_before_destroy = false

      + network {
          + access_network = false
          + fixed_ip_v4    = (known after apply)
          + fixed_ip_v6    = (known after apply)
          + floating_ip    = (known after apply)
          + mac            = (known after apply)
          + name           = "private"
          + port           = (known after apply)
          + uuid           = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

**Create VM instance on OpenStack using the commands below.**

```sh
$ terraform apply -auto-approve
openstack_compute_instance_v2.server: Creating...
openstack_compute_instance_v2.server: Still creating... [10s elapsed]
openstack_compute_instance_v2.server: Creation complete after 19s [id=03ed76cb-1cf5-4259-9427-56aabc0d1a38]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

serverip = "172.10.10.70"
```
## Step 5: Associating instance with a floating IP

**Since we launched our instance on the private network we’ll need to associate a floating IP for external access.**

```sh
# Add Floating ip

resource "openstack_networking_floatingip_v2" "fip1" {
  pool = "public"
}

resource "openstack_compute_floatingip_associate_v2" "fip1" {
  floating_ip = openstack_networking_floatingip_v2.fip1.address
  instance_id = openstack_compute_instance_v2.server.id
}

# Output VM IP Addresses
output "server_private_ip" {
 value = openstack_compute_instance_v2.server.access_ip_v4
}
output "server_floating_ip" {
 value = openstack_networking_floatingip_v2.fip1.address
}
```

**Apply updated configurations.**

```sh
....
Plan: 3 to add, 0 to change, 1 to destroy.

Changes to Outputs:
  ~ server_private_ip = "172.10.10.152" -> (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

openstack_networking_floatingip_v2.fip1: Creating...
openstack_compute_instance_v2.server: Destroying... [id=3f1c20e6-390e-4848-9fb8-1e04d85b7251]
openstack_networking_floatingip_v2.fip1: Creation complete after 9s [id=d322da14-226c-4f00-880e-d8d137a33fb4]
openstack_compute_instance_v2.server: Still destroying... [id=3f1c20e6-390e-4848-9fb8-1e04d85b7251, 10s elapsed]
openstack_compute_instance_v2.server: Destruction complete after 12s
openstack_compute_instance_v2.server: Creating...
openstack_compute_instance_v2.server: Still creating... [10s elapsed]
openstack_compute_instance_v2.server: Creation complete after 16s [id=3e102058-4cc6-479b-8682-0035ebfb6b96]
openstack_compute_floatingip_associate_v2.fip1: Creating...
openstack_compute_floatingip_associate_v2.fip1: Creation complete after 4s [id=135.181.187.56/3e102058-4cc6-479b-8682-0035ebfb6b96/]

Apply complete! Resources: 3 added, 0 changed, 1 destroyed.

Outputs:

server_private_ip = "172.10.10.152"
server_floating_ip = "publicip"
```

**Check if you can ssh to the instance:**

```sh
$ ssh debian@<floatingip>
Warning: Permanently added '<floatingip>' (ECDSA) to the list of known hosts.
Enter passphrase for key '/Users/jkmutai/.ssh/id_rsa':
Linux ubuntu20 4.19.0-13-cloud-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

debian@ubuntu20:~$ cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 10 (buster)"
NAME="Debian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

We’ve successfully created an instance on OpenStack using Terraform. This is the most basic resource creation on OpenStack using Terraform for more examples and documentation on OpenStack provider read the official documentation.

Similar Terraform guides:

- [How To Store Terraform State in Consul KV Store](https://computingforgeeks.com/how-to-store-terraform-state-in-consul-kv-store/)
- [How To Provision VMs on oVirt / RHEV with Terraform](https://computingforgeeks.com/how-to-provision-vms-on-ovirt-rhev-with-terraform/)
- [Deploy VM Instances on Hetzner Cloud with Terraform](https://computingforgeeks.com/deploy-vm-instances-on-hetzner-cloud-with-terraform/)
- [How To Provision VMs on KVM with Terraform](https://computingforgeeks.com/how-to-provision-vms-on-kvm-with-terraform/)