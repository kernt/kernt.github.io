---
tags:
  - guacamole
---

# Install Guacamole on Debian

## Step 1: Install Abhängigkeiten

*Let’s start by updating our system and installing the dependencies required by Guacamole Remote Desktop.*

```sh
sudo apt update
sudo apt install -y vim build-essential libcairo2-dev libjpeg62-turbo-dev libpng-dev \
libtool-bin libossp-uuid-dev libavcodec-dev libavformat-dev libavutil-dev libswscale-dev \
libpango1.0-dev libssh2-1-dev libvncserver-dev libtelnet-dev \
libssl-dev libvorbis-dev libwebp-dev libpulse-dev
```

*Another tool we need to install is FreeRDP2 which is hosted in the Remmina PPA*

```
echo "deb http://deb.debian.org/debian $(lsb_release -cs)-backports main" | sudo tee /etc/apt/sources.list.d/backports.list
sudo apt update
sudo apt install freerdp2-x11 freerdp2-dev
```

## Step 2: Install Apache Tomcat on Debian

Since we are using Apache Tomcat to run the Guacamole Java war file we need to install Java on our Debian system.

```sh
sudo apt install openjdk-11-jdk
```

**Check the installed version**

```java
$ java --version
openjdk 11.0.16 2022-07-19
OpenJDK Runtime Environment (build 11.0.16+8-post-Debian-1deb11u1)
OpenJDK 64-Bit Server VM (build 11.0.16+8-post-Debian-1deb11u1, mixed mode, sharing)
```

### Install Apache Tomcat on Debian

*To install Tomcat on Debian 11 / Debian 10, issue the command:*

```sh
sudo apt install tomcat9 tomcat9-admin tomcat9-common tomcat9-user
```

*Ensure that the service has been started and enabled:*

```sh
sudo systemctl enable --now tomcat9
```

Check if Tomcat is running:

```sh
$ systemctl status tomcat9
 tomcat9.service - Apache Tomcat 9 Web Application Server
     Loaded: loaded (/lib/systemd/system/tomcat9.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2023-08-16 13:27:41 EDT; 2min 48s ago
       Docs: https://tomcat.apache.org/tomcat-9.0-doc/index.html
   Main PID: 18458 (java)
      Tasks: 29 (limit: 4660)
     Memory: 101.3M
        CPU: 5.938s
     CGroup: /system.slice/tomcat9.service
             └─18458 /usr/lib/jvm/java-11-openjdk-amd64/bin/java -Djava.util.logging.config.file=/var/lib/tomcat9/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLo>
~
....
```

Tomcat listens on port 8080, so we need to allow this port on the firewall. First, install ufw using `sudo apt install ufw`

```sh
sudo ufw allow 8080/tcp
```

## Step 3: Download Guacamole Remote Server

Guacamole Remote Server contains all the native and server components required for remote desktop connections. It provides all the libraries which all native components depend on as well as guacd which is the hub of Guacamole.

