---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 basics future parser

Die Puppet Sprache entwickelt sich im Moment;
Viele Features, die voraussichtlich in die nächste Hauptversion (4 oder auch 5) aufgenommen werden, sind verfügbar, wenn Sie den futureParser aktivieren.

System bereit machen
* Stellen Sie Sicher , dass `rgen` gem installiert ist.
* Setzen Sie `parser= fure` in dem `[main]` Abschnitt ihrer `puppet.conf`.
* Für temporäre test mit dem future parse  kann man das Argument`--parser-future` benutzen.

Viele der experimentellen Merkmale behandeln, wie Code ausgewertet wird, zum Beispiel in einem früheren Beispiel verglichen wir den Wert der `$::facterversion` fact mit einer Zahl, aber der Wert wird als String behandelt, so dass der Code nicht kompiliert wird.
Mit dem zukünftigen Parser wird der Wert konvertiert und es wird kein Fehler gemeldet, wie in der folgenden Befehlszeilenausgabe gezeigt:

```
#../puppet/.pupp../puppet/manifests$ puppet apply --parser=future version.pp
Notice: Compiled catalog for cookbook.example.com in environment production in 0.36 seconds
Notice: Finished catalog run in 0.03 seconds
```

## Anhängen und Verknüpfen von Arrays in Puppet 4

Sie können Arrays mit dem `+` Operator verketten oder sie mit dem `<<` Operator anhängen.
Im folgenden Beispiel verwenden wir den ternären Operator, um der Variablen `$apache` einen bestimmten Paketnamen zuzuordnen.
Dann fügen wir diesen Wert zu einem Array mit dem `<<` Betreiber hinzu:

```ruby
$apache = $::osfamily ? {
  'Debian' => 'apache2',
  'RedHat' => 'httpd'
}
$packages = ['memcached'] << $apache
package {$packages: ensure => installed}
```

Wenn wir zwei Arrays haben, können wir den `+` Operator verwenden, um die beiden Arrays zu verketten.
In diesem Beispiel definieren wir ein Array von Systemadministratoren `$sysadmins` und ein anderes Array von Applikationseigentümern `$appowner`.
Wir können dann das Array verketten und es als Argument für unsere erlaubten Benutzer verwenden:

```ruby
$sysadmins = [ 'thomas','john','josko' ]
$appowners = [ 'mike', 'patty', 'erin' ]
$users = $sysadmins + $appowners
notice ($users)
```

Wenn wir dieses Manifest anwenden, sehen wir, dass die beiden Arrays wie in der folgenden Befehlszeilenausgabe gezeigt wurden:

```s
#../puppet/.pupp../puppet/manifests$ puppet apply --parser=future concat.pp Notice: [thomas, john, josko, mike, patty, erin]
Notice: Compiled catalog for cookbook.example.com in environment production in 0.36 seconds
Notice: Finished catalog run in 0.03 seconds
Merging Hashes
```

Wenn wir zwei Hashes haben, können wir sie mit demselben `+` Operator zusammenführen, den wir für Arrays verwendet haben.
Betrachten wir unseren `$interfaces` Hash aus einem früheren Beispiel; Wir können dem Hash eine weitere Schnittstelle hinzufügen:

```ruby
$iface = {
  'name' => 'eth0',
  'ip'   => '192.168.4.1',
  'mac'  => '52:54:00:4a:60:07'
}  + {'route' => '192.168.4.254'}
notice ($iface)
```

Wenn wir dieses Manifest anwenden, sehen wir, dass das route attribut in den Hash verschmolzen ist (Ihre Ergebnisse können sich unterscheiden, die Reihenfolge, in der die Hash ausgabe sind  unvorhersehbar), wie folgt:

```s
#../puppet/.pupp../puppet/manifests$ puppet apply --parser=future hash2.pp
Notice: {route => 192.168.4.254, name => eth0, ip => 192.168.4.1, mac => 52:55:00:4a:60:07}
Notice: Compiled catalog for cookbook.example.com in environment production in 0.36 seconds
Notice: Finished catalog run in 0.03 seconds

```

### Lambada Funktionen

Lambda-Funktionen sind Iteratoren, die auf Arrays oder Hashes angewendet werden.
Sie iterieren durch das Array oder Hash und Sie wenden eine Iterator-Funktion wie `each`, `map`, `filter`, `reduce` oder `slice`, für jedes Element des Arrays oder Schlüssel des Hashs an.
Einige der Lambda-Funktionen geben ein berechnetes Array oder Wert zurück.
Andere wie `each` gibt nur das Array oder den Hash zurück mit jedem wert der drin ist.

