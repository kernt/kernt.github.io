---
tags:
  - puppet
  - konfigurationsmanagement
---
# Prüfen der manifests mit Puppet-lint

Zu den Spezifikationen die gehört noch laut puppetlabs um Puppet-lint eine aussagekräftige Rückmeldung zu bekommen.
Einige dieser Regeln haben wir in den [Puppet4 Basics]()

* es müssen 2 Leerzeichen verwendet werden 
* es Dürfen kein Tabs benutzt werden
* es darf am ende einer Zeile kein Leerzeichen sein
* es darf keine line über 80 Zeichen geben
* innerhalb der anweisungs Blöcke werden Parameter mit `=>` zugewiesen

Mit dem Style Guide wird sichergestellt, dass ihr Puppet Code einfach zu lesen und zu Administrieren ist wenn Sie Planen ihren code der Community zur Verfügung zu stellen ist es sehr wichtig dies Grundlagen einzuhalten.

# Puppet-lint Installieren 

Für Puppet-lint gibt es in ihr Linux Distribution bestimmt ein Packet aber wir werden um dem Thema Praxis einzuhalten [Puppet](puppet.md) einsetzen. 

Erstellen sie die Datei `puppet-lint.pp` mit folgendem Inhalt.

```
  package {'puppet-lint':
    ensure => 'installed',
    provider => 'gem',
  }
```

Nun kann man mit `puppet apply puppet-lint.pp` Puppet veranlast werden Puppet-lint zu installieren
Weitere Details unter  [Puppet4 Package ](../puppet/puppet4-basics-package).

# Puppet-lint einsetzen

Zum testen benutze wir mal unser Beispiel `puppet-lint.pp` aber mit einem eingebautem Fehler.
Ich habe da mal einen Leerzeichen bei `provider => 'gem', ` eingefügt.
Das sollte zur folgenden Ausgabe führen wen alles wie erwartet funktioniert hat.
```
# puppet-lint puppet-lint.pp 
WARNING: indentation of => is not properly aligned on line 2
ERROR: trailing whitespace found on line 4
```
Nach dem korrigieren Unserer Datei sollte es folgender maßen aussehen: 
```
[[puppet-lint]] puppet-lint.pp 
# ~$
```

# Puppet-lint kann mehr
Für weitere Informationen [puppet-lint](http../puppet//github.c../puppet/rodj../puppet/puppet-lint).
