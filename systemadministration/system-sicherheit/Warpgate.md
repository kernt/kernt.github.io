## What is Warpgate?

**Warpgate** is an easy-to-configure service that supports smart SSH, HTTPS and MySQL. This service does not require any special client applications to work. Once deployed on a Linux host, it will accept SSH, HTTPS and MySQL connections and provide an (optional) web admin UI.

The below image illustrates a simple Warpgate set-up.

![Warpgate setup](https://computingforgeeks.com/wp-content/uploads/2022/12/Warpgate-setup.png?ezimgfmt=rs:484x258/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 1")

The amazing features and benefits associated with Warpgate are:

- It is written in 100% safe Rust.
- It has a single binary with no dependencies. This makes the installation so easy.
- It records every session for you to view (live) and replay later through a built-in admin web UI.
- It does not act as a jump host, it forwards your connections straight to the target instead.
- It supports Native 2FA and SSO (TOTP & OpenID Connect).
- It allows users to set it up in their DMZ, add user accounts and easily assign them to specific hosts and URLs within the network.

## #1. Install Warpgate on Linux

Warpgate can be installed easily on your desired Linux host easily using a single binary with no dependencies. The latest binary can be downloaded from the [GitHub Releases](https://github.com/warp-tech/warpgate/releases).

You can also download the binary from your terminal. First, export the desired version:

```
ver=v0.7.0
```

At the time of this documentation, the latest release version was at **0.7.0**. Proceed and download the exported version with the command:

```
##For x86_64-linux
wget https://github.com/warp-tech/warpgate/releases/download/$ver/warpgate-$ver-x86_64-linux

##For arm64-linux
wget https://github.com/warp-tech/warpgate/releases/download/$ver/warpgate-$ver-arm64-linux
```

Once downloaded, make the binary executable:

```
chmod +x warpgate-*
```

Now move the binary to your PATH:

```
sudo mv warpgate-* /usr/bin/warpgate
```

## #2. Configure Warpgate on Linux

Once installed, Warpgate can be configured using to work as desired. If you want to use a non-default configuration file, (other than /etc/warpgate.yaml) you need to pass it using the `--config <path>` argument. You can also use an external database other than the built-in SQLite using `-database-url mysql://…`or `--database-url postgres://...`

In this guide, we will use the interactive setup mode by running the below command:

```
sudo warpgate setup
```

Proceed as shown:

```
08:49:56  INFO Welcome to Warpgate 0.7.0
08:49:56  INFO Let's do some basic setup first.
08:49:56  INFO The new config will be written in /etc/warpgate.yaml.
08:49:56  INFO * Paths can be either absolute or relative to /etc.
? Directory to store app data (up to a few MB) in (/var/lib/warpgate) › Press Enter
? Endpoint to listen for HTTP connections on (0.0.0.0:8888) › 
08:53:35  INFO You will now choose specific protocol listeners to be enabled.
08:53:35  INFO 
08:53:35  INFO NB: Nothing will be exposed by default -
08:53:35  INFO     you'll set target hosts in the config file later.
? Accept SSH connections? (y/n) › yes
? Endpoint to listen for SSH connections on (0.0.0.0:2222) › 
? Accept MySQL connections? (y/n) › yes
? Endpoint to listen for MySQL connections on (0.0.0.0:33306) › 
? Do you want to record user sessions? (y/n) › yes
? Set a password for the Warpgate admin user ›  ********
08:54:20  INFO Generated configuration:
---
sso_providers: []
recordings:
....
08:54:20  INFO Saved into /etc/warpgate.yaml
08:54:20  INFO Using config: "/etc/warpgate.yaml"
```

At this point, you will have a configuration generated in **/etc/warpgate.yaml**. If you want to delete, or start over again, rerun the command `warpgate setup`.

Allow the set ports through the firewall:

```
#For UFW
sudo ufw allow 8888
sudo ufw allow 2222
sudo ufw allow 33306

##For Firewalld
sudo firewall-cmd --add-port=8888/tcp --permanent
sudo firewall-cmd --add-port=2222/tcp --permanent
sudo firewall-cmd --add-port=33306/tcp --permanent
sudo firewall-cmd --reload
```

Once configured, you can start Warpgate with the command

```
$ sudo warpgate run
08:58:02  INFO Warpgate version=0.7.0
08:58:02  INFO Using config: "/etc/warpgate.yaml"
08:58:02  INFO --------------------------------------------
08:58:02  INFO Warpgate is now running.
08:58:02  INFO Accepting SSH connections on 0.0.0.0:2222
08:58:02  INFO Accepting HTTP connections on https://0.0.0.0:8888
08:58:02  INFO Accepting MySQL connections on 0.0.0.0:33306
08:58:02  INFO --------------------------------------------
08:58:02  INFO Listening address=0.0.0.0:2222
08:58:02  INFO Listening address=0.0.0.0:8888
08:58:02  INFO Listening address=0.0.0.0:33306
```

If you have a config in another directory, use:

```
sudo warpgate run --config <path>
```

You can now access the web admin UI using the URL [https://IP_Address:8888/@warpgate/admin](https://IP_Address:8888/@warpgate/admin)

![SSH and MySQL Bastion Server using Warpgate](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-1024x753.png?ezimgfmt=rs:484x356/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 2")

Login using the created user and you will see the below dashboard.

![SSH and MySQL Bastion Server using Warpgate 1](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-1-1024x552.png?ezimgfmt=rs:484x261/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 3")

## #3. Create a Systemd Service for Warpgate

To avoid starting the Warpgate service manually, we will create a Systemd Service. Begin by creating the service file with the command:

```
sudo tee /etc/systemd/system/warpgate.service<<EOF
[Unit]
Description=Warpgate
After=network.target
StartLimitIntervalSec=0

[Service]
Type=notify
Restart=always
RestartSec=5
ExecStart=/usr/bin/warpgate --config /etc/warpgate.yaml run

[Install]
WantedBy=multi-user.target
EOF
```

Once the file has been created, reload the system daemon:

```
sudo systemctl daemon-reload
```

Ensure that the service is stopped and nothing is running on ports 2222 and 8888.

```
sudo killall -9 warpgate
```

On Rhel-based systems, modify SELinux:

```
sudo /sbin/restorecon -v /usr/bin/warpgate
```

Now start and enable the service:

```
sudo systemctl enable --now warpgate
```

Check the status of the service:

```
$ systemctl status warpgate
● warpgate.service - Warpgate
     Loaded: loaded (/etc/systemd/system/warpgate.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-12-11 12:05:28 EAT; 12s ago
   Main PID: 2839 (warpgate)
      Tasks: 9 (limit: 4575)
     Memory: 8.1M
     CGroup: /system.slice/warpgate.service
             └─2839 /usr/bin/warpgate --config /etc/warpgate.yaml run
```

## #4. Adding an SSH Target on Warpgate

Now to be able to SSH to other systems using Warpgate, we need to add the target. Warpgate has its own set of SSH keys which the target host must trust for the connections to be established. The keys can be viewed in the **SSH** tab on the admin UI.

![SSH and MySQL Bastion Server using Warpgate 2](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-2-1024x577.png?ezimgfmt=rs:484x273/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 4")

You can also view the keys from the command line with the command:

```
sudo warpgate client-keys
```

These keys are listed in order of preference. Copy one of these keys and paste in `~/.ssh/authorized_keys` on your client machine. If the machine does not have the file, create it _manually_ and add the keys.

```
##On the target host
vim ~/.ssh/authorized_keys
```

In the file, paste the copied keys at the end and save it.

Aside from using the SSH keys, you have an alternative **password authentication** method which is not recommended.

Now on the admin UI, add the target host by navigating to **Config** > **Targets** > **Add target**

![SSH and MySQL Bastion Server using Warpgate 3](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-3-1024x731.png?ezimgfmt=rs:484x346/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 5")

On the next page, provide the username, and IP address for the remote system.

![SSH and MySQL Bastion Server using Warpgate 4](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-4-1024x878.png?ezimgfmt=rs:484x415/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 6")

Once the changes have been made, click **Update configuration**. Once added, the system should appear on the home page as shown.

![SSH and MySQL Bastion Server using Warpgate 5](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-5-1024x712.png?ezimgfmt=rs:484x337/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 7")

Now to connect to the remote system from the Warpgate host, use the command with the below syntax:

```
ssh admin:<target-name>@<Host> -p 2222
```

The values here are:

- **Host**: the Warpgate host
- **Port**: the Warpgate SSH port (default: 2222)
- **Username**: `admin:<target-name>` in this example: **_admin:ubuntu11_**
- **Password**: your Warpgate admin password

To view the exact connection command, click on the added host.

![SSH and MySQL Bastion Server using Warpgate 6](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-6-1024x640.png?ezimgfmt=rs:484x303/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 8")

For example:

```
ssh admin:ubuntu11@192.168.205.11 -p 2222
```

Provide the Warpgate admin password for the connection to happen.

![SSH and MySQL Bastion Server using Warpgate 7](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-7-1024x728.png?ezimgfmt=rs:484x344/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 9")

## #5. Adding MySQL Target on Warpgate

Ensure that your installed database supports **TLS** before you proceed with this set up. On your MySQL server, you need to make the below configurations:

```
sudo vim /etc/mysql/mariadb.conf.d/50-server.cnf
##OR
sudo vim /etc/my.cnf.d/mariadb-server.cnf
```

In the file, modify the below lines:

```
[mysqld]
# localhost which is more compatible and is not less secure.
bind-address            = *

tls_version = TLSv1.2,TLSv1.3
```

Save the file and restart the service:

```
sudo systemctl restart mariadb
```

Alow the service through the firewall:

```
#For UFW
sudo fw allow 3306

##For Firewalld
sudo firewall-cmd --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```

Now you can create a user to use for remote connections:

```
sudo mysql -u root -p
```

Crate the user with the commands:

```
CREATE USER 'sample_user'@'%' IDENTIFIED BY 'Passw0rd';
GRANT ALL PRIVILEGES ON *.* TO 'sample_user'@'%' IDENTIFIED BY 'Passw0rd';
FLUSH PRIVILEGES;
EXIT;
```

On the desired host that you want to run the SQL queries from, ensure that you have **MySQL client** installed with:

- TLS: enabled
- Cleartext password authentication: _allowed_

Also install the additional package below for _cleartext password authentication_

```
##On Debian/Ubuntu
sudo yum install mariadb-libs

##On Rhel/Rocky Linux/CentOS/Alma Linux
sudo apt install libmariadb3
```

We have generated our certificates on the Warpgate server for MySQL connection in the below paths:

```
$ sudo vim /etc/warpgate.yaml 
---
......
mysql:
  enable: true
  listen: "0.0.0.0:33306"
  certificate: /var/lib/warpgate/tls.certificate.pem
  key: /var/lib/warpgate/tls.key.pem
...
```

For the connection form the client to happen, we need to copy these certificates to the **MySQL client**. You can do this using the SCP command:

```
##From Warpgate server
cd /var/lib/warpgate/
sudo scp tls.* username@IP_Address:~/
```

In the command, replace the **username** and **IP address** of your MySQL client. Once copied, move the certs to a preferred directory on the MySQL client.

```
##On the MySQL client
sudo mv tls-* /etc/mysql
```

Now update your MySQL client config file with the certs:

```
sudo vim /etc/mysql/my.cnf
```

In the file, add the lines:

```
[client-server]
ssl-cert=/etc/mysql/tls.certificate.pem
ssl-key=/etc/mysql/tls.key.pem
tls_version = TLSv1.2,TLSv1.3
```

Now back to the Wapgate admin UI, we will use steps almost similar to SSH. Navigate to **Config** > **Targets** > **Add target**, set the target name and type as shown.

![SSH and MySQL Bastion Server using Warpgate 8](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-8-1024x713.png?ezimgfmt=rs:484x337/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 10")

Make adjustments to the config by providing the host, user and password as shown.

![SSH and MySQL Bastion Server using Warpgate 9](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-9-1024x1006.png?ezimgfmt=rs:484x476/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 11")

Once added, **update the configuration** and it should appear on the home page as shown.

![SSH and MySQL Bastion Server using Warpgate 10](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-10-1024x816.png?ezimgfmt=rs:484x386/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 12")

Now to connect to the database using a URL, the syntax will be:

```
mysql://<username>#<target>:<password>@<warpgate host>:<warpgate mysql port>?sslMode=required
```

Where:

- **Host**: the Warpgate host
- **Port**: the Warpgate MySQL port (default: 33306)
- **Username**: `admin#<target>` or `admin:<target-name>` in this example: **admin#mariaDB**
- **Password**: your Warpgate admin password

You can obtain the required URL or connection command form the admin UI

![SSH and MySQL Bastion Server using Warpgate 11](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-11.png?ezimgfmt=rs:484x434/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 13")

Now connect to the database form the Warpgate server

```
mysql -u 'admin#mariaDB' --host '192.168.205.2' --port 33306 --ssl -p
```

Sample Output:

![SSH and MySQL Bastion Server using Warpgate 12](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-12-1024x695.png?ezimgfmt=rs:484x329/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 14")

Once connected, you can view the logs in the Admin UI under **sessions**

![SSH and MySQL Bastion Server using Warpgate 13](https://computingforgeeks.com/wp-content/uploads/2022/12/SSH-and-MySQL-Bastion-Server-using-Warpgate-13-1024x573.png?ezimgfmt=rs:484x271/rscb23/ng:webp/ngcb23 "How To Setup SSH and MySQL Bastion Server using Warpgate 15")