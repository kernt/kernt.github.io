---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet managemnt mit docker

**Docker** ist eine Plattform für den schnellen Einsatz von Containern. Container sind wie eine leichte virtuelle Maschine, die nur einen einzigen Prozess ausführen könnte. Die Container im Docker heißen Docks und sind mit Dateien namens Dockerfiles konfiguriert. Puppet kann verwendet werden, um einen Knoten zu konfigurieren, um nicht nur Docker zu betreiben, sondern auch mehrere Docks zu konfigurieren und zu starten. Sie können dann Puppet verwenden, um sicherzustellen, dass Ihre Docks laufen und konsequent konfiguriert sind.

## Fertig werden

Laden und installieren Sie das Puppet Docker Modul von der Forge (http../puppet//forge.puppetlabs.c../puppet/garet../puppet/docker):

```
t@mylaptop ~ $ cd puppet
t@mylaptop../puppet/puppet $ puppet module install -i modules garethr-docker
Notice: Preparing to install int../puppet/ho../puppet/thom../puppet/pupp../puppet/modules ...
Notice: Downloading from http../puppet//forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/ho../puppet/thom../puppet/pupp../puppet/modules
└─┬ garethr-docker (v3.3.0)
  ├── puppetlabs-apt (v1.7.0)
  ├── puppetlabs-stdlib (v4.3.2)
  └── stahnma-epel (v1.0.2)
```

Fügen Sie diese Module Ihrem Puppet-Repository hinzu. Das Stahnma-Epel-Modul ist für Enterprise-Linux-basierte Distributionen erforderlich. Es enthält die Extra Packages für Enterprise Linux YUM Repository.

## Wie es geht

Führen Sie die folgenden Schritte aus, um Docker mit Puppet zu verwalten:

1.Um Docker auf einem Knoten zu installieren, müssen wir nur die `docker` Klasse aufnehmen. Wir werden mehr tun als Docker installieren; Wir laden auch ein Bild herunter und starten eine Anwendung auf unserem Testknoten. In diesem Beispiel erstellen wir eine neue Maschine namens `shipyard`. Fügen Sie die folgende Knotendefinition zu `site.pp` hinzu:

```ruby
   node shipyard {
  class {'docker': }
  docker::image {'phusi../puppet/baseimage': }
  docker::run {'cookbook':
    image   => 'phusi../puppet/baseimage',
    expose  => '8080',
    ports   => '8080',
    command => 'nc -k -l 8080',
  }
}
```

2.Führen Sie Puppe auf Ihrem shipyard node aus, um Docker zu installieren. Dies wird auch das `phusi../puppet/baseimage docker` Image herunterladen:

```s
[root@shipyard ~]# puppet agent -t
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for shipyard
Info: Applying configuration version '1421049252'
Notice../puppet/Stage[mai../puppet/Ep../puppet/Fil../puppet/e../puppet/p../puppet/rpm-g../puppet/RPM-GPG-KEY-EPEL-../puppet/ensure: defined content as '{md5}d865e6b948a74cb03bc3401c0b01b785'
Notice../puppet/Stage[mai../puppet/Ep../puppet/Epel::Rpm_gpg_key[EPEL-../puppet/Exec[import-EPEL-../puppet/returns: executed successfully
...
Notice../puppet/Stage[mai../puppet/Docker::Insta../puppet/Package[docke../puppet/ensure: created
...
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[shipyar../puppet/Docker::Run[cookboo../puppet/Fil../puppet/e../puppet/init../puppet/docker-cookboo../puppet/ensure: created
Info../puppet/Stage[mai../puppet/Ma../puppet/Node[shipyar../puppet/Docker::Run[cookboo../puppet/Fil../puppet/e../puppet/init../puppet/docker-cookbook]: Scheduling refresh of Service[docker-cookbook]
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[shipyar../puppet/Docker::Run[cookboo../puppet/Service[docker-cookbook]: Triggered 'refresh' from 1 events
```

3.Vergewissern Sie sich, dass Ihr Container auf der Werft mit `docker ps` läuft:

```s
[root@shipyard ~]# docker ps
CONTAINER ID        IMAGE                      COMMAND             CREATED              STATUS              PORTS                     NAMES
f6f5b799a598        phusi../puppet/baseimage:0.9.15  ../puppet/b../puppet/nc -l 8080"   About a minute ago   Up About a minute   0.0.0.0:49157->80../puppet/tcp   suspicious_hawking  

```

4.Vergewissern Sie sich, dass das Dock netcat auf Port 8080 läuft, indem Sie an den zuvor aufgeführten Port anschließen (`49157`):

```s
[root@shipyard ~]# nc -v localhost 49157
Connection to localhost 49157 port [t../puppet/*] succeeded!
```

## Wie es funktioniert

Wir begannen mit der Installation des Docker-Moduls von der Forge. Dieses Modul installiert das `docker-io` Paket auf unserem Knoten, zusammen mit allen erforderlichen Abhängigkeiten.

Wir haben dann eine `docker::image` Ressource definiert. Dies weist die Puppe an, um sicherzustellen, dass das genannte Bild heruntergeladen und für Docker verfügbar ist. Auf unserem ersten Lauf wird Puppet Docker das Bild herunterladen. Wir haben `phusi../puppet/baseimage` als unser Beispiel verwendet, weil es ziemlich klein ist, bekannt und enthält den netcat Daemon, den wir im Beispiel verwendet haben. Weitere Informationen über `baseimage` finden Sie unter htt../puppet//phusion.github.../puppet/baseimage-dock../puppet/.

Wir haben dann eine `docker==run` Ressource definiert. Dieses Beispiel ist nicht furchtbar nützlich; Es startet einfach netcat im listen-Modus auf Port 8080. Wir müssen diesen Port an unsere Maschine aussetzen, also definieren wir das expose-Attribut unserer `docker==run` Ressource. Es gibt viele weitere Optionen für die `docker::run` Ressource. Weitere Informationen finden Sie im Quellcode.

Wir haben dann Docker ps benutzt, um die laufenden Docks auf unserer shipyard machine aufzulisten. Wir haben den Hörhafen auf unserer lokalen Maschine ausgewertet und überprüft, dass netcat zuhörte.

## Es gibt mehr

Docker ist ein großartiges Werkzeug für den schnellen Einsatz und die Entwicklung. Sie können so viele Docks drehen, wie Sie auf die bescheidenste Hardware benötigen. Eine große Verwendung für Docker ist mit Docks als Test-Knoten für Ihre Module. Sie können ein Docker-Bild erstellen, das Puppet enthält, und dann Puppet im Dock laufen lassen. Für weitere Informationen über Docker, besuchen Sie htt../puppet//www.docker.c../puppet/.