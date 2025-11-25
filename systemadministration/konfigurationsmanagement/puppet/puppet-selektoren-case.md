---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet-selektoren-case

Sie irgendeine 'if' bedingte Aussage schreiben, Puppet bietet aber ein paar zusätzliche ausdrücke, um Ihnen zu helfen, Bedingungen leichter auszudrücken: die `selector` und die `case` Anweisung.

Hier einige Beispiele der `selector` und `case` Anweisungen:

1.Schreiben Sie folgenden Inhalt in ihre manifest Datei:

```pp
$systemtype = $::operatingsystem ? {
  'Ubuntu' => 'debianlike',
  'Debian' => 'debianlike',
  'RedHat' => 'redhatlike',
  'Fedora' => 'redhatlike',
  'CentOS' => 'redhatlike',
  default  => 'unknown',
}

notify { "You have a ${systemtype} system": }

```

2.und den Folgenden:

```pp
class debianlike {
  notify { 'Special manifest for Debian-like systems': }
}

class redhatlike {
  notify { 'Special manifest for RedHat-like systems': }
}

case $::operatingsystem {
  'Ubuntu',
  'Debian': {
    include debianlike
  }
  'RedHat',
  'Fedora',
  'CentOS',
  'Springdale': {
    include redhatlike
  }
  default: {
    notify { "I don't know what kind of system you have!":
    }
  }
}
```

## Wie es funktioniert

### Selector

Im ersten Beispiel haben wir einen Selektor (der `?` Operator) verwendet, um einen Wert für die Variable `$ systemtype` abhängig vom Wert von `$::operatingsystem` auszuwählen.
Dies ähnelt dem ternären Operator in C oder Ruby, aber anstatt zwischen zwei möglichen Werten zu wählen, kannst du so viele Werte haben wie du willst.

Puppet vergleicht den Wert von `$::operatingsystem` mit jedem der möglichen Werte, die wir in Ubuntu, Debian und so weiter geliefert haben.
Diese Werte könnten reguläre Ausdrücke sein (z. B. für eine Teilstring-Übereinstimmung oder zur Verwendung von Wildcards), aber in unserem Fall haben wir nur litarale Strings (Buchstaben genauer UTF-8 ) verwendet.

Sobald es eine Übereinstimmung findet, gibt der selector-Ausdruck den Wert zurück, der mit dem übereinstimmenden String verknüpft ist. Wenn der Wert von `$::operatingsystem` Fedora ist, wird der selector beispielsweise den `redhatlike` String zurückgeben und dieser wird der Variablen `$systemtype` zugewiesen.

### Die case Anweisung

Im Gegensatz zu selektoren gibt die `case` keinen Wert zurück.
`case` Anweisungen kommen praktisch zur Anwendung, wenn Sie verschiedenen Code abhängig von dem Wert eines Ausdrucks ausführen sollen.

Also eine form `if` und `ifelse` die aber viel kürce sein kann.

In unserem zweiten Beispiel haben wir die `case` Anweisung verwendet, um entweder die `debianlike` oder `redhatlike` Klasse zu incudiren, abhängig vom Wert von `$::operatingsystem`.

Wieder vergleicht Puppet den Wert von `$::operatingsystem` mit einer Liste von möglichen Übereinstimmungen. Diese könnten reguläre Ausdrücke oder Strings sein, oder wie in unserem Beispiel kommagetrennte Listen von Strings.
Wenn es eine Übereinstimmung findet, wird der zugehörige Code zwischen den geschweiften Klammern ausgeführt. Also, wenn der Wert von `$::operatingsystem` Ubuntu ist, dann wird der Code einschließlich `debianlike` ausgeführt werden.

Sobald Sie die grundlegende Verwendung von Selektoren und `case` Anweisungen in den Griff bekommen haben, können die folgenden Tipps hilfreich finden.

### Reguläre ausdrücke

Wie bei `if` Anweisungen können Sie reguläre Ausdrücke mit Selektoren und `case` Anweisungen verwenden, und Sie können auch die Werte der übereinstimmenden Gruppen erfassen und auf sie mit `$1`, `$2` und so weiter verweisen:

```pp
case $::lsbdistdescription {
../puppet/Ubuntu (.../puppet/: {
    notify { "You have Ubuntu version ${1}": }
  }
../puppet/CentOS (.../puppet/: {
    notify { "You have CentOS version ${1}": }
  }
  default: {}
}
```

## Defaults

Sowohl Selektoren als auch `case` Anweisungen können Sie einen Default Wert angeben, der gewählt wird, wenn keine der anderen Optionen übereinstimmt (die Style-Anleitung schlägt vor, dass Sie immer eine Default-Klausel definiert haben):

```pp
$lunch = 'Filet mignon.'
$lunchtype =  $lunch ? {
../puppet/fri../puppet/ => 'unhealthy',
../puppet/sal../puppet/ => 'healthy',
  default => 'unknown',
}

notify { "Your lunch was ${lunchtype}": }
```

Die Ausgabe ist folgende:

```s
# ~ $ puppet apply lunchtype.pp
Notice: Your lunch was unknown
Notice../puppet/Stage[mai../puppet/Ma../puppet/Notify[Your lunch was unknow../puppet/message: defined 'message' as 'Your lunch was unknown'
```

Wenn die Standardaktion normalerweise nicht auftritt, verwenden Sie die Funktion `fail()`, um den Puppet run zu stoppen.
