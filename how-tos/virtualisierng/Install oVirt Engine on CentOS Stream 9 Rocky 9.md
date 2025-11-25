---
tags:
  - virtualisierung
  - kvm
  - 0virt
---
oVirt is a free to use and open source virtualization management solution whose development is backed by Red Hat. oVirt is build for management of KVM (Kernel-based Virtual Machine) virtualization environments. It ships with features that enables you to manage storage, networking, and virtual machines, all this from a centralized web-based administrative interface.

The key components of oVirt project are the oVirt Engine and oVirt Node. oVirt Engine is a component that provides a graphical user interface and a REST API for the purpose of managing all resources in the virtualized environment. You can install oVirt Engine on a physical or virtual machine running any Enterprise Linux.

This article will provide the guidance on the manual installation of standalone oVirt Engine on a CentOS Stream 9 / Rocky Linux 9 Linux system. The OS must be installed before running the script called `engine-setup` that will perform the configuration of oVirt Engine. Once it’s setup you can then add compute hosts and configure storage for running Virtual Machines.

[![oVirt Engine](https://computingforgeeks.com/wp-content/uploads/2024/02/oVirt-Engine-2048x1078.png?ezimgfmt=ng%3Awebp%2Fngcb23%2Frs%3Adevice%2Frscb23-1 "Install oVirt Engine on CentOS Stream 9 / Rocky 9 1")](https://computingforgeeks.com/wp-content/uploads/2024/02/oVirt-Engine.png?ezimgfmt=ng%3Awebp%2Fngcb23%2Frs%3Adevice%2Frscb23-1)

Here are the hardware requirements for installing oVirt Engine on standalone VM or dedicated server.

|_Resource_|_Minimum_|_Recommended_|
|---|---|---|
|CPU|A dual core x86_64 CPU.|A quad core x86_64 CPU or multiple dual core x86_64 CPUs.|
|Memory|4 GB of available system RAM if Data Warehouse is not installed and if memory is not being consumed by existing processes.|16 GB of system RAM.|
|Hard Disk|25 GB of locally accessible, writable disk space.|50 GB of locally accessible, writable disk space.You can use the [RHV Engine History Database Size Calculator](https://access.redhat.com/labs/rhevmhdsc/) to calculate the appropriate disk space for the Engine history database size.|
|Network Interface|1 Network Interface Card (NIC) with bandwidth of at least 1 Gbps.|1 Network Interface Card (NIC) with bandwidth of at least 1 Gbps.|
## 1. Pull updates and set ntp

**Login to your Linux server instance.**

```sh
ssh root@ServerIP or username@ServerIP
```

**Run the commands below to ensure all system packages are up-to-date.**

```sh
sudo dnf -y update
```

**When kernel updates are applied, a reboot is required.**

```sh
sudo reboot
```

**Change the currently used time zone**

```sh
sudo timedatectl set-timezone your_time_zone
```

**To list all available time zones, type the following at a shell prompt:**

```sh
timedatectl list-timezones
```

**To change the time zone to _Africa/Nairobi_, I will type:**

```sh
sudo timedatectl set-timezone Africa/Nairobi
```

**Also enable automatic synchronization of the system clock with a remote server.**

```sh
sudo timedatectl set-ntp yes
```

**Install Chrony time synchronization.**

```sh
sudo dnf -y install chrony
sudo systemctl enable --now chronyd
```

**Manually sync time.**

```sh
sudo chronyc sources
```

**Confirm your system local time value.**

```sh
timedatectl
               Local time: Thu 2024-02-08 10:36:19 EAT
           Universal time: Thu 2024-02-08 07:36:19 UTC
                 RTC time: Thu 2024-02-08 07:36:19
                Time zone: Africa/Nairobi (EAT, +0300)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

## 2. Add oVirt Engine RPM Repositories

**Let’s enable the correct repositories which contain oVirt Engine packages.**

```sh
sudo dnf install -y centos-release-ovirt45
```

**Expected installation output:**

```sh
...
Dependencies resolved.
======================================================================================================================================================================================================
....

Transaction Summary
======================================================================================================================================================================================================
Install  12 Packages

Total download size: 105 k
Installed size: 37 k
Downloading Packages:
```

For Rocky Linux 9, update repository configurations file to replace `$stream` with `9-stream`. This will enable us to use CentOS Stream 9 repository on Rocky Linux 9 system.

```sh
for repo in oVirt-4.5 Storage-common OpenStack-yoga Messaging-rabbitmq NFV-OpenvSwitch Ceph-Pacific Gluster-10 OpsTools; do
 sudo sed -i 's/$stream/9-stream/' /etc/yum.repos.d/CentOS-$repo.repo
done
```

**List the repositories currently enabled on the system.**

```sh
sudo dnf repolist
repo id                                                                                repo name
appstream                                                                              CentOS Stream 9 - AppStream
baseos                                                                                 CentOS Stream 9 - BaseOS
centos-ceph-pacific                                                                    CentOS-9-stream - Ceph Pacific
centos-gluster10                                                                       CentOS-9-stream - Gluster 10
centos-nfv-openvswitch                                                                 CentOS Stream 9 - NFV OpenvSwitch
centos-openstack-yoga                                                                  CentOS-9 - OpenStack yoga
centos-opstools                                                                        CentOS Stream 9 - OpsTools - collectd
centos-ovirt45                                                                         CentOS Stream 9 - oVirt 4.5
centos-rabbitmq-38                                                                     CentOS-9 - RabbitMQ 38
crb                                                                                    CentOS Stream 9 - CRB
extras-common                                                                          CentOS Stream 9 - Extras packages
ovirt-45-upstream                                                                      oVirt upstream for CentOS Stream 9 - oVirt 4.5
resilientstorage                                                                       CentOS Stream 9 - ResilientStorage
```

**Update cache metadata.**

```sh
sudo dnf makecache -y
```

## 3. Installing oVirt Engine

**Set correct hostname for the machine.**

```sh
sudo hostnamectl set-hostname ovirt.mylab.io
```

**Add the IP and its hostname to `/etc/hosts` file.**

```sh
sudo vim /etc/hosts
ovirt.mylab.io 192.168.1.8
```

**Now that we’ve configured the repositories required, let’s install the package and dependencies for the oVirt Engine.**

```sh
sudo dnf install ovirt-engine
```

**Accept installation when prompted to proceed.**

```sh
....
Transaction Summary
======================================================================================================================================================================================================
Install  687 Packages

Total download size: 845 M
Installed size: 2.6 G
Is this ok [y/N]: y
```

**Import all GPG keys as guided during installation.**

```sh
...
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                  28 MB/s | 845 MB     00:30
CentOS Stream 9 - NFV OpenvSwitch                                                                                                                                     1.0 MB/s | 1.0 kB     00:00
Importing GPG key 0x9D2A76A7:
 Userid     : "CentOS NFV SIG (https://wiki.centos.org/SpecialInterestGroup/NFV) <security@centos.org>"
 Fingerprint: 3515 4228 1749 01BE FA8E 69A6 2146 5E28 9D2A 76A7
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-NFV
Is this ok [y/N]: y
Key imported successfully
CentOS-9 - OpenStack yoga                                                                                                                                             1.0 MB/s | 1.0 kB     00:00
Importing GPG key 0x764429E6:
 Userid     : "CentOS Cloud SIG (http://wiki.centos.org/SpecialInterestGroup/Cloud) <security@centos.org>"
 Fingerprint: 736A F511 6D9C 40E2 AF6B 074B F9B9 FEE7 7644 29E6
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Cloud
Is this ok [y/N]: y
Key imported successfully
CentOS Stream 9 - OpsTools - collectd                                                                                                                                 622 kB/s | 1.0 kB     00:00
Importing GPG key 0x51BC2A13:
 Userid     : "CentOS OpsTools SIG (https://wiki.centos.org/SpecialInterestGroup/OpsTools) <security@centos.org>"
 Fingerprint: 7872 8176 9AD7 3878 85EE A649 4FD9 5327 51BC 2A13
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-OpsTools
Is this ok [y/N]: y
Key imported successfully
CentOS Stream 9 - oVirt 4.5                                                                                                                                           1.0 MB/s | 1.0 kB     00:00
Importing GPG key 0x61E8806C:
 Userid     : "CentOS Virtualization SIG (http://wiki.centos.org/SpecialInterestGroup/Virtualization) <security@centos.org>"
 Fingerprint: A7C8 E761 309D 2F1C 92C5 0B62 7AEB BE82 61E8 806C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Virtualization
Is this ok [y/N]: y
Key imported successfully
oVirt upstream for CentOS Stream 9 - oVirt 4.5                                                                                                                        1.2 MB/s | 1.6 kB     00:00
Importing GPG key 0x24901D0C:
 Userid     : "oVirt <infra@ovirt.org>"
 Fingerprint: 3C98 E81D B93D EA6D 54DE 690E 44E4 75CB 2490 1D0C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-oVirt-4.5
Is this ok [y/N]: y
```

**Confirm successful installation by querying for package details using `rpm` command.**

```sh
$ rpm -qi ovirt-engine
Name        : ovirt-engine
Version     : 4.5.5
Release     : 1.el9
Architecture: noarch
Install Date: Thu 08 Feb 2024 09:49:46 AM EAT
Group       : Virtualization/Management
Size        : 39491334
License     : ASL 2.0
Signature   : RSA/SHA256, Fri 01 Dec 2023 12:12:24 PM EAT, Key ID 7aebbe8261e8806c
Source RPM  : ovirt-engine-4.5.5-1.el9.src.rpm
Build Date  : Fri 01 Dec 2023 11:01:32 AM EAT
Build Host  : x86-03.mbox.rdu2.centos.org
Packager    : CBS <cbs@centos.org>
Vendor      : CentOS Community Build Service
URL         : http://www.ovirt.org
...
```

## 4. Configure oVirt Engine

**To configure oVirt Engine, we need to run the `engine-setup` command:**

```sh
sudo engine-setup
```

**Initialization starts immediately.**

```sh
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: /etc/ovirt-engine-setup.conf.d/10-packaging-jboss.conf, /etc/ovirt-engine-setup.conf.d/10-packaging.conf
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20240208065045-1nrdhj.log
          Version: otopi-1.10.4 (otopi-1.10.4-1.el9)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment setup (late)
[ INFO  ] Stage: Environment customization
...
```

**Configure key options required to deploy oVirt Engine.**

```sh
          --== PRODUCT OPTIONS ==--      
          Configure Cinderlib integration (Currently in tech preview) (Yes, No) [No]:Yes
          Configure Engine on this host (Yes, No) [Yes]:Yes

          Configuring ovirt-provider-ovn also sets the Default cluster's default network provider to ovirt-provider-ovn.
          Non-Default clusters may be configured with an OVN after installation.
          Configure ovirt-provider-ovn (Yes, No) [Yes]:<Enter>
          Configure WebSocket Proxy on this host (Yes, No) [Yes]:<Enter>

          * Please note * : Data Warehouse is required for the engine.
          If you choose to not configure it on this host, you have to configure
          it on a remote host, and then configure the engine on this host so
          that it can access the database of the remote Data Warehouse host.
          Configure Data Warehouse on this host (Yes, No) [Yes]:<Enter>

          * Please note * : Keycloak is now deprecating AAA/JDBC authentication module.
          It is highly recommended to install Keycloak based authentication.
          Configure Keycloak on this host (Yes, No) [Yes]:<Enter>
          Configure VM Console Proxy on this host (Yes, No) [Yes]:<Enter>
          Configure Grafana on this host (Yes, No) [Yes]:<Enter>
```

**It will check if any updates are available.**

```sh
          --== PACKAGES ==--

[ INFO  ] Checking for product updates...
[ INFO  ] No product updates found
```

**Under Network configurations confirm hostname or set a new one.**

```sh
--== NETWORK CONFIGURATION ==--

          Host fully qualified DNS name of this server [ovirt.mylab.io]: ovirt.mylab.io
```

**Database customization has many options. Work with what suits your use. Here we will go with all default values.**

```sh
 --== DATABASE CONFIGURATION ==--

          Where is the DWH database located? (Local, Remote) [Local]:<Enter>

          Setup can configure the local postgresql server automatically for the DWH to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create DWH database, or prefer to perform that manually? (Automatic, Manual) [Automatic]:<Enter>

          Where is the Keycloak database located? (Local, Remote) [Local]:<Enter>

          Setup can configure the local postgresql server automatically for the Keycloak to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Keycloak database, or prefer to perform that manually? (Automatic, Manual) [Automatic]:<Enter>
          Where is the ovirt cinderlib database located? (Local, Remote) [Local]:<Enter>
          Setup can configure the local postgresql server automatically for the CinderLib to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create CinderLib database, or prefer to perform that manually? (Automatic, Manual) [Automatic]:<Enter>
          Where is the Engine database located? (Local, Remote) [Local]:<Enter>

          Setup can configure the local postgresql server automatically for the engine to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]:<Enter>
```

**Set Engine admin user password.**

```sh
--== OVIRT ENGINE CONFIGURATION ==--

          Engine admin password:<Set-New-Password>
          Confirm engine admin password:<Confirm-New-Password>
          Application mode (Virt, Gluster, Both) [Both]:<Enter>
          Use Engine admin password as initial keycloak admin password (Yes, No) [Yes]:<Enter>
```

**Complete by settings values for the remaining options.**

```sh
 --== STORAGE CONFIGURATION ==--

          Default SAN wipe after delete (Yes, No) [No]:<Enter>
  --== PKI CONFIGURATION ==--

          Organization name for certificate [mylab.io]:<Enter>

 --== APACHE CONFIGURATION ==--

          Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
          Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]:<Enter>

 --== SYSTEM CONFIGURATION ==--


          --== MISC CONFIGURATION ==--

          Please choose Data Warehouse sampling scale:
          (1) Basic
          (2) Full
          (1, 2)[1]:<Enter>
          Use Engine admin password as initial Grafana admin password (Yes, No) [Yes]:<Enter>

          --== END OF CONFIGURATION ==--
```

**At the end you will get an output with all the configurations set.**

```sh
 --== CONFIGURATION PREVIEW ==--

          Application mode                        : both
          Default SAN wipe after delete           : False
          Host FQDN                               : ovirt.mylab.io
          Update Firewall                         : False
          CinderLib database host                 : localhost
          CinderLib database port                 : 5432
          CinderLib database secured connection   : False
          CinderLib database host name validation : False
          CinderLib database name                 : ovirt_cinderlib
          CinderLib database user name            : ovirt_cinderlib
          Set up Cinderlib integration            : True
          Configure local CinderLib database      : True
          Configure local Engine database         : True
          Set application as default page         : True
          Configure Apache SSL                    : True
          Keycloak installation                   : True
          Engine database host                    : localhost
          Engine database port                    : 5432
          Engine database secured connection      : False
          Engine database host name validation    : False
          Engine database name                    : engine
          Engine database user name               : engine
          Engine installation                     : True
          PKI organization                        : mylab.io
          Set up ovirt-provider-ovn               : True
          DWH installation                        : True
          DWH database host                       : localhost
          DWH database port                       : 5432
          DWH database secured connection         : False
          DWH database host name validation       : False
          DWH database name                       : ovirt_engine_history
          Configure local DWH database            : True
          Grafana integration                     : True
          Grafana database user name              : ovirt_engine_history_grafana
          Keycloak database host                  : localhost
          Keycloak database port                  : 5432
          Keycloak database secured connection    : False
          Keycloak database host name validation  : False
          Keycloak database name                  : ovirt_engine_keycloak
          Keycloak database user name             : ovirt_engine_keycloak
          Configure local Keycloak database       : True
          Configure VMConsole Proxy               : True
          Configure WebSocket Proxy               : True
```

**Just hit _Enter_ to begin configurations of oVirt Engine.**

```
Please confirm installation settings (OK, Cancel) [OK]:
```

**Wait for the configuration process to complete**

```
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping engine service
[ INFO  ] Stopping ovirt-fence-kdump-listener service
[ INFO  ] Stopping dwh service
[ INFO  ] Stopping vmconsole-proxy service
[ INFO  ] Stopping websocket-proxy service
[ INFO  ] Stage: Misc configuration (early)
[ INFO  ] Stage: Package installation
[ INFO  ] Stage: Misc configuration
[ INFO  ] Initializing PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_cinderlib' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Upgrading CA
[ INFO  ] Creating PostgreSQL 'engine' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_engine_history' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_engine_keycloak' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating CA: /etc/pki/ovirt-engine/ca.pem
[ INFO  ] Creating CA: /etc/pki/ovirt-engine/qemu-ca.pem
[ INFO  ] Creating a user for Grafana
[ INFO  ] Allowing ovirt_engine_history_grafana to read data on ovirt_engine_history
[ INFO  ] Setting up ovirt-vmconsole proxy helper PKI artifacts
[ INFO  ] Setting up ovirt-vmconsole SSH PKI artifacts
[ INFO  ] Configuring WebSocket Proxy
[ INFO  ] Creating/refreshing Engine database schema
```

**Successful installation end summary report is printed in your screen.**

```

--== END OF SUMMARY ==--

[ INFO  ] Restarting httpd
[ INFO  ] Start with setting up Keycloak for Ovirt Engine
[ INFO  ] Done with setting up Keycloak for Ovirt Engine
[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20240208065045-1nrdhj.log
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20240208070615-setup.conf'
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of setup completed successfully
```

The `engine-setup` will also display details about how to access your environment.

## 5. Configuring firewalld

**You can install and activate firewalld to keep your services safe.**

```sh
sudo dnf -y install firewalld
sudo systemctl enable --now firewalld
```

**Copy xml files with ports needed by oVirt Engine defined.**

```sh
sudo cp /etc/ovirt-engine/firewalld/* /etc/firewalld/services
```

**Reload firewalld rules.**

```
sudo firewall-cmd --reload
```

**Allow other oVirt services in the firewalld.**

```
for service in ovn-central-firewall-service ovirt-provider-ovn ovirt-http \
  ovirt-https ovirt-vmconsole-proxy  ovirt-websocket-proxy \
  ovirt-fence-kdump-listener ovirt-imageio-proxy ovirt-postgres; do
sudo firewall-cmd --permanent --add-service $service;
done
```

**Reload and confirm if they were added to allow list.**

```sh
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

## 6. Access oVirt Engine web interface

The FQDN value used in setting up hostname should have an A record in your DNS server. Alternatively, create a record in local workstation `/etc/hosts` file.

```sh
IP FQDN
```

To access oVirt Engine on web browser, use **_https://manager-fqdn/ovirt-engine_**. Ignore SSL certificate risk warning.

![install ovirt engine 01](https://computingforgeeks.com/wp-content/uploads/2024/02/install-ovirt-engine-01-1024x426.png?ezimgfmt=rs:696x290/rscb23/ng:webp/ngcb23 "Install oVirt Engine on CentOS Stream 9 / Rocky 9 2")

You can get the certificate authority’s certificate on below URL. Replacing **_manager-fqdn_** with the FQDN that you provided during the installation.

```sh
http://<manager-fqdn>/ovirt-engine/services/pki-resource?resource=ca-certificate&format=X509-PEM-CA
```

See [Configuring oVirt / RHEV Manager Certificate Security on browser](https://computingforgeeks.com/configuring-ovirt-rhev-manager-certificate-security-on-browser/)

To access admin panel, use “**Administration Portal”** under “_Portals_”

![install ovirt engine 02](https://computingforgeeks.com/wp-content/uploads/2024/02/install-ovirt-engine-02-1024x432.png?ezimgfmt=rs:696x294/rscb23/ng:webp/ngcb23 "Install oVirt Engine on CentOS Stream 9 / Rocky 9 3")

Login with admin user account created.

- Username: **admin@ovirt**
- Password: Use password that you specified during installation.

![install ovirt engine 03](https://computingforgeeks.com/wp-content/uploads/2024/02/install-ovirt-engine-03-1024x632.png?ezimgfmt=rs:696x430/rscb23/ng:webp/ngcb23 "Install oVirt Engine on CentOS Stream 9 / Rocky 9 4")

You can add alternate host names or IP addresses for access the Administration Portal.

```sh
sudo vim /etc/ovirt-engine/engine.conf.d/99-custom-sso-setup.conf
SSO_ALTERNATE_ENGINE_FQDNS="_alias1.example.com alias2.example.com_"
```

The list of alternate host names needs to be separated by spaces. You can also add the IP address of the Engine to the list, but using IP addresses instead of DNS-resolvable host names is not recommended.

To add compute node use:

- [Install oVirt Compute Node on CentOS Stream 9 / Rocky 9](https://computingforgeeks.com/install-ovirt-compute-node-on-centos-stream-rocky/)
## 7. Access Grafana Dashboard

Grafana web dashboard is available at **_https://manager-fqdn/_ovirt-engine-grafana/**

![install ovirt engine 04](https://computingforgeeks.com/wp-content/uploads/2024/02/install-ovirt-engine-04-1024x650.png?ezimgfmt=rs:696x442/rscb23/ng:webp/ngcb23 "Install oVirt Engine on CentOS Stream 9 / Rocky 9 5")

Login with **admin** user and the password that you specified during installation.

**Next articles to read on oVirt management:**

- [Add NFS Data, ISO and Export Storage Domain to oVirt / RHEV](https://computingforgeeks.com/add-nfs-data-iso-export-storage-domain-to-ovirt-rhev/)
- [Creating VM and Storage Logical Networks in oVirt/RHEV](https://computingforgeeks.com/creating-vm-and-storage-logical-networks-in-ovirt-rhev/)
- [Configure oVirt / RHEV User Authentication using FreeIPA LDAP](https://computingforgeeks.com/configure-ovirt-rhev-user-authentication-using-freeipa-ldap/)
- [How To Create Data Center and Cluster in oVirt / RHEV](https://computingforgeeks.com/how-to-create-data-center-and-cluster-in-ovirt-rhev/)
- [Upload and Use ISO Images on oVirt / RHEV Storage Domain](https://computingforgeeks.com/upload-and-use-iso-images-on-ovirt-rhev-storage-domain/)
- [How To Add Compute Host to oVirt Virtualization](https://computingforgeeks.com/add-compute-host-to-ovirt-virtualization/)
- [How To Provision VMs on oVirt / RHEV with Terraform](https://computingforgeeks.com/how-to-provision-vms-on-ovirt-rhev-with-terraform/)