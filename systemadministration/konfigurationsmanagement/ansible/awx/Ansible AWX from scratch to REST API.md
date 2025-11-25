# part 1 of 8

Before installing Ansible AWX, we have to install the Operating System.

CentOS is a great open source RedHat-like community operating system.

# Concepts

## What is CentOS

> _The CentOS Project is a community-driven free software effort focused on delivering a robust open source ecosystem around a Linux platform.  
> We offer two Linux distros:  
> – CentOS Linux is a consistent, manageable platform that suits a wide variety of deployments. For some open source communities, it is a solid, predictable base to build upon.  
> – The new CentOS Stream is a rolling-release distro that tracks just ahead of Red Hat Enterprise Linux (RHEL) development, positioned as a midstream between Fedora Linux and RHEL. For anyone interested in participating and collaborating in the RHEL ecosystem, CentOS Stream is your reliable platform for innovation._ [_¹_](https://www.centos.org)

# System Requirements

- 20GB disk space;
- 4GB of memory (AWX on Docker Compose requires at least 4GB).

A detailed system requirements is provided from CentOS documentation. [²](https://wiki.centos.org/About/Product)

# Download CentOS ISO

Download the [CentOS ISO](https://www.centos.org/download/).

# Installing CentOS

Boot your machine with the previous CentOS ISO file.

## Initial Screen

Select “Install CentOS Linux…”

## Select Language

Configure the system language.

## Select “Installation Destination”

Configure the file system layout.

For the bellow screen, there is no modifications, then click **[Done]**

## Select “Network & Host Name”

Configure network settings.

## Define “Host Name”

Configure Host Name, click **[Apply]**

## Setup “NAT NIC”

Configure the NAT Network (internet access) using DHCP.

## Setup “Private NIC”

Configure the Private Network (host-only) using static IP address.

## Select “Software Selection”

Configure the software profile for this installation.

## Select “Time & Date”

Configure region.

## Installation Preview

In _Installation Summary_, click **[Begin Installation]**

## Installation Running

## Select “Root Password”

Set root password.

## Installation Summary

Once your installation is done, click **[Reboot]**


# Part 2 of 8

This topic is about how to install Ansible AWX on Docker Compose.

Nowadays the most applications are container-based design, Ansible AWX has support to multiple deployment types such as OpenShift, Kubernetes, Minishift, and Docker Compose.

# Concepts

## What is Docker Compose

> _Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see the list of features._ [_¹_](https://docs.docker.com/compose/)

# System Requirements

The system requirements for Ansible AWX deployment on Docker Compose. [²](https://github.com/ansible/awx/blob/devel/INSTALL.md)

- At least 4GB of memory;
- At least 2 cpu cores;
- At least 20GB of space.

# Prerequisites

Install docker-ce repository.

`curl [https://download.docker.com/linux/centos/docker-ce.repo](https://download.docker.com/linux/centos/docker-ce.repo) -o /etc/yum.repos.d/docker-ce.repo`

Install EPEL repository.

`dnf -y install [https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm](https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm)`

Install packages.

`dnf -y install python3-pip git docker-ce ansible --nobest`

Install docker-compose.

`pip3 install docker-compose`

# Setup Docker

Enable and start docker service.

`systemctl enable --now docker`

Create a exclusive network for Docker Compose.

`docker network create --opt com.docker.network.bridge.name=awxcompose0 awxcompose`

# Setup Firewall

Allow the following firewall policies.

```sh
firewall-cmd --zone=trusted --add-interface=awxcompose0 --permanent  
firewall-cmd --zone=trusted --add-interface=awxcompose0  
firewall-cmd --add-service=http --add-service=https --permanent  
firewall-cmd --add-service=http --add-service=https
```

# Download and Setup AWX

Get AWX source code.

`git clone [https://github.com/ansible/awx](https://github.com/ansible/awx)`

Change directory.

`cd awx/installer/`

Define Docker Compose network to be used by template.

```sh
cat << EOF >> roles/local_docker/templates/docker-compose.yml.j2networks:  
  default:  
    external:  
      name: awxcompose  
EOF
```

Define AWX admin password.

`sed -E -i 's|^(#)?admin_password=.*|admin_password=mypassword|g' inventory`

Create SSL certificate.

```sh
mkdir -vp ~/.awx/sslopenssl req -subj \  
  '/C=BR/ST=Sao Paulo/L=Sao Paulo/O=AWX Example/OU=IT/CN=awx.example.com/emailAddress=awx@example.com' \  
  -new -nodes -x509 -days 365 -newkey rsa:2048 \  
  -keyout ~/.awx/ssl/awx.key \  
  -out ~/.awx/ssl/awx.crtcat ~/.awx/ssl/awx.crt ~/.awx/ssl/awx.key >> ~/.awx/ssl/awx.pem
```

Adjust inventory for SSL certificate.

`sed -E -i 's|^(#)?ssl_certificate=.*|ssl_certificate="~/.awx/ssl/awx.pem"|g' inventory`

You can adjust other parameters, check inventory file.  
By default the user’s home directory is the default path for storing container files, as well as for the PostgreSQL database.

`grep -Ev '^$|^#' inventory`

![](1_jUbAqF8MAO16lt2R9rUDJg.webp)

Run Ansible Playbook to configure AWX.

`ansible-playbook -i inventory -e ansible_python_interpreter=python3 install.yml`

Wait for playbook to finish.

![](1_GBXaG2el3nsBscKsuNQI5w.webp)

While containers are initializing, index page remains in status _“AWX is Upgrading”_, that page will be automatically refreshed and redirected to login page, then it will be able to accept user credentials.

NOTE: it’s strongly recommended to use Safari or Firefox browsers, recent Chrome version does not work due self-signed certificate.

![](1_vaR_NNSoBjY9SuCS6_5LVg.webp)

![](1_XkehNUefqHnIG-XE4h7nUg.webp)

![](1_pY-ntETju6arRNK7v5Wvcg.webp)

# Summary

In this topic was presented:

- Installing AWX prerequisites;
- Download and setup Ansible AWX for Docker Compose deployment on CentOS 8.

# References

[1] - [https://docs.docker.com/compose/](https://docs.docker.com/compose/)  
[2] - [https://github.com/ansible/awx/blob/devel/INSTALL.md](https://github.com/ansible/awx/blob/devel/INSTALL.md)


# Part 3  of 8

## What is REST API

> _REST stands for Representational State Transfer and is sometimes spelled as “ReST”. It relies on a stateless, client-server, and cacheable communications protocol, usually the HTTP protocol._ [_¹_](https://docs.ansible.com/ansible-tower/latest/html/towerapi/tools.html)

# Exploring AWX REST API with cURL

Let’s explore and understand an interaction example with AWX REST API, this topic is intended to explain, no direct action is required.

First, you need to know which resources are available through API interface.

All resources are detailed described [here](https://docs.ansible.com/ansible-tower/latest/html/towerapi/api_ref.html).

![](1_gsxoX6oKSqc4C4eWsuvCQQ.webp)

Expand [_Inventories_](https://docs.ansible.com/ansible-tower/latest/html/towerapi/api_ref.html#/Inventories/Inventories_inventories_create) resource.

A practical example is provided from documentation.

![](1_z5Vgni5ivz2gE5QSgyPbdQ.webp)

There are 3 available response status code for this resource:

```
201 - created  
400 - bad request  
403 - forbidden
```

cURL example:

![](1_MzpkjL5FMT6mfOMaNDvlPw.webp)

![](1_-NQsxU3EiWijAHIlyaxJeQ.webp)

The next topics are hands-on!

# Prerequisites

Before working with API, you need to install some tools.

Install **ansible-tower-cli** using pip3.

`sudo pip3 install ansible-tower-cli`

Install **perl-JSON-PP** and **jq** packages.

`sudo dnf install -y perl-JSON-PP jq`

# CLI first steps

The **tower-cli** is the recommended tool to manage Ansible AWX and Tower, but you can use **awx** or **awx-cli** as well, the **awx-cli** will be successor of tower-cli.

Initial **tower-cli** configuration. [²](https://github.com/ansible/tower-cli/tree/master/docs/source/cli_ref/examples)

`tower-cli config`  
`tower-cli config verify_ssl false`  
`tower-cli config host [https://awx.example.com](https://awx.example.com)`  
`tower-cli config username admin`  
`tower-cli config password mypassword`

The _tower-cli config_ results:

![](1_EOUTfZnDncnZ-lyuNkLm2w.webp)

Create the static inventory.

```sh
tower-cli inventory create \  
  --organization "Default" \  
  --name "Example Inventory" \  
  --variables '{  
                  "api_awx_url": "https://awx.example.com",  
                  "api_awx_username": "admin",  
                  "api_awx_password": "mypassword"  
              }'
```

![](1_h2yyX_DrGQ2UE5KWv1kk2w.webp)

Here is another way to do the same task using curl command (REST API):

```sh
curl -k -s --user admin:mypassword -X POST -H "Content-Type: application/json" \  
     "https://awx.example.com/api/v2/inventories/" \  
     --data '{  
               "name": "Example Inventory",  
               "organization": "1",  
               "variables": "api_awx_url: \"https://awx.example.com\"\napi_awx_username: admin\napi_awx_password: mypassword"  
            }' | python3 -m json.tool
```

Here is another way to do the same task using Web UI:

> _Left Menu (Inventories) > click [+] (select Inventory) > fill the form > click [SAVE]_

![](1_afyZFgYid1H867wUs90-IQ.webp)

# Adding hosts to the static inventory

Add hosts to the inventory.

```sh
tower-cli host create \  
  --name "nodea" \  
  --inventory "Example Inventory" \  
  --variables 'ansible_host: 172.16.100.151'
```

![](1_Cc7czeEaE8xrUh6e4G530Q.webp)

```sh
tower-cli host create \  
  --name "nodeb" \  
  --inventory "Example Inventory" \  
  --variables '{  
                 "ansible_host": "172.16.100.152",  
                 "api_inventory_disable": "yes"  
              }'
```

![](1_1NGm1xNCio1Dnd7vccRfVg.webp)

```sh
tower-cli host create \  
  --name "nodec" \  
  --inventory "Example Inventory" \  
  --variables '{  
                 "ansible_host": "172.16.100.153",  
                 "api_inventory_remove": "yes"  
              }'
```

![](1_qD04cQU_CJuFrw9deF9A8w.webp)

```sh
tower-cli host create \  
  --name "noded" \  
  --inventory "Example Inventory" \  
  --variables '{  
                 "ansible_host": "172.16.100.154",  
                 "api_inventory_disable": "yes",  
                 "api_inventory_remove": "yes"  
              }'
```

![](1_NNW2zZEG_0N1f1bk36P_6Q.webp)

```sh
tower-cli host create \  
  --name "nodee" \  
  --inventory "Example Inventory"
```

![](1_5DefZsOsbl-CkKARknEC6g.webp)

Here is another way to do the same task using curl command (REST API):

```sh
curl -k -s --user admin:mypassword -X POST -H "Content-Type: application/json" \  
     "https://awx.example.com/api/v2/inventories/2/hosts/" \  
     --data '{  
               "name": "nodea",  
               "variables": "ansible_host: 172.16.100.151"  
            }' | python3 -m json.toolcurl -k -s --user admin:mypassword -X POST -H "Content-Type: application/json" \  
     "https://awx.example.com/api/v2/inventories/2/hosts/" \  
     --data '{  
               "name": "nodeb",  
               "variables": "ansible_host: 172.16.100.152\napi_inventory_disable: yes"  
            }' | python3 -m json.toolcurl -k -s --user admin:mypassword -X POST -H "Content-Type: application/json" \  
     "https://awx.example.com/api/v2/inventories/2/hosts/" \  
     --data '{  
               "name": "nodec",  
               "variables": "ansible_host: 172.16.100.153\napi_inventory_remove: yes"  
            }' | python3 -m json.toolcurl -k -s --user admin:mypassword -X POST -H "Content-Type: application/json" \  
     "https://awx.example.com/api/v2/inventories/2/hosts/" \  
     --data '{  
               "name": "noded",  
               "variables": "ansible_host: 172.16.100.154\napi_inventory_disable: yes\napi_inventory_remove: yes"  
            }' | jqcurl -k -s --user admin:mypassword -X POST -H "Content-Type: application/json" \  
     "https://awx.example.com/api/v2/inventories/2/hosts/" \  
     --data '{ "name": "nodee" }' | json_pp
```

Here is another way to do the same task using Web UI:

> _Left Menu (Inventories) > click (Example Inventory) > click [HOSTS] > click [+] > fill the form > click [SAVE]_

![](wk49-PQjoDfdbyQU4lTEsQ.webp)

# Identifying inventory by ID

Before make any changes to the static inventory, you’ll need to check the inventory id first.

`tower-cli inventory list --organization "Default" --name "Example Inventory"`

![](1_Fqowl7ygwhxPkroSGTB5Hw.webp)

Here is another way to do the same task using curl command (REST API):

```sh
curl -k -s --user admin:mypassword -X GET \  
     "https://awx.example.com/api/v2/inventories/?name=Example%20Inventory" | python3 -m json.tool
```

# Identifying host by ID

Now, you’ll need to check only enabled hosts.

`tower-cli host list --inventory 2 --enabled true`

![](h1_NKPBaCE-In4oNpFneN9XHQ.webp)

Here is another way to do the same task using curl command (REST API):

```sh
curl -k -s --user admin:mypassword -X GET \  
     "https://awx.example.com/api/v2/hosts/?enabled=true&inventory=2" | python3 -m json.tool
```

# Disable host by ID

Once our target host is **“nodee”** having the **“6”** id, you can disable it on inventory.

tower-cli host modify 6 --enabled false

![](1_uFOk0A_QGhyJuJytqNkquw.webp)

Here is another way to do the same task using curl command (REST API):

```sh
curl -k -s --user admin:mypassword -X PATCH -H "Content-Type: application/json" \  
     "https://awx.example.com/api/v2/hosts/6/" \  
     --data '{ "enabled": "False" }' | python3 -m json.tool
```

Here is another way to do the same task using Web UI:

> _Left Menu (Inventories) > click (Example Inventory) > click [HOSTS] > deselect switch key_

![](1_uFOk0A_QGhyJuJytqNkquw.webp)

# Delete host by ID

Delete the **“nodee”** from inventory.

`tower-cli host delete 6`

![](1_GY-WlJfQ4f0SDNfR8x1Brw.webp)

Here is another way to do the same task using curl command (REST API):

NOTE: there are two ways, the first **_curl_** command is recommended.

```sh
curl -k -s --user admin:mypassword -X POST -H "Content-Type: application/json" \  
     "https://awx.example.com/api/v2/inventories/2/hosts/" \  
     --data '{ "id": 6, "disassociate": "True" }'curl -k -s --user admin:mypassword -X DELETE "https://awx.example.com/api/v2/hosts/6/"
```

Here is another way to do the same task using Web UI:

> _Left Menu (Inventories) > click (Example Inventory) > click [HOSTS] > click [trash icon]_

![](1_svFve4M6piGlREtYSvZ-SQ.webp)

# Summary

In this topic was presented:

- Understanding AWX REST API resources and how to interacts with them;
- Installing and configuring **tower-cli** tool;
- Working with CLI, REST API and Web UI to manage AWX.

# References

[1] - [https://docs.ansible.com/ansible-tower/latest/html/towerapi/tools.html](https://docs.ansible.com/ansible-tower/latest/html/towerapi/tools.html)  
[2] - [https://github.com/ansible/tower-cli/tree/master/docs/source/cli_ref/examples](https://github.com/ansible/tower-cli/tree/master/docs/source/cli_ref/examples)

# Part4 of 8

REST API interactions shown in the previous topic have been converted to playbook, which will be used by Workflow later.

In real envinroment that uses Ansible for configuration management, playbooks are always preferred instead scripts.

# Concepts

## What is Ansible Playbook

> _Playbooks are Ansible’s configuration, deployment, and orchestration language. They can describe a policy you want your remote systems to enforce, or a set of steps in a general IT process._ [_¹_](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)

# Playbook directory structure

Two playbooks has been developed for this guide, see **README.md** for more details at:

[https://github.com/cdomingos/ansible-playbook-awxapiinventory.git](https://github.com/cdomingos/ansible-playbook-awxapiinventory.git)

![](1_fGOihCh26qQ_puc2kADQjg.webp)

# The awx-api-inventory playbook

A playbook which interacts with REST API that is able to remove or disable host in inventory.

This playbook uses [**uri**](https://docs.ansible.com/ansible/latest/modules/uri_module.html) ansible module.

```yaml
---  
- name: MANAGE HOST IN AWX/TOWER INVENTORY USING REST API  
  hosts: all  
  connection: local  
  gather_facts: false  
  
  vars:  
    api_awx_inventory: "{{ awx_inventory_name | default(tower_inventory_name, True) }}"  
  
  tasks:  
    - name: Check REST API credentials  
      uri:  
        url: "{{ api_awx_url | regex_replace('(/+)$') }}/api/v2/me/"  
        method: GET  
        validate_certs: false  
        user: "{{ api_awx_username }}"  
        password: "{{ api_awx_password }}"  
        return_content: true  
        force_basic_auth: true  
        status_code: 200  
  
    - name: Get inventory id  
      uri:  
        url: "{{ api_awx_url | regex_replace('(/+)$') }}/api/v2/inventories/?name={{ api_awx_inventory | urlencode }}"  
        method: GET  
        validate_certs: false  
        user: "{{ api_awx_username }}"  
        password: "{{ api_awx_password }}"  
        return_content: true  
        force_basic_auth: true  
        status_code: 200  
      register: reg_uri_get_inventory_id  
      failed_when: reg_uri_get_inventory_id.json.count == 0  
  
    - name: Set fact_inventory_id  
      set_fact:  
        fact_inventory_id: "{{ item.id | int }}"  
      loop: "{{ reg_uri_get_inventory_id.json.results }}"  
      loop_control:  
        label: "{{ item.id | int }}"  
      when:  
        - item.name == api_awx_inventory  
  
    - name: Check if fact_inventory_id is valid  
      assert:  
        that:  
          - fact_inventory_id is defined  
          - fact_inventory_id | length > 0  
  
    - name: Check if managed node exists in inventory  
      uri:  
        url: "{{ api_awx_url | regex_replace('(/+)$') }}/api/v2/inventories/{{ fact_inventory_id }}/hosts/?name={{ inventory_hostname }}"  
        method: GET  
        validate_certs: false  
        user: "{{ api_awx_username }}"  
        password: "{{ api_awx_password }}"  
        return_content: false  
        force_basic_auth: true  
      register: reg_uri_host_check  
  
    - when: reg_uri_host_check.json.count > 0  
      block:  
  
        # the loop maybe run without arguments when entire block is skipped,  
        # this is an elegant way to avoid this behavior  
        - name: Set fact_host_id  
          set_fact:  
            fact_host_id: "{{ item.id | int }}"  
          loop: "{{ host_id_loopvar }}"  
          loop_control:  
            label: "{{ item.id | default('skip', True) }}"  
          when:  
            - item.name == inventory_hostname  
          vars:  
            host_id_loopvar: "{{ reg_uri_host_check.json.results | default(['skip'], True) }}"  
  
        - name: Check if fact_host_id is valid  
          assert:  
            that:  
              - fact_host_id is defined  
              - fact_host_id | length > 0  
              - fact_host_id != "skip"  
  
        - name: Check if managed node id exists  
          uri:  
            url: "{{ api_awx_url | regex_replace('(/+)$') }}/api/v2/hosts/{{ fact_host_id | int }}/"  
            method: GET  
            validate_certs: false  
            user: "{{ api_awx_username }}"  
            password: "{{ api_awx_password }}"  
            return_content: true  
            force_basic_auth: true  
            status_code: 200  
  
        - name: Disable managed node in inventory  
          uri:  
            url: "{{ api_awx_url | regex_replace('(/+)$') }}/api/v2/hosts/{{ fact_host_id | int }}/"  
            method: PATCH  
            validate_certs: false  
            user: "{{ api_awx_username }}"  
            password: "{{ api_awx_password }}"  
            return_content: false  
            force_basic_auth: true  
            status_code: 200  
            body:  
              enabled: false  
            body_format: json  
          changed_when: true  
          when:  
            - api_inventory_disable is defined  
            - api_inventory_disable | bool  
  
        - name: Disassociate managed node from inventory  
          uri:  
            url: "{{ api_awx_url | regex_replace('(/+)$') }}/api/v2/inventories/{{ fact_inventory_id }}/hosts/"  
            method: POST  
            validate_certs: false  
            user: "{{ api_awx_username }}"  
            password: "{{ api_awx_password }}"  
            return_content: true  
            force_basic_auth: true  
            status_code: 204  
            body: "{'id': {{ fact_host_id | int }}, 'disassociate': true }"  
            body_format: json  
          register: reg_uri_host_disassociate  
          changed_when: true  
          when:  
            - api_inventory_remove is defined  
            - api_inventory_remove | bool  
  
        - name: Check if managed host still exists in inventory  
          uri:  
            url: "{{ api_awx_url | regex_replace('(/+)$') }}/api/v2/inventories/{{ fact_inventory_id }}/hosts/{{ fact_host_id | int }}/"  
            method: GET  
            validate_certs: false  
            user: "{{ api_awx_username }}"  
            password: "{{ api_awx_password }}"  
            return_content: false  
            force_basic_auth: true  
          register: reg_uri_host_checkagain  
          failed_when: reg_uri_host_checkagain.status == 200  
          when:  
            - api_inventory_remove is defined  
            - api_inventory_remove | bool
```

This playbook could be rewritten using [**Ansible_Tower**](https://docs.ansible.com/ansible/latest/modules/list_of_web_infrastructure_modules.html#ansible-tower) modules, but I choose the **uri** module to expand API usage.

# The basic playbook

A basic playbook which do simple tasks.

```yaml
---  
- name: BASIC TASKS  
  hosts: all  
  gather_facts: false  
  
  tasks:  
    - name: Ensure ansible user exists  
      user:  
        name: ansible  
        state: present  
  
    - name: Test host connectivity  
      ping:
```

# Summary

In this topic was presented:

- Review an example playbook to automate API interactions using **_uri_** ansible module.

# References

[1] - [https://docs.ansible.com/ansible/latest/user_guide/playbooks.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)

# Continue Reading

Next topic is **_Creating a remote user for Ansible_**:

# part 5 of 8

This topic is about bootstrap an user to be used by Ansible in Managed Nodes.

It is recommended to use a regular user in managed nodes instead root, common user name as **_ansible_** or **_devops_** is not a good idea for production environments.

For a good _traceability_ and secure environment compliance, consider using solutions like FreeIPA or Red Hat IDM, each administrator should use their personal accounts.

# Environment

Use the first topic _“Operating System Installation”_ to install the following machines: **nodea**, **nodeb**, **nodec** and **noded**.

![](1_m9-q7qQ2TlEEabHQg6LRLQ.webp)

NOTE: if you do not want to install the managed nodes, skip all configurations related to “JT - GitHub AWX Basic” that uses basic.yml playbook in AWX Web UI, and go to the next part: [AWX Workflow Use Case](https://medium.com/@claudio.domingos/ansible-awx-from-scratch-to-rest-api-part-6-of-8).

# Hostname resolution for awx_task container

As shown the **awx-api-inventory.yml** playbook have the declared variable `api_awx_url`, as a good practice you should use FQDN instead of IP Address.

If you do not have a DNS previously configured, you can execute the following command:

`sudo docker exec awx_task sh -c "echo 172.16.100.150 awx.example.com >> /etc/hosts"`

There is a persistent way that you can make changes in your **docker-compose.yml**, adding **_extra_hosts_**:

```sh
vi ~/.awx/awxcompose/docker-compose.yml  
awk '/task/,/awx.example.com/' ~/.awx/awxcompose/docker-compose.yml  
docker-compose -f ~/.awx/awxcompose/docker-compose.yml up -d
```

![](1_xz1ARlkYTMlMr7F6IhJ__g.webp)

See [extra_hosts](https://docs.docker.com/compose/compose-file/#extra_hosts) Docker Compose documentation.

# Creating a local inventory

Create a initial inventory.

```sh
cat >> inventory << EOF  
nodea ansible_password=centos ansible_host=172.16.100.151  
nodeb ansible_password=centos ansible_host=172.16.100.152  
nodec ansible_password=centos ansible_host=172.16.100.153  
noded ansible_password=centos ansible_host=172.16.100.154  
EOF
```

# Create the user

Create **devops** user on managed nodes.

```sh
ANSIBLE_LOAD_CALLBACK_PLUGINS=yes ANSIBLE_STDOUT_CALLBACK=yaml ANSIBLE_HOST_KEY_CHECKING=no \  
    ansible -i inventory all -u root -m user -a "name=devops state=present"
```

![](1_Y9UTLYVW7C7h5ivR5K0Axg.webp)

# Create the sudoers file

Allow **devops** to use passwordless sudo on managed nodes.

```sh
ANSIBLE_LOAD_CALLBACK_PLUGINS=yes ANSIBLE_STDOUT_CALLBACK=yaml \  
    ansible -i inventory all -u root -m copy \  
      -a "dest=/etc/sudoers.d/devops content='devops ALL=(ALL) NOPASSWD:ALL'"
```

![](1_vbyPqFBXG3JuQNebRmxfAQ.webp)

# Create the OpenSSH keypair

Create OpenSSH keypair.

```sh
ansible localhost -m file -a "path=~/.ssh mode=0700 state=directory"  
ansible localhost -m openssh_keypair -a "path=~/.ssh/id_rsa size=2048"
```

![](1_h0lTXpKAD86AU13k9noW6Q.webp)

# Deploy the OpenSSH Public key

Deploy OpenSSH Public key to managed nodes for **devops** user.

```sh
ANSIBLE_LOAD_CALLBACK_PLUGINS=yes ANSIBLE_STDOUT_CALLBACK=yaml \  
    ansible -i inventory all -u root -m authorized_key \  
      -a "user=devops state=present key={{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```

![](1_bq75cNDzKl2vCwByHc-5dg.webp)

# Check privilege scalation

Check if **devops** user has privilege escalation permissions.

ansible -i inventory all -u devops -b -m command -a "id"

![](1_H4F5DMBNwP7olA7iVjLPMQ.webp)

# Summary

In this topic was presented:

- Create a user in Managed Nodes to be used by Ansible for Controller Node.


# part 6 of 8

# Concepts

## What is Workflows

> _Workflows allow you to configure a sequence of disparate job templates (or workflow templates) that may or may not share inventory, playbooks, or permissions. However, workflows have ‘admin’ and ‘execute’ permissions, similar to job templates. A workflow accomplishes the task of tracking the full set of jobs that were part of the release process as a single unit._ [_¹_](https://docs.ansible.com/ansible-tower/latest/html/userguide/workflows.html)

# Accessing AWX

Log in to AWX, in my case **“https://awx.example.com"**

Access login page, insert your credentials > click **[SIGN IN]**

![](1_ST1JhN7A9LJzfGTB1Dw46Q.webp)

# Create a machine credential

Create the **“DevOps User”** machine credential.  
Use the OpenSSH Private Key generated by previous topic.

> _Left Menu (Credentials) > click [+] > Fill the form > click [SAVE]_

![](1__CBRy0Suw3kntwnbg6kmhw.webp)

Here is another way to do the same task using _tower-cli_:

```sh
tower-cli credential create --organization "Default" --name "DevOps User" \  
  --credential-type "Machine" \  
  --inputs "{'username': 'devops', 'ssh_key_data': \"$( sed -z 's/\n/\\n/g' ~/.ssh/id_rsa )\", 'become_method': 'sudo', 'become_username': 'root'}"

```
# Create a project

Create the **“GitHub AWX”** project.  
Use: [https://github.com/cdomingos/ansible-playbook-awxapiinventory.git](https://github.com/cdomingos/ansible-playbook-awxapiinventory.git)

> _Left Menu (Projects) > click [+] > Fill the form > click [SAVE]_

![](1_-_oZWVEscS4Orh2m-mBD6w.webp)

Here is another way to do the same task using _tower-cli_:

```sh
tower-cli project create --organization "Default" --name "GitHub AWX" \  
  --scm-type git \  
  --scm-url [https://github.com/cdomingos/ansible-playbook-awxapiinventory.git](https://github.com/cdomingos/ansible-playbook-awxapiinventory.git) \  
  --scm-update-on-launch true \  
  --wait
```

# Create a job template for _basic_ playbook

Create the **“JT - GitHub AWX Basic”** job template.

> _Left Menu (Templates) > click [+] (Job Template) > Fill the form > click [SAVE]_

![](1_svbsos_lVlI0SR14Xh0CMA.webp)

Here is another way to do the same task using _tower-cli_:

```sh
tower-cli job_template create --name="JT - GitHub AWX Basic" --inventory="Example Inventory" \  
  --credential="DevOps User" \  
  --project="GitHub AWX" --become-enabled true --playbook=basic.yml
```

# Create a job template for awx-api-inventory playbook

Create the **“JT - GitHub AWX API”** job template.

NOTE: no credentials are required.

> _Left Menu (Templates) > click [+] (Job Template) > Fill the form > click [SAVE]_

![](1_ZUTu7InB3ivk42yjuR372A.webp)

Here is another way to do the same task using _tower-cli_:

```sh
tower-cli job_template create --name="JT - GitHub AWX API" --inventory="Example Inventory" \  
    --project="GitHub AWX" --playbook=awx-api-inventory.yml
```

# Create a workflow template

Create the **“WORKFLOW - GitHub AWX”** workflow template.

> _Left Menu (Templates) > click [+] (Workflow Template) > Fill the form > click [SAVE]_

![](1_nP3HZzmvFDnPqsdlEKz6aA.webp)

> _After [SAVE] will redirect to workflow visualizer > click [START] > Select (JT - GitHub AWX Basic) / RUN: ALWAYS > click [SELECT] > click (+) in (JT - GitHub AWX Basic) > Select (JT - GitHub AWX API) / RUN: On Success > click [SELECT] > click [SAVE]_

![](1_gDi36bzSDBVyKNzAOtqCCA.webp)

Here is another way to do the same task using _tower-cli_ [²](https://github.com/ansible/tower-cli/blob/master/docs/source/cli_ref/usage/WORKFLOWS.rst)_:_

```sh
tower-cli workflow create --name="WORKFLOW - GitHub AWX"tower-cli node create --workflow-job-template "WORKFLOW - GitHub AWX" \  
  --job-template "JT - GitHub AWX Basic"tower-cli node create --workflow-job-template "WORKFLOW - GitHub AWX" \  
  --job-template "JT - GitHub AWX API"tower-cli node associate_success_node 1 2
```

# Run workflow

Run the **“WORKFLOW - GitHub AWX”** workflow.

> _Left Menu (Templates) > click [rocket icon] in (WORKFLOW - GitHub AWX)_

![](1_Sb6JwjTaCkkLkMnpRNQAxA.webp)

Here is another way to do the same task using _tower-cli_:

```sh
tower-cli workflow_job launch --workflow-job-template "WORKFLOW - GitHub AWX" --monitor
```

# Check workflow results

Check the **“WORKFLOW - GitHub AWX”** workflow and their jobs results.

> _Left Menu (Jobs) > click last job for (WORKFLOW - GitHub AWX)_

![](1_EjdjGKkO2qrNP6IYVTDauA.webp)
![](1_5yEw0-smxBIB0R2M4X0ceg.webp)
![](1_KEBiWDQbs3KryAuy98r6sw.webp)

Here is another way to do the same task using _tower-cli_:

tower-cli job list --job-template "JT - GitHub AWX Basic"tower-cli job list --job-template "JT - GitHub AWX API"tower-cli workflow_job list --workflow-job-template "WORKFLOW - GitHub AWX"

# Check inventory

Check the **“Example Inventory”** inventory status.

> _Left Menu (Inventories) > click (Example Inventory) > click [HOSTS]_

![](1_r5I96a2kYJZiL0bO2QKOBg.webp)

Here is another way to do the same task using _tower-cli_:

tower-cli host list --inventory "Example Inventory"

# Summary

In this topic was presented:

- Working with AWX using Web UI and CLI;
- Create a complete **workflow** in AWX;
- Execute a **workflow** in AWX.

# References

[1] - [https://docs.ansible.com/ansible-tower/latest/html/userguide/workflows.html](https://docs.ansible.com/ansible-tower/latest/html/userguide/workflows.html)  
[2] - [https://github.com/ansible/tower-cli/blob/master/docs/source/cli_ref/usage/WORKFLOWS.rst](https://github.com/ansible/tower-cli/blob/master/docs/source/cli_ref/usage/WORKFLOWS.rst)

# part 7 of 8

This topic is about encrypt sensitive data that are defined as variables in inventory.

Sensitive data such as credentials can be encrypted when used by Playbooks, in real scenario the Ansible AWX or Red Hat Ansible Tower have many users accessing playbooks, inventories, variables, etc.

Ansible Vault is the official tool to encrypt any sensitive data used by Ansible.

# Concepts

## What is Ansible Vault

> _Ansible Vault is a feature of ansible that allows you to keep sensitive data such as passwords or keys in encrypted files, rather than as plaintext in playbooks or roles. These vault files can then be distributed or placed in source control._ [_¹_](https://docs.ansible.com/ansible/latest/user_guide/vault.html)

# Create a vault credential

Create the **“Vault AWX”** vault credential.

> _Left Menu (Credentials) > click [+] > Fill the form > click [SAVE]_

![](1_Y9v1kUFw1Cd1jAWpPaxXsg.webp)

Here is another way to do the same task using _tower-cli_:

```sh
tower-cli credential create --organization "Default" --name="Vault AWX" --credential-type="Vault" \  
  --inputs='{"vault_password": "myvaultpass"}'
```

# Encrypt sensitive data using ansible-vault

For requested _“New Vault password:”_ use **_myvaultpass_** according previous task.

`ansible-vault encrypt_string "admin" --name "api_awx_username"`

![](1_0eksS8hAzJtlBjALfOteCQ.webp)

```sh
ansible-vault encrypt_string "mypassword" --name "api_awx_password"

ansible-vault encrypt_string "mypassword" --name "api_awx_password"
```

![](1_m6UWjOwD58a5hq033xMHyw.webp)

# Adjust inventory

Adjust the **“Example Inventory”** inventory.

NOTE: before copying results, convert it to the following format:

## Before

api_awx_username: **!vault |**  
          $ANSIBLE_VAULT;1.1;AES256  
          64313237613433336139303839353030356336346236333139336331373462303335326662376665  
...output omitted...

## After

```
api_awx_username:  
  **__ansible_vault: |**  
          $ANSIBLE_VAULT;1.1;AES256  
          64313237613433336139303839353030356336346236333139336331373462303335326662376665  
...output omitted...
```

Repeat the process for `api_awx_password` variable.

> _Left Menu (Inventories) > click (Example Inventory) > Edit VARIABLES > click [SAVE]_

![](1_kG9q0wRib0fRntge5n7bSw.webp)

![](1_yMQAde-NAk-MnNbOG4sbPw.webp)

Here is another way to do the same task using _tower-cli_:

![](1_xSGjo4m6FGpBTz4-aUnDug.webp)

```sh
tower-cli inventory modify --organization "Default" --name "Example Inventory" \  
  --variables @vault.yml
```

# Adjust job template

Adjust the **“JT - GitHub AWX API”** job template adding the **“Vault AWX”** vault credential.

> _Left Menu (Templates) > click (JT - GitHub AWX API) > Search CREDENTIALS / CREDENTIALS TYPE: Vault (Vault AWX) > click [SAVE]_

![](1_PzoGvOrW71SlghKQQq37RQ.webp)

Here is another way to do the same task using _tower-cli_:

tower-cli job_template modify --name "JT - GitHub AWX API" --credential "Vault AWX"

# Run workflow

Run the **“WORKFLOW - GitHub AWX”** workflow.

> _Left Menu (Templates) > click [rocket icon] in (WORKFLOW - GitHub AWX)_

![](1_1ck-TGyZpFtAkvUFPHrbMg.webp)

Here is another way to do the same task using _tower-cli_:

tower-cli workflow_job launch --workflow-job-template "WORKFLOW - GitHub AWX" --monitor

# Check workflow results

Check the **“WORKFLOW - GitHub AWX”** workflow and their jobs results.

> _Left Menu (Jobs) > click last job for (WORKFLOW - GitHub AWX)_

![](1_sRMCPHzfg3iH6meY1sNbXQ.webp)

![](1_RTiyXynamlwa_DHyi492Nw.webp)

Here is another way to do the same task using _tower-cli_:

tower-cli job list --job-template "JT - GitHub AWX Basic"tower-cli job list --job-template "JT - GitHub AWX API"tower-cli workflow_job list --workflow-job-template "WORKFLOW - GitHub AWX"

# Summary

In this topic was presented:

- Working with AWX using Web UI and CLI;
- Encrypt sensitive data using **ansible-vault;**
- Adjust AWX Job Template to decrypt this data using a **Vault Credential**.

# References

[1] - [https://docs.ansible.com/ansible/latest/user_guide/vault.html](https://docs.ansible.com/ansible/latest/user_guide/vault.html)

# part 8 of 8

You can increase your security access to REST API using OAuth 2 Token Authentication method instead of Basic HTTP:  
[https://docs.ansible.com/ansible-tower/latest/html/administration/oauth2_token_auth.html](https://docs.ansible.com/ansible-tower/latest/html/administration/oauth2_token_auth.html)

It is very recommended that you use dynamic inventories instead of static inventories (e.g. AWS EC2):  
[https://docs.ansible.com/ansible-tower/latest/html/userguide/inventories.html](https://docs.ansible.com/ansible-tower/latest/html/userguide/inventories.html)

- So, do you already know how to automate your application using Ansible?
- What about your pipeline?
- How are you kickstarting your machines?
- How are you preparing your environment?


This guide is also published in my [GitHub](https://github.com/cdomingos/article-towerawx-restapi-inventory).