---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet für fortgeschrittene: gem

Manchmal ist es sehr nützlich, einen Aspekt einer Klasse zu parametrisieren.
Zum Beispiel müssen Sie möglicherweise verschiedene Versionen eines `gem` Pakets verwalten und anstatt separate Klassen für jedes einzelne Paket zu machen, die sich nur in der Versionsnummer unterscheiden, können Sie die Versionsnummer als Parameter übergeben.

## Wie es geht…

In diesem Beispiel erstellen wir eine Definition, die Parameter akzeptiert:

1.Deklariere den Parameter als Teil der Klassendefinition:

```ruby
  class eventmachine($version) {
    package { 'eventmachine': provider => gem, ensure   => $version,
    }
  }

```

2.Verwenden Sie die folgende Syntax, um die Klasse auf einen Node aufzunehmen:

```ruby
  class { 'eventmachine':
    version => '1.0.3',
  }
```

## Wie es funktioniert…

Die Klasse Definition `class eventmachine($ version) {` ist genau wie eine normale Klasse Definition außer es gibt an, dass die Klasse einen Parameter: `$version` annimmt. Innerhalb der Klasse haben wir eine Paketressource definiert:

```ruby
  package { 'eventmachine':
    provider => gem,
    ensure   => $version,
  }
```

Dies ist ein `gem` Paket, und wir fordern, Version `$version` zu installieren.

Füge die Klasse auf einen Knoten ein, anstelle der üblichen Include-Syntax:
`include eventmachine`

## Auf diese Weise wird es eine `class` aussage geben

```ruby
  class { 'eventmachine':
    version => '1.0.3',
  }
```

Dies hat die gleiche Wirkung, setzt aber auch einen Wert für den Parameter als Version.

## Es gibt mehr

Sie können mehrere Parameter für eine Klasse angeben:

`class mysql($package, $socket, $port) {`

Dann liefern sie auf die gleiche Weise:

```ruby
  class { 'mysql':
    package => 'percona-server-server-5.5',
    socket  =>../puppet/v../puppet/r../puppet/mysq../puppet/mysqld.sock',
    port    => '3306',
  }
```

## Festlegen von Standardwerten

Sie können auch Standardwerte für einige Ihrer Parameter angeben. Wenn Sie die Klasse ohne Einstellung eines Parameters einfügen, wird der Standardwert verwendet.
Zum Beispiel, wenn wir eine `mysql` Klasse mit drei Parametern erstellt haben, könnten wir Standardwerte für alle oder alle Parameter wie im Code-Snippet angegeben:

`class mysql($package, $socket, $port='3306') {`

oder alle:

```ruby
  class mysql(
    package = percona-server-server-5.5",
    socket  =../puppet/v../puppet/r../puppet/mysq../puppet/mysqld.sock',
    port    = '3306') {
```

Mit den Standardeinstellungen können Sie einen Standardwert verwenden und diesen Standardwert überschreiben, wo Sie ihn benötigen.

Im Gegensatz zu einer Definition kann nur eine Instanz einer parametrisierten Klasse auf einem Node existieren. Also, wo Sie mehrere verschiedene Instanzen der Ressource haben müssen, verwenden Sie stattdessen `define`.