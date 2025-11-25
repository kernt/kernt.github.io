---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 basics regex substitutions

Puppet's `regsubst` Funktion bietet eine einfache Möglichkeit, Text zu manipulieren, zu suchen und zu ersetzen, oder extrahieren des Musters aus Strings. Wir müssen dies oft mit Daten aus `fatcts` ([Puppet facts](../puppet/puppet4-facts))  machen, oder aus externen Programmen.

In diesem Beispiel sehen wir, wie man `regsubst` benutzt, um die ersten drei Oktette einer IPv4-Adresse zu extrahieren (der Netzwerkteil../puppet/24`, vorausgesetzt,handelt sich um eine Adresse der Klasse C).

Es folgt ein Beispiel


1. Füge den folgenden Code in deine manist Datein ein :
```
$class_c = regsubst($::ipaddress, '(.*)\..*', '\1.0')
notify { "The network part of ${::ipaddress} is ${class_c}": }
```

2. Führe einen Puppet run aus 

```
#../puppet/.pupp../puppet/manifests$ puppet apply ipaddress.pp 
Notice: Compiled catalog for cookbook.example.com in environment production in 0.02 seconds
Notice: The network part of 192.168.122.148 is
  192.168.122.0
Notice../puppet/Stage[mai../puppet/Ma../puppet/Notify[The network part of 192.168.122.148 is
  192.168.122.../puppet/message: defined 'message' as 'The network part of 192.168.122.148 is
  192.168.122.0'
Notice: Finished catalog run in 0.03 seconds
```

### Wie es funktioniert

Die `regsubst` Funktion benötigt mindestens drei Parameter: 
Quelle(Source), Muster(pattern) und Ersatz(replace).

In unserem Beispiel haben wir den Quellstring als `$::ipaddress` angegeben, der auf dieser Maschine wie folgt lautet:
`192.168.122.148`

Wir geben einen Muster(pattern) an, wie den folgenden :
`(.*)\..*`

Wir geben den Ersatz(replace) an 
`\1.0`

Das Muster(pattern) erfasst den gesamten String bis zur letzten Periode (`\.`) In der Variable `\1`.
Wir passen dann mit `.*` an, Das alles mit dem Ende der Zeichenfolge übereinstimmt. 
Wenn wir also den String am Ende mit `\1.0` ersetzen, gelangen zum Netzwerkteil der IP-Adresse, der die folgenden Ausgabe liefert:
`192.168.122.0`

Wir könnten natürlich, das gleiche Ergebnis auf andere Weise haben, einschließlich der folgenden:
`$class_c = regsubst($::ipaddress, '\.\d+$', '.0')`

Hier passt der Ausdruck nur auf das letzte Oktett und wird es ersetzen mit `.0`, das das gleiche Ergebnis ohne aufnahme erreicht.

