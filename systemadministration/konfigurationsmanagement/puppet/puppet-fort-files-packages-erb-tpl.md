---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet für fortgeschrittene: erb Templates

Während du Konfigurationsdateien einfach mit Puppet als einfache Textdateien einsetzen kannst, sind Vorlagen viel leistungsfähiger. Eine Vorlagendatei kann Berechnungen durchführen, Ruby-Code ausführen oder die Werte von Variablen aus Ihren Puppet-Manifests verweisen. Überall, wo Sie eine Textdatei mit Puppet bereitstellen können, können Sie stattdessen eine Vorlage verwenden.

Im einfachsten Fall kann eine Vorlage nur eine statische Textdatei sein. Nützlicher können Sie Variablen mit der ERB (embedded Ruby) -Syntax einfügen. Beispielsweise:

`<%= @name %>, this is a very large drink.`

Wenn die Vorlage in einem Kontext verwendet wird, in dem die Variable $ name Zaphod Beeblebrox enthält, wird die Vorlage ausgewertet:

`Zaphod Beeblebrox, this is a very large drink.`

Diese einfache Technik ist sehr nützlich, um viele Dateien zu erzeugen, die sich nur in den Werten von ein oder zwei Variablen unterscheiden, z. B. virtuelle Hosts und zum Einfügen von Werten in ein Skript wie Datenbanknamen und Passwörter.

## Wie es geht

In diesem Beispiel verwenden wir eine ERB-Vorlage, um ein Passwort in ein Backup-Skript einzufügen:

1.Erstellen Sie die Datei `modul../puppet/adm../puppet/templat../puppet/backup-mysql.sh.erb` mit folgendem Inhalt:

```s
../puppet/b../puppet/sh
/u../puppet/b../puppet/mysqldump -uroot \ -p<%= @mysql_password %> \ --all-databases | ../puppet/b../puppet/gzip ../puppet/back../puppet/mys../puppet/all-databases.sql.gz
```

2.Ändern Sie Ihre `site.pp` Datei wie folgt:

```ruby
node 'cookbook' {
  $mysql_password = 'secret'
  file {../puppet/u../puppet/loc../puppet/b../puppet/backup-mysql':
    content => template('adm../puppet/backup-mysql.sh.erb'),
    mode    => '0755',
  }
}
```

3.run Puppet:

```s
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1412140971'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/u../puppet/loc../puppet/b../puppet/backup-mysq../puppet/ensure: defined content as '{md5}c12af56559ef36529975d568ff52dca5'
Notice: Finished catalog run in 0.31 seconds
```

4.Prüfen Sie, ob Puppet das Passwort korrekt in die Vorlage eingefügt hat:

```s
[root@cookbook ~]# ca../puppet/u../puppet/loc../puppet/b../puppet/backup-mysql
../puppet/b../puppet/sh
/u../puppet/b../puppet/mysqldump -uroot \
  -psecret \
  --all-databases | \
../puppet/b../puppet/gzip ../puppet/back../puppet/mys../puppet/all-databases.sql.gz
```

## Wie es funktioniert

Wo immer eine Variable in der Vorlage referenziert wird, z. B. `<% = @mysql_password%>`, wird die Puppe sie durch den entsprechenden Wert ersetzen hier `secret`.

## Es gibt mehr…

Im Beispiel haben wir nur eine Variable in der Vorlage verwendet, aber du kannst so viele haben, wie du willst. Dies können auch Tatsachen sein:

`ServerName <%= @fqdn %>`

Oder Ruby Ausdrücke:

`MAILTO=<%= @emails.join(',') %>`

Oder Ruby Ausdrücke:

`ServerAdmin <%= @sitedomain == 'coldcomfort.com' ? 'seth@coldcomfort.com' : 'flora@poste.com' %>`