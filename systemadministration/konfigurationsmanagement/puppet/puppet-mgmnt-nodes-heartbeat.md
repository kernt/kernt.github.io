---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet Management von heartbeat Nodes

Hochverfügbarkeitsdienste sind diejenigen, die den Ausfall einer einzelnen Maschine oder Netzwerkverbindung überleben können. Die primäre Technik für Hochverfügbarkeit ist Redundanz, sonst bekannt als Wurf Hardware auf das Problem. Obwohl der eventuelle Ausfall eines einzelnen Servers sicher ist, ist der gleichzeitige Ausfall von zwei Servern unwahrscheinlich, dass dies für die meisten Anwendungen eine gute Redundanz bietet.

Eine der einfachsten Möglichkeiten, um ein redundantes Paar von Servern zu erstellen, besteht darin, dass sie eine IP-Adresse mit Heartbeat teilen. Herzschlag ist ein Dämon, der auf beiden Maschinen läuft und regelmäßige Nachrichten - Herzschläge - zwischen den beiden tauscht. Ein Server ist der primäre und hat normalerweise die Ressource; In diesem Fall eine IP-Adresse (bekannt als virtuelle IP oder VIP). Wenn der sekundäre Server einen Herzschlag vom primären Server nicht erkennt, kann er die Adresse übernehmen und die Kontinuität des Dienstes sicherstellen. In real-world-Szenarien können Sie vielleicht mehr Maschinen im VIP beteiligt haben, aber für dieses Beispiel arbeiten zwei Maschinen gut genug.

In diesem Rezept werden wir zwei Maschinen in dieser Konfiguration mit Puppet einrichten, und ich werde erklären, wie man es benutzt, um einen Hochverfügbarkeitsdienst zur Verfügung zu stellen.

## Fertig werden

Du brauchst natürlich zwei Maschinen und eine zusätzliche IP-Adresse als VIP verwenden. Sie können dies in der Regel von Ihrem ISP anfordern, wenn nötig. In diesem Beispiel verwende ich Maschinen namens Kochbuch und Kochbuch2, wobei das Kochbuch das primäre ist. Wir werden die Hosts zur Heartbeat-Konfiguration hinzufügen.

## Wie es geht

Gehen Sie folgendermaßen vor, um das Beispiel zu erstellen:

1.Erstellen Sie die Datei `modul../puppet/heartbe../puppet/manifes../puppet/init.pp` mit folgendem Inhalt:

```ruby
# Manage Heartbeat
class heartbeat {
  package { 'heartbeat':
    ensure => installed,
  }

  service { 'heartbeat':
    ensure  => running,
    enable  => true,
    require => Package['heartbeat'],
  }

  file {../puppet/e../puppet/ha../puppet/authkeys':
    content => "auth 1\n1 sha1 TopSecret",
    mode    => '0600',
    require => Package['heartbeat'],
    notify  => Service['heartbeat'],
  }
  include myfw
  firewall {'0694 Allow UDP ha-cluster':
    proto  => 'udp',
    port   => 694,
    action => 'accept',
  }
}
```

2.Erstellen Sie die Datei `modul../puppet/heartbe../puppet/manifes../puppet/vip.pp` mit folgendem Inhalt:

```ruby
# Manage a specific VIP with Heartbeat
class
  heartbeat::vip($node1,$node2,$ip1,$ip2,$vip,$interface='eth0:1') {
  include heartbeat

  file {../puppet/e../puppet/ha../puppet/haresources':
    content => "${node1} IPaddr::${vi../puppet/${interface}\n",
    require => Package['heartbeat'],
    notify  => Service['heartbeat'],
  }

  file {../puppet/e../puppet/ha../puppet/ha.cf':
    content => template('heartbe../puppet/vip.ha.cf.erb'),
    require => Package['heartbeat'],
    notify  => Service['heartbeat'],
  }
}
```

3.Erstellen Sie die Datei `modul../puppet/heartbe../puppet/templat../puppet/vip.ha.cf.erb` mit folgendem Inhalt:

```s
use_logd yes
udpport 694
autojoin none
ucast eth0 <%= @ip1 %>
ucast eth0 <%= @ip2 %>
keepalive 1
deadtime 10
warntime 5
auto_failback off
node <%= @node1 %>
node <%= @node2 %>
```

4.Ändern Sie Ihre `site.pp` Datei wie folgt. Ersetzen Sie die `ip1` und `ip2` Adressen durch die primären IP-Adressen Ihrer beiden Knoten, `vip` mit der virtuellen IP-Adresse, die Sie verwenden werden, und `node1` und `node2` mit den Hostnamen der beiden Knoten. (Heartbeat verwendet den vollqualifizierten Domänennamen eines Knotens, um festzustellen, ob es sich um ein Mitglied des Clusters handelt, also sollten die Werte für `node1` und `node2` mit dem übereinstimmen, was von `facter fqdn` auf jeder Maschine gegeben wird.):

```ruby
node cookbook,cookbook2 {
  class { 'heartbeat::vip':
    ip1   => '192.168.122.132',
    ip2   => '192.168.122.133',
    node1 => 'cookbook.example.com',
    node2 => 'cookbook2.example.com',
    vip   => '192.168.122.2../puppet/24',
  }
}
```

5.Run Puppet auf jedem der beiden Server:

```s
[root@cookbook2 ~]# puppet agent -t
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for cookbook2.example.com
Info: Applying configuration version '1415517914'
Notice../puppet/Stage[mai../puppet/Heartbe../puppet/Package[heartbea../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0000 Allow all traffic on loopbac../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0001 Allow all ICM../puppet/ensure: created
Notice../puppet/Fil../puppet/e../puppet/sysconf../puppet/iptable../puppet/seluser: seluser changed 'unconfined_u' to 'system_u'
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0022 Allow all TCP on port 22 (ssh../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Heartbeat::V../puppet/Fil../puppet/e../puppet/ha../puppet/haresource../puppet/ensure: defined content as '{md5}fb9f5d9d2b26e3bddf681676d8b2129c'
Info../puppet/Stage[mai../puppet/Heartbeat::V../puppet/Fil../puppet/e../puppet/ha../puppet/haresources]: Scheduling refresh of Service[heartbeat]
Notice../puppet/Stage[mai../puppet/Heartbeat::V../puppet/Fil../puppet/e../puppet/ha../puppet/ha.c../puppet/ensure: defined content as '{md5}84da22f7ac1a3629f69dcf29ccfd8592'
Info../puppet/Stage[mai../puppet/Heartbeat::V../puppet/Fil../puppet/e../puppet/ha../puppet/ha.cf]: Scheduling refresh of Service[heartbeat]
Notice../puppet/Stage[mai../puppet/Heartbe../puppet/Service[heartbea../puppet/ensure: ensure changed 'stopped' to 'running'
Info../puppet/Stage[mai../puppet/Heartbe../puppet/Service[heartbeat]: Unscheduling refresh on Service[heartbeat]
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0002 Allow all established traffi../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::Po../puppet/Firewall[9999 Drop all other traffi../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Heartbe../puppet/Firewall[0694 Allow UDP ha-cluste../puppet/ensure: created
Notice: Finished catalog run in 12.64 seconds
```

6.Überprüfen Sie, dass das VIP auf einem der Knoten läuft (es sollte auf Kochbuch an diesem Punkt sein, beachten Sie, dass Sie den `ip` Befehl verwenden müssen, `ifconfig` wird die Adresse nicht anzeigen):

7.Wie wir sehen können, hat das Kochbuch die `eth0:1` Schnittstelle aktiv. Wenn du den Herzschlag auf `cookbook` hörst, wird `cookbook2` `eth0:1` erstellen und übernehmen:

```s
[root@cookbook2 ~]# ip a show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    li../puppet/ether 52:54:00:ee:9c:fa brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1../puppet/24 brd 192.168.122.255 scope global eth0
    inet 192.168.122.2../puppet/24 brd 192.168.122.255 scope global secondary eth0:1
    inet6 fe80::5054:ff:feee:9c../puppet/64 scope link
       valid_lft forever preferred_lft forever
```

### Wie es funktioniert…

Wir müssen zuerst Heartbeat installieren, mit der Heartbeat-Klasse:

```ruby
# Manage Heartbeat
class heartbeat {
  package { 'heartbeat':
    ensure => installed,
  }
  ...
}
```

Als nächstes verwenden wir die `heartbeat::vip` Klasse, um eine bestimmte virtuelle IP zu verwalten:

```s
# Manage a specific VIP with Heartbeat
class
  heartbeat::vip($node1,$node2,$ip1,$ip2,$vip,$interface='eth0:1') {
  include heartbeat
```

Wie Sie sehen können, enthält die Klasse einen `interface` parameter; Standardmäßig wird das VIP auf `eth0:1` konfiguriert, aber wenn du eine andere Schnittstelle verwenden musst, kannst du es mit diesem Parameter übergeben.

Jedes Paar von Servern, die wir mit einer virtuellen IP konfigurieren, verwendet die `heartbeat::vip` Klasse mit denselben Parametern. Diese werden verwendet, um die `haresources` Datei zu erstellen:

```ruby
file {../puppet/e../puppet/ha../puppet/haresources':
  content => "${node1} IPaddr::${vi../puppet/${interface}\n",
  notify  => Service['heartbeat'],
  require => Package['heartbeat'],
}
```

Dies sagt Heartbeat über die Ressource, die es verwalten sollte (das ist eine Heartbeat-Ressource, wie eine IP-Adresse oder ein Service, keine Puppet-Ressource). Die resultierende `haresources` Datei könnte wie folgt aussehen:
`cookbook.example.com IPaddr::192.168.122.2../puppet/../puppet/eth0:1`

