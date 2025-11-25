---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschrittene: Tags einsetzen

Eine häufige Anforderung ist, eine bestimmte Gruppe von Ressourcen vor anderen Gruppen anzuwenden (zum Beispiel einer Paket-Repository oder eine benutzerdefinierte Ruby-Version zu installieren), oder nach der anderen (zum Beispiel der Bereitstellung eine Anwendung, sobald ihre Abhängigkeiten installiert sind). Puppet der Laufstufe Funktion ermöglicht es Ihnen, dies zu tun.

Standardmäßig werden alle Ressourcen in Ihrem Manifest in einer einzigen Stufe genannt angewandt main. Wenn Sie eine Ressource müssen vor allen anderen angewendet werden, können Sie es zu einem neuen Fahrstufe zuweisen, bevor kommen angegeben wird main. In ähnlicher Weise könnten Sie einen Lauf Stufe definieren , die nach kommt main. In der Tat können Sie so viele Laufstufen definieren , wie Sie Puppet benötigen und erklären , welche Reihenfolge sie in angewendet werden.

In diesem Beispiel werden wir Stufen verwenden Sie eine Klasse, um sicherzustellen, angewendet erste und eine andere letzte.

## Wie es geht

Hier sind die Schritte , um ein Beispiel für die Verwendung Lauf zu erstellen stages:

1.Erstellen Sie die Datei `modul../puppet/adm../puppet/manifes../puppet/stages.pp` mit folgendem Inhalt:

```ruby
  class admin::stages {
    stage { 'first': before => Stage['main'] }
    stage { 'last': require => Stage['main'] }
    class me_first {
      notify { 'This will be done first': }
    }
    class me_last {
      notify { 'This will be done last': }
    }
    class { 'me_first':
      stage => 'first',
    }
    class { 'me_last':
      stage => 'last',
    }
  }
```

Ändern Sie bitte Ihre site.ppDatei wie folgt:

```ruby
  node 'Kochbuch' {
    Klasse { 'first_class':}
    Klasse { 'second_class':}
    Admin umfassen :: Stufen
  }
```

3.Puppet run:

```s
root@cookbook:~# puppet agent -t
Info: Applying configuration version '1411019225'
Notice: This will be done first
Notice: Second Class
Notice: First Class
Notice: This will be done last
Notice: Finished catalog run in 0.43 seconds
```

## Wie es funktioniert

Lassen Sie sich diesen Code im Detail untersuchen , um zu sehen , was passiert. Zuerst erklären wir die Laufphasen firstund last, wie folgt:

```ruby
  stage { 'first': before => Stage['main'] }
  stage { 'last': require => Stage['main'] }
```

Für die `first` Stage, haben wir festgelegt , dass es vor kommen sollte `main`. Das heißt, jede Ressource markiert in der als `first` in dem vor jeder Ressource angewandt Stufe `main` stage (die Standard - stage).

Die `last` Stage erfordert die `main` stage, so dass keine Ressource in der `last` stage kann in der bis nach jeder Ressource angewandt wird `main` stage.

Wir erklären dann einige Klassen, die wir später in diesem Lauf stages zuordnen werden:

```ruby
  class me_first {
    notify { 'This will be done first': }
  }
  class me_last {
    notify { 'This will be done last': }
  }
```

Wir können es jetzt setzen alle zusammen und umfassen diese Klassen auf dem Knoten, die Laufstufen für jede Angabe, wie wir dies tun:

```ruby
  class { 'me_first': stage => 'first',
  }
  class { 'me_last': stage => 'last',
  }
```

Beachten Sie, dass in den `class` Erklärungen für `me_first` und `me_last` mussten  wir nicht angeben , dass sie einen Parameter `stage` nehmen sollen.
Der `stage` Parameter ist ein weiterer metaparameter, was bedeutet, es kann ohne explizit deklariert werden müssen zu jeder Klasse oder Ressource angewandt werden.
Wenn wir `puppet agent` auf unserem Puppet Node laufen lassen, teilen wir der `me_first` Klasse mit das  `first_class` zuerst Laufen soll und `second_class` danach.
Die `me_last` benachrichtigung wurde nach der angelegten `main` Stage durchgeführt, so dass beide nach dem sie benachrichtigt  wurden von `first_class` und danach die `second_class`.
Wenn der `puppet agent` mehrere Male läuft, werden Sie sehen, dass die Benachrichtigungen aus `first_class` und `second_class` nicht immer in der gleichen Reihenfolge erscheinen , aber die `me_first` Klasse wird immer an erster Stelle und die `me_last` Klasse wird immer zuletzt angezeigt.

## Es gibt mehr

Sie können so viele Stages definieren, wie Sie möchten, und richten Sie jede Bestellung für sie ein. Dies kann ein kompliziertes Manifest, das sonst viele explizite Abhängigkeiten zwischen Ressourcen erfordern würde, erheblich vereinfachen.
Vorsicht vor versehentlicher Einführung von Abhängigkeitszyklen; Wenn du dir etwas zu einer Stage zuweist, machst du automatisch alles in früheren Stadien abhängig.

Sie können Ihre Stationen in der `site.pp` Datei stattdessen definieren, so dass es auf der obersten Ebene des Manifests einfach ist zu sehen, welche Stufen verfügbar sind.

Gary Larizza hat eine hilfreiche Einführung geschrieben die Stufen verwendet, auf ihrer Website:

[using-run-stages-with-puppet](htt../puppet//garylarizza.c../puppet/bl../puppet/20../puppet/../puppet/../puppet/using-run-stages-with-pupp../puppet/)

Ein Vorbehalt: Viele Leute mögen es nicht, Stages zu benutzen, denn das Gefühl, dass die Puppet bereits eine ausreichende Ressourcenkontrolle bereitstellt, und dass die Einsatzstufen ununterbrochen Ihren Code sehr schwer zu befolgen machen können.
Die Verwendung von Stages sollte möglichst auf ein Minimum reduziert werden. Es gibt ein paar wichtige Beispiele, wo die Verwendung von Stages weniger Komplexität schafft.
Die bemerkenswerteste ist, wenn eine Ressource das System verwendet, um Pakete auf dem System zu installieren. Es hilft, eine Paket-Management-Stage zu haben, die vor der Haupt Stage kommt.
Wenn Pakete in der Haupt- (Standard-) Stage definiert sind, können Ihre Manifeste auf die aktualisierten Paketverwaltungs konfigurationsinformationen zählen, die vorhanden sind.
Zum Beispiel, für ein Yum-basiertes System, würden Sie eine yum repos Bühne erstellen, die vor dem Haupt Stage kommt. Sie können diese Abhängigkeit mit Pfeilen `->` angeben, wie im folgenden Code-Snippet gezeigt:

```ruby
  stage {'yumrepos': }
  Stage['yumrepos'] -> Stage['main']
```

Wir können dann eine Klasse erstellen, die ein Yum Repository (yumrepo) Ressource erstellt und sie dem yumrepos Stage wie folgt zuordnet:

```ruby
  class {'yums': stage => 'yumrepos',
  }
  class yums {
    notify {'always before the rest': }
    yumrepo {'testrepo': baseurl => 'fil../puppet///v../puppet/yum', ensure  => 'present',
    }
  }
```

Für Apt-basierte Systeme wäre das gleiche Beispiel ein stage, in dem Apt-Quellen definiert sind. Der Schlüssel mit stages ist es, ihre Definitionen in deiner `site.pp` Datei zu behalten, wo sie sehr sichtbar sind und sie nur sparsam verwenden, wo man garantieren kann, dass man keine Abhängigkeit kreise einführt.