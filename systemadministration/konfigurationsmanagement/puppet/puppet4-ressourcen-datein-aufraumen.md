---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 ressourcen datein aufraumen

Die `tidy` Ressource von Puppet wird Ihnen helfen, alte oder veraltete Dateien aufzuräumen, wodurch der Datenträgerverbrauch reduziert wird. Wenn Sie beispielsweise das Puppet-Reporting aktiviert haben, wie es im Abschnitt zum Erstellen von Berichten beschrieben ist, sollten Sie regelmäßig alte Berichtsdateien löschen.

Wie es geht...

Lass uns anfangen.

1.Ändern Sie Ihre `site.pp` Datei wie folgt:

```pp
node 'cookbook' {
  tidy {../puppet/v../puppet/l../puppet/pupp../puppet/reports':
    age     => '1w',
    recurse => true,
  }
}
```

3.Puppet run:

```pp
[root@cookbook clients]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/repor../puppet/cookbook.example.c../puppet/201409090637.yam../puppet/ensure: removed
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/repor../puppet/cookbook.example.c../puppet/201409100556.yam../puppet/ensure: removed
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/repor../puppet/cookbook.example.c../puppet/201409090631.yam../puppet/ensure: removed
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/repor../puppet/cookbook.example.c../puppet/201408210557.yam../puppet/ensure: removed
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/repor../puppet/cookbook.example.c../puppet/201409080557.yam../puppet/ensure: removed
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/repor../puppet/cookbook.example.c../puppet/201409100558.yam../puppet/ensure: removed
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/repor../puppet/cookbook.example.c../puppet/201408210546.yam../puppet/ensure: removed
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/v../puppet/l../puppet/pupp../puppet/repor../puppet/cookbook.example.c../puppet/201408210539.yam../puppet/ensure: removed
Notice: Finished catalog run in 0.80 seconds
```

## Wie es funktioniert

Puppet durchsucht den angegebenen Pfad für alle Dateien, die mit dem `age` parameter übereinstimmen. In diesem Fall `2w` (zwei Wochen). Es sucht auch Unterverzeichnisse (`recurse => true`).

Alle Dateien, die Ihren Kriterien entsprechen, werden gelöscht.

### Es gibt mehr

Sie können Dateianlagen in Sekunden, Minuten, Stunden, Tagen oder Wochen angeben, indem Sie ein einzelnes Zeichen verwenden, um die Zeiteinheit wie folgt festzulegen:

* 60s

* 180m

* 24h

* 30d

* 4w

Sie können festlegen, dass Dateien, die größer als eine gegebene Größe sind, wie folgt entfernt werden sollten:
`size => '100m',`

Dies entfernt Dateien von 100 Megabyte und mehr. Für Kilobyte verwenden Sie `k` und für Bytes verwenden Sie `b`.

## Hinweis

Beachten Sie, dass, wenn Sie sowohl Alters- als auch Größenparameter angeben, sie als unabhängige Kriterien behandelt werden. Wenn Sie z. B. Folgendes angeben, wird die Puppe alle Dateien entfernen, die mindestens einen Tag alt oder mindestens 512 KB groß sind:

age => "1d",

size => "512k",
