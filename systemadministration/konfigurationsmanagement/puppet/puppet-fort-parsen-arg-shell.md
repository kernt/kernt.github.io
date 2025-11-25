---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet für fortgeschrittene: exec

Wenn Sie Werte in eine Befehlszeile einfügen möchten (z.B die von einer `exec` Ressource ausgeführt werden), müssen sie oft quted sein, besonders wenn sie Leerzeichen enthalten. Die `shellquote` Funktion wird eine beliebige Anzahl von Argumenten, einschließlich Arrays, und jedes Argumente queten und übergibt sie alle als eine Leerzeichen getrennte Zeichenfolge, die Sie an Befehle übergeben können weiter.

In diesem Beispiel möchten wir eine `exec` Ressource einrichten, die eine Datei umbenennent.
Aber sowohl die Quelle als auch der Zielname enthalten Leerzeichen, also müssen sie in der Kommandozeile korrekt gequted werden.

## Wie es geht

Hier ist ein Beispiel für die Verwendung der Shellquote-Funktion:

1.Erstellen Sie ein `shellquote.pp` manifest mit dem folgenden Befehl:

```ruby
$source = 'Hello Jerry'
$target = 'Hello... Newman'
$argstring = shellquote($source, $target)
$command =../puppet/b../puppet/mv ${argstring}"
notify { $command: }
```

2.Run Puppet:

```s
$ puppet apply shellquote.pp
...
Notice../puppet/b../puppet/mv "Hello Jerry" "Hello... Newman"
Notice../puppet/Stage[mai../puppet/Ma../puppet/Notif../puppet/b../puppet/mv "Hello Jerry" "Hello... Newman../puppet/message: defined 'message' as../puppet/b../puppet/mv "Hello Jerry" "Hello... Newman"'
```

## Wie es funktioniert

Zuerst definieren wir die `$source` und `$target` Variablen, die die beiden Dateinamen sind, die wir in der Befehlszeile verwenden möchten:

```ruby
$source = 'Hello Jerry'
$target = 'Hello... Newman'
```

Dann nennen wir Shellquote, um diese Variablen in eine zitierte, raumgetrennte Zeichenfolge wie folgt zu verknüpfen:

`$argstring = shellquote($source, $target)`

Dann stellen wir die letzte Kommandozeile zusammen:

`$command =../puppet/b../puppet/mv ${argstring}"`

Das Ergebnis ist:

`$command =../puppet/b../puppet/mv ${argstring}"`

Diese Befehlszeile kann nun mit einer Exec-Ressource ausgeführt werden. Was würde passieren, wenn wir `shellquote` nicht benutzt hätten?

Dies wird nicht funktionieren, weil `mv` erwartet leerspace-getrennte Argumente , also wird es dies als eine Anforderung interpretieren, um drei Dateien `Hallo`, `Jerry` und `Hallo` ... in ein Verzeichnis namens `Newman` zu verschieben, was wahrscheinlich nicht das ist, was wir wollen.