---
tags:
  - mariadb
  - datenbanken
---
# Galera-Cluster MariaDB Nodes konfigurieren

Erstellen Sie eine `cnf`-Datei im Verzeichnis `/etc/mysql/conf.d` auf jedem Knoten, um die Galera-spezifischen Einstellungen festzulegen.

```shell
nano /etc/mysql/conf.d/galera.cnf
```

Die Datei beinhaltet allgemeine Datenbankeinstellungen, wie das binäre Protokollformat und die **Standard-Speicher-Engine**. Zudem enthält sie Konfigurationen für den Galera-Cluster, einschließlich des Cluster-Namens und der Cluster-Adresse.

Fügen Sie folgende Zeilen für den ersten Knoten ein:

```shell
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://node_1-ip-address,node_2-ip-address,node_3-ip-address"
# Galera Synchronization Configuration
wsrep_sst_method=rsync
# Galera Node Configuration
wsrep_node_address="node_1-ip-address"
wsrep_node_name="node_1"
```

- **Allgemeine Datenbankeinstellungen**: Dazu gehören Einstellungen wie `binlog_format=ROW` für das Format der binären Protokolle und `default-storage-engine=innodb` für die Standard-Speicher-Engine.
- **Galera-Provider-Konfiguration**: Einstellungen wie `wsrep_on=ON` dienen dazu, die Galera-Replikation zu aktivieren und `wsrep_provider=/usr/lib/galera/libgalera_smm.so`, um den Pfad zur Galera-Bibliothek anzugeben.
- **Galera-Cluster-Konfiguration**: Diese umfasst den Cluster-Namen (`wsrep_cluster_name`) und die Cluster-Adresse (`wsrep_cluster_address`), die die IP-Adressen oder Hostnamen der Knoten im Cluster enthält.
- **Galera-Synchronisation**: Konfiguriert die Methode für den State Snapshot Transfer (SST), z. B. `wsrep_sst_method=rsync`.
- **Galera-Knoten-Konfiguration**: Definiert die IP-Adresse oder den Hostnamen des aktuellen Knotens (`wsrep_node_address`) sowie den Namen des Knotens (`wsrep_node_name`).

Nachdem Sie die Datei gespeichert haben, erstellen Sie eine für den zweiten Knoten:

```shell
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://node_1-ip-address,node_2-ip-address,node_3-ip-address"
# Galera Synchronization Configuration
wsrep_sst_method=rsync
# Galera Node Configuration
wsrep_node_address="node_2-ip-address"
wsrep_node_name="node_2"
```

Fahren Sie nun mit dem letzten Knoten fort:

```shell
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://node_1-ip-address,node_2-ip-address,node_3-ip-address"
# Galera Synchronization Configuration
wsrep_sst_method=rsync
# Galera Node Configuration
wsrep_node_address="node_3-ip-address"
wsrep_node_name="node_3"
```

## Firewall auf Servern ändern

Da die Knoten miteinander über bestimmte Ports kommunizieren, müssen Sie die **Firewall-Einstellungen** anpassen.

Öffnen Sie die nachfolgenden **Ports** in Ihrer Firewall:

- **Port 3306**: Dies ist der Standardport für MariaDB. Er wird für die Datenbankkommunikation und -anfragen verwendet.
- **Galera-Ports**: Zusätzlich zum Standardport 3306 benutzt Galera auch andere Ports für die interne Kommunikation zwischen den Knoten. Der Standardbereich für Galera-Ports liegt in der Regel bei 4567, 4568 und 4444 für den State Snapshot Transfer (SST).

Sie können die Firewall-Einstellungen auf Ihrem Ubuntu-Server mit dem folgenden Befehl festlegen:

```shell
sudo ufw allow 3306,4567,4568,4444/tcp
sudo ufw allow 4567/udp
```

## Galera-Cluster MariaDB starten

