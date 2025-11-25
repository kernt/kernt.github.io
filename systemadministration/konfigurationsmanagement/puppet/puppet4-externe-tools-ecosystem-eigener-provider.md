---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 externe tools ecosystem eigener provider

Im vorherigen Abschnitt haben wir eine neue benutzerdefinierte Art namens `gitrepo` und sagte Puppet, dass es zwei Parameter, `source` und `path`. Doch so weit, haben wir nicht gesagt, Puppet, wie man tatsächlich auschecken die Repo; Mit anderen Worten, wie man eine bestimmte Instanz dieses Typs erstellt. Dort kommt der Anbieter herein.

Wir haben gesehen, dass ein Typ oft mehrere mögliche Anbieter haben wird. In unserem Beispiel gibt es nur einen vernünftigen Weg, um einen Git Repo zu instanziieren, also werden wir nur einen Provider liefern: `git`. Wenn du diesen Typ verallgemeinern würdest, um es einfach zu sagen, dann ist es nicht schwer, sich vorzustellen, dass mehrere verschiedene Anbieter je nach Art des Repo, zB `git`, `svn`, `cvs`, und so weiter erstellt werden.

## Wie es geht

Wir werden den `Git`-Provider hinzufügen und eine Instanz einer `gitrepo` Ressource erstellen, um zu überprüfen, ob das alles funktioniert. Sie müssen Git installiert haben, damit dies funktioniert, aber wenn Sie das Git-basierte Manifest Management Setup verwenden, das in [Puppet4 Infrastruktur](puppet4-infrastruktur.md) beschrieben ist, können wir sicher davon ausgehen, dass Git verfügbar ist.

1.Erstellen Sie die Datei `modul../puppet/cookbo../puppet/l../puppet/pupp../puppet/provid../puppet/gitre../puppet/git.rb` mit folgendem Inhalt:

```pp
require 'fileutils'

Puppet::Type.type(:gitrepo).provide(:git) do
  commands :git => "git"

  def create
    git "clone", resource[:source], resource[:path]
  end

  def exists?
    File.directory? resource[:path]
  end
end
```

2.Ändern Sie Ihre `site.pp` Datei wie folgt:

```pp
node 'cookbook' {
  gitrepo { 'http../puppet//github.c../puppet/puppetla../puppet/puppetlabs-git':
    ensure => present,
    path   =>../puppet/t../puppet/puppet',
  }
}
```

3. Puppet run:

```pp
[root@cookbook ~]# puppet agent -t
Notice../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/l../puppet/pupp../puppet/ty../puppet/gitrepo.r../puppet/ensure: defined content as '{md5}6471793fe2b4372d40289ad4b614fe0b'
Notice../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/l../puppet/pupp../puppet/provid../puppet/gitrep../puppet/ensure: created
Notice../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/l../puppet/pupp../puppet/provid../puppet/gitre../puppet/git.r../puppet/ensure: defined content as '{md5}f860388234d3d0bdb3b3ec98bbf5115b'
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1416378876'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Gitrepo[http../puppet//github.c../puppet/puppetla../puppet/puppetlabs-gi../puppet/ensure: created
Notice: Finished catalog run in 2.59 seconds
```

Wie es funktioniert...

Benutzerdefinierte Provider können in jedem Modul, in einem `l../puppet/pupp../puppet/provid../puppet/TYPE_NAME` Unterverzeichnis in einer Datei nach dem Anbieter benannt leben. (Der Provider ist das eigentliche Programm, das auf dem System läuft, in unserem Beispiel ist das Programm Git und der Provider ist in den Modulen../puppet/cookbo../puppet/l../puppet/pupp../puppet/provid../puppet/gitre../puppet/git.rb`. Beachten Sie, dass der Name des Moduls Ist irrelevant.)

Nach einer ntitialen Pflichtlinie in `git.rb`, sagen wir Puppet, um einen neuen Provider für den `gitrepo` Typ mit der folgenden Zeile zu registrieren:
`Puppet::Type.type(:gitrepo).provide(:git) do`

Wenn du eine Instanz des `gitrepo` Typs in deinem Manifest deklarierst, wird die Puppe zunächst prüfen, ob die Instanz bereits existiert, indem sie die `exists?` Methode auf dem Anbieter aufruft . Also müssen wir diese Methode mit dem Code versorgen, um zu prüfen, ob eine Instanz des `gitrepo` Typs bereits vorhanden ist:

```pp
def exists?
  File.directory? resource[:path]
end
```

Dies ist nicht die anspruchsvollste Umsetzung; Es gibt einfach `true` zurück, wenn ein Verzeichnis existiert, das dem `path` parameter der Instanz entspricht. Eine bessere Umsetzung `exists?` Könnte z. B. prüfen, ob es ein `.git` Unterverzeichnis gibt und dass es gültige Git-Metadaten enthält. Aber das wird jetzt tun.

Wenn `exists?` Kehrt `true`, dann wird die Puppe keine weiteren Maßnahmen ergreifen, weil die angegebene Ressource existiert (soweit Puppet weiß). Wenn es `false` zurückgibt, geht Puppet davon aus, dass die Ressource noch nicht existiert und versucht, sie zu erstellen, indem sie die `create` Methode des Anbieters aufruft.

Dementsprechend liefern wir einen Code für die `create` Methode, die den Befehl `git clone` aufruft, um das Repo zu erstellen:

```pp
def create
  git "clone", resource[:source], resource[:path]
end
```

Die Methode hat Zugriff auf die Parameter der Instanz, die wir wissen müssen, wo wir das Repo ausprobieren können und welches Verzeichnis es in erstellen soll. Wir erhalten dies durch Betrachten von `ressource[:source]` und `resource[:path]`.

## Es gibt mehr

Sie können sehen, dass benutzerdefinierte Typen und Anbieter in Puppet sehr mächtig sind. Tatsächlich können sie alles tun - zumindest was Ruby tun kann. Wenn Sie einige Teile Ihrer Infrastruktur mit komplizierten `define`Definitions anweisungen und `exec`Ausführungsressourcen verwalten, können Sie erwägen, diese mit einem benutzerdefinierten Typ zu ersetzen. Allerdings, wie bereits erwähnt, ist es lohnt sich zu sehen, ob jemand anderes hat dies bereits getan, bevor Sie Ihre eigenen.

Unser Beispiel war sehr einfach, und es gibt viel mehr zu lernen, schreiben Sie Ihre eigenen Typen. Wenn du deinen Code für andere verteilen willst, oder wenn du es nicht tust, dann ist es eine gute Idee, Tests mitzunehmen. Puppetlabs hat eine nützliche Seite auf der Schnittstelle zwischen benutzerdefinierten Typen und Providern:

[Puppet Custom Types](Htt../puppet//docs.puppetlabs.c../puppet/guid../puppet/custom_types.html)

Bei der Implementierung von Anbietern:

[Puppet Provider Development](Htt../puppet//docs.puppetlabs.c../puppet/guid../puppet/provider_development.html)

Und ein komplettes gearbeitetes Beispiel für die Entwicklung eines benutzerdefinierten Typs und Provider, ein wenig fortgeschrittener als das in diesem Buch vorgestellt:

[Puppet Respurce example](Htt../puppet//docs.puppetlabs.c../puppet/guid../puppet/complete_resource_example.html)