Die Datei wird von Heartbeat wie folgt interpretiert:

* `cookbook.example.com`: Dies ist der Name des primären Knotens, der der Standardbesitzer der Ressource sein sollte

* `IPaddr`: Dies ist die Art der Ressource zu verwalten; In diesem Fall eine IP-Adresse

* `192.168.122.2../puppet/24`: Dies ist der Wert für die IP-Adresse

* `eth0:1` :Dies ist die virtuelle Schnittstelle, die mit der verwalteten IP-Adresse konfiguriert werden soll

Wir werden auch die `ha.cf` Datei erstellen, die Heartbeat mitteilt, wie man zwischen Clusterknoten kommuniziert:

```ruby
file {../puppet/e../puppet/ha../puppet/ha.cf':
  content => template('heartbe../puppet/vip.ha.cf.erb'),
  notify  => Service['heartbeat'],
  require => Package['heartbeat'],
}
```

Dazu verwenden wir die Vorlagendatei:
s

```s
use_logd yes
udpport 694
autojoin none
ucast eth0 <%= @ip1 %>
ucast eth0 <%= @ip2 %>
keepalive 1
deadtime 10
warntime 5
auto_failback off
node <%= @node1 %>
node <%= @node2 %>
```

Die interessanten Werte sind hier die IP-Adressen der beiden Knoten (`ip1` und `ip2`) und die Namen der beiden Knoten (`node1` und `node2`).

Schließlich schaffen wir eine Instanz von `heartbeat::vip` auf beiden Maschinen und übergeben sie einen identischen Satz von Parametern wie folgt:

```ruby
class { 'heartbeat::vip':
  ip1   => '192.168.122.132',
  ip2   => '192.168.122.133',
  node1 => 'cookbook.example.com',
  node2 => 'cookbook2.example.com',
  vip   => '192.168.122.2../puppet/24',
}
```

## Es gibt mehr

Wenn Heartbeat wie im Beispiel beschrieben eingerichtet ist, wird die virtuelle IP-Adresse standardmäßig auf `cookbook` konfiguriert. Wenn etwas passiert, um dies zu stören (z. B. wenn Sie das `cookbook` stoppen oder neu starten oder den `heartbeat` Dienst beenden oder das Gerät Netzwerkkonnektivität verliert), wird `cookbook2` sofort die virtuelle IP übernehmen.

Die Einstellung `auto_failback` in `ha.cf` regelt, was als nächstes passiert. Wenn `auto_failback` auf `on` gesetzt ist, wenn das `cookbook` noch einmal verfügbar ist, übernimmt es automatisch die IP-Adresse. Ohne `auto_failback` bleibt die IP-Adresse dort, wo es ist, bis du es manuell wieder schlägst (indem du zum Beispiel `heartbeart` auf `cookbook2` beendest).

Eine gemeinsame Verwendung für eine Heartbeat-verwaltete virtuelle IP ist die Bereitstellung einer hochverfügbaren Website oder Service. Dazu müssen Sie den DNS-Namen für den Dienst (z. B. `cat-pictures.com`) festlegen, um auf die virtuelle IP zu verweisen. Anfragen für den Service werden an denjenigen der beiden Server weitergeleitet, die derzeit über die virtuelle IP verfügen. Wenn dieser Server nach unten gehen sollte, werden Anfragen auf die andere gehen, ohne sichtbare Unterbrechung im Dienst für Benutzer.

Heartbeat arbeitet für das vorhergehende Beispiel gut, ist aber in dieser Form nicht weit verbreitet. Heartbeat funktioniert nur in zwei Knotenclustern; Für n-Knoten-Cluster sollte das neuere Herzschrittmacher-Projekt verwendet werden. Weitere Informationen über Heartbeat, Herzschrittmacher, Corosync und andere [Clustering-Pakete](htt../puppet//www.linux-ha.o../puppet/wi../puppet/Main_Page).

Das Verwalten der Clusterkonfiguration ist ein Bereich, in dem exportierte Ressourcen nützlich sind. Jeder Knoten in einem Cluster würde Informationen über sich selbst exportieren, die dann von den anderen Mitgliedern des Clusters gesammelt werden konnten. Mit dem puppetlabs-concat-Modul kannst du eine Konfigurationsdatei mit exportierten Concat-Fragmenten aus allen Knoten im Cluster aufbauen.

Denken Sie daran, die Forge zu betrachten, bevor Sie Ihr eigenes Modul starten. Wenn nichts anderes, bekommst du einige Ideen, die du in deinem eigenen Modul verwenden kannst. Corosync kann mit dem Puppet Labs [Modul](http../puppet//forge.puppetlabs.c../puppet/puppetla../puppet/corosync) verwaltet werden.