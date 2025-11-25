---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 externe tools ecosystem facts env

Ein weiterer praktischer Weg, um Informationen in Puppet und Facter zu bekommen, ist, es mit Umgebungsvariablen zu übergeben. Jede Umgebungsvariable, deren Name mit `FACTER_` beginnt, wird als Tatsache interpretiert. Zum Beispiel, fragen Sie den Wert von hallo mit dem folgenden Befehl:

```s
[root@cookbook ~]# facter -p hello
Hello, world
```

Überschreiben Sie nun den Wert mit einer Umgebungsvariablen und fragen Sie noch einmal:

```s
[root@cookbook ~]# FACTER_hello='Howdy!' facter -p hello
Howdy!
```

Es funktioniert genauso gut mit Puppet, also lasst uns ein Beispiel durchlaufen.

## Wie es geht

In diesem Beispiel legen wir eine Tatsache fest, die eine Umgebungsvariable verwendet:

1.Halten Sie die Knotendefinition für Kochbuch das gleiche wie unser letztes Beispiel:

```pp
node cookbook {
  notify {"$::hello": }
}
```

2.Führen Sie den folgenden Befehl aus:

```pp
[root@cookbook ~]# FACTER_hello="Hallo Welt" puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1416212026'
Notice: Hallo Welt
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Notify[Hallo Wel../puppet/message: defined 'message' as 'Hallo Welt'
Notice: Finished catalog run in 0.27 seconds
```
