---
tags:
  - datenbanken
  - galera
  - cluster
---
# Installation auf Centos

```sh
sudo yum install mysql-server
```

wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.12/yum/centos8-amd64/rpms/galera-4-26.4.2.el8.x86_64.rpm

`sudo rpm -Uvh galera-4-26.4.2.el8.x86_64.rpm`

**my.cnf**

```cnf
 [mysqld]  
 binlog_format=ROW  
 default-storage-engine=innodb  
 innodb_autoinc_lock_mode=2  
 bind-address=0.0.0.0  
  
 # Galera Cluster Settings  
 wsrep_on=ON  
 wsrep_provider=/usr/lib/galera4/libgalera_smm.so  
 wsrep_cluster_address=gcomm://IP1,IP2,IP3  
 wsrep_cluster_name=my_cluster  
 wsrep_node_address=IP_current_node  
 wsrep_node_name=node1
```

Replace `IP1`, `IP2`, and `IP3` with the IP addresses of the other MySQL servers. `IP_current_node` is the IP address of your current server.

**Firewall and SELinux**

```sh
setenforce 0  # Disable SELinux  
#This is a temporary way, for permanant way edit the /etc/selinux/config and make the variable SELINUX=disabled  
#save the file and reboot the server
```

Save the `my.cnf` file and restart the MySQL service. Repeat these steps for other MySQL servers.

1. First, run the `galera_new_cluster` command on one MySQL server.
2. On other MySQL servers, run the following commands:

`systemctl start mysql`

Den Cluster testen

Connect to any of the MariaDB server, create a database, and switch

```sql
create database node1;  
USE node1;  
  
#Create a table and insert 100000 rows.  
CREATE TABLE example_table (  
    id INT AUTO_INCREMENT PRIMARY KEY,  
    name VARCHAR(50),  
    value INT  
);  
  
-- Insert 100,000 rows into the table  
INSERT INTO example_table (name, value)  
SELECT  
    'key', nums.n  
FROM (  
    SELECT  
        a.N + b.N * 10 + c.N * 100 + d.N * 1000 + e.N * 10000 + 1 AS n  
    FROM  
        (SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) a  
        ,(SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) b  
        ,(SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) c  
        ,(SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) d  
        ,(SELECT 0 AS N UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) e  
    ORDER BY n  
) nums;
```

Verify the count in the same table on the other nodes

`select count(1) from example_table;`

