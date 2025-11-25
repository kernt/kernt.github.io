---
tags:
  - puppet
  - puppetdb
  - konfigurationsmanagement
---
# Pupper PuppetDB

PuppetDB ist eine Datenbank für Puppet, die verwendet wird, um Informationen über Nodes zu speichern, die mit einem Puppet-Master \(Puppet version &gt;= 4 Puppet Sever\) verbunden sind.
PuppetDB ist auch ein Speicherbereich für exportierte Ressourcen.
**_Exportierte Ressourcen_** sind Ressourcen, die auf Nodes definiert sind, aber auf andere Nodes angewendet werden.
Der einfachste Weg zur Installation von PuppetDB ist die Verwendung des PuppetDB Moduls von Puppet-Labs. Von diesem Punkt an werden wir davon ausgehen, dass Sie die `puppet.example.com` Maschine benutzen und eine Passenger basierte Konfiguration von Puppet haben.

## Fertig werden

Installieren Sie das PuppetDB-Modul in der Produktionsumgebung, die Sie im vorherigen Rezept erstellt haben.
Wenn Sie keine Verzeichnisumgebungen erstellt haben, machen Sie sich keine Sorgen, mit der Puppenmodulinstallation wird das Modul an die richtige Stelle für Ihre Installation mit dem folgenden Befehl installiert:

```ruby
root@puppet:~# puppet module install puppetlabs-puppetdb
Notice: Preparing to install into /etc/puppet/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/etc/puppet/environments/production/modules
└─┬ puppetlabs-puppetdb (v3.0.1)
  ├── puppetlabs-firewall (v1.1.3)
  ├── puppetlabs-inifile (v1.1.3)
  └─┬ puppetlabs-postgresql (v3.4.2)
    ├─┬ puppetlabs-apt (v1.6.0)
    │ └── puppetlabs-stdlib (v4.3.2)
    └── puppetlabs-concat (v1.1.0)
```

### Wie es geht

Nun, da unser Puppet-Master das PuppetDB-Modul installiert hat, müssen wir das PuppetDB-Modul an unseren Puppet-Master anwenden, das können wir im site manifest machen.
Füge folgendes zu deiner \(Produktion\) `site.pp` hinzu:

```ruby
node puppet {
  class { 'puppetdb': }
  class { 'puppetdb==master==config': 
    puppet_service_name => 'apache2',
  }
}
```

Führen Sie den `puppet agent` aus, um die `puppetdb` Klasse und die `puppetdb==master==config` Klasse anzuwenden:

```s
root@puppet:~# puppet agent -t
Info: Caching catalog for puppet
Info: Applying configuration version '1410416952'
...
Info: Class[Puppetdb==Server==Jetty_ini]: Scheduling refresh of Service[puppetdb]
Notice: Finished catalog run in 160.78 seconds

```

### Wie es funktioniert

Das PuppetDB-Modul ist ein hervorragendes Beispiel dafür, wie eine komplexe Konfigurationsaufgabe puppetiert \(ja ein Kunstwort\) werden kann. 
Einfach durch Hinzufügen der `puppetdb` Klasse zu unserem Puppet-Master-Knoten, Puppet installiert und konfiguriert `postgresql` und `puppetdb`.

Als wir die `puppetdb==master==config-Klasse` angerufen haben, setzen wir die Variable `puppet_service_name` auf `apache2`, das ist, weil wir Puppet mit Passenger laufen lassen. 
Ohne diese Zeile würde unser Agent versuchen, den Puppetmaster-Prozess anstelle von `apache2` zu starten.

Der Agent richtet dann die Konfigurationsdateien für PuppetDB und konfiguriert Puppet, um PuppetDB zu verwenden. Wenn Sie auf `/etc/puppet/puppet.conf` schauen, sehen Sie die folgenden zwei neuen Zeilen:

```
storeconfigs = true
storeconfigs_backend = puppetdb
```

### Weiter führendes

Nun, da PuppetDB konfiguriert ist und wir einen erfolgreichen Agenten am laufen gehabt haben, hat PuppetDB Daten, die wir abfragen können:

```s
root@puppet:~# puppet node status puppet
puppet
Currently active
Last catalog: 2014-09-11T06:45:25.267Z
Last facts: 2014-09-11T06:45:22.351Z
```