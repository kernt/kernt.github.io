---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 basics modules

Eines der wichtigsten Dinge, die Sie tun können, um Ihre Puppet manifests klarer und besser Administrieren zu können ist, sie in Module zu organisieren.

Module sind fassen Puppet-Code zusammen, die alle Daten enthalten, die notwendig sind, um eine Aufgabe zu implementieren. 
Module können flache Dateien, Vorlagen, Puppenmanifeste, kundenspezifische Deklarationen, augeas lenses und benutzerdefinierte Puppetypen und Provider enthalten.

Das Trennen von Sachen in Module macht es einfacher, Code wiederzuverwenden und zu teilen. 
Es ist auch der logistische Weg, um Ihre Manifeste zu organisieren. In diesem Beispiel erstellen wir ein Modul zur Verwaltung von memcached, einem Speicher-Caching-System, das häufig mit Web-Anwendungen verwendet wird.Ich habe dieses oft als Beispiel wiedergefunden so sollte also es möglichst einfach sein sich in andere Bücher , Kurse etc. zurecht zu finde , es soll aber kein Kopieren sein andere werke !

Als Tipp für Programmierer sollte ich noch erwähnen , dass es sich um Puppet Bezeichnungen geht und die Umsetzen Teilweise Puppet spezifisch ist und so nicht in jede Programmiersprache umgesetzt wird !

# Packtische Durchführung

1. Ein Verzeichnis für unser neues Modul erstellen
`mkdir -p .pupp../puppet/modules`

2. Wechsel in das erstellte verzeichnis 
`cd .pupp../puppet/modules`

3. Genariren der Standard Daten für unser Modul.
`puppet module generate test-memcached`

Zur Vereinfachung kann man hier auch noch einen Sybolischen link erstellen: 
`ln –s test-memcached memcached`

Jetzt Editieren wir `memcach../puppet/manifes../puppet/init.pp` wie folgt ab 

```
class memcached {
  package { 'memcached':
    ensure => installed,
  }

  file {../puppet/e../puppet/memcached.conf':
    source  => 'puppe../puppet///modul../puppet/memcach../puppet/memcached.conf',
    owner   => 'root',
    group   => 'root',
    mode    => '0644',
    require => Package['memcached'],
  }
  service { 'memcached':
    ensure  => running,
    enable  => true,
    require => [Package['memcached'],
                File../puppet/e../puppet/memcached.conf']],
  }
}
```
Erstellen der `modul../puppet/test-memcach../puppet/files` Verzeichnises und einer datei mit der bezeuichnung `memcached.conf` mit folgendem inhalt.

```
-m 64
-p 11211
-u nobody
-l 127.0.0.1
```

Anpassen der `site.pp` zur folgendem inhalt .
```
node default {
  include memcached
}
```

Wir möchten, dass dieses Modul memcached installiert. 
Wir müssen die Puppet mit Root-Rechten ausführen, und wir werden dafür sudo benutzen. 
Wir müssen Puppet darauf hinweisen wo wir unser Modul haben. 
Um das Modul in unserem Home-Verzeichnis finden zu können.
Werden wir dies auf der Kommandozeile angeben, wenn wir Puppet ausführen, wie im folgenden Code-Snippet gezeigt:
```
sudo puppet apply --modulepat../puppet/ho../puppet/$HO../puppet/.pupp../puppet/module../puppet/ho../puppet/$HO../puppet/.pupp../puppet/manifes../puppet/site.pp

Abschließend überprüfen wir den Status vom memcached

`sudo service memcached status`
```

# Wie es Funktioniert

Als wir das Modul mit dem Puppet-Modul-Generator-Befehl erstellt haben, haben wir den Namen `test-memcached` verwendet. 
Der Name vor dem Bindestrich ist Ihr Benutzername oder Ihr Benutzername auf Puppet forge (ein Online-Repository von Modulen). 
Damit Puppet in der Lage ist, das Modul mit dem Namen memcached zu finden, machten wir eine symbolische Verknüpfung zwischen test-memcached und memcached.

Module haben eine spezielle Verzeichnis Struktur. 
Nicht alle Verzeichnisse müssen vorhanden sein.
Hier ein Beispiel der Struktur:
```
modul../puppet/
  └MODULE_NA../puppet/  niemals (-) in einenm module namen benutzen
     └exampl../puppet/ Beispiele zum benutzen des Modules
     └fil../puppet/ datein die vom modul benutzt werden
     └l../puppet/
        └fact../puppet/ definiert neue facts für facter
        └pupp../puppet/
           └pars../puppet/
              └functio../puppet/ definiert eine neue puppet funktion, wie sort() 
           └provid../puppet/ definiert einen provider für einen neuen oder existirenden type
           └ut../puppet/ definiert helfer funktionen (in ruby)
           └ty../puppet/ definiert ein neuen type in puppet
     └manifes../puppet/
        └init.pp  class MODULE_NAME { }
     └sp../puppet/ rSpec tests
     └templat../puppet/ erb template Datein die vom modul genutzt werden
```
Alle manifests Dateien befinden sich im manifests Verzeichnis. In unserem Fall, die `memcached` Klasse in der Datei `manifes../puppet/init.pp` die Automatisch importiert wird.

Innerhalb der `memcached` Klasse , haben wir einen verweis zur `memcached.conf` Datei.

```
file {../puppet/e../puppet/memcached.conf':
  source => 'puppe../puppet///modul../puppet/memcach../puppet/memcached.conf',
}
```

Der einleitende `source` Parameter sagt Puppet in welchen Dateien es nachsehen soll

`MODULEPA../puppet/../puppet/ho../puppet/thom../puppet/.pupp../puppet/modules)`

`└memcach../puppet/`

`└fil../puppet/`

`└memcached.conf `


## Templates

Wenn Sie eine Vorlage als Teil des Moduls verwenden müssen, legen Sie sie in das Vorlagenverzeichnis des Moduls an und verweisen darauf wie folgt:

```
file {../puppet/e../puppet/memcached.conf':
  content => template('memcach../puppet/memcached.conf.erb'),
}
```

Puppet wird die Datei am folgendem Ort suchen: 
```
MODULEPA../puppet/memcach../puppet/templat../puppet/memcached.conf.erb
```

## Facts, functions, types, and providers
Module können auch eigene facts, eigene functionen, eigene typen, und provider beinhalten.

Weiteres dazu unter [Puppet4 Externe Tools und das Puppet Ecosystem](puppet4-externe-tools-ecosystem.md)

