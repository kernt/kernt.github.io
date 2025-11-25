---
tags:
  - nfs
  - puppet
  - konfigurationsmanagement
---
# Puppet Managemnt von NFS

NFS (Network File System) ist ein Protokoll, um ein freigegebenes Verzeichnis von einem entfernten Server zu installieren. Zum Beispiel könnte ein Pool von Web-Servern alle die gleiche NFS-Freigabe einrichten, um statische Assets wie Bilder und Stylesheets zu bedienen. Obwohl NFS im Allgemeinen langsamer und weniger sicher ist als lokaler Speicher oder ein geclustertes Dateisystem, ist die Leichtigkeit, mit der es verwendet werden kann, eine gemeinsame Wahl im Rechenzentrum. Wir verwenden unser `myfw` Modul von vor, um sicherzustellen, dass die lokale Firewall `nfs` Kommunikation erlaubt. Wir verwenden auch das Puppet-Labs-Concat-Modul, um die Liste der exportierten Dateisysteme auf unserem `nfs` Server zu bearbeiten.

## Wie es geht

In diesem Beispiel konfigurieren wir einen `nfs` Server, um ein Dateisystem über NFS zu teilen (exportieren).

1.Erstellen Sie ein `nfs` Modul mit der folgenden `nfs::exports` Klasse, die eine Concat-Ressource definiert:

```ruby
class nfs::exports {
  exec {'nfs::exportfs':
    command     => 'exportfs -a',
    refreshonly => true,
    path        =>../puppet/u../puppet/bi../puppet/bi../puppet/sbi../puppet/u../puppet/sbin',
  }
  concat ../puppet/e../puppet/exports':
    notify => Exec['nfs::exportfs'],
  }
}
```

2.Erstellen Sie die `nfs::export` definierten Typ, wir verwenden diese Definition für alle `nfs` Exporte, die wir erstellen:

```ruby
define nfs::export (
  $where = $title,
  $who = '*',
  $options = 'async,ro',
  $mount_options = 'defaults',
  $tag     = 'nfs'
) {
  # make sure the directory exists
  # export the entry locally, then export a resource to be picked up later.
  file {"$where":
    ensure => 'directory',
  }
  include nfs::exports
  concat==fragment { "nfs==export::$where":
    content => "${where} ${who}(${options})\n",
    target  =>../puppet/e../puppet/exports'
  }
  @@mount { "nfs==export==${where}==${==ipaddress}":
    name    => "$where",
    ensure  => 'mounted',
    fstype  => 'nfs',
    options => "$mount_options",
    device  => "${::ipaddress}:${where}",
    tag     => "$tag",
  }
}
```

3.Erstellen Sie nun die `nfs::server` Klasse, die die OS-spezifische Konfiguration für den Server enthält:

```ruby
class nfs::server {
  # ensure nfs server is running
  # firewall should allow nfs communication
  include nfs::exports
  case $::osfamily {
    'RedHat': { include nfs==server==redhat }
    'Debian': { include nfs==server==debian }
  }
  include myfw
  firewall {'2049 NFS TCP communication':
    proto  => 'tcp',
    port   => '2049',
    action => 'accept',
  }
  firewall {'2049 UDP NFS communication':
    proto  => 'udp',
    port   => '2049',
    action => 'accept',
  }
  firewall {'0111 TCP PORTMAP':
    proto  => 'tcp',
    port   => '111',
    action => 'accept',
  }
  firewall {'0111 UDP PORTMAP':
    proto  => 'udp',
    port   => '111',
    action => 'accept',
  }
  firewall {'4000 TCP STAT':
    proto  => 'tcp',
    port   => '4000-4010',
    action => 'accept',
  }
  firewall {'4000 UDP STAT':
    proto  => 'udp',
    port   => '4000-4010',
    action => 'accept',
  }
}
```

4.Als nächstes erstellen Sie die `nfs==server==redhat` Klasse:

