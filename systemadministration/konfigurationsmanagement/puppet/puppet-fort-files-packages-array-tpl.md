---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für Fortgeschrittene templates mit Datein und Packages

Im vorherigen Beispiel haben wir gesehen, dass man Ruby verwenden kann, um je nach Ergebnis eines Ausdrucks unterschiedliche Werte in Vorlagen zu interpolieren.
Aber du bist nicht darauf beschränkt, immer einen Wert zu bekommen. Sie können viele von ihnen in ein Puppet-Array setzen und dann die Vorlage generieren einige Inhalte für jedes Element des Arrays mit einer Schleife.

## Wie es geht

Gehen Sie folgendermaßen vor, um ein Beispiel für die Iteration über Arrays zu erstellen:

1.Ändern Sie Ihre `site.pp` Datei wie folgt:

```ruby
  node 'cookbook' {
    $ipaddresses = ['192.168.0.1', '158.43.128.1', '10.0.75.207' ]
    file {../puppet/t../puppet/addresslist.txt':
      content => template('ba../puppet/addresslist.erb')
    }
  }
```

2.Erstellen Sie die Datei `modu../puppet/ba../puppet/templat../puppet/addresslist.erb` mit folgendem Inhalt:

```ruby
<% @ipaddresses.each do |ip| -%>
IP address <%= ip %> is present
<% end -%>
```

3.Run puppet:

```s
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1412141917'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/t../puppet/addresslist.tx../puppet/ensure: defined content as '{md5}073851229d7b2843830024afb2b3902d'
Notice: Finished catalog run in 0.30 seconds

```

4.Überprüfen Sie den Inhalt der generierten Datei:

```s
[root@cookbook ~]# ca../puppet/t../puppet/addresslist.txt
  IP address 192.168.0.1 is present.
  IP address 158.43.128.1 is present.
  IP address 10.0.75.207 is present.
```

### Wie es funktioniert…

In der ersten Zeile der Vorlage verweisen wir auf die Array `ipaddresses` und rufen ihre `each` Methode auf:
`<% @ipaddresses.each do |ip| -%>`

In Ruby erzeugt dies eine Schleife, die einmal für jedes Element des Arrays ausgeführt wird. Jedes Mal um die Schleife wird die Variable `ip` auf den Wert des aktuellen Elements gesetzt.

In unserem Beispiel enthält das  `ipaddresses` Array drei Elemente, so dass die folgende Zeile dreimal ausgeführt wird, einmal für jedes Element:
`IP address <%= ip %> is present.`

Dies führt zu drei Ausgaben:

```s
IP address 192.168.0.1 is present.
IP address 158.43.128.1 is present.
IP address 10.0.75.207 is present.
```

Die letzte Zeile beendet die Schleife:

`<% end -%>`

#### Hinweis

Beachten Sie, dass die erste und die letzte Zeile mit `-%>` statt nur `%>` enden, wie wir vorher gesehen haben. Die Wirkung des `-` ist, die neue Zeile zu unterdrücken, die sonst bei jedem Durchlauf durch die Schleife erzeugt würde, was uns unerwünschte leere Zeilen in der Datei bescheren würde.

#### Es gibt mehr

Vorlagen können auch über Hashes oder Arrays von Hashes iterieren:

```ruby
$interfaces = [ {name => 'eth0', ip => '192.168.0.1'},
  {name => 'eth1', ip => '158.43.128.1'},
  {name => 'eth2', ip => '10.0.75.207'} ]

<% @interfaces.each do |interface| -%>
Interface <%= interface['name'] %> has the address <%= interface['ip'] %>.
<% end -%>

Interface eth0 has the address 192.168.0.1.
Interface eth1 has the address 158.43.128.1.
Interface eth2 has the address 10.0.75.207.
```