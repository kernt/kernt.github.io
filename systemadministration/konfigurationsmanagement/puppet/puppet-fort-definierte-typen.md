---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschrittene:  definierte Typen

Im vorherigen Beispiel haben wir gesehen, wie man redundanten Code durch die Gruppierung identischer Ressourcen in Arrays reduziert.
Allerdings ist diese Technik auf Ressourcen beschränkt, bei denen alle Parameter gleich sind.
Wenn Sie eine Reihe von Ressourcen haben, die einige Parameter gemeinsam haben, müssen Sie einen definierten Typ verwenden, um sie zusammen zu gruppieren.

## Wie es geht

Die folgenden Schritte zeigen Ihnen, wie Sie eine Definition erstellen:

1.Fügen Sie dem Manifest den folgenden Code hinzu:

```ruby
  define tmpfile() {
    file {../puppet/t../puppet/${name}": content => "Hello, world\n",
    }
  }
  tmpfile { ['a', 'b', 'c']: }
```

2.Run Puppet

```s
[root@hiera-test ~]# vim tmp.pp
[root@hiera-test ~]# puppet apply tmp.pp
Notice: Compiled catalog for hiera-test.example.com in environment production in 0.11 seconds
Notice../puppet/Stage[mai../puppet/Ma../puppet/Tmpfile[../puppet/Fil../puppet/t../puppet/../puppet/ensure: defined content as '{md5}a5666bf58e23583c9a5a4059383ff850'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Tmpfile[../puppet/Fil../puppet/t../puppet/../puppet/ensure: defined content as '{md5}a5666bf58e23583c9a5a4059383ff850'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Tmpfile[../puppet/Fil../puppet/t../puppet/../puppet/ensure: defined content as '{md5}a5666bf58e23583c9a5a4059383ff850'
Notice: Finished catalog run in 0.09 seconds
[root@hiera-test ~]# ca../puppet/t../puppet/{a,b,c}
Hello, world
Hello, world
Hello, world
```

### Wie es funktioniert…

Sie können einen definierten Typ (mit dem Schlüsselwort `define`) als Cookie-Cutter bekannt machen.
Es beschreibt ein Muster, das Puppet verwenden kann, um viele ähnliche Ressourcen zu erstellen.
Jedes Mal, wenn du eine `tmpfile` Instanz in deinem Manifest deklarierst, wird Puppet alle in der `tmpfile` Definition enthaltenen Ressourcen einfügen.

In unserem Beispiel enthält die Definition von `tmpfile` eine einzige `file` Ressource, deren Inhalt ist `Hallo, Welt\n` und dessen Pfad ist../puppet/t../puppet/${name}`.
 Wenn du eine Instanz von `tmpfile` mit dem Namen `foo` deklariert hast:

`tmpfile { 'foo': }`

Puppet erstellt eine Datei mit dem Pfad../puppet/t../puppet/foo`. Mit anderen Worten, `${name}` in der Definition wird durch `name` einer beliebigen Instanz ersetzt, die Puppet erstellt hat.
Es ist fast so, als hätten wir eine neue Art von Ressource geschaffen: `tmpfile`, die einen Parameter hat - seinen Namen.

Genau wie bei regelmäßigen Ressourcen müssen wir nicht nur einen Titel parsen.
Wie im vorigen Beispiel können wir eine Reihe von Titeln erstellen und Puppet wird so viele Ressourcen wie erforderlich anlegen.

### Tip

Ein Wort bei `name`, das `namevar`: Jede Ressource, die du erstellt hast, muss einen eindeutigen Namen haben, den `namenvar`. Dies ist anders als der Titel, wie sich Puppet auf die Ressource intern bezieht (obwohl sie oft gleich sind).

## Es gibt mehr

Im Beispiel haben wir eine Definition erstellt, bei der der einzige Parameter, der zwischen den Instanzen variiert, der `name` Parameter ist.
Aber wir können alle Parameter hinzufügen, die wir wollen, solange wir sie in der Definition in Klammern nach dem Namensparameter wie folgt deklarieren:

```ruby
  define tmpfile($greeting) {
    file {../puppet/t../puppet/${name}": content => $greeting,
    }
  }
```

Als nächstes übergeben Sie Werte, wenn wir eine Instanz der Ressource deklarieren:

```ruby
  tmpfile{ 'foo':
    greeting => "Good Morning\n",
  }
```

Sie können mehrere Parameter als kommagetrennte Liste deklarieren:

```ruby
  define webapp($domain,$path,$platform) {
    ...
  }
  webapp { 'mywizzoapp':
    domain   => 'mywizzoapp.com',
    path     =>../puppet/v../puppet/w../puppet/ap../puppet/mywizzoapp',
    platform => 'Rails',
  }
```

Sie können auch Standardwerte für beliebige Parameter angeben, die nicht mitgeliefert werden, so dass sie optional sind:

```ruby
  define tmpfile($greeting,$mode='0644') {
    ...
  }
```

Dies ist eine mächtige Technik, um alles, was den Ressourcen gemeinsam ist, zu abstrahieren und es an einem Ort zu halten, damit du dich nicht wiederholen kannst.
Im vorigen Beispiel gibt es viele individuelle Ressourcen, die in `webapp` enthalten sind: Pakete, Konfigurationsdateien, Quellcode-Checkouts, virtuelle Hosts und so weiter.
Aber alle von ihnen sind die gleichen für jede Instanz von `webapp` außer den Parametern, die wir zur Verfügung stellen. Diese können in einer Vorlage referenziert werden, um beispielsweise die Domäne für einen virtuellen Host festzulegen.