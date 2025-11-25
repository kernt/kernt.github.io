---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 basics in operator

Der `in` Operator prüft, ob ein String einen anderen String enthält. Hier ist ein Beispiel:
`if 'spring' in 'springfield'`

Der vorhergehende Ausdruck ist wahr, wenn der `spring` ein Teilstring von `springfield` ist, was es hier ist ist. 
Der `in` Operator kann auch die Zugehörigkeit zu Arrays wie folgt testen:

`if $crewmember in ['Frank', 'Dave', 'HAL' ]`

Wenn `in` mit einem Hash verwendet wird, prüft es, ob die Zeichenfolge ein Schlüssel des Hash ist:

```
$ifaces = { 'lo'   => '127.0.0.1', 
            'eth0' => '192.168.4.1' }
if 'eth0' in $ifaces {
  notify { "eth0 has address ${ifaces['eth0']}": }
}
```

Die folgenden Schritte zeigen Ihnen, wie Sie den `in` Operator benutzen können:

1. Füge den folgenden Code in dein manifest ein:
```
if $::operatingsystem in [ 'Ubuntu', 'Debian' ] {
  notify { 'Debian-type operating system detected': }
} elseif $::operatingsystem in [ 'RedHat', 'Fedora', 'SuSE', 'CentOS' ] {
  notify { 'RedHat-type operating system detected': }
} else {
  notify { 'Some other operating system detected': }
}
```

2. Führe einen Puppet run aus 
```
#../puppet/.pupp../puppet/manifests$ puppet apply in.pp
Notice: Compiled catalog for cookbook.example.com in environment production in 0.03 seconds
Notice: Debian-type operating system detected
Notice../puppet/Stage[mai../puppet/Ma../puppet/Notify[Debian-type operating system detecte../puppet/message: defined 'message' as 'Debian-type operating system detected'
Notice: Finished catalog run in 0.02 seconds
```

Der Wert eines `in` Ausdrucks ist boolean (true oder false), so dass du ihn einer Variablen zuordnen kannst:
```
$debianlike = $::operatingsystem in [ 'Debian', 'Ubuntu' ]

if $debianlike {
  notify { 'You are in a maze of twisty little packages, all alike': }
}
```