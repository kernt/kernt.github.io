---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 standart bezeichnung

Die Wahl geeigneter und informativer Namen für Ihre Module und Klassen wird eine große Hilfe sein, wenn es um die Lesbarkeit Ihres Codes geht. 
Das ist noch wichtiger, wenn andere Leute ihren Code lesen und an ihren Manifesten arbeiten müssen.

Hier einige tips wie man dinge in ihren manifests benen kann:

1. Namensmodule nach der Software oder dem Service, denn es verwalten soll , z.B. `Apache` oder `haproxy`.
2. Namensklassen in Modulen (Unterklassen) nach der Funktion oder dem Dienst, den sie dem Modul zur Verfügung stellen, z.B. `apache==vhosts` oder `rails==dependencies`.
3. Wenn eine Klasse innerhalb eines Moduls den von diesem Modul bereitgestellten Dienst deaktiviert, benennen Sie ihn `disabled`. Zum Beispiel sollte eine Klasse, die Apache deaktiviert, `Apache::disabled` genannt werden.
4. Erstellen Sie eine Rolle und Profile Hierarchie von Modulen. Jede Node sollte eine einzige Rolle haben, die aus einem oder mehreren Profilen besteht. Jedes Profilmodul sollte einen einzigen Dienst konfigurieren.
5. Das Modul, das Benutzer verwaltet, sollte `user` benannt werden.
6. Innerhalb des Benutzerbausteins deklariere deine virtuellen Benutzer im `user::virtuell` (für mehrere virtuelle Benutzer und andere Ressourcen, siehe [Puppet4 Benutzer und virtuelle Ressourcen](puppet4-benutzer-virtuelleressourcen.md)).
7. Innerhalb des User-Moduls sollten Unterklassen für bestimmte Benutzergruppen nach der Gruppe benannt werden, z.B. `user==sysadmins` oder `user==contractors`.
8. Wenn Sie Puppet verwenden, um die Konfigurationsdateien für verschiedene Dienste bereitzustellen, benennen Sie die Datei nach dem Dienst, aber mit einem Suffix, das angibt, welche Art von Datei es ist, zum Beispiel:

* Apache init script: `apache.init`
* Logrotate config snippet für Rails: `rails.logrotate`
* Nginx vhost Datei for mywizzoapp: `mywizzoapp.vhost.nginx`
* MySQL Konfig für standalone server: `standalone.mysql`

10. Wenn Sie eine andere Version einer Datei je nach Betriebssystem Release bereitstellen müssen, können Sie beispielsweise eine Namenskonvention wie folgt verwenden:
```
memcached.lucid.conf
memcached.precise.conf
```

10. Sie können Puppet nutzen um automatisch die entsprechende Version auszuwählen:
```
source = > "puppe../puppet///modul../puppet/memcached
../puppet/memcached.${::lsbdistrelease}.conf",
```

11. Wenn eine Verwaltung der Releases von z.B Ruby Versionen  notwendig ist, benenne die Klasse gefold von der Release Version. Zum Beispiel `ruby192` oder `ruby186`.

##Weiterführendes

Puppet-Community unterhält eine Reihe von Best-Practice-Richtlinien für Ihre Puppet-Infrastruktur, die einige Hinweise auf Namenskonventionen enthält:
* (Puppet best_practices)[htt../puppet//docs.puppetlabs.c../puppet/guid../puppet/best_practices.html]

Einige Leute bevorzugen es, mehrere Klassen auf einem Node zu enthalten, indem sie eine durch Kommas getrennte Liste verwenden, anstatt getrennte Include-Anweisungen, zum Beispiel:

```
 node 'server014' inherits 'server' {
    include mail==server, repo==gem, repo::apt, zabbix
  }
```