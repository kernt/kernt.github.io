---
tags:
  - mysql
  - berechtigungen
  - datenbanken
  - system-administration
---
# Mysql Berechtigungen Setzen

**DB suchen**

`show databases like '%DB%';`

## User Berechtigungen

* **ALL PRIVILEGES** – Gewährt (grants) alle privilegen zu einen Benutzer.
* **CREATE** – Datenbenken und tabellen anlegen.
* **DROP** - löschen von databases and tables.
* **DELETE** - delete Zeilen von einer speziefischen table.
* **INSERT** - Einfügen in bestimmten Zeilen in tablen.
* **SELECT** – database lesen.
* **UPDATE** - Zeilen in Tablen ändern.

**Neuen Benutzer anlegen**

`CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'user_password';`

**Benutzer Password ändern**

`ALTER USER 'database_user'@'localhost' IDENTIFIED BY 'new_password';`

**Neuen Benutzer anlegen auf ip eingeschränkt**

`CREATE USER 'newuser'@'10.8.0.5' IDENTIFIED BY 'user_password';`

**Neuen Benutzer anlegen ohne einschrenkungen**

`CREATE USER 'newuser'@'%' IDENTIFIED BY 'user_password';`

**Alle Bereatigungen für einen Benutzer an einer Datenebenk setzen**

`GRANT ALL PRIVILEGES ON database_name.* TO 'database_user'@'localhost';`

**ALL Berechtigungen für einen Benutzer zu allen Batenbanken**

`GRANT ALL PRIVILEGES ON *.* TO 'database_user'@'localhost';`

**Gewährt alle Berechtigungen für einen Benutzer über eine Tabelle zu einer Datenbank**

`GRANT ALL PRIVILEGES ON database_name.table_name TO 'database_user'@'localhost';`

**Gewähre mehre Berechtigungen für einen Benutzer zu einer bestimten Datenbak**

`GRANT SELECT, INSERT, DELETE ON database_name.* TO database_user@'localhost';`

**Berechtigungen für einen Benutzer zu anzeigen**

`SHOW GRANTS FOR 'database_user'@'localhost';`

**Wiederrufen von berechtigungen eines Benutzers**

`REVOKE ALL PRIVILEGES ON database_name.* TO 'database_user'@'localhost';`

**Entvernen eines existirenden Benutzes**

`DROP USER 'user'@'localhost'`

alle Berechtigungen werden auch entvernt.