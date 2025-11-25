---
tags:
  - puppet
  - konfigurationsmanagement
---

# puppet4 externe tools ecosystem eigene ressourcen

Wie Sie wissen, hat Puppet eine Reihe von nützlichen integrierten Ressourcentypen: Pakete, Dateien, Benutzer und so weiter.
Normalerweise können Sie alles tun, was Sie tun müssen, indem Sie entweder Kombinationen dieser eingebauten Ressourcen verwenden oder `define`, die Sie mehr oder weniger in der gleichen Weise wie eine Ressource verwenden können (siehe [Puppet4 Bessere Manifests Schreiben](puppet4-bessere-manifests.md) ).

In den frühen Tagen der Puppe, die Schaffung Ihrer eigenen Ressource-Typ war häufiger als die Liste der Kernressourcen war kürzer als es heute ist. Bevor Sie erwägen, Ihre eigene Ressourcentyp zu erstellen, schlage ich vor, die Forge für alternative Lösungen zu suchen. Auch wenn Sie ein Projekt finden können, das Ihr Problem nur teilweise löst, werden Sie besser daran gedient, das Projekt zu erweitern und zu helfen, anstatt zu versuchen, Ihre eigenen zu schaffen. Allerdings, wenn Sie Ihren eigenen Ressourcentyp erstellen müssen, macht Puppet es ganz einfach. Die einheimischen Typen sind in Ruby geschrieben, und du wirst eine grundlegende Vertrautheit mit Ruby brauchen, um deine eigenen zu schaffen.

Lassen Sie uns unser Gedächtnis auf die Unterscheidung zwischen Typen und Anbietern erneuern. Ein Typ beschreibt eine Ressource und die Parameter, die es haben kann (z.B. der `package` typ). Ein Provider sagt Puppet, wie man einen Ressourcentyp für eine bestimmte Plattform oder Situation implementiert (zB die `a../puppet/dpkg` Anbieter implementieren den `package` typ für Debian-ähnliche Systeme).

Ein einzelner Typ (`package`) kann viele Anbieter (APT, YUM, Fink, und so weiter) haben. Wenn Sie bei der Deklaration einer Ressource keinen Provider angeben, wählt die Puppe die am besten geeignete Umgebung aus.

Wir verwenden Ruby in diesem Abschnitt; Wenn Sie nicht mit Ruby vertraut sind, besuchen Sie [ruby-doc](htt../puppet//www.ruby-doc.o../puppet/do../puppet/Tutori../puppet/) oder [codeacademy](htt../puppet//www.codecademy.c../puppet/trac../puppet/ru../puppet/) .

## Wie es geht

In diesem Abschnitt werden wir sehen, wie wir einen benutzerdefinierten Typ erstellen können, den wir verwenden können, um Git-Repositories zu verwalten, und im nächsten Abschnitt werden wir einen Provider schreiben, um diesen Typ zu implementieren.

Erstellen Sie die Datei `modul../puppet/cookbo../puppet/l../puppet/pupp../puppet/ty../puppet/gitrepo.rb` mit folgendem Inhalt:

```pp
Puppet::Type.newtype(:gitrepo) do
  ensurable

  newparam(:source) do
    isnamevar
  end

  newparam(:path)
end
```

## Wie es funktioniert

Benutzerdefinierte Typen können in jedem Modul, in einem `l../puppet/pupp../puppet/type` Unterverzeichnis und in einer Datei für den Typ (in unserem Beispiel, das ist `modul../puppet/cookbo../puppet/l../puppet/pupp../puppet/ty../puppet/gitrepo.rb`) benannt werden.

Die erste Zeile von `gitrepo.rb` erzählt Puppet, um einen neuen Typ namens `gitrepo` zu registrieren:
`Puppet::Type.newtype(:gitrepo) do`

Die `ensurable`(reibungslose) Linie gibt automatisch den Typ eine `ensure`(sichere) Eigenschaft, wie Puppets integrierte Ressourcen:
`ensurable`

Wir geben dem Typ einige Parameter. Für den Moment ist alles, was wir brauchen, ein `source` parameter für die Git-Quell-URL und ein `path` parameter, um Puppet zu erzählen, wo das Repo im Dateisystem erstellt werden soll:

```pp
newparam(:source) do
  isnamevar
end
```

Die `isnamevar`-Deklaration teilt der Puppe mit, dass der `source` parameter der namevar des Typs ist. Also, wenn Sie eine Instanz dieser Ressource deklarieren, was auch immer Sie geben, wird es der Wert der `source`, zum Beispiel:

```pp
gitrepo { 'gi../puppet//github.c../puppet/puppetla../puppet/puppet.git':
  path =>../puppet/ho../puppet/ubun../puppet/d../puppet/puppet',
}
```

Schließlich sagen wir der Puppe, dass der Typ den `path` parameter akzeptiert:
`newparam(:path)`

## Es gibt mehr

Bei der Entscheidung, ob Sie einen benutzerdefinierten Typ erstellen sollten, sollten Sie ein paar Fragen über die Ressource, die Sie zu beschreiben versuchen, wie:

* Ist die Ressource aufzählbar? Kannst du leicht eine Liste aller Instanzen der Ressource auf dem System erhalten?

* Ist die Ressource atomar? Können Sie sicherstellen, dass nur eine Kopie der Ressource auf dem System existiert (dies ist besonders wichtig, wenn Sie sicherstellen wollen => abwesend auf der Ressource)?

* Gibt es eine andere Ressource, die diese Ressource beschreibt? In einem solchen Fall wäre ein definierter Typ, der auf der vorhandenen Ressource basiert, in den meisten Fällen eine einfachere Lösung.

### Dokumentation

Unser Beispiel ist bewusst einfach, aber wenn Sie auf die Entwicklung von echten benutzerdefinierten Typen für Ihre Produktionsumgebung weitergehen, sollten Sie Dokumentationsstrings hinzufügen, um zu beschreiben, was der Typ und seine Parameter tun, zum Beispiel:

```pp
Puppet::Type.newtype(:gitrepo) do
  @doc = "Manages Git repos"

  ensurable

  newparam(:source) do
    desc "Git source URL for the repo"
    isnamevar
  end

  newparam(:path) do
    desc "Path where the repo should be created"
  end
end
```

### Validierung

Sie können die Parametervalidierung verwenden, um nützliche Fehlermeldungen zu generieren, wenn jemand versucht, schlechte Werte an die Ressource zu übergeben. Zum Beispiel könnten Sie bestätigen, dass das Verzeichnis, in dem das Repo erstellt werden soll, tatsächlich existiert:

```pp
newparam(:path) do
  validate do |value|
    basepath = File.dirname(value)
    unless File.directory?(basepath)
      raise ArgumentError , "The path %s doesn't exist" % basepath
    end
  end
end
```

Sie können auch die Liste der zulässigen Werte angeben, die der Parameter ausführen kann:

```pp
newparam(:breakfast) do
  newvalues(:bacon, :eggs, :sausages)
end
```
