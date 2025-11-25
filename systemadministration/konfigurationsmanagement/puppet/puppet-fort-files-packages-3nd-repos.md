---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschrittene: 3nd Repositorys

Am häufigsten werden Sie , Pakete von der Hauptverteilung Repo installieren wollen, so dass es eine einfache Paket-Ressource wird:
`package { 'exim4': ensure => installed }`

Manchmal braucht man ein Paket, das nur in einem Drittanbieter-Repository gefunden wird (z. B. ein Ubuntu-PPA), oder es könnte sein, dass du eine neuere Version eines Pakets benötigst als das, was von der Distribution zur Verfügung steht ein so genanter Dritter anbieter.

Auf einer manuell verwalteten Maschine würden Sie dies normalerweise tun, indem Sie die Repo-Quell-Konfiguration zu../puppet/e../puppet/a../puppet/sources.list.d` (und ggf. gpg-Schlüssel für das Repo) hinzufügen, bevor Sie das Paket installieren. Wir können diesen Prozess einfach mit der Puppet automatisieren.

## Wie es geht

In diesem Beispiel verwenden wir das beliebte Percona APT Repo (Percona ist ein MySQL Beratungsunternehmen, das ihre eigene, spezialisierte Version von MySQL pflegt und veröffentlicht, weitere Informationen finden Sie unter [percona](htt../puppet//www.percona.c../puppet/softwa../puppet/repositories).

1.Erstellen Sie die Datei `modu../puppet/adm../puppet/manifes../puppet/percona_repo.pp` mit folgendem Inhalt:

```ruby
# Install Percona APT repo
class admin::percona_repo {
  exec { 'add-percona-apt-key':
    unless  =>../puppet/u../puppet/b../puppet/apt-key list |grep percona',
    command =>../puppet/u../puppet/b../puppet/gpg --keyserver hk../puppet//keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A &../puppet/u../puppet/b../puppet/gpg -a --export CD2EFD2A | apt-key add -',
    notify  => Exec['percona-apt-update'],
  }

