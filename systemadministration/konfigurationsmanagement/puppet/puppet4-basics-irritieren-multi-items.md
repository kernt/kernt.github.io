---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 basics irritieren multi items

Arrays sind ein mächtiges Feature in Puppet.
Wo immer du die gleiche Operation auf einer Liste von Dingen ausführt werden soll, kann ein Array in der Lage sein zu helfen. 
Sie können ein Array erstellen, indem Sie einfach seinen Inhalt in eckige Klammern setzen:
```
$lunch = [ 'franks', 'beans', 'mustard' ]
```

## Wie man es anwendet in der Praxis

1. Füge den folgenden code deiner manifest Datei hinzu.
```
$packages = [ 'ruby1.8-dev',
  'ruby1.8',
  'ri1.8',
  'rdoc1.8',
  'irb1.8',
  'libreadline-ruby1.8',
  'libruby1.8',
  'libopenssl-ruby' ]

package { $packages: ensure => installed }
```

2. Führe einen Puppet run durch und prüfe , dass jedes Packt installiert ist.

Obwohl Arrays mit Puppet lange ausrechend sind und uns immmer bgleiten werden , ist es auch sinnvoll, sich über eine noch flexiblere Datenstruktur zu informieren: den Hash.

## hashes einsetzen

Ein Hash ist wie ein Array, aber jedes der Elemente kann gespeichert und nach Namen (z.B. als Schlüssel) nachgeschlagen werden, zum Beispiel erstellen wir die Datei `hash.pp` mit folgendem inhalt:
```
$interface = {
  'name' => 'eth0',
  'ip'   => '192.168.4.1',
  'mac'  => '52:5F:00:4a:60:07' 
}
notify { "(${interface['ip']}) at ${interface['mac']} on ${interface['name']}": }
```
Wenn wir den Puppet run ausführen , können wie folgende ausgabe sehen: 
```
tkern@tobkern-desktop../puppet/.pupp../puppet/manifests$ puppet apply hash.pp
Notice: (192.168.4.1) at 52:5F:00:4a:60:07 on etho
```

Hash-Werte können alles sein, was Sie Variablen, Strings, Funktionsaufrufen, Ausdrücken und sogar anderen Hashes oder Arrays zuordnen können.
Hashes sind nützlich, um eine Reihe von Informationen über eine bestimmte Sache zu speichern, weil durch den Zugriff auf jedes Element der Hash mit einem Schlüssel referenziert wird, können wir schnell die Informationen finden, die wir suchen.

## Arrays mit einer split Funktion erstellen
Sie könnet literale arrays erstellen wenn Sie eckige Klammern wie folgt benutzen:
```
define lunchprint() {
  notify { "Lunch included ${name}":}": }
}

$lunch = ['egg', 'beans', 'chips']
lunchprint { $lunch: }
```

Wenn wir jetzt Puppet auf den vorherigen Code laufen lassen, sehen wir die folgenden Meldungen in der Ausgabe:
```
#$ puppet apply lunchprint.pp
...
Notice: Lunch included chips
Notice: Lunch included beans
Notice: Lunch included egg

```

Allerdings kann Puppet auch Arrays für Sie aus Strings, mit der `split` Funktion, wie folgt erstellen:
```
$menu = 'egg beans chips'
$items = split($menu, ' ')
lunchprint { $items: }
```
Nun lassen wir wider `puppet apply` diesis neue manifest laufen, und wir sehen die Gleichen Meldungen.
```
# ~ $ puppet apply lunchprint2.pp 
...
Notice: Lunch included chips
Notice: Lunch included beans
Notice: Lunch included egg.

```
Beachten Sie, dass `split` zwei Argumente annimmt: 
Das erste Argument ist die zu spaltende Zeichenfolge. 
Das zweite Argument ist der Charakter, der sich aufspaltet; 
In diesem Beispiel ein Leerzeichen. 
Wenn Puppet sich durch den Code arbeitet, und auf ein Leerzeichen trifft, interpretiert Puppet ihn als das Ende eines Items und den Anfang des nächsten. 
Also, wird der String 'Eier Bohnen Chips', in drei Elemente aufgeteilt werden.
Ausgabe :
```
Eier
Bohnen
Chips
```

Der character zum teilen kann jeder character oder string sein :
```
$menu = 'egg and beans and chips'
$items = split($menu, ' and ')
```

Der character kann auch ein regulärer Ausdruck sein, zum Beispiel eine wird ein Set aus alternativen getrennt mit einem `|` (pipe) symbol.
```
$lunch = 'egg:beans,chips'
$items = split($lunch, ':|,')
```