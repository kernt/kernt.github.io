---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet für fortgeschrittene: Virtuelle Resourcen expotieren

Alle unseren Rezepten bis zu diesem Punkt hat sich um eine einzelne maschine gehandelt. Es ist möglich mit Puppet, Ressourcen von einem Nodes zu haben, die einen anderen Node beeinflussen. Diese Interaktion wird mit exportierten Ressourcen verwaltet. Exportierte Ressourcen sind genau wie jede Ressource, die Sie für einen Knoten definieren könnten, aber anstatt auf den Node anzuwenden, auf dem sie erstellt wurden, werden sie für die Verwendung von allen Nodes in der Umgebung exportiert. Ausgeführte Ressourcen können als virtuelle Ressourcen betrachtet werden, die einen Schritt weiter gehen und über den Node hinausgehen, auf dem sie definiert wurden.

Es gibt zwei Aktionen mit exportierten Ressourcen. Wenn eine exportierte Ressource erstellt wird, soll sie definiert werden. Wenn alle exportierten Ressourcen geerntet werden, sollen sie gesammelt werden. Das Definieren von exportierten Ressourcen ähnelt virtuellen Ressourcen; Die Ressource in Frage hat zwei `@` Symbole vorangestellt. Um beispielsweise eine Datei-Ressource als extern zu definieren, verwenden Sie `@@file`. Das Sammeln von Ressourcen erfolgt mit dem Raumschiff-operator(space ship operator), `<< | | >>`; So genant  da wie ein Raumschiff aussieht. Um die exportierte Datei Ressource (`@@file`) zu sammeln, würden Sie   `File << | | >>` verwenden.

Es gibt viele Beispiele, die exportierte Ressourcen verwenden. Die häufigste beinhaltet SSH-Host-Schlüssel. Mit exportierten Ressourcen ist es möglich, dass jede Maschine, auf der Puppet ausgeführt wird, ihre SSH-Host-Schlüssel mit den anderen verbundenen Knoten teilen. Die Idee hier ist, dass jede Maschine einen eigenen Hostschlüssel exportiert und dann alle Schlüssel von den anderen Maschinen sammelt. In unserem Beispiel erstellen wir zwei Klassen; Zuerst eine Klasse, die den SSH-Hostschlüssel von jedem Node aus exportiert. Wir werden diese Klasse in unsere Basisklasse aufnehmen. Die zweite Klasse wird eine Sammlklasse sein, die die SSH-Host-Schlüssel sammelt. Wir werden diese Klasse auf unsere Jumpboxes oder SSH Login Server anwenden.


## Hinweis

Jumpboxen sind Maschinen, die spezielle Firewall-Regeln haben, damit sie sich an verschiedenen Standorten anmelden können.

## Fertig werden