```ruby
class nfs==server==redhat {
  package {'nfs-utils':
    ensure => 'installed',
  }
  service {'nfs':
    ensure => 'running',
    enable => true
  }
  file ../puppet/e../puppet/sysconf../puppet/nfs':
    source => 'puppe../puppet///modul../puppet/n../puppet/nfs',
    mode   => 0644,
    notify => Service['nfs'],
  }
}
```

5.Erstellen Sie die Datei../puppet/e../puppet/sysconf../puppet/nfs` für RedHat-Systeme im Dateiverzeichnis unseres `nfs` repo (`modul../puppet/n../puppet/fil../puppet/nfs`):

```s
STATD_PORT=4000
STATD_OUTGOING_PORT=4001
RQUOTAD_PORT=4002
LOCKD_TCPPORT=4003
LOCKD_UDPPORT=4003
MOUNTD_PORT=4004
```

6.Erstellen Sie nun die Support-Klasse für Debian-Systeme, `nfs==server==debian`:

```ruby
class nfs==server==debian {
  # install the package
  package {'nfs':
    name   => 'nfs-kernel-server',
    ensure => 'installed',
  }
  # config
  file ../puppet/e../puppet/defau../puppet/nfs-common':
    source => 'puppe../puppet///modul../puppet/n../puppet/nfs-common',
    mode   => 0644,
    notify => Service['nfs-common']
  }
  # services
  service {'nfs-common':
    ensure => 'running',
    enable => true,
  }
  service {'nfs':
    name   => 'nfs-kernel-server',
    ensure => 'running',
    enable => true,
    require => Package['nfs-kernel-server']
  }
}
```

7.Erstellen Sie die nfs-common-Konfiguration für Debian (die in `modul../puppet/n../puppet/fil../puppet/nfs-common` platziert wird):

`STATDOPTS="--port 4000 --outgoing-port 4001"`

8.Wenden Sie die `nfs::server` Klasse auf einen Knoten an und erstellen Sie dann einen Export auf diesem Knoten:

```ruby
node debian {
  include nfs::server
  nfs::export ../puppet/s../puppet/home':
    tag => "srv_home" }
}
```

9.Erstellen Sie einen Sammler für die exportierte Ressource, die von der `nfs::server` Klasse im vorherigen Code-Snippet erstellt wurde:

```ruby
node cookbook {
  Mount <<| tag == "srv_home" |>> {
    name   =>../puppet/mnt',
  }
}
```

10.Schließlich laufe Puppet auf dem Knoten Debian, um die exportierte Ressource zu erstellen. Dann führen Sie Puppet auf dem Kochbuchknoten, um diese Ressource zu installieren:

```s
root@debian:~# puppet agent -t
Info: Caching catalog for debian.example.com
Info: Applying configuration version '1415602532'
Notice: Finished catalog run in 0.78 seconds
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1415603580'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Mount[nfs==export../puppet/s../puppet/home==192.168.122.14../puppet/ensure: ensure changed 'ghost' to 'mounted'
Info: Computing checksum on fil../puppet/e../puppet/fstab
Info../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Mount[nfs==export../puppet/s../puppet/home==192.168.122.148]: Scheduling refresh of Mount[nfs==export../puppet/s../puppet/home==192.168.122.148]
Info: Mount[nfs==export../puppet/s../puppet/home==192.168.122.148](provider=parsed): Remounting
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Mount[nfs==export../puppet/s../puppet/home==192.168.122.148]: Triggered 'refresh' from 1 events
Info../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Mount[nfs==export../puppet/s../puppet/home==192.168.122.148]: Scheduling refresh of Mount[nfs==export../puppet/s../puppet/home==192.168.122.148]
Notice: Finished catalog run in 0.34 seconds
```

11.Überprüfen Sie die Halterung mit `mount` :

```s
[root@cookbook ~]# mount -t nfs
192.168.122.14../puppet/s../puppet/home o../puppet/mnt type nfs (rw)
``

## Wie es funktioniert