Lambda-Funktionen wie `map` und `reduce` Verwendung temporäre Variablen, die weggeworfen werden, nachdem die Lambda beendet ist. Die Verwendung von Lambda-Funktionen ist am besten am Beispiel. In den nächsten Abschnitten zeigen wir eine Beispielnutzung der Lambda-Funktionen.

### Iterator-Funktion Reduce

`reduce` wird verwendet, um das Array auf einen einzigen Wert zu reduzieren. Dies kann verwendet werden, um das Maximum oder Minimum des Arrays zu berechnen, oder in diesem Fall die Summe der Elemente des Arrays:

```ruby
$count = [1,2,3,4,5]
$sum = reduce($count) | $total, $i | { $total + $i }
notice("Sum is $sum")
```

Dieser vorangehende Code berechnet die Summe des `$count` Arrays und speichert sie in der Variable `$sum` wie folgt:

```s
#../puppet/.pupp../puppet/manifests$ puppet apply --parser future lambda.pp
Notice: Sum is 15
Notice: Compiled catalog for cookbook.example.com in environment production in 0.36 seconds
Notice: Finished catalog run in 0.03 seconds

```

### Iterator-Funktion Filter

Filter wird verwendet, um das Array oder Hash basierend auf einem Test innerhalb der Lambda-Funktion zu filtern.
Zum Beispiel, um unser `$count` Array wie folgt zu filtern:

`$filter = filter ($count) | $i | { $i > 3 } notice("Filtered array is $filter")`

Wenn wir dieses Manifest anwenden, sehen wir, dass nur die Elemente 4 und 5 im Ergebnis sind:

`Notice: Filtered array is [4, 5]`

#### Iterator-Funktion Map
`map` wird verwendet, um eine Funktion auf jedes Element des Arrays anzuwenden.
Zum Beispiel, wenn wir (aus irgendeinem unbekannten Grund) das Quadrat aller Elemente des Arrays berechnen wollten, würden wir die Karte wie folgt verwenden:
`$map = map ($count) | $i | { $i * $i } notice("Square of array is $map")`

Das Ergebnis der Anwendung dieses Manifests ist ein neues Array mit jedem Element des ursprünglichen Arrays quadriert (multipliziert mit sich selbst), wie in der folgenden Befehlszeilenausgabe gezeigt:
`Notice: Square of array is [1, 4, 9, 16, 25]`

#### Iterator-Funktion Slice
Slice ist nützlich, wenn Sie verwandte Werte im selben Array in einer sequentiellen Reihenfolge gespeichert haben.
Zum Beispiel, wenn wir die Ziel- und Portinformationen für eine Firewall in einem Array hatten, konnten wir sie in Paare aufteilen und Operationen auf diesen Paaren durchführen:
```
$firewall_rules = ['192.168.0.1','80','192.168.0.10','443']
slice ($firewall_rules,2) |$ip, $port| { notice("Allow $ip on $port") }
```

Wenn es angewendet wird, wird dieses Manifest die folgenden Hinweise hervorbringen:
```
Notice: Allow 192.168.0.1 on 80
Notice: Allow 192.168.0.10 on 443
```

Um dies zu einem nützlichen Beispiel zu machen, erstellen Sie eine neue Firewall-Ressource im Block des Slice anstelle von notice:
```
slice ($firewall_rules,2) |$ip, $port| { firewall {"$port from $ip": dport  => $port, source => "$ip", action => 'accept', }
}
```

#### Iterator-Funktion each
`each` wird verwendet, um über die Elemente des Arrays zu iterieren, aber es fehlt die Fähigkeit, die Ergebnisse wie die anderen Funktionen zu erfassen.
`each` ist der einfachste Fall, wo man einfach etwas mit jedem Element des Arrays machen möchte, wie im folgenden Code-Snippet gezeigt:
`each ($count) |$c| { notice($c) }`

Wie erwartet, führt dies die Benachrichtigung für jedes Element des `$count` Arrays wie folgt aus:
```
Notice: 1
Notice: 2
Notice: 3
Notice: 4
Notice: 5
```

## Andere Futures
Bei Puppet gibt es immer wieder neue futures die man unter [experiments_overview](http../puppet//docs.puppet.c../puppet/pupp../puppet/4../puppet/experiments_overview.html) finden kann.
