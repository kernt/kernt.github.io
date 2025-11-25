---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschrittene: Packages Paketverionen

Paketversionsnummern sind seltsame Dinge. Sie sehen aus wie Dezimalzahlen, aber sie sind nicht: eine Versionsnummer ist oft in Form von `2.6.4`, zum Beispiel. Wenn du eine Versionsnummer mit einer anderen vergleichen musst, kannst du keinen direkten String-Vergleich machen: `2.6.4` wäre so interpretiert wie größer als `2.6.12`. Und ein numerischer Vergleich funktioniert nicht, weil sie keine gültigen Zahlen sind.

Puppets `versioncmp` Funktion kommt zur Rettung. Wenn du zwei Dinge passierst, die wie Versionsnummern aussehen, wird sie sie vergleichen und einen Wert zurückgeben, der größer ist:

`versioncmp( A, B )`

gibt aus:

* 0 if A und B gleich sind

* Größer als 1 if A is größer als B

* Weniger als 0 if A weniger als B ist

## Wie es geht

Hier ist ein Beispiel mit der `versioncmp` Funktion:

1. Puppet classe

```ruby
node 'cookbook' {
  $app_version = '1.2.2'
  $min_version = '1.2.10'

  if versioncmp($app_version, $min_version) >= 0 {
    notify { 'Version OK': }
  } else {
    notify { 'Upgrade needed': }
  }
}
```

2.Run Puppet:

```s
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Notice: Upgrade needed

```

3.Ändern Sie nun den Wert von `$app_version`:
`$app_version = '1.2.14'`

4. Puppet run erneut  durchführen:

```s
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Notice: Version OK
```

## Wie es funktioniert…

Wir haben angegeben, dass die mindestens akzeptable Version (`$min_version`) `1.2.10` ist. Also, im ersten Beispiel wollen wir es mit `$app_version` von `1.2.2` vergleichen. Ein einfacher alphabetischer Vergleich dieser beiden Strings (z.B in Ruby) würde das falsche Ergebnis liefern, aber `versioncmp` bestimmt korrekt, dass `1.2.2` kleiner als `1.2.10` ist und warnt uns, dass wir ein Upgrade durchführen müssen.

Im zweiten Beispiel ist `$app_version` jetzt `1.2.14`, welches `versioncmp` korrekt als größer als `$min_version` erkennt und so bekommt man die Meldung Version OK.
