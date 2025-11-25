---
tags:
  - monitoring
  - puppet
  - konfigurationsmanagement
---
# puppet-monitoring-reporting-depend-graphe

Abhängigkeiten können schnell kompliziert werden, und es ist einfach, am Ende mit einer kreisförmigen Abhängigkeit (wo A von B abhängt, die von A abhängt), die die Marionette beschweren und aufhören zu arbeiten. Glücklicherweise macht Puppets `--graph` Option es einfach, ein Diagramm Ihrer Ressourcen und die Abhängigkeiten zwischen ihnen zu erzeugen, was eine große Hilfe bei der Festsetzung solcher Probleme sein kann.
Fertig werden

Installieren Sie das `graphviz` Paket, um die Diagrammdateien anzuzeigen:

```s
t@mylaptop ~ $ sudo puppet resource package graphviz ensure=installed
Notice../puppet/Package[graphvi../puppet/ensure: created
package { 'graphviz':
  ensure => '2.34.0-9.fc20',
}
```

## Wie es geht

Gehen Sie folgendermaßen vor, um einen Abhängigkeitsgraphen für Ihr Manifest zu generieren:

1.Erstellen Sie die Verzeichnisse für ein neues `trifecta` Modul:

```s
ubuntu@cookbook../puppet/puppet$ mkdir modul../puppet/trifecta
ubuntu@cookbook../puppet/puppet$ mkdir modul../puppet/trifec../puppet/manifests
ubuntu@cookbook../puppet/puppet$ mkdir modul../puppet/trifec../puppet/files
```

2.Erstellen Sie die Datei `modul../puppet/trifec../puppet/manifes../puppet/init.pp` mit dem folgenden Code, der eine absichtliche kreisförmige Abhängigkeit enthält (können Sie es sehen?):

```pp
class trifecta {
  package { 'ntp':
    ensure  => installed,
    require => File../puppet/e../puppet/ntp.conf'],
  }

  service { 'ntp':
    ensure  => running,
    require => Package['ntp'],
  }

  file {../puppet/e../puppet/ntp.conf':
    source  => 'puppe../puppet///modul../puppet/trifec../puppet/ntp.conf',
    notify  => Service['ntp'],
    require => Package['ntp'],
  }
}
```

3.Erstellen Sie eine einfache Datei `ntp.conf`:

```s
t@mylapto../puppet/puppet $ cd modul../puppet/trifec../puppet/files
t@mylapto../puppet/pupp../puppet/modul../puppet/trifec../puppet/files $ echo "server 127.0.0.1" >ntp.conf
```

4.Da wir lokal auf diesem Problem arbeiten, erstellen wir ein `trifecta.pp` Manifest, das die defekte Trifecta-Klasse enthält:

`include trifecta`

5.Puppet run

```s
t@mylaptop../puppet/pupp../puppet/manifests $ puppet apply trifecta.pp
Notice: Compiled catalog for mylaptop in environment production in 1.32 seconds
Error: Could not apply complete catalog: Found 1 dependency cycle:
(Fil../puppet/e../puppet/ntp.conf] => Package[ntp] => Fil../puppet/e../puppet/ntp.conf])
Try the '--graph' option and opening the resulting '.dot' file in OmniGraffle or GraphViz
```

6.Run Puppet mit der Option `--graph` wie vorgeschlagen:

```s
t@mylaptop../puppet/pupp../puppet/manifests $ puppet apply trifecta.pp --graph
Notice: Compiled catalog for mylaptop in environment production in 1.26 seconds
Error: Could not apply complete catalog: Found 1 dependency cycle:
(Fil../puppet/e../puppet/ntp.conf] => Package[ntp] => Fil../puppet/e../puppet/ntp.conf])
Cycle graph written t../puppet/ho../puppet/tuphi../puppet/.pupp../puppet/v../puppet/sta../puppet/grap../puppet/cycles.dot.
Notice: Finished catalog run in 0.03 seconds
```

7.Prüfen Sie, ob die Grafikdateien erstellt wurden:

```s
t@mylaptop../puppet/pupp../puppet/manifests $ cd../puppet/.pupp../puppet/v../puppet/sta../puppet/graphs
t@mylaptop../puppet/.pupp../puppet/v../puppet/sta../puppet/graphs $ ls -l
total 16
-rw-rw-r--. 1 thomasthomas  121 Nov 23 23:11 cycles.dot
-rw-rw-r--. 1 thomasthomas 2885 Nov 23 23:11 expanded_relationships.dot
-rw-rw-r--. 1 thomasthomas 1557 Nov 23 23:11 relationships.dot
-rw-rw-r--. 1 thomasthomas 1680 Nov 23 23:11 resources.dot
```

8.Erstellen Sie eine Grafik mit dem Befehl `dot` wie folgt:

