---
tags:
  - cluster
  - mariadb
  - datenbanken
---
# maxscale Einrichtung

```
db-node1 : 172.160.18.12
db-node2 : 172.160.18.11
db-node3 : 172.160.18.10 
MaxScale-node4 : 172.160.18.3
```

**NTP einrichten**

```sh
sudo apt install chrony -y
sudo systemctl enable chronyd
sudo systemctl start chronyd
sudo systemctl status chronyd
timedatectl set-timezone Asia/Jakarta
timedatectl
```

**Mariadb einrichten**

`apt install mariadb-server mariadb-client mariadb-backup galera-4 -y`

**DB starten**

```sh
systemctl enable mariadb
systemctl start mariadb
systemctl status mariadb
```

Auf allen Nodes die secure installation durchführen
 
```
Switch to unix_socket authentication [Y/n] n  
Change the root password? [Y/n] y  
Remove anonymous users? [Y/n] y  
Disallow root login remotely? [Y/n] y  
Remove test database and access to it? [Y/n] y  
Reload privilege tables now? [Y/n] y
```

Galera Cluster /etc/hosts

```
172.160.18.12 node1  
172.160.18.11 node2  
172.160.18.10 node3
```

etc/mysql/mariadb.conf.d/60-galera.cnf Konfiguration einrichten

```sh
[galera]  
pid-file        = /var/run/mysqld/mysqld.pid  
socket          = /var/run/mysqld/mysqld.sock  
datadir         = /var/lib/mysql  
  
binlog_format=ROW  
default-storage-engine=innodb  
innodb_autoinc_lock_mode=2  
bind-address=0.0.0.0  
  
# Galera Provider Configuration  
wsrep_on=ON  
wsrep_provider=/usr/lib/galera/libgalera_smm.so  
  
# Galera Cluster Configuration  
wsrep_cluster_name="mariadb_cluster" # NAMA GROUP CLUSTER  
wsrep_cluster_address="gcomm://172.160.18.12,172.160.18.11,172.160.18.10 " # IP NODE  
  
# Galera Synchronization Configuration  
wsrep_sst_method=rsync  
  
# Galera Node Configuration  
wsrep_node_address="172.160.18.12" # IP SERVER DB-NODE1  
wsrep_node_name="db-node1" # Anpassen für NODE1,NODE2
```

**Anpassung für den Server /etc/mysql/mariadb.conf.d/50-server.cnf**

```ini
[server]  
default-time-zone=+07:00  
  
[mysqld]  
pid-file                = /run/mysqld/mysqld.pid  
basedir                 = /usr  
bind-address            = 0.0.0.0  
max_connections         = 3000  
general_log_file        = /var/log/mysql/mysql.log  
general_log             = 1
```

wsrep_node_address
wsrep_node_name

Initialize the Galera Cluster on the first node

```sh
galera_new_cluster  
mysql -u root -p -e "show status like 'wsrep_cluster_size'"
```

Zustand von node1 Prüfen

```sql
mysql -u root -p -e "show status like 'wsrep_cluster_size'"
```

Wen alles läuft auf allen nodes wiederholen

Password setzen

```sql
sudo mysql -u root -p  
  
MariaDB [(none)]> CREATE USER 'maxscale_user'@'%' IDENTIFIED BY 'maxscale_password';  
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'maxscale_user'@'%';  
MariaDB [(none)]> CREATE USER 'admin'@'%' IDENTIFIED BY 'foo123';  
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%';  
MariaDB [(none)]> FLUSH PRIVILEGES;
```

Testen das die DB verteilt wird

```sql
CREATE DATABASE db_testing;  
SHOW DATABASES;
```

Einrichten von Maxscale Loadbalancing

```sh
sudo apt install curl software-properties-common -y
sudo curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | sudo bash -s -- --mariadb-server-version=10.5 --mariadb-maxscale-version=24.02
```

> –-mariadb-maxscale-version=24.02 Richtige version angeben

**install service maxscale**

`sudo apt install maxscale mariadb-client  -y`

*Die /etc/maxscale.cnf anpassen*

```ini
[maxscale]  
threads=auto  
admin_host=0.0.0.0  
admin_port=8989  
admin_secure_gui=false  
  
  
[Galera-Monitor]  
type=monitor  
module=galeramon  
servers=node1,node2,node3  
user=maxscale_user  
password=maxscale_password  
monitor_interval=2s  
disable_master_failback=true  
available_when_donor=true  
  
[node1]  
type=server  
address=172.160.18.12  
port=3306  
protocol=MariaDBBackend  
  
[node2]  
type=server  
address=172.160.18.11  
port=3306  
protocol=MariaDBBackend  
  
[node3]  
type=server  
address=172.160.18.10  
port=3306  
protocol=MariaDBBackend  
  
[Galera-Service]  
type=service  
router=readconnroute  
router_options=synced  
servers=node1,node2,node3  
user=maxscale_user  
password=maxscale_password  
  
[Galera-Listener]  
type=listener  
service=Galera-Service  
protocol=MariaDBClient  
address=0.0.0.0  
port=3306
```

**maxscale config Prüfen**

`maxscale -c -U maxscale`

**maxscale systemctl einrichten**

```
systemctl enable maxscale   
systemctl start maxscale  
systemctl status maxscale
```

**Usereinrichten für maxscale**

```sh
maxctrl create user “noc” “foo123” --type=admin
```

