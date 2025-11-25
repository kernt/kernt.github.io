---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet für fortgeschrittene: Virtuelle resourcen benutzen

Virtuelle Ressourcen in der Puppet mögen kompliziert und verwirrend sein, aber in Wirklichkeit sind sie sehr einfach. Sie sind genau wie reguläre Ressourcen, aber sie werden nicht wirklich wirksam, bis sie realisiert sind (im Sinne von "real gemacht"); Wohingegen eine reguläre Ressource nur einmal pro Knoten deklariert werden kann (so können zwei Klassen die gleiche Ressource beispielsweise nicht deklarieren). Eine virtuelle Ressource kann so oft wie notwendig realisiert werden.

Dies ist praktisch, wenn Sie Anwendungen und Dienste zwischen Maschinen verschieben müssen. Wenn zwei Anwendungen, die dieselbe Ressource verwenden, eine Maschine beenden, würden sie einen Konflikt verursachen, es sei denn, Sie machen die Ressource virtuell.

Um dies zu erklären, schauen wir uns eine typische Situation an, in der virtuelle Ressourcen nützlich sein könnten.

Sie sind verantwortlich für zwei beliebte Web-Anwendungen: WordPress und Drupal. Beide sind Web-Apps, die auf dem Apache laufen, also benötigen beide das installierte Apache-Paket . Die Definition für WordPress könnte so aussehen wie die folgenden:

```ruby
class wordpress {
  package {'httpd':
    ensure => 'installed',
  }
  service {'httpd':
    ensure => 'running',
    enable => true,
  }
}
```

Die Definition für Drupal könnte so aussehen:

```ruby
class drupal {
  package {'httpd':
    ensure => 'installed',
  }
  service {'httpd':
    ensure => 'running',
    enable => true,
  }
}
```

Alles ist gut, bis man beide Apps auf einem einzigen Server konsolidieren muss (was meist gründen der Optimierung vorkommt):

```ruby
node 'bigbox' {
  include wordpress
  include drupal
}
```

Jetzt wird sich Puppet beschweren, weil Sie versucht haben, zwei Ressourcen mit dem gleichen Namen zu definieren hier: httpd.

![error](http../puppet//www.packtpub.c../puppet/graphi../puppet/97817882976../puppet/graphi../puppet/B03643_05_01.jpg)

```s
Error: Could not retrieve catalog from remote server: Error 400 on SERVER: Duplicate declaration: Package[httpd] is already declared in fil../puppet/e../puppet/pupp../puppet/environme../puppet/pupp../puppet/environmen../puppet/producti../puppet/modul../puppet/drup../puppet/manifse../puppet/init.pp:4 on node bigbox.example.com
Warning: Not using cache on failed catalog
Error: Clould not retrieve catalog; skipping run
```

Sie könnten die doppelte Apache-Paketdefinition aus einer der Klassen entfernen, aber dann werden Nodes ohne die Klasse einschließlich Apache fehlschlagen.

Sie können dieses Problem umgehen, indem Sie das Apache-Paket in seine eigene Klasse und dann mit `include apache` überall, wo es benötigt wird hinzufügen, Puppet hat nichts dagegen, dass Sie die gleiche Klasse mehrmals verwenden.
In Wirklichkeit löst Apache in seiner eigenen Klasse die meisten Probleme aus, aber im Allgemeinen hat diese Methode den Nachteil, dass jede potenziell widersprüchliche Ressource eine eigene Klasse haben muss was aber nicht immer vorteilhaft ist.

Virtuelle Ressourcen können verwendet werden, um dieses Problem zu lösen. Eine virtuelle Ressource ist genau wie eine normale Ressource, außer dass es mit einem `@` Zeichen beginnt:

`@package { 'httpd': ensure => installed }`

Sie können es sich als einene Platzhalter-Ressource vorstellen ; Du willst es definieren, aber du bist nicht sicher, dass du es auch benutzen wirst. Puppe liest und merkt sich virtuelle Ressource Definitionen, aber wird nicht wirklich die Ressource benutzen, bis Sie die Ressource einrichten.

Um die Ressource zu erstellen, benutzen wir die `realize` funktion:

`realize(Package['httpd'])`

Sie können `realize` aufrufen, so oft wie Sie wollen um auf die Ressource zuzugreifen und es wird nicht zu einem Konflikt führen. So sind virtuelle Ressourcen der Weg , wenn mehrere verschiedene Klassen alle die gleiche Ressource benötigen, und sie müssen möglicherweise auf dem gleichen Knoten koexistieren.

## Wie es geht

Hier ist, ein Beispiel wie man das mit virtuellen Ressourcen umsetzt:

1.Erstellen Sie das virtuelle Modul mit folgendem Inhalt:

```ruby
class virtual {
  @package {'httpd': ensure => installed }
  @service {'httpd':
    ensure  => running,
    enable  => true,
    require => Package['httpd']
  }
}
```

2.Erstellen Sie das Drupal-Modul mit folgendem Inhalt:

```ruby
class drupal {
  include virtual
  realize(Package['httpd'])
  realize(Service['httpd'])
}
```

3.Erstelle das WordPress module mit dem folgendem Inhalt:

```ruby
class wordpress {
  include virtual
  realize(Package['httpd'])
  realize(Service['httpd'])
}
```

4.Ändern Sie Ihre `site.pp` Datei wie folgt:

```ruby
node 'bigbox' {
  include drupal
  include wordpress
}
```

5.Run Puppet:

```s
bigbox# puppet agent -t
Info: Caching catalog for bigbox.example.com
Info: Applying configuration version '1413179615'
Notice../puppet/Stage[mai../puppet/Virtu../puppet/Package[http../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Virtu../puppet/Service[http../puppet/ensure: ensure changed 'stopped' to 'running'
Info../puppet/Stage[mai../puppet/Virtu../puppet/Service[httpd]: Unscheduling refresh on Service[httpd]
Notice: Finished catalog run in 6.67 seconds
```

## Wie es funktioniert

Sie definieren das Paket und den Service als virtuelle Ressourcen an einem Ort: die `virtual` Klasse.
Alle Knoten können diese Klasse einschließen und Sie können alle Ihre virtuellen Dienste und Pakete in dieser Klasse refinanzieren.
Keines der Pakete wird tatsächlich auf einem Knoten installiert oder Dienste gestartet, bis Sie `relize` anrufen:

```ruby
class virtual {
  @package { 'httpd': ensure => installed }
}
```

Jede Klasse, die das Apache-Paket benötigt, kann auf diese virtuellen Ressource aufrufen:

```ruby
class drupal {
  include virtual
  realize(Package['httpd'])
}
```

Puppet weiß, weil Sie die Ressource virtuell gemacht haben, dass Sie beabsichtigten, mehrere Verweise auf das gleiche Paket zu haben und nicht nur zufällig zwei Ressourcen mit dem gleichen Namen zu erstellen.
So macht Puppet das Richtige.

## Es gibt mehr

Um virtuelle Ressourcen zu realisieren, kannst du auch die Signatur-Raumschiff-Syntax verwenden:
`Package <| title = 'httpd' |>`

Der Vorteil dieser Syntax ist, dass Sie nicht auf den Ressourcennamen beschränkt sind. Sie könnten auch ein Tag verwenden, zum Beispiel:
`Package <| tag = 'web' |>`

Alternativ können Sie auch alle Instanzen des Ressourcentyps angeben, indem Sie den Abfragebereich leer lassen:
`Package <| |>`