Check for the latest stable available version of Guacamole Server from the [release page.](https://archive.apache.org/dist/guacamole/)

Alternatively, download using Wget as below. set the Version variable

```sh
VER=1.5.3
```

Then download it:

```sh
wget https://archive.apache.org/dist/guacamole/$VER/source/guacamole-server-$VER.tar.gz
```

Extract the downloaded file.

```sh
tar xzf guacamole-server-$VER.tar.gz
```

Navigate into the Guacamole directory.

```sh
cd guacamole-server-$VER
```

Then issue the configure script, which checks the available dependencies and adapts the Guacamole server to them.

```sh
./configure --with-init-dir=/etc/init.d
```

Sample Output for the above command:

```
...
------------------------------------------------
guacamole-server version 1.5.3
------------------------------------------------

   Library status:

     freerdp2 ............ yes
     pango ............... yes
     libavcodec .......... yes
     libavformat.......... yes
     libavutil ........... yes
     libssh2 ............. yes
     libssl .............. yes
     libswscale .......... yes
     libtelnet ........... yes
     libVNCServer ........ yes
     libvorbis ........... yes
     libpulse ............ yes
     libwebsockets ....... no
     libwebp ............. yes
     wsock32 ............. no

   Protocol support:

      Kubernetes .... no
      RDP ........... yes
      SSH ........... yes
      Telnet ........ yes
      VNC ........... yes

   Services / tools:

      guacd ...... yes
      guacenc .... yes
      guaclog .... yes

   FreeRDP plugins: /usr/lib/x86_64-linux-gnu/freerdp2
   Init scripts: /etc/init.d
   Systemd units: no

Type "make" to compile guacamole-server.
```

## Step 4: Install Guacamole Remote Desktop on Debian

After making the above check, now it is time to install Guacamole into our Debian system. We need to compile Guacamole-server by issuing the make command as below.

```
make
```

The make command takes some time, once it is complete, now proceed to install Guacamole-server.

```
sudo make install
```

Now issue the ldconfig command, this command links the cache to the recently shared libraries

```
sudo ldconfig
```

Create the required Guacamole directories:

```
sudo mkdir  -p /etc/guacamole/{extensions,lib}
```

Create guacd.conf configuration file:

```
$ sudo vim /etc/guacamole/guacd.conf
[daemon]
pid_file = /var/run/guacd.pid
#log_level = debug

[server]
#bind_host = localhost
bind_host = 127.0.0.1
bind_port = 4822

#[ssl]
#server_certificate = /etc/ssl/certs/guacd.crt
#server_key = /etc/ssl/private/guacd.key
```

Then reload daemons to find the added guacd service.

```
sudo systemctl daemon-reload
```

Start and enable guacd to run on boot

```
sudo systemctl start guacd
sudo systemctl enable guacd
```

Verify if the process is running.

```
$ systemctl status guacd
● guacd.service - LSB: Guacamole proxy daemon
     Loaded: loaded (/etc/init.d/guacd; generated)
     Active: active (running) since Wed 2023-08-16 13:34:38 EDT; 5s ago
       Docs: man:systemd-sysv-generator(8)
      Tasks: 1 (limit: 4660)
     Memory: 9.9M
        CPU: 12ms
     CGroup: /system.slice/guacd.service
             └─32087 /usr/local/sbin/guacd -p /var/run/guacd.pid
```

Download Guacamole client binary same version. set the version variable:

```
VER=1.5.3
```

Pull the archive:

```
wget https://archive.apache.org/dist/guacamole/$VER/binary/guacamole-$VER.war
```

Copy the file to the Tomcat web app directory:

```
sudo mv guacamole-$VER.war /var/lib/tomcat9/webapps/guacamole.war
```

## Step 5: Configure Apache Guacamole on Debian

Guacamole has two main config files i.e

- stored at **/etc/guacamole** referenced by GACAMOLE_HOME environment variable
- stored at **/etc/guacamole/guacamole.properties** this is the main file used by Guacamole and its extensions.

Create a GUACAMOLE_HOME environment variable.

```
sudo echo "GUACAMOLE_HOME=/etc/guacamole" | sudo tee -a /etc/default/tomcat
```

Then define how Guacamole communicates with guacd by creating the guacamole.properties file under **/etc/guacamole** as shown.

```
sudo vim /etc/guacamole/guacamole.properties
```

Edit your file as below:

```
guacd-hostname: localhost
guacd-port:    4822
#user-mapping:    /etc/guacamole/user-mapping.xml
#auth-provider:    net.sourceforge.guacamole.net.basic.BasicFileAuthenticationProvider
```

## Step 6: Set Guacamole Database Authentication Method

By default, Guacamole’s authentication method reads all users and connections from a single file named **user-mapping.xml**. In this file, all users to access Guacamole web UI, servers to connect to as well as the connection methods are defined. But this method of defining authentication is not recommended. For production, you can use database, LDAP or DUO authentication.

In this guide, we will use database authentication. First, install MySQL or MariaDB on your Debian system.

- [Install MariaDB on Debian](https://computingforgeeks.com/?s=install+MariaDB+on+Debian)
- [Install MySQL on Debian](https://computingforgeeks.com/?s=install+MySQL+on+Debian)

Once installed, access the shell as the root user:

```
sudo mysql -u root -p
```

Create a user and database for Guacamole with the SQL commands below:

```
CREATE DATABASE guacamole_db;
CREATE USER 'guacamole_user'@'localhost' IDENTIFIED BY 'Passw0rd!';
GRANT SELECT,INSERT,UPDATE,DELETE ON guacamole_db.* TO 'guacamole_user'@'localhost';
FLUSH PRIVILEGES;
QUIT
```

Next, download the [MySQL Java Connector](https://dev.mysql.com/downloads/connector/j/). You can also export the latest version:

```
VER=8.1.0
```

Then download it with the command:

```
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-$VER.tar.gz
```

Extract the file and copy it to /etc/guacamole/lib/

```
tar -xf mysql-connector-j-*.tar.gz
sudo cp mysql-connector-j-$VER/mysql-connector-j-$VER.jar /etc/guacamole/lib/
```

The other thing required is the [JDBC auth](https://guacamole.apache.org/releases/) plugin. On the site check the latest available version:

```
VER=1.5.3
```

Download the specified version above:

```
wget https://downloads.apache.org/guacamole/$VER/binary/guacamole-auth-jdbc-$VER.tar.gz
```

Extract it and copy it to the /etc/guacamole/extensions/ directory:

```
tar -xf guacamole-auth-jdbc-$VER.tar.gz
sudo mv guacamole-auth-jdbc-$VER/mysql/guacamole-auth-jdbc-mysql-$VER.jar /etc/guacamole/extensions/
```

We now need to import the SQL schema for Guacamole. Navigate to the JDBC path with the command:

```
cd guacamole-auth-jdbc-*/mysql/schema
```

Import the schemas

```
cat *.sql | sudo mysql -u root -p guacamole_db
```

You will be required to provide the MySQL root password to proceed. Once imported, you need to modify Guacamole settings:

```
sudo vim /etc/guacamole/guacamole.properties
```

In the opened file, add these lines:

```
###MySQL properties
mysql-hostname: 127.0.0.1
mysql-port: 3306
mysql-database: guacamole_db
mysql-username: guacamole_user
mysql-password: Passw0rd!
```

Save the file and restart the related services:

```
sudo systemctl restart tomcat9 guacd
```

## Step 7: Accessing Guacamole Remote Desktop Web Interface

Now Apache Guacamole is set up, we can now access it from the browser using the URL:

```
http://server-IP:8080/guacamole
```

You should be able to see the login screen below:

![Install Guacamole Remote Desktop on Debian](https://computingforgeeks.com/wp-content/uploads/2023/08/Install-Guacamole-Remote-Desktop-on-Debian.png?ezimgfmt=rs:484x473/rscb23/ng:webp/ngcb23 "Install Guacamole Remote Desktop on Debian 11 / Debian 10 1")

Login using the default creds **guacadmin** as the username and **guacadmin** as the password. Once connected, it is recommended to delete the default admin user and create a new one.

To create a new admin user, navigate to **Settings** ->**User**->**New User**.

![Install Guacamole Remote Desktop on Debian 1](https://computingforgeeks.com/wp-content/uploads/2023/08/Install-Guacamole-Remote-Desktop-on-Debian-1-1024x928.png?ezimgfmt=rs:484x439/rscb23/ng:webp/ngcb23 "Install Guacamole Remote Desktop on Debian 11 / Debian 10 2")

Once the user has been created, you can **log out and log in** using the new user. Then proceed and delete the old default user:

![Install Guacamole Remote Desktop on Debian 2](https://computingforgeeks.com/wp-content/uploads/2023/08/Install-Guacamole-Remote-Desktop-on-Debian-2-1024x840.png?ezimgfmt=rs:484x397/rscb23/ng:webp/ngcb23 "Install Guacamole Remote Desktop on Debian 11 / Debian 10 3")

### Create New Guacamole Connections

To be able to make SSH, VNC, RDP connections, we need to define them on Guacamole. To achieve that, navigate to **Settings** ->**Connection**->**New Connection**

![Install Guacamole Remote Desktop on Debian 5](https://computingforgeeks.com/wp-content/uploads/2023/08/Install-Guacamole-Remote-Desktop-on-Debian-5.png?ezimgfmt=rs:484x628/rscb23/ng:webp/ngcb23 "Install Guacamole Remote Desktop on Debian 11 / Debian 10 4")

When creating the connection, provide the protocol and also the IP/hostname and port for the server, username and password under the **Parameters**->**Network**.

If you have SSH key authentication configured previously on the remote systems, you need to make the below adjustments to avoid an issue with SSH “**ssh handshake failed**.”

```
$ sudo vim /etc/ssh/sshd_config
HostKeyAlgorithms +ssh-rsa
PubkeyAcceptedKeyTypes +ssh-rsa
```

Apply the changes:

```
sudo systemctl restart sshd
```

Now your connections will appear on your Guacamole Home as shown:

![Install Guacamole Remote Desktop on Debian 4](https://computingforgeeks.com/wp-content/uploads/2023/08/Install-Guacamole-Remote-Desktop-on-Debian-4-1024x410.png?ezimgfmt=rs:484x194/rscb23/ng:webp/ngcb23 "Install Guacamole Remote Desktop on Debian 11 / Debian 10 5")

Click on the desired connection to initiate it. For example, for ssh login to the Rocky8 server, click on it and you will see the login prompt as shown:

![Install Guacamole Remote Desktop on Debian 3](https://computingforgeeks.com/wp-content/uploads/2023/08/Install-Guacamole-Remote-Desktop-on-Debian-3.png?ezimgfmt=rs:484x263/rscb23/ng:webp/ngcb23 "Install Guacamole Remote Desktop on Debian 11 / Debian 10 6")

End the session using exit on your terminal you can still **reconnect**/ go to the **Home page**/ **Logout** from the Guacamole server whenever you want from any device i.e. computer, phone, tablet etc.

![Install and Use Guacamole Remote Desktop on Debian 10 Buster 2](https://computingforgeeks.com/wp-content/uploads/2021/08/Install-and-Use-Guacamole-Remote-Desktop-on-Debian-10-Buster-2.png?ezimgfmt=rs:484x220/rscb23/ng:webp/ngcb23 "Install Guacamole Remote Desktop on Debian 11 / Debian 10 7")

You can also use other Authentication Methods as shown here:

- [Guacamole Integration with Active Directory, OTP, and Duo 2FA](https://computingforgeeks.com/guacamole-integration-with-active-directory-otp-duo-2fa/)

To configure SSL check out our article:

- [Configure Nginx Proxy For Guacamole With Let’s Encrypt SSL](https://computingforgeeks.com/configure-nginx-proxy-for-guacamole-with-lets-encrypt-ssl/)

# Guacamole Integration with Active Directory, OTP, and Duo 2FA

1. Configure Active Directory/LDAP authentication on Guacamole

Guacamole supports Active Directory/LDAP authentication using a plugin available on the main project site. This makes it possible to authenticate using users stored in AD/LDAP. This makes it easier for existing users to log in to Guacamole.

Before you proceed, you need to have AD/LDAP installed and configured. This can be achieved using any of the below guides:

- **Active Directory**
    - [Setup Active Directory Domain Services on Windows Server](https://computingforgeeks.com/?s=Active+Directory+Domain+Services+in+Windows+Server)
- **LDAP**
    - [Install and Configure OpenLDAP Server](https://computingforgeeks.com/?s=Install+LDAP)

Once set up, proceed and download the AD/LDAP extension from the [Official page](http://guacamole.apache.org/releases/). You can also pull the extension using the commands:

Export the version:

```sh
export VER=1.5.3
```

Download des archives:

```sh
wget https://dlcdn.apache.org/guacamole/$VER/binary/guacamole-auth-ldap-$VER.tar.gz
```

Extraieren des archives:

```sh
tar -xzf guacamole-auth-ldap-$VER.tar.gz
```

Create the extensions directory for Guacamole if it doesn’t exist and copy the **_.jar_** file into it:

```sh
sudo cp ~/guacamole-auth-ldap-$VER/guacamole-auth-ldap-$VER.jar /etc/guacamole/extensions
```

Anpassen der Guacamole properties

```sh
sudo vim /etc/guacamole/guacamole.properties
```

Here, provide your AD/LDAP properties as shown:

```
##LDAP SETTINGS
ldap-hostname:          WIN-PLMH2KF0VT2.computingforgeeks.com
ldap-port:              389
ldap-encryption-method: none

ldap-user-base-dn:       DC=computingforgeeks,DC=com
ldap-username-attribute: sAMAccountName

#DN (Distinguished Name) of the user to bind as when authenticating users that are attempting to log in
ldap-search-bind-dn:       CN=Administrator,CN=Users,DC=computingforgeeks,DC=com
#The password to provide to the LDAP server when binding as ldap-search-bind-dn to authenticate other users
ldap-search-bind-password: Passw0rd!

#ldap-user-search-filter: (&(objectClass=user)(memberOf:1.2.840.113556.1.4.1941:GuacamoleUsers,CN=Users,DC=computingforgeeks,DC=com))
```

In the above command, I have used a user Administrator with their password for my bind user to the AD. You can use any other user in your system for authentication.

Save the changes and restart Tomcat:

```
sudo systemctl restart tomcat*
```

Now you can access the Guacamole site using any user on your AD/LDAP. For example on my AD, I have the following user:

![Guacamole Integration with Active Directory OTP and Duo 2FA 2](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-2.png?ezimgfmt=rs:484x582/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 1")

On the Guacamole site, let’s test authentication using the user:

![Guacamole Integration with Active Directory OTP and Duo 2FA](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA.png?ezimgfmt=rs:484x535/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 2")

If all goes well, you should access Guacamole as shown:

![Guacamole Integration with Active Directory OTP and Duo 2FA 1](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-1-1024x808.png?ezimgfmt=rs:484x382/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 3")

## 2. Configure OTP Authentication on Guacamole

Guacamole supports OTP, a _One-Time Password_ as a second authentication method on top of any existing method. Before we use OTP, we need to enable the database authentication extension.

### a. Enable Database authentication for Guacamole

For that reason, install MariaDB/MySQL using any of the below guides:

- [How to Install MariaDB](https://computingforgeeks.com/?s=Install+MariaDB)
- [How to Install MySQL](https://computingforgeeks.com/?s=Install+MySQL)

Once installed, access the shell:

```
mysql -u root -p
```

Create a database and user:

```
create database guacd;
create user guacd_admin@localhost identified by 'Passw0rd!';
```

Next, grant the required permissions to the user:

```
grant SELECT,UPDATE,INSERT,DELETE on guacd.* to guacd_admin@localhost;
```

Save the changes and exit:

```
flush privileges;
quit
```

Next, you need to install the [JDBC auth](https://guacamole.apache.org/releases/) database extension for Guacamole:

```
export VER=1.5.3
wget https://downloads.apache.org/guacamole/$VER/binary/guacamole-auth-jdbc-$VER.tar.gz
```

Extract the archive and copy it to the extensions directory:

```
tar -xf guacamole-auth-jdbc-$VER.tar.gz
sudo mv guacamole-auth-jdbc-$VER/mysql/guacamole-auth-jdbc-mysql-$VER.jar /etc/guacamole/extensions/
```

We now need to import the database schemas:

```
cd guacamole-auth-jdbc-*/mysql/schema
cat *.sql | sudo mysql -u root -p guacd
```

Provide your MySQL root password to import the schemas. We also need to install the [MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/)

```
VER=8.1.0
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-$VER.tar.gz
```

Extract the file and copy it to the /etc/guacamole/lib/ directory:

```
tar -xf mysql-connector-j-*.tar.gz
sudo cp mysql-connector-j-$VER/mysql-connector-j-$VER.jar /etc/guacamole/lib/
```

Next, enable authentication by making the below definitions in the **_guacamole.properties_** file:

```
sudo vim /etc/guacamole/guacamole.properties
```

Add and modify the below lines. Ensure the correct database credentials are provided.

```
###MySQL properties
mysql-hostname: localhost
mysql-database: guacd
mysql-username: guacd_admin
mysql-password: Passw0rd!
```

Save the file and restart Tomcat:

```
sudo systemctl restart tomcat* guacd
```

Access Guacamole using the default creds:

```
Username: guacadmin
Password: guacadmin
```

Sample:

![Guacamole Integration with Active Directory OTP and Duo 2FA 3](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-3.png?ezimgfmt=rs:484x457/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 4")

Once authenticated, create a new user and make them the admin user.

![Guacamole Integration with Active Directory OTP and Duo 2FA 4](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-4-1024x754.png?ezimgfmt=rs:484x356/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 5")

The user will be available as shown:

![Guacamole Integration with Active Directory OTP and Duo 2FA 5](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-5.png?ezimgfmt=rs:484x405/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 6")

### b. Configure TOTP for Guacamole

Now we can proceed and configure OTP for Guacamole. Begin by downloading the extension.

Export the version:

```
export VER=1.5.3
```

Now download the TOTP extension:

```
wget https://downloads.apache.org/guacamole/$VER/binary/guacamole-auth-totp-$VER.tar.gz
```

Extract the extension and copy it to the /etc/guacamole/extensions/ directory

```
tar -zxf guacamole-auth-totp-$VER.tar.gz
sudo cp guacamole-auth-totp-$VER/guacamole-auth-totp-$VER.jar /etc/guacamole/extensions/
```

Now you need to configure TOTP for Guacamole

```
sudo vim /etc/guacamole/guacamole.properties
```

In the file, add the below lines:

```
##OTP SETTINGS
##entity issuing user accounts, default "Apache Guacamole"
#totp-issuer:  Apache Guacamole

#The number of digits which should be included in each generated TOTP code.
totp-digits: 8

#The duration that each generated code should remain valid, in seconds
totp-period: 30

#The hash algorithm that should be used to generate TOTP codes. Legal values are “sha1”, “sha256”, and “sha512”. By default, “sha1” is used.
#totp-mode: sha1
```

Once the settings have been made, save the file and restart Tomcat and GUACD.

```
sudo systemctl restart tomcat* guacd
```

Now validate if OTP is working, by login in using the user created.

![Guacamole Integration with Active Directory OTP and Duo 2FA 6](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-6.png?ezimgfmt=rs:484x461/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 7")

Once authenticated, you will land on this page, use a mobile phone to scan this and obtain the code.

![Guacamole Integration with Active Directory OTP and Duo 2FA 8](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-8.png?ezimgfmt=rs:484x575/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 8")

If all is okay and the code has been provided, you should be authenticated as shown:

![Guacamole Integration with Active Directory OTP and Duo 2FA 9](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-9.png?ezimgfmt=rs:484x265/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 9")

## 3. Configure Duo 2FA Authentication on Guacamole

You can also use Duo as a second-layer authentication for Guacamole. This extension allows users to verify themselves using the DUO service before being allowed.

To use it, you need to download the extension. First, export the version:

```
export VER=1.5.3
```

Download the extension with the command:

```
wget https://downloads.apache.org/guacamole/$VER/binary/guacamole-auth-duo-$VER.tar.gz 
```

Extract the archive and copy it to the extensions directory:

```
tar -zxf guacamole-auth-duo-$VER.tar.gz
sudo cp guacamole-auth-duo-$VER/guacamole-auth-duo-$VER.jar /etc/guacamole/extensions/
```

The next thing we need to do is log in to your [Duo account](https://admin.duosecurity.com/) and add a new **Web SDK** application in the Applications tab.

![Guacamole Integration with Active Directory OTP and Duo 2FA 10](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-10-1024x498.png?ezimgfmt=rs:484x235/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 10")

The app will be added as shown:

![Guacamole Integration with Active Directory OTP and Duo 2FA 11](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-11-1024x498.png?ezimgfmt=rs:484x235/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 11")

Scroll down and rename the application with a friendly name:

![Guacamole Integration with Active Directory OTP and Duo 2FA 12](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-12-1024x602.png?ezimgfmt=rs:484x285/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 12")

Save the changes then proceed and update Guacamole to use the DUO 2FA.

```
sudo vim /etc/guacamole/guacamole.properties
```

In the file, you need to add several variables obtained from the **_Web SDK_** app created:

```
##DUO 2FA
#The hostname of the Duo API endpoint to be used to verify user identities
duo-api-hostname: api-XXXXXXXX.duosecurity.com

#The integration key/client ID provided for Guacamole by Duo.
duo-integration-key: must-be-EXACTLY-20-characters

#The secret key provided for Guacamole by Duo
duo-secret-key: must-be-EXACTLY-20-characters

#An arbitrary, random key which you manually generated for Guacamole
duo-application-key: a-random-key-here
```

Make sure the details are provided correctly, to obtain the **random key**, you can use:

```
$ pwgen 40 1
ienohdiePhuj1veiqueiVie3aila4pahpiesegho
```

Once the changes have been saved, restart Tomcat and GUACD:

```
sudo systemctl restart tomcat* guacd
```

Now access Guacamole and see if Duo 2FA is working as desired:

![Guacamole Integration with Active Directory OTP and Duo 2FA 13](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-13.png?ezimgfmt=rs:484x486/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 13")

Begin by setting up DUO for the first time:

![Guacamole Integration with Active Directory OTP and Duo 2FA 14](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-14.png?ezimgfmt=rs:484x400/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 14")

For my case, I will add a Tablet or phone with the DUO app installed.

![Guacamole Integration with Active Directory OTP and Duo 2FA 15](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-15.png?ezimgfmt=rs:484x400/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 15")

You will be required to scan and add the device, once added, you can proceed to log in.

![Guacamole Integration with Active Directory OTP and Duo 2FA 16](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-16.png?ezimgfmt=rs:484x400/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 16")

Now you can send a push to your added device:

![Guacamole Integration with Active Directory OTP and Duo 2FA 17](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-17.png?ezimgfmt=rs:484x400/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 17")

If all is okay, you should be authenticated.

![Guacamole Integration with Active Directory OTP and Duo 2FA 18](https://computingforgeeks.com/wp-content/uploads/2023/06/Guacamole-Integration-with-Active-Directory-OTP-and-Duo-2FA-18.png?ezimgfmt=rs:484x392/rscb23/ng:webp/ngcb23 "Guacamole Integration with Active Directory, OTP, and Duo 2FA 18")

## Verdict

Today, we have learned the Guacamole integration with Active Directory, OTP, and Duo 2FA. This can be vital if you are running Guacamole in a production environment with many users and security required. I hope this was significant to you.