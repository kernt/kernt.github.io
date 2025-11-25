---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 externe tools ecosystem nenutzer fats

Während Facter's eingebaute Fakten nützlich sind, ist es eigentlich ganz einfach, Ihre eigenen Fakten hinzuzufügen. Wenn Sie z. B. Maschinen in verschiedenen Rechenzentren oder Hosting-Anbietern haben, können Sie dazu eine eigene Tatsache hinzufügen, damit die Puppe feststellen kann, ob lokale Einstellungen angewendet werden müssen (z. B. lokale DNS-Server oder Netzwerkrouten).

## Wie es geht

Hier ist ein Beispiel für eine einfache kundenspezifische Tatsache:

1.Erstellen Sie die Verzeichnis `modul../puppet/fac../puppet/l../puppet/facter` und erstellen Sie dann die Datei `modul../puppet/fac../puppet/l../puppet/ fact../puppet/hello.rb` mit folgendem Inhalt:

```pp
Facter.add(:hello) do
  setcode do
    "Hello, world"
  end
end
```

2.Ändern Sie Ihre `site.pp` Datei wie folgt:

```pp
node 'cookbook' {
  notify { $::hello: }
}
```

3.Puppet run:

```pp
[root@cookbook ~]# puppet agent -t
Notice../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/l../puppet/fact../puppet/hello.r../puppet/ensure: defined content as '{md5}f66d5e290459388c5ffb3694dd22388b'
Info: Loading facts
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1416205745'
Notice: Hello, world
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Notify[Hello, worl../puppet/message: defined 'message' as 'Hello, world'
Notice: Finished catalog run in 0.53 seconds
```

## Wie es funktioniert

Facter Fakten sind in Ruby-Dateien definiert, die mit facter verteilt sind. Puppe kann zusätzliche Fakten hinzufügen, um Dateien zu erstellen, indem sie Dateien innerhalb des `l../puppet/facter` Unterverzeichnisses eines Moduls erstellen. Diese Dateien werden dann auf Client-Knoten übertragen, wie wir früher mit dem puppetlabs-stdlib-Modul gesehen haben. Um das Kommandozeilen-Fame zu verwenden, verwenden Sie diese `puppet` Fakten, fügen Sie die Option `-p` hinzu, wie in der folgenden Befehlszeile gezeigt:

```s
[root@cookbook ~]# facter hello

[root@cookbook ~]# facter -p hello
Hello, world
```

Tip
Wenn Sie eine ältere Version von Puppet (älter als 3.0) verwenden, müssen Sie `pluginsync` in Ihrer Datei `puppet.conf` aktivieren, wie in der folgenden Befehlszeile gezeigt:

```pp
[main]
pluginsync = true
```

Fakten können jeden Ruby-Code enthalten, und der letzte Wert, der innerhalb des `setcodes do ... end`ausgewertet wird, endet, wird der Wert sein, der von der Tatsache zurückgegeben wird. Zum Beispiel könnten Sie eine nützliche Tatsache machen, die die Anzahl der aktuell angemeldeten Benutzer an das System zurückgibt:

```pp
Facter.add(:users) do
  setcode do
    %../puppet/u../puppet/b../puppet/who |wc -l}.chomp
  end
end
```

Um die Tatsache in Ihren Manifesten zu verweisen, benutze einfach ihren Namen wie eine eingebaute Tatsache:

```pp
notify { "${::users} users logged in": }
Notice:  2 users logged in
```

Sie können benutzerdefinierte Fakten zu jedem Puppet-Modul hinzufügen. Bei der Erstellung von Fakten, die von mehreren Modulen verwendet werden, kann es sinnvoll sein, sie in einem Facts-Modul zu platzieren. In den meisten Fällen ist die kundenspezifische Tatsache auf ein bestimmtes Modul bezogen und sollte in diesem Modul platziert werden.

## Es gibt mehr

Der Name der Ruby-Datei, die die Tatsachendefinition enthält, ist irrelevant. Sie können diese Datei benennen, was auch immer Sie wünschen Der Name der Tatsache stammt aus dem `Facter.add()` - Funktionsaufruf. Sie können diese Funktion auch mehrere Male innerhalb einer einzigen Ruby-Datei aufrufen, um mehrere Fakten zu definieren. Zum Beispiel könnten Sie die Datei `grep`../puppet/pr../puppet/meminfo` und mehrere Fakten basierend auf Speicherinformationen zurückgeben, wie in der Datei `meminfo.rb` im folgenden Code-Snippet gezeigt:

```pp
File.open../puppet/pr../puppet/meminfo') do |f|
  f.each_line { |line|
  if (lin../puppet/^Activ../puppet/])
    Facter.add(:memory_active) do
      setcode do line.split(':')[1].to_i
      end
    end
  end
  if (lin../puppet/^Inactiv../puppet/])
    Facter.add(:memory_inactive) do
      setcode do line.split(':')[1].to_i
      end
    end
  end
  }
end
```

Nach dem Synchronisieren dieser Datei an einen Knoten stehen die `memory_active` und `memory_inactive` Fakten wie folgt zur Verfügung:

```pp
[root@cookbook ~]# facter -p |grep memory_
memory_active => 63780
memory_inactive => 58188
```

Sie können die Verwendung von Fakten erweitern, um eine völlig nodeless Puppet-Konfiguration zu bauen; Mit anderen Worten: Die Marionette kann entscheiden, welche Ressourcen für eine Maschine gelten sollen, die ausschließlich auf den Tatsachenfakten beruht. [Jordan Sissel]( htt../puppet//www.semicomplete.c../puppet/bl../puppet/geeke../puppet/puppet-nodeless-configuration.html) hat über diesen Ansatz auf geschrieben.

Sie können mehr über kundenspezifische Fakten erfahren, einschließlich, wie Sie sicherstellen, dass OS-spezifische Fakten nur auf den relevanten Systemen funktionieren und wie man Tatsachen wiegt, so dass sie in einer bestimmten Reihenfolge auf der [Puppetlabs-Website](Htt../puppet//docs.puppetlabs.c../puppet/guid../puppet/custom_facts.html) ausgewertet werden:
