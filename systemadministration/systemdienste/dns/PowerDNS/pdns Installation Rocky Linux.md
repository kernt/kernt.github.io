# Install und Konfiguration MariaDB unter Rocky Linux 9|8

PowerDNS unterstützt diverse database backends wie MySQL, PostgreSQL, Oracle e.t.c. In diesem beispiel werden wir MariaDB als backend storage benutzen.

**Installation von MariaDB on Rocky Linux 9|8**

```sh
sudo dnf install mariadb-server mariadb
```

**Start and enable the service**

```sh
sudo systemctl enable --now mariadb
```

**Insdanz wie folgt härten**

```sh
$ sudo mysql_secure_installation
....
Enter current password for root (enter for none): Just press Enter
......
Switch to unix_socket authentication [Y/n] Y
.....
Change the root password? [Y/n] Y
New password:  New-root-password
Re-enter new password: Re-enter New-root-password
....
Remove anonymous users? [Y/n] Y
....
Disallow root login remotely? [Y/n] Y
.....
Remove test database and access to it? [Y/n] Y
......
Reload privilege tables now? [Y/n] Y
...
Thanks for using MariaDB!
```

**Login in die shell**

```sh
sudo mysql -u root -p 
```

**Anlegen einer Datenbank für PowerDNS**

```sh
CREATE DATABASE powerdns;
GRANT ALL ON powerdns.* TO 'powerdns_user'@'%' IDENTIFIED BY 'Strongpassword';
FLUSH PRIVILEGES;
```

Zur erinnerung benutze ein password ohne special characters, weil es zu einem Fehler führen kann “**Access denied for user ‘powerdns_user’@’localhost’ (using password: YES)**“ 

`Anlegen der PowerDNS tablen`

```sql
use powerdns;
```

**Datenbanken tablen anlegen**

```sql
CREATE TABLE domains (
   id                    INT AUTO_INCREMENT,
   name                  VARCHAR(255) NOT NULL,
   master                VARCHAR(128) DEFAULT NULL,
   last_check            INT DEFAULT NULL,
   type                  VARCHAR(6) NOT NULL,
   notified_serial       INT DEFAULT NULL,
   account               VARCHAR(40) DEFAULT NULL,
   PRIMARY KEY (id)
 ) Engine=InnoDB;
 
 CREATE UNIQUE INDEX name_index ON domains(name);
 
 
 CREATE TABLE records (
   id                    BIGINT AUTO_INCREMENT,
   domain_id             INT DEFAULT NULL,
   name                  VARCHAR(255) DEFAULT NULL,
   type                  VARCHAR(10) DEFAULT NULL,
   content               VARCHAR(64000) DEFAULT NULL,
  ttl                   INT DEFAULT NULL,
   prio                  INT DEFAULT NULL,
   change_date           INT DEFAULT NULL,
   disabled              TINYINT(1) DEFAULT 0,
   ordername             VARCHAR(255) BINARY DEFAULT NULL,
   auth                  TINYINT(1) DEFAULT 1,
   PRIMARY KEY (id)
 ) Engine=InnoDB;
 
 CREATE INDEX nametype_index ON records(name,type);
 CREATE INDEX domain_id ON records(domain_id);
 CREATE INDEX recordorder ON records (domain_id, ordername);
 
 
 CREATE TABLE supermasters (
   ip                    VARCHAR(64) NOT NULL,
   nameserver            VARCHAR(255) NOT NULL,
   account               VARCHAR(40) NOT NULL,
   PRIMARY KEY (ip, nameserver)
 ) Engine=InnoDB;
 
 
 CREATE TABLE comments (
   id                    INT AUTO_INCREMENT,
   domain_id             INT NOT NULL,
   name                  VARCHAR(255) NOT NULL,
   type                  VARCHAR(10) NOT NULL,
   modified_at           INT NOT NULL,
   account               VARCHAR(40) NOT NULL,
   comment               VARCHAR(64000) NOT NULL,
   PRIMARY KEY (id)
 ) Engine=InnoDB;
CREATE INDEX comments_domain_id_idx ON comments (domain_id);
 CREATE INDEX comments_name_type_idx ON comments (name, type);
 CREATE INDEX comments_order_idx ON comments (domain_id, modified_at);
 
 
 CREATE TABLE domainmetadata (
   id                    INT AUTO_INCREMENT,
   domain_id             INT NOT NULL,
   kind                  VARCHAR(32),
   content               TEXT,
   PRIMARY KEY (id)
 ) Engine=InnoDB;
 
 CREATE INDEX domainmetadata_idx ON domainmetadata (domain_id, kind);
 
 
 CREATE TABLE cryptokeys (
 id                    INT AUTO_INCREMENT,
   domain_id             INT NOT NULL,
   flags                 INT NOT NULL,
   active                BOOL,
   content               TEXT,
   PRIMARY KEY(id)
 ) Engine=InnoDB;
 
 CREATE INDEX domainidindex ON cryptokeys(domain_id);
 
 
 CREATE TABLE tsigkeys (
   id                    INT AUTO_INCREMENT,
   name                  VARCHAR(255),
   algorithm             VARCHAR(50),
   secret                VARCHAR(255),
   PRIMARY KEY (id)
 ) Engine=InnoDB;
 
 CREATE UNIQUE INDEX namealgoindex ON tsigkeys(name, algorithm);
```

