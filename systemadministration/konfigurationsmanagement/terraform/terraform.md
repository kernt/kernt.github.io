---
tags:
  - terraform
---
# terraform

Infrastructure as Code zum Provisionieren und managen von beliebigen cloud Umgebungen, infrastructure, oder service

_Resource Strukturen _

`{output}/{provider}/{service}/{resource}.tf`

- {output} 
- {provider} z.B. Grafana
- {service}  Config ders Providers
- {resource}  Vars Token etc.

## terrraform plugins

`~/.terraform.d/plugins/`

# Terraform Providers

### DNS Provider

#### acme-dns

#### Vega DNS

#### Duck DNS

#### do.de

#### rfc2136

## Grafana Provider

## ICINGA2 Provider

## Prometheus Provider

## Setup nfs Client

1. Install the Helm provider for Terraform

```tf
provider "helm" {  
  kubernetes {  
    config_path = "~/.kube/config"  
  }  
}
```

2. Add the `nfs-subdir-external-provisioner` Helm chart repository

```tf
data "helm_repository" "nfs" {  
  name = "nfs-subdir-external-provisioner"  
  url  = "https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/"  
}
```

3. Install the NFS client provisioner. Replace `<nfs-server>` and `<nfs-path>` placeholders with your NFS server IP and exported NFS directory path:

```tf
resource "helm_release" "nfs_client_provisioner" {  
  name       = "nfs-client-provisioner"  
  repository = data.helm_repository.nfs.metadata[0].name  
  chart      = "nfs-subdir-external-provisioner"  
  set {  
    name  = "nfs.server"  
    value = "<nfs-server>"  
  }  
  set {  
    name  = "nfs.path"  
    value = "<nfs-path>"  
  }  
}
```

4. Apply the Terraform script. Here’s an example of what this might look like in your shell

```sh
terraform init  
terraform apply
```

## Verifying the Installation

**Check if the NFS client provisioner pod is running**

`kubectl get pods`

**Check if the storage class has been created**

`kubectl get sc`

## Install Postgresql wit our provisioner

**First, we need to declare the `helm_release` for PostgreSQL**

```tf
data "helm_repository" "bitnami" {  
  name = "bitnami"  
  url  = "https://charts.bitnami.com/bitnami"  
}  
  
resource "helm_release" "postgresql" {  
  name       = "postgresql"  
  chart      = "bitnami/postgresql"  
  repository = data.helm_repository.bitnami.metadata[0].name  
  set {  
    name  = "persistence.storageClass"  
    value = "nfs-client"  
  }  
  set {  
    name  = "persistence.size"  
    value = "1Gi"  
  }  
}
```

# cloud init

**main.tf**

```tf

terraform {
    required_providers {
        #...
        cloudinit = {
            source = "hashicorp/cloudinit"
            version = ">=2.2.0"
        }
    }
}

```

**cloud config file cloud-init-ansible.yaml**

```
# Configure proxy server so install packages etc. will work behind proxy
# based on https://bugs.launchpad.net/cloud-init/+bug/1089405/comments/15
bootcmd:
- |
  cloud-init-per once env sh -c "mkdir -p /etc/systemd/system/cloud-config.service.d &&
  mkdir -p /etc/systemd/system/cloud-final.service.d && { cat > /etc/cloud/env <<-EOF
  http_proxy=http://192.168.1.10:7890
  https_proxy=http://192.168.1.10:7890
  no_proxy=localhost,127.0.0.1
  EOF
  } && { cat > /etc/systemd/system/cloud-config.service.d/override.conf <<-EOF
  [Service]
  EnvironmentFile=/etc/cloud/env
  EOF
  } && { cat > /etc/systemd/system/cloud-final.service.d/override.conf <<-EOF
  [Service]
  EnvironmentFile=/etc/cloud/env
  EOF
  } && systemctl daemon-reload"

runcmd:
  # Can't do this with packages as runcmd runs before packages so we cannot use software installed by packages
  - [ yum, -y, install, python3, python-virtualenv, git ]
  - [ mkdir, -p, /opt/ansible/virtualenv, /opt/ansible/configuration ]
  - [ chown, azureuser, /opt/ansible/virtualenv, /opt/ansible/configuration ]
  - [ chgrp, azureuser, /opt/ansible ]
  - [ chmod, "0750", /opt/ansible ]
  - [ su, -c, "git clone ${ansible_source} /opt/ansible/configuration", azureuser ]
  - [ su, -c, virtualenv -ppython3 /opt/ansible/virtualenv, azureuser ]
  - [ su, -c, /opt/ansible/virtualenv/bin/pip install --upgrade pip, azureuser]
  - [ su, -c, /opt/ansible/virtualenv/bin/pip install ansible, azureuser]
  - [ su, -c, /opt/ansible/virtualenv/bin/ansible-galaxy install azure.azure_modules, azureuser]
  - [ su, -c, /opt/ansible/virtualenv/bin/pip install -r $HOME/.ansible/roles/azure.azure_modules/files/requirements-azure.txt, azureuser]
  - [ su, -c, bash -c "export AZURE_SUBSCRIPTION_ID=${azure_subscription_id}; export AZURE_CLIENT_ID=${azure_client_id}; export AZURE_SECRET='${azure_client_secret}'; export AZURE_TENANT=${azure_tenant_id}; /opt/ansible/virtualenv/bin/azure-playbook -i /opt/ansible/configuration/Ansible/inventory/all.azure_rm.yml --limit $HOSTNAME -c local /opt/ansible/configuration/Ansible/site.yml", azureuser] ]
```

created a resource for the cloud-init configuration, in a file called `cloud-init.tf`

```
data "cloudinit_config" "ansible" {
  gzip = true
  base64_encode = true

  part {
    filename = "cloud-init-ansible"
    content_type = "text/cloud-config"
    content = templatefile(
        "cloud-init-ansible.yaml",
        {
            ansible_source = "https://deployuser:deploykey@git.host.domain.tld/group/repo"
            azure_subscription_id = vars.azure_subscription_id
            azure_client_id = vars.azure_client_id
            azure_client_secret = vars.azure_client_secret
            azure_tenant_id = vars.azure_tenant_id
        }
    )
  }
}
```

VM’s `custom_data`

```
resource "azurerm_linux_virtual_machine" "vm-1" {
  #...
  custom_data = data.cloudinit_config.ansible.rendered
  #...
}
```

https://blog.entek.org.uk/notes/2021/09/29/terraform-cloud-init-and-ansible.html

# Reverse Terraform

https://medium.com/@jinvishal2011/reverse-terraform-infrastructure-as-code-iac-from-existing-infrastructure-1fffdc462366

https://medium.com/@xpiotrkleban/reverse-engineering-existing-infrastructure-to-terraform-hcl-files-453420059ef9


# Terraform Providers

# [terraform-provider-contabo](https://github.com/contabo/terraform-provider-contabo)
