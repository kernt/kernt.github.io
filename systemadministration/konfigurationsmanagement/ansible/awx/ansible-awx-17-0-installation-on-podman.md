**Prerequisites**

Before we begin, we must make sure that all tools required like installed in your machine

- Docker/ Podman — as our container
- Ansible — to run the installation playbook
- Python3-pip- to run

Step 1

Clone or download the awx 17 into your machine.

```sh
cd ~  
git clone -b 17.1.0 https://github.com/ansible/awx.git  
cd awx

```
Step 2

Once awx is downloaded, you can go to the installation directory and you will noticed 3 files and 1 directory

build.yml — AWX Docker image

install.yml — This playbook will be use later to install AWX

inventory — This is a config file where it will saved all configuration including Postgres credential, Podman, Docker resources, AWX Password.

Edit inventory files and you may change it if you like.

```sh
admin_password=Admin@123 #AWX Password  
pg_database=awx          #AWX DB  
pg_password=Admin@123    #AWX DB PASSWORD  
host_port= 80   #This is your awx port  
awx_alternate_dns_servers="8.8.8.8,8.8.4.4"   
postgres_data_dir="/var/lib/awx/pgdocker"      #POSTGRES DATA  
docker_compose_dir="/var/lib/awx/awxcompose"   #DOCKER COMPOSE FILE  
project_data_dir="/var/lib/awx/projects"       #PLAYBOOKS WILL BE SAVE HERE LATER
```

Step 3

Run the playbook to install AWX, Run the command if you in the installation directory.

`ansible-playbook -i inventory install.yml`

This will help you to create 3containers and every container has their own purpose

1. AWX-_WEB — is your AWX UI (Run in port 8052 and forward to 80)
2. AWX_POST — For AWX Database
3. AWX_POSTGRES

Once installation is done, you should able to access your AWX in the browser using port 80 or if you want to change to another port, just go to inventory files and change **`host_port = 80**` to another port you like.