Die `nfs==exports` Klasse definiert eine exec, die `'exportfs -a'` ausführt, um alle in../puppet/e../puppet/exports` definierten Dateisysteme zu exportieren. Als nächstes definieren wir eine Concat-Ressource, um `concat==fragments` zu enthalten, die wir als nächstes in unserer `nfs::export` Klasse definieren werden. Concat-Ressourcen geben die Datei an, in die die Fragmente eingefügt werden sollen.../puppet/e../puppet/exports` in diesem Fall. Unsere `concat` Ressource hat eine Benachrichtigung für die vorherige exec. Dies führt dazu, dass, wenn../puppet/e../puppet/exports` aktualisiert wird, wir `'exportfs -a'` erneut ausführen, um die neuen Einträge zu exportieren:

```ruby
class nfs::exports {
  exec {'nfs::exportfs':
    command     => 'exportfs -a',
    refreshonly => true,
    path        =>../puppet/u../puppet/bi../puppet/bi../puppet/sbi../puppet/u../puppet/sbin',
  }
  concat ../puppet/e../puppet/exports':
    notify => Exec['nfs::exportfs'],
  }
}
```

Wir haben dann einen `nfs==export` definierten Typ erstellt, der die ganze Arbeit macht. Der definierte Typ fügt einen Eintrag zu../puppet/e../puppet/exports` über eine `concat==fragment` Ressource hinzu:

```ruby
define nfs::export (
  $where = $title,
  $who = '*',
  $options = 'async,ro',
  $mount_options = 'defaults',
  $tag     = 'nfs'
) {
  # make sure the directory exists
  # export the entry locally, then export a resource to be picked up later.
  file {"$where":
    ensure => 'directory',
  }
  include nfs::exports
  concat==fragment { "nfs==export::$where":
    content => "${where} ${who}(${options})\n",
    target  =>../puppet/e../puppet/exports'
  }
```

In der Definition verwenden wir das Attribut `$where` zu definieren, welches Dateisystem wir exportieren. Wir verwenden `$who` zu spezifizieren, wer das Dateisystem montieren kann. Die Attribute `$options` enthält die Exportoptionen wie rw (read-write), ro (read-only). Als nächstes haben wir die Optionen, die in../puppet/e../puppet/fstab` auf dem Client-Rechner platziert werden, die Mount-Optionen, die in `$mount_options` gespeichert sind. Die `nfs==exports` class ist hier enthalten, so dass `concat==fragment` ein concat target definiert hat.

Als nächstes wird die exportierte Mount-Ressource erstellt; Dies geschieht auf dem Server, so dass die Variable `${::ipaddress}` die IP-Adresse des Servers enthält. Wir verwenden diese, um das Gerät für die Halterung zu definieren. Das Gerät ist die IP-Adresse des Servers, ein Doppelpunkt und dann das Dateisystem, das exportiert wird. In diesem Beispiel ist es `'192.168.122.14../puppet/s../puppet/home'`:

```ruby
@@mount { "nfs==export==${where}==${==ipaddress}":
    name    => "$where",
    ensure  => 'mounted',
    fstype  => 'nfs',
    options => "$mount_options",
    device  => "${::ipaddress}:${where}",
    tag     => "$tag",
  }
