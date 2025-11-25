---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet4 if und Reguläre ausdrücke

Eine andere Art von Ausdruck, den Sie testen können, `if` Aussagen und andere Bedingungen der reguläre Ausdruck sind.
Ein regelmäßiger Ausdruck ist eine leistungsfähige Möglichkeit, Strings mit Muster zu vergleichen.

## Pracktische umsetzung

Dies ist ein Beispiel für die Verwendung eines regulären Ausdrucks in einer bedingten Anweisung. Dazu fügen folgendes unserer manifests Datei hinzu:

```sh
if $::architecture =../puppet/../puppet/ {
  notify { '64Bit OS Installed': }
} else {
  notify { 'Upgrade to 64Bit': }
  fail('Not 64 Bit')
}
```

## Wie es Funktioniert

Die Puppet behandelt den Text, der zwischen den Schrägstrichen als regulärer Ausdruck geliefert wird, und spezifiziert den dazu übereinstimmenden Text.
Wenn der Ausdruck passt wird die `if` Anweisung erfolgreich und der Code zwischen den ersten zwei geschweiften Klammern abgearbeitet andernfalls wird der Code zwischen den zweiten geschweiften Klammern abgearbeitet.
Bei diesem Beispiel haben wir einen regulären Ausdruck benutzt um unterschiedliche Linux Distributionen und die unterschiedlichen Bezeichnungen zur 64bit Unterstützung abzufragen die sich je nach Distribution `amd64` oder `x86_64` nennen.

Wenn wir möchten das etwas ausgeführt wird, wenn etwas nicht passt, benutzt man `!~` statt `=~`
`~` ist hier ein Parameter der recht häufig zu Verwirrung führt `!~` bedeutet "nicht ungefähr" und `=~` in etwa "gleich ungefähr" was etwas uneindeutig ist und zu nicht vorhersagbaren Ergebnissen führen kann.

Ein der wenigen Praktischen einsetze zur Administration für `=~`  ist z.B Wir suchen einen Benutzer den Namen kennen wir nur ungefähr und möchten alle alternativen Namen angezeigt bekommen, was man hiermit erreicht würde aber kein definierter zustand , dass würde etwas Intelligents an dieser stelle benötigen was theoretisch möglich ist aber viel zu aufwendig ist im Operativen Betrieb.

## Erfassen von Mustern

Sie können nicht nur Text mit einem regulären Ausdruck anpassen, sondern auch den übereinstimmenden Text erfassen und in einer Variablen speichern:

```sh
$input = 'Puppet is better than manual configuration'
if $input =../puppet/(.*) is better than (.../puppet/ {
  notify { "You said '${0}'. Looks like you're comparing ${1}
    to ${2}!": }
}
```

Der vorangehende Code erzeugt diese Ausgabe:

> You said 'Puppet is better than manual configuration'.
> Looks like you're comparing Puppet to manual configuration!

Die Variable `$0` speichert den gesamten übereinstimmenden Text (vorausgesetzt, der Gesamtausdruck war erfolgreich).
Wenn Sie Klammern um irgendeinen Teil des regulären Ausdrucks setzen, schafft es eine Gruppe, und alle zusammenpassenden Gruppen werden auch in Variablen gespeichert.
Die erste abgestimmte Gruppe wird `$1` gespeichert , die zweite `$2`, und so weiter, wie im vorigen Beispiel gezeigt.

Als Basis dient die Ruby Syntax mit allen Besonderheiten von Ruby !