Um exportierte Ressourcen zu verwenden, musst du Storeconfigs auf deinen Puppet-Mastern aktivieren. Es ist möglich, exportierte Ressourcen mit einer masterless (dezentralen) Bereitstellung zu verwenden. 
Wir gehen jedoch davon aus, dass Sie für dieses Beispiel ein zentrales Modell verwenden.Beim Thema [Puppet4 Infrastruktur](puppet4-infrastruktur.md), haben wir puppetdb mit dem puppetdb Modul aus der forge Quelle konfiguriert. 
Es ist möglich, andere Backends zu benutzen, wenn Sie es wünschen; Aber alle außer puppetdb sind veraltet. Weitere Informationen finden Sie unter folgendem Link: [Using_Stored_Configuration](htt../puppet//projects.puppetlabs.c../puppet/projec../puppet/pupp../puppet/wi../puppet/Using_Stored_Configuration).

Stellen Sie sicher, dass Ihre Puppet-Master so konfiguriert sind, dass sie puppetdb als storeconfigs-Container verwenden.

## Wie es geht

Wir erstellen eine `ssh_host` Klasse, um die `ssh` Schlüssel eines Hosts zu exportieren und sicherzustellen, dass es in unserer Basisklasse enthalten ist.

1.Erstellen Sie die erste Klasse, `base::ssh_host`, die wir in unsere Basisklasse aufnehmen werden:

```ruby
class base::ssh_host {
  @@sshkey{"$::fqdn":
    ensure       => 'present',
    host_aliases => ["$==hostname","$==ipaddress"],
    key          => $::sshdsakey,
    type         => 'dsa',
  }
}
```

2.Denken Sie daran, diese Klasse in die Basisklassendefinition aufzunehmen:

```ruby
class base {
  ...
  include ssh_host
}
```

3.Erstellen Sie eine Definition für `Jumpbox`, entweder in einer Klasse oder innerhalb der Node definition für `Jumpbox`:

```ruby
node 'jumpbox' {
  Sshkey <<| |>>
}
```

4.Führen Sie nun Puppet auf ein paar Nodes aus, um die exportierten Ressourcen zu erstellen. In meinem Fall lief Puppet auf meinem Puppet-Server und meinem zweiten Beispielknoten (`node2`). Schließlich lief Puppet auf `jumpbox`, um zu überprüfen, dass die SSH-Host-Tasten für unsere anderen Nodes gesammelt wurden:

```s
[root@jumpbox ~]# puppet agent -t
Info: Caching catalog for jumpbox.example.com
Info: Applying configuration version '1413176635'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[jumpbo../puppet/Sshkey[node2.example.co../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[jumpbo../puppet/Sshkey[puppe../puppet/ensure: created
Notice: Finished catalog run in 0.08 seconds
```

## Wie es funktioniert

Wir haben eine `sshkey` Ressource für den Knoten mit den facter facts `fqdn`, `hostname`, `ipaddress` und `sshdsakey` erstellt. 
Wir verwenden den `fqdn` als den Titel für unsere exportierte Ressource, da jede exportierte Ressource einen eindeutigen Namen haben muss. Wir können davon ausgehen, dass der Node eines Knotens in unserer Organisation einzigartig sein wird (obwohl es manchmal auch nicht der Fall ist, wenn man es am wenigsten erwartet). Wir definieren dann Aliase, mit denen unser Node bekannt sein kann. 
Wir verwenden die Hostname-Variable für einen Alias ​​und die Haupt-IP-Adresse der Maschine als die andere. Wenn du andere Namenskonventionen für deine Knoten hättest, könntest du hier auch andere Aliase anschließen. Wir gehen davon aus, dass Hosts DSA-Schlüssel verwenden, also verwenden wir die Variable sshdsakey in unserer Definition. 
In einer großen Installation würden Sie diese Definition in Tests verpacken, um sicherzustellen, dass die DSA-Schlüssel existierten. Sie würden auch die RSA-Schlüssel verwenden, wenn sie auch existierten.

Mit hilfe der `sshkey` Ressource defination und des exportes , haben wir dann eine `Jumpbox` Node definition erstellt. In dieser Definition haben wir die Raumschiff-Syntax verwendet(spaceship syntax) `Sshkey << | | >>`, um alle definierten exportierten `sshkey` Ressourcen zu sammeln.

## Es gibt mehr

Wenn Sie die exportierten Ressourcen definieren, können Sie der Ressource Tag-Attribute hinzufügen, um Untermengen von exportierten Ressourcen zu erstellen. Wenn Sie zum Beispiel einen Entwicklungs- und Produktionsbereich Ihres Netzwerks hatten, können Sie für jeden Bereich verschiedene Gruppen von sshkey-Ressourcen erstellen, wie im folgenden Code-Snippet gezeigt:

```ruby
@@sshkey{"$::fqdn":
    host_aliases => ["$==hostname","$==ipaddress"],
    key          => $::sshdsakey,
    type         => 'dsa',
    tag          => "$::environment",
  }
```

Sie können dann `jumpbox` ändern, um nur Ressourcen für die Produktion zu sammeln, wie zum Beispiel:
`Sshkey <<| tag == 'production' |>>`

Zwei wichtige Dinge zur erinnerung, wenn Sie mit exportierten Ressourcen Arbeiten:
Erstens muss jede Ressource einen eindeutigen Namen über Ihre gesamte Installation hinweg haben.
Mit dem `fqdn` Domain-Namen innerhalb des Titels ist in der Regel schonn genug, um Ihre Definitionen einzigartig zu halten.
Zweitens kann jede Ressource virtuell gemacht werden. Auch definierte Typen, die Sie erstellt haben, können exportiert werden.
Exportierte Ressourcen können verwendet werden, um einige ziemlich komplexe Konfigurationen zu erreichen, die sich automatisch anpassen, wenn Maschinen sich ändern.

### Hinweis zu Nodes

Ein Wort der Vorsicht bei der Arbeit mit einer extrem großen Anzahl von Nodes (mehr als 5.000), exportierte Ressourcen können sich eine lange Zeit zu sammeln und gültig sein, vor allem, wenn jede exportierte Ressource  eine Datei erstellt.