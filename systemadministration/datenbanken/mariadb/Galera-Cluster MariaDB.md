---
tags:
  - mariadb
  - datenbanken
---
# Einrichten einer MariaDB Galera Cluster

## Schrit 1: MariaDB Repositories

Einricten eines mariadb Repositories _/etc/yum.repos.d/mariadb.repo_ 

```sh
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

## Schrit 2 Setzen des SELinux permissive mode

Für die Datenbank sollte SELinux deaktiviert werden:

`sudo setenforce0`

## Schrit 3 Installation MariaDB Galera Cluster 10

socat  benutzen mit:

```sh
sudo yum install socat
```

und Galera samt MariaDB Installieren

```sh
sudo yum install MariaDB-Galera-server MariaDB-client rsync galera
```

## Schrit 4 Einrichten der Sicherheits vorkehrungen für MariaDB

Den service starten:

`sudo systemctl start mariadb`

Sichere Konfiguration von der MariaDB

`mysql_secure_installation`

## Schrit 5 Benutzer für MariaDB Galera Cluster einrichten

```sh
mysql -u root -p

mysql> DELETE FROM mysql.user WHERE user='';
mysql> GRANT ALL ON *.* TO 'root'@'%' IDENTIFIED BY 'dbpass';
mysql> GRANT USAGE ON *.* to sst_user@'%' IDENTIFIED BY 'dbpass';
mysql> GRANT ALL PRIVILEGES on *.* to sst_user@'%';
mysql> FLUSH PRIVILEGES;
mysql> quit
```

## Schrit 6 Erstellen einer MariaDB Galera Cluster Konfiguration

Den service Stopen:

`sudo systemctl stop mariadb`

Follgende Konfiguration erfolgt auf allen nodes:

```sh
sudo cat >> /etc/my.cnf.d/server.cnf << EOF
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
innodb_locks_unsafe_for_binlog=1
query_cache_size=0
query_cache_type=0
bind-address=0.0.0.0
datadir=/var/lib/mysql
innodb_log_file_size=100M
innodb_file_per_table
innodb_flush_log_at_trx_commit=2
wsrep_provider=/usr/lib64/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://1.1.1.1,1.1.1.2,1.1.1.3"
wsrep_cluster_name='galera_cluster'
wsrep_node_address='1.1.1.1'
wsrep_node_name='db1'
wsrep_sst_method=rsync
wsrep_sst_auth=sst_user:dbpass
EOF
```

Hier muss dann aber noch der _wsrep_node_name_ , _wsrep_sst_auth_ , _wsrep_node_address_ und _wsrep_cluster_address_ Parameter für die Nodes angepasst werden.

Wenn alle Nodes Eingerichetet sind, kann man den Cluster starten

```sh
galera_new_cluster
```

Status überprüfen

`ps -f -u mysql | more`

und mit der DB

```sh
MariaDB [(none)]> show global status like 'wsrep_cluster_size';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+
1 row in set (0.00 sec)
```

**Maridb servie starten**

`systemctl start mariadb.service`

**Cluster größe Prüfen**

`mysql -u root -p -e "show status like 'wsrep%'"`

### Replikation Überprüfen

Testdatenbank _clustertest_ anlegen

```sh
mysql -u root -p -e 'CREATE DATABASE clustertest;'
mysql -u root -p -e 'CREATE TABLE clustertest.mycluster ( id INT NOT NULL AUTO_INCREMENT, name VARCHAR(50), ipaddress VARCHAR(20), PRIMARY KEY(id));'
mysql -u root -p -e 'INSERT INTO clustertest.mycluster (name, ipaddress) VALUES ("db1", "1.1.1.1");'
mysql -u root -p -e 'SELECT * FROM clustertest.mycluster;'

```

Testen ob alle Nodes die Datenbank übernaommen haben

```sh
mysql -u root -p -e 'SELECT * FROM clustertest.mycluster;'
```

###Quellen

* [getting-started-with-mariadb-galera-and-mariadb-maxscale-on-centos](https://mariadb.com/resources/blog/getting-started-with-mariadb-galera-and-mariadb-maxscale-on-centos/)
