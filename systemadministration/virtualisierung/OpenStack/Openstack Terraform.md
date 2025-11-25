---
tags:
  - openstack
  - terraform
---

variables.tf

```
variable "openstack_user_name" {
    description = "The username for the Tenant."
    default  = "myuser"
}

variable "openstack_tenant_name" {
    description = "The name of the Tenant."
    default  = "my_tenant"
}

variable "openstack_password" {
    description = "The password for the Tenant."
    default  = "my_password"
}

variable "openstack_auth_url" {
    description = "The endpoint url to connect to OpenStack."
    default  = "http://<HOSTNAME>:5000/v2.0"
}

variable "openstack_keypair" {
    description = "The keypair to be used."
    default  = "my_keypair"
}

variable "tenant_network" {
    description = "The network to be used."
    default  = "my_network"
}
```

provider.tf

```
provider "openstack" {
  user_name = "${var.openstack_user_name}"
  tenant_name = "${var.openstack_tenant_name}"
  password  = "${var.openstack_password}"
  auth_url  = "${var.openstack_auth_url}"
}
```

deploy.tf

```
variable "count" {
  default = 2
}
resource "openstack_compute_instance_v2" "web" {
  count = "${var.count}"
  name = "${format("web-%02d", count.index+1)}"
  image_name = "CentOS6"
  availability_zone = "AZ1"
  flavor_id = "2"
  key_pair = "${openstack_keypair}"
  security_groups = ["default"]
  network {
    name = "${tenant_network}"
  }
  user_data = "${file("bootstrapweb.sh")}"
}
```

**bootstrapweb.sh**

```sh
#!/bin/bash

yum install -y httpd
chkconfig --level 345 httpd on

cat <<EOF > /var/www/html/index.html
<html>
<body>
<p>hostname is: $(hostname)</p>
</body>
</html>
EOF
chown -R apache:apache /var/www/html
service httpd start
```

**Anzeigen was gemacht werden würde**

```
terraform plan
```


Quellen:

[how-to-use-terraform-to-deploy-openstack-workloads](https://superuser.openinfra.dev/articles/how-to-use-terraform-to-deploy-openstack-workloads/)

https://elatov.github.io/2018/09/deploying-an-openstack-instance-with-terraform/

https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs

