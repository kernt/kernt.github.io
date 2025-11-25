---
tags:
  - openldap
---

## Configure the OpenLDAP Container

When running OpenLDAP, there are quite a number of configurations you need to make. The Bitnami Docker container supports a number of configurations or environment variables. Below are some of the supported variables:

- **LDAP_PORT_NUMBER**: This is the port on which OpenLDAP is listening for requests. The default supported port here is 1389
- **LDAP_ROOT**: This is the baseDN of your LDAP tree. For example _dc=example,dc=org_
- **LDAP_ADMIN_USERNAME**: This is the admin user for the LDAP database
- **LDAP_ADMIN_PASSWORD**: The desired password for the admin user.
- **LDAP_ADMIN_PASSWORD_FILE**: You can use this to point to the file that contains the LDAP database admin user password instead of specifying the password directly in the YAML
- **LDAP_CONFIG_ADMIN_ENABLED**: This is used to specify whether to create a configuration admin user. Default: no
- **LDAP_CONFIG_ADMIN_USERNAME**: The username for the LDAP configuration admin user. This is separate from LDAP_ADMIN_USERNAME. Default: admin.
- **LDAP_CONFIG_ADMIN_PASSWORD**: A password for the config admin.
- **LDAP_CONFIG_ADMIN_PASSWORD_FILE**: This points to a file containing the LDAP configuration admin user password.
- **LDAP_USERS**: This is a list of users on your LDAP separated by commas. The users will be created in the default tree. For example user01,user02
- **LDAP_PASSWORDS**: A list of passwords to use for the LDAP users. For example: bitnami1,bitnami2
- **LDAP_USER_DC**: This is the users’ organizational unit. The default value is users.
- **LDAP_GROUP**: This is the group used for newly created users. Default: readers
- **LDAP_EXTRA_SCHEMAS**: This is used to add extra schemas among the OpenLDAP’s distributed schemas. Default: cosine, inetorgperson, nis
- **LDAP_SKIP_DEFAULT_TREE**: Used to specify whether to skip creating the default LDAP tree based on LDAP_USERS, LDAP_PASSWORDS, LDAP_USER_DC and LDAP_GROUP.
- **LDAP_CUSTOM_LDIF_DIR**: Used to specify the location of LDIF files that should be used to bootstrap the database.
- **LDAP_CUSTOM_SCHEMA_FILE**: The location of a schema file that could not be added as a custom ldif file.
- **LDAP_CUSTOM_SCHEMA_DIR**: This is the directory of the custom schemas that could not be added as custom ldif files.
- **LDAP_ULIMIT_NOFILES**: This is the maximum number of open file descriptors. Default: 1024.
- **LDAP_PASSWORD_HASH**: The Hash that you want to be used in the generation of user passwords. Must be one of {SSHA}, {SHA}, {SMD5}, {MD5}, {CRYPT}, and {CLEARTEXT}. Default: {SSHA}.
- **LDAP_PPOLICY_HASH_CLEARTEXT**: Used to specify if you want plaintext passwords should be hashed automatically. Will only be applied with LDAP_CONFIGURE_PPOLICY active. Default: no.

**You also have the option to secure OpenLDAP using the following variables**

- **LDAP_ENABLE_TLS**: used to specify whether to enable TLS for traffic or not. Defaults to no.
- **LDAP_REQUIRE_TLS**: Used to set whether connections must use TLS. Will only be applied with LDAP_ENABLE_TLS active. Defaults to no.
- **LDAP_LDAPS_PORT_NUMBER**: This is the port used for TLS secure traffic. Priviledged port is supported (e.g. 636). Default: 1636 (non-privileged port).
- **LDAP_TLS_CERT_FILE**: This is the file that contains the certificate file for the TLS traffic. No defaults.
- **LDAP_TLS_KEY_FILE**: the file that contains the key for the certificate. No defaults.
- **LDAP_TLS_CA_FILE**: The file with the CA of the certificate. No defaults.
- **LDAP_TLS_DH_PARAMS_FILE**: The file that has the DH parameters. No defaults.

For example, you can use a YAML file with the below syntax to secure your OpenLDAP instance.

```yaml
services:
  openldap:
  ...
    environment:
      ...
      - LDAP_ENABLE_TLS=yes
      - LDAP_TLS_CERT_FILE=/opt/bitnami/openldap/certs/openldap.crt
      - LDAP_TLS_KEY_FILE=/opt/bitnami/openldap/certs/openldap.key
      - LDAP_TLS_CA_FILE=/opt/bitnami/openldap/certs/openldapCA.crt
    ...
    volumes:
      - /path/to/certs:/opt/bitnami/openldap/certs
      - /path/to/openldap-data-persistence:/bitnami/openldap/
  ...
```

In this guide, we will use Docker Compose to configure and run the OpenLDAP Bitnami container. First, we will download the sample Docker Compose file:

```
curl -sSL https://raw.githubusercontent.com/bitnami/containers/main/bitnami/openldap/docker-compose.yml > docker-compose.yml
```

You can then proceed and modify the container as desired.

```
vim docker-compose.yml
```

In the file, you can update values as desired. In this guide, we will use the latest available Docker image and update the configs to suit our environment.

```yaml
# Copyright VMware, Inc.
# SPDX-License-Identifier: APACHE-2.0

version: '2'

services:
  openldap:
    image: docker.io/bitnami/openldap:latest
    ports:
      - '389:1389'
      - '636:1636'
    environment:
      - LDAP_ADMIN_USERNAME=admin
      - LDAP_ADMIN_PASSWORD=adminpassword
      - LDAP_USERS=user01,user02
      - LDAP_PASSWORDS=password1,password2
      - LDAP_ROOT=dc=computingforgeeks,dc=org 
      - LDAP_ADMIN_DN=cn=admin,dc=computingforgeeks,dc=org 

    volumes:
      - 'openldap_data:/bitnami/openldap'

volumes:
  openldap_data:
    external: true
```

Once all the desired settings have been made. Save the file and proceed as shown below.
## 3. Create the OpenLDAP Persistent Volume

For the container to persist its data after the system reboots, we need to have persistent storage created. Begin by creating the path on your host machine:

```sh
sudo mkdir -p /data/openldap
```

Set the required permission and ownership:

```sh
sudo chmod 775 -R /data/openldap
sudo chown -R $USER:docker /data/openldap
```

On Rhel-base systems, you need to configure SELinux as shown:

```sh
sudo setenforce 0
sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
```

Now create the volume as shown:

```sh
docker volume create --driver local \
     --opt type=none \
     --opt device=/data/openldap \
     --opt o=bind openldap_data
```

Check if the volume exists:

```sh
$ docker volume list
DRIVER    VOLUME NAME
local     openldap_data
```

## 4. Start OpenLDAP in Bitnami Docker Container

Once all the above configurations have been made, we can proceed and start the container using the command:

```sh
docker compose up -d
```

Sample Output:

```sh
[+] Running 2/2
 ✔ openldap 1 layers [⣿]      0B/0B      Pulled                                          5.6s 
   ✔ f041216ccfa5 Pull complete                                                          1.2s 
[+] Running 2/2
 ✔ Network debian_default         Created                                                0.1s 
 ✔ Container debian-openldap-1    Started                                                0.5s 
```

Check if the container is up:

```sh
$ docker ps
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS          PORTS                                                                              NAMES
bc79a32a90dc   bitnami/openldap:latest   "/opt/bitnami/script…"   42 seconds ago   Up 41 seconds   0.0.0.0:389->1389/tcp, :::389->1389/tcp, 0.0.0.0:636->1636/tcp, :::636->1636/tcp   debian-openldap-1
```

Now all the ports through the firewall:

```sh
##For UFW
sudo ufw allow 389/tcp
sudo ufw allow 636/tcp

##For Firewalld
sudo firewall-cmd --add-port=389/tcp --permanent
sudo firewall-cmd --add-port=636/tcp --permanent
sudo firewall-cmd --reload
```
## 5. Setup OpenLDAP Client

To test if all is okay, we can set the OpenLDAP client on the desired system. There are several guides on how to set up your OpenLDAP client.

