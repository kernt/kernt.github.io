---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet4 if 

Puppets `if` Anweisung erlaubt Ihnen, das manifest Verhalten auf der Grundlage des Wertes einer Variablen oder eines Ausdrucks zu ändern.
Damit können Sie je nach facts Daten über den Node, z. B. das Betriebssystem oder die Speichergröße, unterschiedliche Ressourcen oder Parameterwerte anwenden.

Sie können auch Variablen innerhalb des Manifests setzen, die das Verhalten der enthaltenen Klassen ändern können. 
Zum Beispiel müssen Nodes im Rechenzentrum A möglicherweise andere DNS-Server als Nodes im Rechenzentrum B verwenden, oder Sie müssen möglicherweise einen  von Klassen für ein Ubuntu-System und einen anderen Satz für andere Systeme enthalten.

## Praktische Umsetzung

Hier ein einfaches Beispiel mit einer `if` anweisung:

```sh
  if $::timezone == 'UTC' {
    notify { 'Universal Time Coordinated':}
  } else {
    notify { "$::timezone is not UTC": }
  }
```

## Wie funktioniert

Puppet behandelt, was auch immer ein `if` Schlüsselwort als Ausdruck folgt und auswertet es aus. 
Wenn der Ausdruck wahr ist, wird Puppet den Code innerhalb der geschweiften Klammern ausführen.

Optional können Sie einen anderen Zweig hinzufügen, der ausgeführt wird, wenn der Ausdruck `false` ausgewertet wird.

## Elseif zweige

Sie können weitere Tests mit dem `elseif` Schlüsselwort hinzufügen:

```sh
if $::timezone == 'UTC' {
  notify { 'Universal Time Coordinated': }
} elseif $::timezone == 'GMT' {
  notify { 'Greenwich Mean Time': }
} else {
  notify { "$::timezone is not UTC": }
}
```

## Vergleiche durchführen

Sie können zwei werten auf gleichheit prüfen wenn Sie die `==` syntax folgender maßen einsetzen:

```sh
if $::timezone == 'UTC' {
  
}
```

Alternativ können mit `<` und `>` auch numerische werte vergleichen werden :

```sh
if $::uptime_days > 365 {
  notify { 'Time to upgrade your kernel!': }
}
```

Um zu Test welcher größer oder gleich einen numerischen wert ist kann `<=` oder `>=` benutzt werden.

```sh
if $::mtu_eth0 <= 1500 {
  notify {"Not Jumbo Frames": }
}
```

## Ausdrücke Kombinieren

Sie können nun alles miteinander in einem komplexen Logischen Ausdruck indem Sie `and`, `or`, und `not` folgender maßen einsetzen:

```sh
if ($==uptime_days > 365) and ($==kernel == 'Linux') {
  …
}

if ($role == 'webserver') and ( ($datacenter == 'A') or ($datacenter == 'B') ) {
  …
}
```