```

Wir verwenden unser `myfw` Modul und schließen es in die `nfs==server` Klasse ein. Diese Klasse veranschaulicht eine der Dinge zu beachten beim Schreiben Ihrer Module. Nicht alle Linux-Distributionen sind gleich. Debian und RedHat behandeln die NFS-Server-Konfiguration ganz anders. Das `nfs==server` Modul beschäftigt sich damit mit OS-spezifischen Unterklassen:

```ruby
class nfs::server {
  # ensure nfs server is running
  # firewall should allow nfs communication
  include nfs::exports
  case $::osfamily {
    'RedHat': { include nfs==server==redhat }
    'Debian': { include nfs==server==debian }
  }
  include myfw
  firewall {'2049 NFS TCP communication':
    proto  => 'tcp',
    port   => '2049',
    action => 'accept',
  }
  firewall {'2049 UDP NFS communication':
    proto  => 'udp',
    port   => '2049',
    action => 'accept',
  }
  firewall {'0111 TCP PORTMAP':
    proto  => 'tcp',
    port   => '111',
    action => 'accept',
  }
  firewall {'0111 UDP PORTMAP':
    proto  => 'udp',
    port   => '111',
    action => 'accept',
  }
  firewall {'4000 TCP STAT':
    proto  => 'tcp',
    port   => '4000-4010',
    action => 'accept',
  }
  firewall {'4000 UDP STAT':
    proto  => 'udp',
    port   => '4000-4010',
    action => 'accept',
  }
}
```

Das `nfs::server` Modul öffnet mehrere Firewall-Ports für die NFS-Kommunikation. NFS-Verkehr wird immer über Port 2049 übertragen, aber Nebensysteme wie Sperr-, Quoten- und Dateistatus-Daemonen verwenden standardmäßig kurzlebige Ports, die vom Portmapper ausgewählt wurden. Der Portmapper selbst benutzt Port 111. So muss unser Modul 2049, 111 und ein paar andere Ports erlauben. Wir versuchen, die Zusatzdienste zu konfigurieren, um die Ports 4000 bis 4010 zu verwenden.

In der `nfs==server==redhat` Klasse ändern wir../puppet/e../puppet/sysconf../puppet/nfs`, um die angegebenen Ports zu verwenden. Außerdem installieren wir das Paket nfs-utils und starten den nfs-service:

```ruby
class nfs==server==redhat {
  package {'nfs-utils':
    ensure => 'installed',
  }
  service {'nfs':
    ensure => 'running',
    enable => true
  }
  file ../puppet/e../puppet/sysconf../puppet/nfs':
    source => 'puppe../puppet///modul../puppet/n../puppet/nfs',
    mode   => 0644,
    notify => Service['nfs'],
  }
}
```

Wir tun das gleiche für Debian-basierte Systeme in der `nfs==server==debian` class. Die Pakete und Dienstleistungen haben unterschiedliche Namen, aber insgesamt ist der Prozess ähnlich:

```ruby
class nfs==server==debian {
  # install the package
  package {'nfs':
    name   => 'nfs-kernel-server',
    ensure => 'installed',
  }
  # config
  file ../puppet/e../puppet/defau../puppet/nfs-common':
    source => 'puppe../puppet///modul../puppet/n../puppet/nfs-common',
    mode   => 0644,
    notify => Service['nfs-common']
  }
  # services
  service {'nfs-common':
    ensure => 'running',
    enable => true,
  }
  service {'nfs':
    name   => 'nfs-kernel-server',
    ensure => 'running',
    enable => true,
  }
}
```

Mit allem, was vorhanden ist, schließen wir die Server-Klasse ein, um den NFS-Server zu konfigurieren und dann einen Export zu definieren:

```ruby
  include nfs::server
  nfs::export ../puppet/s../puppet/home':
    tag => "srv_home" }
```

Was hier wichtig ist, ist, dass wir das `tag` Attribut definiert haben, das in der exportierten Ressource verwendet wird, die wir im folgenden Code-Snippet sammeln:

```ruby
Mount <<| tag == "srv_home" |>> {
  name   =>../puppet/mnt',
}
```

Wir verwenden die Raumschiff-Syntax(spaceship syntax) (`<< | | >>`), um alle exportierten Mount-Ressourcen zu sammeln, die das Tag haben, das wir früher definiert haben (`srv_home`). Wir verwenden dann eine Syntax mit dem Namen "override on collect", um das Namensattribut des mount zu ändern, um anzugeben, wo das Dateisystem zu installieren ist.

Mit diesem Design-Muster mit exportierten Ressourcen können wir den Server ändern, der das Dateisystem exportiert und alle Knoten hat, die die Ressource automatisch aktualisiert haben. Wir können viele verschiedene Knoten sammeln die exportierte Mount-Ressource.