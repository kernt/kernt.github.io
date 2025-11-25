---
tags:
  - konfigurationsmanagement
  - puppet
  - mysql
---
# Puppet MySQL Modul benutzen

MySQL ist ein sehr weit verbreiteter Datenbankserver, und es ist ziemlich sicher, dass Sie irgendwann einen MySQL Server installieren und konfigurieren müssen. Das `puppetlabs-mysql` Modul kann Ihre MySQL-Implementierungen vereinfachen.

## Wie es geht

Gehen Sie folgendermaßen vor, um das Beispiel zu erstellen:

1.Installiere das `puppetlabs-mysql` Modul:

```s
t@mylaptop../puppet/puppet $ puppet module install -i modules puppetlabs-mysql
Notice: Preparing to install int../puppet/ho../puppet/thom../puppet/pupp../puppet/modules ...
Notice: Downloading from http../puppet//forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/ho../puppet/thom../puppet/pupp../puppet/modules
└─┬ puppetlabs-mysql (v2.3.1)
  └── puppetlabs-stdlib (v4.3.2)
```

2.Erstellen Sie eine neue Knotendefinition für Ihren MySQL Server:

```ruby
node dbserver {
  class { '==mysql==server':
    root_password    => 'PacktPub',
    override_options => {
      'mysqld' => { 'max_connections' => '1024' }
    }
  }
}
```

3.Führen Sie die Puppe aus, um den Datenbankserver zu installieren und das neue Root-Passwort anzuwenden:

```s
[root@dbserver ~]# puppet agent -t
Info: Caching catalog for dbserver.example.com
Info: Applying configuration version '1414566216'
Notice../puppet/Stage[mai../puppet/Mysql==Server==Insta../puppet/Package[mysql-serve../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Mysql==Server==Servi../puppet/Service[mysql../puppet/ensure: ensure changed 'stopped' to 'running'
Info../puppet/Stage[mai../puppet/Mysql==Server==Servi../puppet/Service[mysqld]: Unscheduling refresh on Service[mysqld]
Notice../puppet/Stage[mai../puppet/Mysql==Server==Root_passwo../puppet/Mysql_user[root@localhos../puppet/password_hash: defined 'password_hash' as '*6ABB0D4A7D1381BAEE4D078354557D495ACFC059'
Notice../puppet/Stage[mai../puppet/Mysql==Server==Root_passwo../puppet/Fil../puppet/ro../puppet/.my.cn../puppet/ensure: defined content as '{md5}87bc129b137c9d613e9f31c80ea5426c'
Notice: Finished catalog run in 35.50 seconds
```

4.Vergewissern Sie sich, dass Sie eine Verbindung zur Datenbank herstellen können:

```s
[root@dbserver ~]# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle a../puppet/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation a../puppet/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## Wie es funktioniert

Das MySQL-Modul installiert den MySQL-Server und stellt sicher, dass der Server läuft. Es konfiguriert dann das Root-Passwort für MySQL. Das Modul macht noch viele andere Dinge für Sie. Es erstellt eine `.my.cnf` Datei mit dem Root-Benutzer-Passwort. Wenn wir den `mysql` Client ausführen, setzt die .`my.cnf` Datei alle Vorgaben, so dass wir keine Argumente liefern müssen.
Es gibt mehr...

Im nächsten Abschnitt zeigen wir, wie man Datenbanken und Benutzer erstellt.