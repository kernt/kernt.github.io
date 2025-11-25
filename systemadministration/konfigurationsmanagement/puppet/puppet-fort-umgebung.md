---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschrittene: Umgebung

Oft in einem Puppet Manifests, müssen Sie wissen, wo einige lokale Informationen über die Maschine zu finden sind. Facter ist das Werkzeug, das Puppet begleitet, um eine Standardmethode bereit zu stellen, um Informationen (facts) aus der Umgebung zu bekommen, wie solche:

* Betriebssystem
* Speichergröße
* Die Architektur
* Prozessor zählen

Um eine vollständige Liste der auf Ihrem System verfügbaren Fakten zu sehen, führen Sie:

```s
$ sudo facter
architecture => amd64
augeasversion => 0.10.0
domain => compute-1.internal
ec2_ami_id => ami-137bcf7a
ec2_ami_launch_index => 0
```

## Hinweis

Während es praktisch sein kann, diese Informationen von der Kommandozeile zu erhalten, liegt die wirkliche Kraft von Facter darin, auf diese Fakten in deinen Puppenmanifesten zuzugreifen.

Some modules define their own facts; to see any facts that have been defined locally, add the -p (pluginsync) option to facter as follows:

`$ sudo facter -p`

## Wie es geht…

Hier ist ein Beispiel für die Verwendung von facter Fakten in einem Manifest:

1.Verweis auf eine Fache Tatsache in deinem Manifest wie jede andere Variable. Fakten sind globale Variablen in Puppet, also sollten sie mit einem doppelten Doppelpunkt vorangestellt werden (`::`), wie im folgenden Code-Snippet:

```ruby
notify { "This is $==operatingsystem version $==operatingsystemrelease, on $==architecture architecture, kernel version $==kernelversion": }

```

2.Wenn Puppet läuft, werden die entsprechenden Werte für den aktuellen Nodes ausgefüllt:

```s
[root@hiera-test ~]# puppet agent -t
...
Info: Applying configuration version '1411275985'Notice: This is RedHat version 6.5, on x86_64 architecture, kernel version 2.6.32
...
Notice: Finished catalog run in 0.40 seconds
```

## Wie es funktioniert

Facter bietet eine Standardmethode für Manifestationen, um Informationen über die Knoten zu erhalten, auf die sie angewendet werden. Wenn Sie sich auf eine Tatsache in einem Manifest beziehen, wird die Puppe Facter abfragen, um den aktuellen Wert zu erhalten und ihn in das Manifest einzufügen. Futt Fakten sind Top-Umfang Variablen.

### Tip

Verweisen Sie immer auf fatcts mit führenden Doppelkolonien, um sicherzustellen, dass Sie die Tatsache und nicht eine lokale Variable verwenden:

`$::hostname` nicht `$hostname`

## Es gibt mehr…

Sie können auch Fakten in ERB-Vorlagen verwenden. Beispielsweise möchten Sie den Hostnamen des Knotens in eine Datei einfügen oder eine Konfigurationseinstellung für eine Anwendung ändern, die auf der Speichergröße des Nodes basiert. Wenn Sie fact Namen in Vorlagen verwenden, denken Sie daran, dass sie kein Dollarzeichen brauchen, denn das ist Ruby, nicht Puppet:

```s
$KLogPath <%= case @kernelversion when '2.6.31' then
'/v../puppet/r../puppet/rsysl../puppet/kmsg' else../puppet/pr../puppet/kmsg' end %>
```

Wenn Sie auf fatcts verweisen, verwenden Sie die `@` Syntax. Variablen, die im gleichen Umfang wie der Funktionsaufruf zur Vorlage definiert sind, können auch mit der `@` Syntax referenziert werden. Out of scope Variablen sollten die `scope` Funktion verwenden.
Um beispielsweise auf die `mysql::port` Variable zu verweisen, die wir früher in den `mysql´ Modulen definiert haben, verwenden Sie Folgendes:

`MySQL Port = <%= scope['==mysql==port'] %>`

Wenn Sie diese Vorlage anwenden, wird folgende Datei angezeigt:

```s
[root@hiera-test ~]# puppet agent -t
...
Info: Caching catalog for hiera-test.example.com
Notice../puppet/Stage[mai../puppet/E../puppet/Fil../puppet/t../puppet/template-tes../puppet/ensure: defined content as '{md5}96edacaf9747093f73084252c7ca7e67'
Notice: Finished catalog run in 0.41 seconds [root@hiera-test ~]# ca../puppet/t../puppet/template-test
MySQL Port = 3306
```

### Siehe auch

* [Puppet4 Externe Tools und das Puppet Ecosystem](puppet4-externe-tools-ecosystem.md)