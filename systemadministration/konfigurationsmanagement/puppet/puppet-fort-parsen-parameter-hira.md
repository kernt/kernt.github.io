---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet für fortgeschrittene: Hira Parameter benutzen

Wie die Parameter-Vorgaben, die wir im vorherigen Kapitel eingeführt haben, kann Hiera verwendet werden, um Standardwerte für Klassen bereitzustellen. Diese Funktion erfordert Puppet Version 3 und höher.

## Fertig werden

Installieren und konfigurieren Sie hiera wie in Kapitel 2, Puppeninfrastruktur. Erstellen Sie eine globale oder gemeinsame Yaml-Datei; Dies dient als Voreinstellung für alle Werte.

## Wie es geht

1.Erstellen Sie eine Klasse mit Parametern und ohne Standardwerte:

```s
t@mylaptop../puppet/puppet $ mkdir -p modul../puppet/mys../puppet/manifests t@mylaptop../puppet/puppet $ vim modul../puppet/mys../puppet/manifes../puppet/init.pp
class mysql ( $port, $socket, $package ) {
  notify {"Port: $port Socket: $socket Package: $package": }
}

```

2.Aktualisieren Sie Ihre .yaml Datei in Hiera mit den Standardwerten für die `mysql` Klasse:

```s
---
mysql::port: 3306
mysql::package: 'mysql-server'
mysql::socket:../puppet/v../puppet/l../puppet/mys../puppet/mysql.sock'
```

Wenden Sie die Klasse an einen Node an, Sie können die MySQL-Klasse zu Ihrer Standard Node ab jetzt hinzufügen.

```ruby
node default {
  class {'mysql': }
}
```

3.Führen Sie `puppet run` aus und überprüfen Sie die Ausgabe:

```s
[root@hiera-test ~]# puppet agent -t
Info: Caching catalog for hiera-test.example.com
Info: Applying configuration version '1411182251'
Notice: Port: 3306 Socket../puppet/v../puppet/l../puppet/mys../puppet/mysql.sock Package: mysql-server
Notice../puppet/Stage[mai../puppet/Mys../puppet/Notify[Port: 3306 Socket../puppet/v../puppet/l../puppet/mys../puppet/mysql.sock Package: mysql-serve../puppet/message: defined 'message' as 'Port: 3306 Socket../puppet/v../puppet/l../puppet/mys../puppet/mysql.sock Package: mysql-server'
Notice: Finished catalog run in 1.75 seconds
```

## Wie es funktioniert

Wenn wir die `mysql` Klasse in unserem Manifest instanziieren, haben wir keine Werte für irgendwelche der Attribute bereitgestellt.
Puppet weiß das es nach einem Wert in Hiera, der mit `class_name==parameter_name:` übereinstimmt oder `==class_name::parameter_name:` suchen muss.

Wenn Puppet einen Wert findet, verwendet er ihn als Parameter für die Klasse. Wenn Puppet keinen Wert in Hiera findet und kein Standard definiert ist, führt ein Katalogfehler zu der folgenden Ausgabe:

> Error: Could not retrieve catalog from remote server: Error 400 on SERVER: Must pass package to Class[Mysql] a../puppet/e../puppet/pupp../puppet/environmen../puppet/producti../puppet/manifes../puppet/site.pp:6 on node hiera-test.example.com

Dieser Fehler weist darauf hin, dass Puppet einen Wert für den Parameter `package` wünscht.

## Es gibt mehr

Sie können eine Hiera-Hierarchie definieren und verschiedene Werte für Parameter basierend auf Fakten liefern.
Sie könnten z. B. in Ihrer Hierarchie `%{::osfamily}` haben und haben unterschiedliche `yaml` Dateien, die auf dem Parameter `osfamily` (RedHat, Suse und Debian) basieren.