Beenden Sie den MariaDB-Dienst, falls er bereits läuft:

```shell
sudo systemctl stop mariadb
```

Dieser Befehl startet den **MariaDB-Server** und initialisiert einen neuen Galera-Cluster auf dem ersten Knoten:

```shell
sudo galera_new_cluster
```

Prüfen Sie die Anzahl der Knoten im Cluster:

```shell
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

Sie sollten folgende Ausgabe erhalten:

```shell
+---------------------------+-------------+
| Variable_name        | Value       |
+--------------------------+--------------+
| wsrep_cluster_size | 1              |
+--------------------------+------------ -+
```

Der erste Knoten wurde erfolgreich gestartet.

Aktivieren Sie den zweiten Knoten:

```shell
systemctl start mariadb
```

Kontrollieren Sie, ob sich die Anzahl der Knoten erhöht hat:

```shell
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

In der Konsole sehen wir:

```shell
+---------------------------+-------------+
| Variable_name        | Value       |
+--------------------------+--------------+
| wsrep_cluster_size | 2               |
+--------------------------+--------------+
```
Nun starten wir den dritten Knoten:

```shell
systemctl start mariadb
```

Schauen Sie, ob der Knoten ordnungsgemäß läuft:

```shell
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

Es sollten sich jetzt drei Nodes im Cluster befinden:

```shell
+---------------------------+-------------+
| Variable_name        | Value       |
+--------------------------+--------------+
| wsrep_cluster_size | 3              |
+--------------------------+------------ -+
```

## Replikation testen

Stellen Sie sicher, dass Sie eine Verbindung zu jedem Knoten im Cluster herstellen können. Verwenden Sie den **MariaDB-Client**, um sich als Root-User oder als eine andere Benutzerin bzw. ein anderer Benutzer mit ausreichenden Rechten anzumelden.

```shell
mysql -u root -p
```

Erstellen Sie eine neue Testdatenbank auf einem der Knoten im Cluster:

```sql
CREATE DATABASE test_db;
```

Melden Sie sich bei den anderen Knoten an und überprüfen Sie, ob die Testdatenbank vorhanden ist:

```sql
SHOW DATABASES;
```

Die Testdatenbank sollte in der Liste der Datenbanken erscheinen:

```sql
+-------------------------------+
| Database                        |
+-------------------------------+
| information_schema   |
| mysql                              |
| performance_schema |
| test_db                          | 
| sys                                  |
+------------------------------+
```

Fügen Sie eine neue Testtabelle in der Testdatenbank hinzu:

```sql
USE test_db;
CREATE TABLE test_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50)
);
```

Geben Sie einige Testdaten in die Spalte `name` der Testtabelle ein:

```sql
INSERT INTO test_table (name) VALUES ('Alice'), ('Bob'), ('Charlie');
```

Kontrollieren Sie auf den anderen Knoten, ob die Testtabelle und die eingefügten Daten übernommen wurden:

```sql
USE test_db;
SELECT * FROM test_table;
```

Die Ausgabe zeigt uns die Liste der Personen mit ihren Namen und der ID an:

```sql
+----+-----------+
| id | name    |
+----+-----------+
| 1  | Alice     |
| 2  | Bob       |
| 3  | Charlie |
+----+----------+
```

So aktualisieren Sie einen Datensatz in der Testtabelle:

```sql
UPDATE test_table SET name = 'David' WHERE name = 'Alice';
```

Probieren Sie, einen Datensatz zu löschen:

```sql
DELETE FROM test_table WHERE name = 'Bob';
```

Überprüfen Sie auf den anderen Knoten, ob die Aktualisierungen und Löschungen repliziert wurden:

```sql
SELECT * FROM test_table;
```

Die Änderungen erscheinen erfolgreich auf jedem Knoten:

```sql
+----+------------+
| id | name     |
+----+-----------+
| 1  | David    |
| 3  | Charlie  |
+----+-----------+
```