- [How to Set Up OpenLDAP Client](https://computingforgeeks.com/?s=LDAP+client)

In this guide, we will setup an Ubuntu client. The first thing to do is setup the hostname for the client:

```sh
sudo hostnamectl set-hostname ldapclient.computingforgeeks.org
```

Next, configure resolution updating /etc/hosts:

```
$ sudo vim /etc/hosts
##OpenLDAP server
192.168.200.56 ldapmaster.computingforgeeks.org

##OpenLDAP Client
192.168.200.52 ldapclient.computingforgeeks.org
```

Ensure that the client can reach the OpenLDAP server:

```sh
$ sudo ping -c3 ldapmaster.computingforgeeks.org
PING ldapmaster.computingforgeeks.org (192.168.200.56) 56(84) bytes of data.
64 bytes from ldapmaster.computingforgeeks.org (192.168.200.56): icmp_seq=1 ttl=64 time=0.232 ms
64 bytes from ldapmaster.computingforgeeks.org (192.168.200.56): icmp_seq=2 ttl=64 time=0.248 ms
64 bytes from ldapmaster.computingforgeeks.org (192.168.200.56): icmp_seq=3 ttl=64 time=0.232 ms

--- ldapmaster.computingforgeeks.org ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2032ms
rtt min/avg/max/mdev = 0.232/0.237/0.248/0.007 ms
```

Then install all the required packages:

```sh
sudo apt update -y && sudo apt -y install libnss-ldap libpam-ldap ldap-utils
```

Proceed and configure the LDAP URI. For our case, we will use _ldap://:hostname or IP_ syntax as shown:

![Run OpenLDAP in Bitnami Docker Container](https://computingforgeeks.com/wp-content/uploads/2023/10/Run-OpenLDAP-in-Bitnami-Docker-Container.png?ezimgfmt=rs:484x257/rscb23/ng:webp/ngcb23 "How To Run OpenLDAP in Bitnami Docker Container 1")

Set the ROOT DN of your search base

![Run OpenLDAP in Bitnami Docker Container 1](https://computingforgeeks.com/wp-content/uploads/2023/10/Run-OpenLDAP-in-Bitnami-Docker-Container-1.png?ezimgfmt=rs:484x208/rscb23/ng:webp/ngcb23 "How To Run OpenLDAP in Bitnami Docker Container 2")

Set the LDAP version to be used:

![Run OpenLDAP in Bitnami Docker Container 2](https://computingforgeeks.com/wp-content/uploads/2023/10/Run-OpenLDAP-in-Bitnami-Docker-Container-2.png?ezimgfmt=rs:484x208/rscb23/ng:webp/ngcb23 "How To Run OpenLDAP in Bitnami Docker Container 3")

Proceed and make local root Database admin

![Run OpenLDAP in Bitnami Docker Container 3](https://computingforgeeks.com/wp-content/uploads/2023/10/Run-OpenLDAP-in-Bitnami-Docker-Container-3.png?ezimgfmt=rs:484x208/rscb23/ng:webp/ngcb23 "How To Run OpenLDAP in Bitnami Docker Container 4")

Answer the next question “Does the LDAP database require login?: with **NO**.

![Run OpenLDAP in Bitnami Docker Container 4](https://computingforgeeks.com/wp-content/uploads/2023/10/Run-OpenLDAP-in-Bitnami-Docker-Container-4.png?ezimgfmt=rs:484x208/rscb23/ng:webp/ngcb23 "How To Run OpenLDAP in Bitnami Docker Container 5")

Set the root account:

![Run OpenLDAP in Bitnami Docker Container 5](https://computingforgeeks.com/wp-content/uploads/2023/10/Run-OpenLDAP-in-Bitnami-Docker-Container-5.png?ezimgfmt=rs:484x208/rscb23/ng:webp/ngcb23 "How To Run OpenLDAP in Bitnami Docker Container 6")

Provide the password for the admin account:

![Run OpenLDAP in Bitnami Docker Container 6](https://computingforgeeks.com/wp-content/uploads/2023/10/Run-OpenLDAP-in-Bitnami-Docker-Container-6.png?ezimgfmt=rs:484x208/rscb23/ng:webp/ngcb23 "How To Run OpenLDAP in Bitnami Docker Container 7")

Once the installation is complete, you will have your config saved at **_/etc/ldap.conf_**

Now open the below file and make these changes to the  `passwd` and `group` lines:

```sh
$ sudo vim /etc/nsswitch.conf
passwd: compat systemd ldap
group: compat systemd ldap
shadow: compat
```

You also need to modify the file below and modify line 26 as shown:

```sh
$ sudo vim /etc/pam.d/common-password
##Line 26
password [success=1 user_unknown=ignore default=die] pam_ldap.so try_first_pass
```

You also need to allow the user’s home directory to be created on the first login. To achieve that, add the below line at the end of the file:

```sh
$ sudo vim /etc/pam.d/common-session
session optional pam_mkhomedir.so skel=/etc/skel umask=077
```

Now save the changes and test if you can log in to the system using the users created on OpenLDAP. For example:

```sh
ssh user01@localhost
```

Sample Output:

![Run OpenLDAP in Bitnami Docker Container 7](https://computingforgeeks.com/wp-content/uploads/2023/10/Run-OpenLDAP-in-Bitnami-Docker-Container-7.png?ezimgfmt=rs:484x315/rscb23/ng:webp/ngcb23 "How To Run OpenLDAP in Bitnami Docker Container 8")