Verify if the tables exist:

```sql
MariaDB [powerdns]> show tables; 
+--------------------+
| Tables_in_powerdns |
+--------------------+
| comments           |
| cryptokeys         |
| domainmetadata     |
| domains            |
| records            |
| supermasters       |
| tsigkeys           |
+--------------------+
7 rows in set (0.000 sec)

MariaDB [powerdns]> quit;
```

## Install PowerDNS unter Rocky Linux 9|8

UNter Rocky Linux 8, muss der service  _systemd-resolved_ deaktiviert werden weil dieser auch auf dem port **53** lauscht.

```sh
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

**Löschen vom symbolic link**

```sh
$ ls -lh /etc/resolv.conf 
-rw-r--r--. 1 root root 55 Aug 24 06:46 /etc/resolv.conf
$ sudo unlink /etc/resolv.conf
```

**Erstellen eines neuen symbolic link**

```sh
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

 **`dnsmasq` muss auch gestoppt und disabled werden** 

```sh
sudo systemctl disable dnsmasq
sudo systemctl stop dnsmasq
```

**Neustart des network managers**

```sh
sudo service NetworkManager restart
```

**Kill  _dnsmasq_ services**

```sh
sudo killall dnsmasq
```

**Seit PowerDNS in den EPEL repositories aufgenommen wurde kann man PDNS mit dnf installieren**

```sh
sudo dnf -y install pdns pdns-backend-mysql bind-utils
```

## Konfiguration der PowerDNS Database unter Rocky Linux 9|8

SInce we already have a database created on MariaDB, we will simply edit configuration file to allow PowerDNS to use the database:

```sh
sudo vim  /etc/pdns/pdns.conf
```

Make the below changes to the file, ensure launch=bind is commented out.

```sh
# launch=bind
launch=gmysql
# gmysql parameters
gmysql-host=127.0.0.1
gmysql-port=3306
gmysql-dbname=powerdns
gmysql-user=powerdns_user
gmysql-password=Strongpassword
```

Save the changes, and verify the connection to the database:

```sh
sudo systemctl stop pdns.service
sudo pdns_server --daemon=no --guardian=no --loglevel=9
```

Start and enable PowerDNS:

```sh
sudo killall -9 pdns_server
sudo systemctl start pdns
sudo systemctl enable pdns
```

Verify if the service is running:

```sh
$ sudo ss -alnp4 | grep pdns
udp   UNCONN 0      0            0.0.0.0:53         0.0.0.0:*    users:(("pdns_server",pid=33244,fd=5))
tcp   LISTEN 0      128          0.0.0.0:53         0.0.0.0:*    users:(("pdns_server",pid=33244,fd=7))
```

Allow the DNS service through the firewall:

```sh
sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --reload
```

Also verify if PowerDNS is responding to requests:

```sh
$ dig @127.0.0.1

; <<>> DiG 9.16.23-RH <<>> @127.0.0.1
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 18968
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;.				IN	NS

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Aug 24 13:15:24 CEST 2022
;; MSG SIZE  rcvd: 28
```

## Install PowerDNS Admin unter Rocky Linux 9|8

Now we can install PowerDNS Admin, a web admin interface to easily manage the PowerDNS server. Bu now, we will install the required build tools. Add the REMI repo and enable PowerTools

```sh
##On Rocky Linux 8
sudo dnf -y install http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo dnf config-manager --set-enabled powertools

##On Rocky Linux 9
sudo dnf -y install http://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo dnf config-manager --set-enabled crb
```

Install the Python development and other packages.

```
sudo yum install python3-devel gcc python3-pip git mariadb-connector-c-devel xmlsec1 openldap-devel xmlsec1-openssl-devel postgresql-devel libxslt-devel libffi-devel cyrus-sasl-devel libtool-ltdl-devel
```

Uisng PIP, install Flask and virtualenv

```sh
sudo pip3 install virtualenv flask
```

Also install Node.js on Rocky Linux 9|8

```sh
curl -sL https://rpm.nodesource.com/setup_18.x | sudo -E bash -
sudo yum install -y nodejs
```

Also install the Yarn package:

```sh
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
sudo yum install yarn
```

Now clone the PowerDNS admin source code to _/var/www/html/pdns_

