---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet für fortgeschrittene: Packages codeteile

Manchmal können Sie nicht eine ganze Konfigurationsdatei in einem Stück bereitstellen, aber das Zeilen-Zeilen-Editieren ist nicht genug. Oft müssen Sie eine Konfigurationsdatei aus verschiedenen Bits der Konfiguration erstellen, die von verschiedenen Klassen verwaltet wird. Sie können in eine Situation laufen, in der lokale Informationen auch in die Datei importiert werden müssen. In diesem Beispiel erstellen wir eine Konfigurationsdatei mit einer lokalen Datei sowie Snippets, die in unseren Manifesten definiert sind.
Fertig werden

Obwohl es möglich ist, unser eigenes System zu erstellen, um Dateien aus Stücken zu bauen, verwenden wir das Puppetlabs unterstützte Concat-Modul. Wir beginnen mit der Installation des Concat-Moduls, in einem früheren Beispiel haben wir das Modul auf unsere lokale Maschine installiert. In diesem Beispiel ändern wir die Puppet-Server-Konfiguration und laden das Modul zum Puppet-Server herunter.

In Ihrem Git-Repository erstellen Sie eine ¸environment.conf` Datei mit folgendem Inhalt:

```s
modulepath = public:modules
manifest = manifes../puppet/site.pp
```

Erstellen Sie das öffentliche Verzeichnis und laden Sie das Modul wie folgt in dieses Verzeichnis ein:

```s
t@mylaptop../puppet/puppet $ mkdir public && cd public
t@mylaptop../puppet/pupp../puppet/public $ puppet module install puppetlabs-concat --modulepath=.
Notice: Preparing to install int../puppet/ho../puppet/thom../puppet/pupp../puppet/public ...
Notice: Downloading from http../puppet//forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/ho../puppet/thom../puppet/pupp../puppet/public
└─┬ puppetlabs-concat (v1.1.1)
  └── puppetlabs-stdlib (v4.3.2)
```

Fügen Sie nun die neuen Module zu unserem Git-Repository hinzu:

```s
t@mylaptop../puppet/pupp../puppet/public $ git add .
t@mylaptop../puppet/pupp../puppet/public $ git commit -m "adding concat"
[production 50c6fca] adding concat
 407 files changed, 20089 insertions(+)
```

dann einen git push zum Git server:

`t@mylaptop../puppet/pupp../puppet/public $ git push origin production`

## Wie es geht

Nun, da wir das `concat` Modul auf unserem Server zur Verfügung haben, können wir in unserem `base` modul eine `concat` Container-Ressource erstellen:

```ruby
  concat {'hosts.allow':
    path =>../puppet/e../puppet/hosts.allow',
    mode => 0644
  }
```

Erstellen Sie ein `concat::fragment` Modul für den Header der neuen Datei:

```ruby
  concat::fragment {'hosts.allow header':
    target  => 'hosts.allow',
    content => "# File managed by puppet\n",
    order   => '01'
  }
```

Erstellen Sie ein `concat::fragment`, das eine lokale Datei enthält:

```ruby
  concat::fragment {'hosts.allow local':
    target => 'hosts.allow',
    source =>../puppet/e../puppet/hosts.allow.local',
    order  => '10',
  }
```

Erstellen Sie ein `concat::fragment` Modul, das an das Ende der Datei gehen wird:

```ruby
  concat::fragment {'hosts.allow tftp':
    target  => 'hosts.allow',
    content => "in.ftpd: .example.com\n",
    order   => '50',
  }
```

On the node, creat../puppet/e../puppet/hosts.allow.local with the following contents:

`in.tftpd: .example.com`

Führen Sie die Puppe aus, um die Datei zu erstellen:

```ruby
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1412138600'
Notice../puppet/Stage[mai../puppet/Ba../puppet/Concat[hosts.allo../puppet/File[hosts.allo../puppet/ensure: defined content as '{md5}b151c8bbc32c505f1c4a98b487f7d249'
Notice: Finished catalog run in 0.29 seconds
```

Überprüfen Sie den Inhalt der neuen Datei als:

```ruby
[root@cookbook ~]# ca../puppet/e../puppet/hosts.allow
# File managed by puppet
in.tftpd: .example.com
in.ftpd: .example.comc
```

## Wie es funktioniert

Die `concat` Ressource definiert einen Container, der alle nachfolgenden `concat==fragment` Ressourcen enthält. Jede `concat==fragment` Ressource verweist auf die Concat-Ressource als Ziel. Jedes `concat:: fragment` enthält auch ein `order` attribut. Das `order` attribut wird verwendet, um die Reihenfolge anzugeben, in der die Fragmente der endgültigen Datei hinzugefügt werden. Unsere../puppet/e../puppet/hosts.allow` Datei wird mit der Kopfzeile, dem Inhalt der lokalen Datei und schließlich der `in.tftpd` Zeile, die wir definiert haben, gebaut.