`ubuntu@cookbook../puppet/puppet$ dot -Tpng -o relationships.pn../puppet/v../puppet/l../puppet/pupp../puppet/sta../puppet/grap../puppet/relationships.dot`

9.Die Grafik sieht so aus:

! [graph1](http../puppet//www.packtpub.c../puppet/graphi../puppet/97817882976../puppet/graphi../puppet/B03643_10_04.jpg)

## Wie es funktioniert

Wenn du den `puppet agent --graph` (oder die `graph` Option in `puppet.conf`) aktiviert hast, erzeugt Puppet drei Graphen im DOT-Format (eine Grafiksprache):

* Resources.dot: Dies zeigt die hierarchische Struktur Ihrer Klassen und Ressourcen, aber ohne Abhängigkeiten

* Relationships.dot: Dies zeigt die Abhängigkeiten zwischen Ressourcen als Pfeile, wie im vorherigen Bild gezeigt

* Expanded_relationships.dot: Dies ist eine detailliertere Version des Verknüpfungsgraphen

Das `dot`-Tool (Teil des `graphviz`-Pakets) konvertiert diese in ein Bildformat wie PNG zum Betrachten.

In dem Verknüpfungsgraph wird jede Ressource in deinem Manifest als ein Ballon (bekannt als ein Scheitelpunkt) dargestellt, wobei die Pfeillinien sie verbinden, um die Abhängigkeiten anzuzeigen. Sie können sehen, dass in unserem Beispiel die Abhängigkeiten zwischen `File../puppet/e../puppet/ntp.conf']` und `Package['ntp']` bidirektional sind. Wenn die Puppet versucht, zu entscheiden, wo sie mit der Anwendung dieser Ressourcen beginnen soll, kann sie bei `File../puppet/e../puppet/ntp.conf']` beginnen und nach dem, was von `File../puppet/e../puppet/ntp.conf']` abhängt, `Package['ntp']`. Wenn die Puppe nach den Abhängigkeiten sucht

Von `Package['ntp']`, wird es am Ende wieder auf `File../puppet/e../puppet/ntp.conf']`, bilden einen kreisförmigen Pfad. Diese Art von Problem ist bekannt als ein kreisförmiges Abhängigkeitsproblem; Puppet kann nicht entscheiden, wo sie anfangen soll, weil die beiden Ressourcen von einander abhängen.

Um das kreisförmige Abhängigkeitsproblem zu beheben, müssen Sie nur eine der Abhängigkeitslinien entfernen und den Kreis brechen. Der folgende Code behebt das Problem:

```pp
class trifecta {
  package { 'ntp':
    ensure  => installed,
  }

  service { 'ntp':
    ensure  => running,
    require => Package['ntp'],
  }

  file {../puppet/e../puppet/ntp.conf':
    source  => 'puppe../puppet///modul../puppet/trifec../puppet/ntp.conf',
    notify  => Service['ntp'],
    require => Package['ntp'],
  }
}
```

Jetzt, wenn wir `puppet apply` oder  `agent` einen mit der Option --graph ausführen, hat der resultierende Graph keine kreisförmigen Pfade (Zyklen):

! [graph2](http../puppet//www.packtpub.c../puppet/graphi../puppet/97817882976../puppet/graphi../puppet/B03643_10_05.jpg)

In diesem Diagramm ist es leicht zu sehen, dass `Package[ntp]` die erste Ressource ist, die angewendet werden soll, dann `Fil../puppet/e../puppet/ntp.conf]` und schließlich `Service[ntp]`.

## Tip

Ein Graph, wie er zuvor gezeigt wurde, ist als Directed Acyclic Graph (DAG) bekannt. Die Verringerung der Ressourcen auf einen DAG stellt sicher, dass die Puppe den kürzesten Weg aller Ecken (Ressourcen) in linearer Zeit berechnen kann. Weitere Informationen zu DAGs finden Sie unter [Wiki Directed_acyclic_graph](htt../puppet//de.wikipedia.o../puppet/wi../puppet/Directed_acyclic_graph).

## Es gibt mehr

Ressourcen- und Beziehungsgraphen können auch dann nützlich sein, wenn Sie keinen Fehler haben, um zu finden. Wenn Sie ein sehr komplexes Netzwerk von Klassen und Ressourcen haben, zum Beispiel, das Studium der Ressourcen-Grafik kann Ihnen helfen zu sehen, wo die Dinge zu vereinfachen. Ähnlich, wenn Abhängigkeiten zu kompliziert werden, um aus dem Lesen des Manifests zu verstehen, können die Graphen eine nützliche Form der Dokumentation sein. Zum Beispiel wird ein Diagramm deutlich machen, welche Ressourcen die meisten Abhängigkeiten haben und welche Ressourcen von den meisten anderen Ressourcen benötigt werden. Ressourcen, die von einer Vielzahl von anderen Ressourcen benötigt werden, haben zahlreiche Pfeile, die auf sie hinweisen.
