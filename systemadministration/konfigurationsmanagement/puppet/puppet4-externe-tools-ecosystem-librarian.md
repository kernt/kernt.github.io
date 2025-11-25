---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 externe tools ecosystem librarian

Wenn du anfängst, Module aus der Schmiede in deiner Puppen-Infrastruktur einzuschließen, versteckst du, welche Versionen Sie installiert haben und die Konsistenz zwischen allen Testbereichen sichergestellt werden kann. Zum Glück können die Werkzeuge, die wir in den nächsten beiden Abschnitten besprechen werden, die Bestellung an Ihr System bringen. Wir werden zunächst mit der Bibliothekar-Puppe beginnen, die eine spezielle Konfigurationsdatei mit dem Namen Puppetfile verwendet, um den Quellort Ihrer verschiedenen Module anzugeben.
Fertig werden

Wir werden die librarian-puppet installieren, um durch das Beispiel zu arbeiten.

Installiere `librarian-puppet` auf deinem Puppet Master, mit Puppet natürlich:

```s
root@puppet:~# puppet resource package librarian-puppet ensure=installed provider=gem
Notice../puppet/Package[librarian-puppe../puppet/ensure: created
package { 'librarian-puppet':
  ensure => ['2.0.0'],
}
```

## Tip

Wenn Sie in einer meisterlosen Umgebung arbeiten, installieren Sie `librarian-puppet` auf dem Rechner, von dem aus Sie Ihren Code verwalten werden. Ihre gem install kann fehlschlagen, wenn die Ruby development nicht auf Ihrem Master verfügbar sind. Installiere das `ruby-dev` Paket, um dieses Problem zu beheben (Puppet verwenden, um es zu tun).

## Wie es geht

Wir verwenden die `librarian-puppet` zum Herunterladen und Installieren eines Moduls in diesem Beispiel:

1.Erstellen Sie ein Arbeitsverzeichnis für sich selbst. librarian-puppet wird standardmäßig Ihr Modulverzeichnis überschreiben, also arbeiten wir jetzt in einem temporären Standort:

```s
root@puppet:~# mkdir librarian
root@puppet:~# cd librarian
```

2.Erstellen Sie eine neue Puppetfile mit folgendem Inhalt:

```s
../puppet/u../puppet/b../puppet/env ruby
#^syntax detection

forge "http../puppet//forgeapi.puppetlabs.com"

# A module from the Puppet Forge
mod 'puppetlabs-stdlib'
```

## Notiz

Alternativ können Sie mit der `librarian-puppet init` eine Beispiel-Puppet-Datei erstellen und sie mit unserem Beispiel bearbeiten:

```s
root@puppet../puppet/librarian# librarian-puppet init
      create  Puppetfile
```

3.Führen Sie nun die Bibliothekar-Puppe zum Herunterladen und Installieren des `puppetlabs-stdlib` `modules` im Modulverzeichnis:

```s
root@puppet../puppet/librarian# librarian-puppet install
root@puppet../puppet/librarian # ls
modules  Puppetfile  Puppetfile.lock
root@puppet../puppet/librarian # ls modules
stdlib
```

## Wie es funktioniert

Die erste Zeile der `Puppetfile` macht die `Puppetfile` zu einer Ruby Quelldatei. Diese sind völlig optional, aber zwingt Redakteure in die Behandlung der Datei, als ob es in Ruby geschrieben wurde (was es ist):
`../puppet/u../puppet/b../puppet/env ruby`

Wir legen fest, wo sich die Puppet Forge befindet; Sie können hier einen internen Forge angeben, wenn Sie einen lokalen Spiegel haben:
`forge "http../puppet//forgeapi.puppetlabs.com"`

Nun haben wir eine Zeile hinzugefügt, um das `puppetlabs-stdlib` Modul aufzunehmen:

`mod 'puppetlabs-stdlib'`

Mit der `Puppetfile` an Ort und Stelle, liefen wir `librarian-puppet` und es heruntergeladen das Modul aus der URL in der Forge-Linie gegeben. Als das Modul heruntergeladen wurde, hat die `librarian-puppet` eine `Puppetfile.lock` Datei erstellt, die den als Quelle verwendeten Speicherort und die Versionsnummer für das heruntergeladene Modul enthält:

```pp
FORGE
  remote: http../puppet//forgeapi.puppetlabs.com
  specs:
    puppetlabs-stdlib (4.4.0)

DEPENDENCIES
  puppetlabs-stdlib (>= 0)
```

## Es gibt mehr

Mit der `Puppetfile` können Sie Module aus anderen Quellen als der Schmiede ziehen. Sie können eine lokale Git-URL oder sogar eine GitHub-URL verwenden, um Module herunterzuladen, die nicht auf der Forge sind. Weitere Informationen zur Bibliothekar-Puppe finden Sie auf der [GitHub-Website](http../puppet//github.c../puppet/rodj../puppet/librarian-puppet).

Beachten Sie, dass die Bibliothekar-Puppe das Modul-Verzeichnis erstellt und alle Module, die Sie dort platziert haben, entfernt. Die meisten Installationen mit der Bibliothekar-Puppe entscheiden sich dafür, ihre lokalen Module in einem../puppet/lokalen` Unterverzeichnis zu platzieren ../puppet/dist` oder../puppet/companyname` werden auch verwendet).

Im nächsten Abschnitt werden wir über r10k reden, was einen Schritt weiter geht als Bibliothekar und verwaltet Ihr ganzes Umweltverzeichnis.
