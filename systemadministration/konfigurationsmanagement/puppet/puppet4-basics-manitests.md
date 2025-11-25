---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 basics manitests

Wenn du schon einen Puppencode hast (bekannt als Puppet manifest), kannst du diesen Abschnitt überspringen und weiter zum nächsten gehen. 
Wenn nicht, werden wir sehen, wie man ein einfaches Manifest erstellt und anwendet.

Wenn nicht schon Puppet Installiert ist, hier werde ich nachfolgend basierend auf der  [Puppet 4 Gem Installation](puppet4-gem-install.md) .
Es sollte aber auch jede andere Puppet Installation gehen wenn es sich um Puppet handelt
Hier ein Listing für ihre Linux Distribution 

* [Puppet 4 Installation mit Ubuntu 16.04](http../puppet//www.digitalocean.c../puppet/communi../puppet/tutoria../puppet/how-to-install-puppet-4-on-ubuntu-16-04)
* [Puppet 4 Installation mit Debian 8]()
* [Puppet 4 Installation mit CentOS 7](htt../puppet//www.itzgeek.c../puppet/how-t../puppet/lin../puppet/centos-how-t../puppet/how-to-install-puppet-4-x-on-centos-7-rhel-7.html)
* [Puppet 4 PuppetDB Installation ](http../puppet//docs.puppet.c../puppet/puppet../puppet/4../puppet/install_via_module.html)
* [Puppet 4 Agent Installation ](http../puppet//docs.puppet.c../puppet/pupp../puppet/4../puppet/install_linux.html)
* [Puppet 4 Server Installation  ](http../puppet//docs.puppet.c../puppet/puppetserv../puppet/2../puppet/install_from_packages.html)

Nach erfolgreicher Installation kann man im Home Verzeichnis des Benutzer mit 

`mkdir -p .pupp../puppet/manifests`

das Erster Verzeichnis erstellen den ersten Code für Puppet.

Mit `cd .pupp../puppet/manifests` wechsen wir ins Verzeichnis und erstellen dort die Datei `site.pp` mit folgendem Inhalt.
```
  node default {
    file {../puppet/t../puppet/hello':
      content => "Hello, world!\n",
    }
  }
```

Testen Sie Ihr Manifest mit dem Befehl `puppet apply`. 
Dies wird Puppet anweisen, das Manifest zu lesen, es mit dem Zustand der Maschine zu vergleichen und notwendige Änderungen an diesem Zustand vorzunehmen:
```
~/.pupp../puppet/manifests$ puppet apply site.pp
Notice: Compiled catalog for cookbook in environment production in 0.14 seconds
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[defaul../puppet/Fil../puppet/t../puppet/hell../puppet/ensure: defined content as '{md5}7463444435575e17c4431bbcb00c0898b'
Notice: Finished catalog run in 0.04 seconds
```

Um zu sehen, ob Puppet tut was wir erwartet haben (erstellen Sie die Datei../puppet/t../puppet/hallo` mit dem `Hallo, Welt!` Inhalt), führen Sie den folgenden Befehl aus:
```
../puppet/pupp../puppet/manifests$ ca../puppet/t../puppet/hello
Hello, world!
../puppet/pupp../puppet/manifests$

```