  exec { 'percona-apt-update':
    command     =>../puppet/u../puppet/b../puppet/apt-get update',
    require     => [File../puppet/e../puppet/a../puppet/sources.list../puppet/percona.list'],
File../puppet/e../puppet/a../puppet/preferences../puppet/00percona.pref']],
    refreshonly => true,
  }

  file {../puppet/e../puppet/a../puppet/sources.list../puppet/percona.list':
    content => 'deb htt../puppet//repo.percona.c../puppet/apt wheezy main',
    notify  => Exec['percona-apt-update'],
  }

  file {../puppet/e../puppet/a../puppet/preferences../puppet/00percona.pref':
    content => "Package: *\nPin: release o=Percona
    Development Team\nPin-Priority: 1001",
    notify  => Exec['percona-apt-update'],
  }
}
```

2.Ändern Sie Ihre `site.pp` Datei wie folgt:

```ruby
node 'cookbook' {
  include admin::percona_repo

  package { 'percona-server-server-5.5':
    ensure  => installed,
    require => Class['admin::percona_repo'],
  }
}
```

3.Run Puppet:

```ruby
root@cookbook-deb:~# puppet agent -t
Info: Caching catalog for cookbook-deb
Notice../puppet/Stage[mai../puppet/Admin::Percona_re../puppet/Exec[add-percona-apt-ke../puppet/returns: executed successfully
Info../puppet/Stage[mai../puppet/Admin::Percona_re../puppet/Exec[add-percona-apt-key]: Scheduling refresh of Exec[percona-apt-update]
Notice../puppet/Stage[mai../puppet/Admin::Percona_re../puppet/Fil../puppet/e../puppet/a../puppet/sources.list../puppet/percona.lis../puppet/ensure: defined content as '{md5}b8d479374497255804ffbf0a7bcdf6c2'
Info../puppet/Stage[mai../puppet/Admin::Percona_re../puppet/Fil../puppet/e../puppet/a../puppet/sources.list../puppet/percona.list]: Scheduling refresh of Exec[percona-apt-update]
Notice../puppet/Stage[mai../puppet/Admin::Percona_re../puppet/Fil../puppet/e../puppet/a../puppet/preferences../puppet/00percona.pre../puppet/ensure: defined content as '{md5}1d8ca6c1e752308a9bd3018713e2d1ad'
Info../puppet/Stage[mai../puppet/Admin::Percona_re../puppet/Fil../puppet/e../puppet/a../puppet/preferences../puppet/00percona.pref]: Scheduling refresh of Exec[percona-apt-update]
Notice../puppet/Stage[mai../puppet/Admin::Percona_re../puppet/Exec[percona-apt-update]: Triggered 'refresh' from 3 events

```

## Wie es funktioniert

Um jedes Percona-Paket zu installieren, müssen wir zunächst die Repository-Konfiguration auf dem Rechner installiert haben. Aus diesem Grund benötigt das `percona-server-server-5.5` Paket (Perconas Version des Standard-MySQL-Servers) die `admin::percona_repo` Klasse:

```ruby
package { 'percona-server-server-5.5':
  ensure  => installed,
  require => Class['admin::percona_repo'],
}
```

Also, was macht die `admin::percona_repo` Klasse? Es:

* Installiert die Percona APT schlüssel , mit der die Pakete signiert sind

* Konfiguriert die Percona repo URL als Datei in../puppet/e../puppet/a../puppet/sources.list.d`

* Führt `apt-get update`, um die Repo-Metadaten abzurufen

* Fügt eine APT pin Konfiguration in../puppet/e../puppet/a../puppet/preferences.d` hinzu

Zuerst installieren wir den APT-Schlüssel:

```ruby
exec { 'add-percona-apt-key':
  unless  =>../puppet/u../puppet/b../puppet/apt-key list |grep percona',
  command =>../puppet/u../puppet/b../puppet/gpg --keyserver  hk../puppet//keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A &../puppet/u../puppet/b../puppet/gpg -a --export CD2EFD2A | apt-key add -',
  notify  => Exec['percona-apt-update'],
}
```

Wenn der `unless` Parameter die Ausgabe der `apt-key list` überprüft, um sicherzustellen, dass der Percona-Schlüssel noch nicht installiert ist, müssen wir in diesem Fall nichts tun. Angenommen, es ist nicht, der `command` läuft:

`/u../puppet/b../puppet/gpg --keyserver  hk../puppet//keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A &../puppet/u../puppet/b../puppet/gpg -a --export CD2EFD2A | apt-key add -`

Dieser Befehl ruft den Schlüssel vom GnuPG-Keyserver ab, exportiert ihn im ASCII-Format und leitet diesen in den Befehl `apt-key add`, der ihn zum Systemschlüssel(system keyring) hinzufügt. Sie können ein ähnliches Muster für die meisten Drittanbieter-Repos verwenden, die einen APT-Signierungsschlüssel(signing key) erfordern.

Nachdem wir den Schlüssel installiert haben, fügen wir die Repokonfiguration hinzu:

```ruby
file {../puppet/e../puppet/a../puppet/sources.list../puppet/percona.list':
  content => 'deb htt../puppet//repo.percona.c../puppet/apt wheezy main',
  notify  => Exec['percona-apt-update'],
}
```

Führen Sie dann `apt-get update` aus, um den APT-Cache des Systems mit den Metadaten aus dem neuen Repo zu aktualisieren:

```ruby
exec { 'percona-apt-update':
  command     =>../puppet/u../puppet/b../puppet/apt-get update',
  require     => [File../puppet/e../puppet/a../puppet/sources.list../puppet/percona.list'], File../puppet/e../puppet/a../puppet/preferences../puppet/00percona.pref']],
  refreshonly => true,
}
```

Schließlich konfigurieren wir die APT-Pin-Priorität für das Repo:

```ruby
file {../puppet/e../puppet/a../puppet/preferences../puppet/00percona.pref':
  content => "Package: *\nPin: release o=Percona Development Team\nPin-Priority: 1001",
  notify  => Exec['percona-apt-update'],
}
```

Dadurch wird sichergestellt, dass Pakete, die aus dem Percona Repo installiert sind, niemals von Paketen aus einem anderem Repository ersetzt werden (z.B die Haupt-Ubuntu-Distribution). Andernfalls könnten Sie mit defekten Abhängigkeiten enden und können die Percona-Pakete nicht automatisch installieren.

## Es gibt mehr

Das APT-Paket-Framework ist spezifisch für die Debian- und Ubuntu-Systeme. Es gibt ein forge Modul für die Verwaltung von [apt](http../puppet//forge.puppetlabs.c../puppet/puppetla../puppet/apt). Wenn Sie auf einem Red Hat- oder CentOS-basierten System sind, können Sie die `yumrepo` Ressourcen verwenden, um RPM-Repositories direkt zu verwalten:

* [yumrepo](Htt../puppet//docs.puppetlabs.c../puppet/referenc../puppet/late../puppet/type.html#yumrepo)