```sh
sudo su -
git clone https://github.com/ngoduykhanh/PowerDNS-Admin.git /var/www/html/pdns
```

Switch to the directory and create the virtual environment:

```sh
cd /var/www/html/pdns/
virtualenv -p python3 flask
```

Active the virtual environment and install the required packages.

```sh
source ./flask/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

Once complete, deactivate the environment:

```
deactivate
```

Edit the below file to allow PowerDNS admin to connect to the database;

```sh
vim /var/www/html/pdns/powerdnsadmin/default_config.py
```

Make the below adjustments

```sh
### DATABASE CONFIG
SQLA_DB_USER = 'powerdns_user'
SQLA_DB_PASSWORD = 'Strongpassword'
SQLA_DB_HOST = '127.0.0.1'
SQLA_DB_NAME = 'powerdns'
SQLALCHEMY_TRACK_MODIFICATIONS = True
```

**Erstellen eines database schemas**

```sh
cd /var/www/html/pdns/
source ./flask/bin/activate
export FLASK_APP=powerdnsadmin/__init__.py
flask db upgrade
```

**Einfacher Output**

```sh
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 787bdba9e147, Init DB
INFO  [alembic.runtime.migration] Running upgrade 787bdba9e147 -> 59729e468045, Add view column to setting table
INFO  [alembic.runtime.migration] Running upgrade 59729e468045 -> 1274ed462010, Change setting.value data type
INFO  [alembic.runtime.migration] Running upgrade 1274ed462010 -> 4a666113c7bb, Adding Operator Role
INFO  [alembic.runtime.migration] Running upgrade 4a666113c7bb -> 31a4ed468b18, Remove all setting in the DB
INFO  [alembic.runtime.migration] Running upgrade 31a4ed468b18 -> 654298797277, Upgrade DB Schema
INFO  [alembic.runtime.migration] Running upgrade 654298797277 -> 0fb6d23a4863, Remove user avatar
INFO  [alembic.runtime.migration] Running upgrade 0fb6d23a4863 -> 856bb94b7040, Add comment column in domain template record table
INFO  [alembic.runtime.migration] Running upgrade 856bb94b7040 -> b0fea72a3f20, Update domain serial columns type
INFO  [alembic.runtime.migration] Running upgrade b0fea72a3f20 -> 3f76448bb6de, Add user.confirmed column
INFO  [alembic.runtime.migration] Running upgrade 3f76448bb6de -> 0d3d93f1c2e0, Add domain_id to history table
INFO  [alembic.runtime.migration] Running upgrade 0d3d93f1c2e0 -> 0967658d9c0d, add apikey account mapping table
INFO  [alembic.runtime.migration] Running upgrade 0967658d9c0d -> fbc7cf864b24, update history detail quotes
INFO  [alembic.runtime.migration] Running upgrade fbc7cf864b24 -> 6ea7dc05f496, Fix typo in history detail
```

**Now generate the asset files with Yarn**

```
yarn install --pure-lockfile
flask assets build
```

**Deactivate the environment**

```
deactivate
```

## Enable PowerDNS API access on Rocky Linux 9|8

**To enable the PowerDNS API access, we need to edit the below file**

```sh
sudo vim /etc/pdns/pdns.conf
```

**Make the below changes to the file**

```sh
# api   Enable/disable the REST API (including HTTP listener)
#
# api=no
api=yes

#################################
# api-key       Static pre-shared authentication key for access to the REST API
#
# api-key=
api-key=3ce1af6c-981d-4190-a559-1e691d89b90e #You can generate one from https://codepen.io/corenominal/pen/rxOmMJ
```

**Save the file and restart the service**

```sh
sudo systemctl restart pdns
```

## Install and Configure Nginx for Powerdns

To be able to load the PowerDNS web interface, we need to install and configure Nginx.

```sh
sudo yum install nginx
```

Once installed, create a virtualhost file for PowerDNS admin.

```sh
vim /etc/nginx/conf.d/powerdns-admin.conf
```

ADd the below lines and make changes where required:

```c
server {
  listen	*:80;
  server_name               pdnsadmin.computingforgeeks.com;

  index                     index.html index.htm index.php;
  root                      /var/www/html/pdns;
  access_log                /var/log/nginx/pdnsadmin_access.log combined;
  error_log                 /var/log/nginx/pdnsadmin_error.log;

  client_max_body_size              10m;
  client_body_buffer_size           128k;
  proxy_redirect                    off;
  proxy_connect_timeout             90;
  proxy_send_timeout                90;
  proxy_read_timeout                90;
  proxy_buffers                     32 4k;
  proxy_buffer_size                 8k;
  proxy_set_header                  Host $host;
  proxy_set_header                  X-Real-IP $remote_addr;
  proxy_set_header                  X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_headers_hash_bucket_size    64;

  location ~ ^/static/  {
    include  /etc/nginx/mime.types;
    root /var/www/html/pdns/powerdnsadmin;

    location ~*  \.(jpg|jpeg|png|gif)$ {
      expires 365d;
    }

    location ~* ^.+.(css|js)$ {
      expires 7d;
    }
  }

  location / {
    proxy_pass            http://unix:/run/pdnsadmin/socket;
    proxy_read_timeout    120;
    proxy_connect_timeout 120;
    proxy_redirect        off;
  }

}
```

Set the correct ownership for the path:

```sh
chown -R nginx: /var/www/html/pdns
```

Restart Nginx:

```sh
systemctl restart nginx
```

Allow the service through the firewall:

```sh
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```

## Create a Service file for PowerDNS Admin unter Rocky Linux 9|8

To be able to manage the PowerDNS Admin service, we need to create a systemd service file;

```
vim /etc/systemd/system/pdnsadmin.service
```

The file will contain the below lines:

```
[Unit]
Description=PowerDNS-Admin
Requires=pdnsadmin.socket
After=network.target

[Service]
PIDFile=/run/pdnsadmin/pid
User=pdns
Group=pdns
WorkingDirectory=/var/www/html/pdns
ExecStart=/var/www/html/pdns/flask/bin/gunicorn --pid /run/pdnsadmin/pid --bind unix:/run/pdnsadmin/socket 'powerdnsadmin:create_app()'
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Create a socket file for PowerDNS admin;

```
sudo vim /etc/systemd/system/pdnsadmin.socket
```

Add the below lines to the file;

```
[Unit]
Description=PowerDNS-Admin socket

[Socket]
ListenStream=/run/pdnsadmin/socket

[Install]
WantedBy=sockets.target
```

Finally, create an environment file

```
mkdir /run/pdnsadmin/
echo "d /run/pdnsadmin 0755 pdns pdns -" >> /etc/tmpfiles.d/pdnsadmin.conf
```

Set the correct ownership of the files:

```
chown -R pdns: /run/pdnsadmin/
chown -R pdns: /var/www/html/pdns/powerdnsadmin/
```

Reload the system deamon;

```
systemctl daemon-reload
```

Start and enable PowerDNS Admin on Rocky Linux 9|8

```
systemctl enable --now pdnsadmin.service pdnsadmin.socket
```

Verify if the service is running:

```
$ systemctl status pdnsadmin.service pdnsadmin.socket
 pdnsadmin.service - PowerDNS-Admin
     Loaded: loaded (/etc/systemd/system/pdnsadmin.service; enabled; vendor preset: disabled)
     Active: active (running) since Wed 2022-08-24 14:42:36 CEST; 9s ago
TriggeredBy: ● pdnsadmin.socket
   Main PID: 92303 (gunicorn)
      Tasks: 2 (limit: 23441)
     Memory: 69.4M
        CPU: 690ms
     CGroup: /system.slice/pdnsadmin.service
             ├─92303 /var/www/html/pdns/flask/bin/python /var/www/html/pdns/flask/bin/gunicorn --pid /run/pdnsadmin/pid --bind unix:/run/pd>
             └─92304 /var/www/html/pdns/flask/bin/python /var/www/html/pdns/flask/bin/gunicorn --pid /run/pdnsadmin/pid --bind unix:/run/pd>

Aug 24 14:42:36 localhost.localdomain systemd[1]: Started PowerDNS-Admin.
Aug 24 14:42:36 localhost.localdomain gunicorn[92303]: [2022-08-24 14:42:36 +0200] [92303] [INFO] Starting gunicorn 20.0.4
Aug 24 14:42:36 localhost.localdomain gunicorn[92303]: [2022-08-24 14:42:36 +0200] [92303] [INFO] Listening at: unix:/run/pdnsadmin/socket >
Aug 24 14:42:36 localhost.localdomain gunicorn[92303]: [2022-08-24 14:42:36 +0200] [92303] [INFO] Using worker: sync
Aug 24 14:42:36 localhost.localdomain gunicorn[92304]: [2022-08-24 14:42:36 +0200] [92304] [INFO] Booting worker with pid: 92304

● pdnsadmin.socket - PowerDNS-Admin socket
     Loaded: loaded (/etc/systemd/system/pdnsadmin.socket; enabled; vendor preset: disabled)
     Active: active (running) since Wed 2022-08-24 14:42:36 CEST; 9s ago
      Until: Wed 2022-08-24 14:42:36 CEST; 9s ago
   Triggers: ● pdnsadmin.service
     Listen: /run/pdnsadmin/socket (Stream)
     CGroup: /system.slice/pdnsadmin.socket

Aug 24 14:42:36 localhost.localdomain systemd[1
```
