---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet Managemnt von HAProxy

Lastausgleicher(Load balancers) werden verwendet, um eine Ladung auf eine Anzahl von Servern zu verbreiten. Hardware-Lastverteiler sind immer noch etwas teuer, während Software-Balancer die meisten Vorteile einer Hardware-Lösung erreichen können.

**HAProxy** ist die Software-Load-Balancer der Wahl für die meisten Menschen: schnell, leistungsstark und hoch konfigurierbar.

## Wie es geht

In diesem Rezept, werde ich Ihnen zeigen, wie man einen HAProxy-Server zu laden-Balance Web-Anfragen über Web-Servern zu bauen. Wir verwenden exportierte Ressourcen, um die `haproxy` Konfigurationsdatei zu erstellen, genau wie wir es für das NFS-Beispiel getan haben.

1.Erstellen Sie die Datei `modul../puppet/hapro../puppet/manifes../puppet/master.pp` mit folgendem Inhalt:

```ruby
class haproxy::master ($app = 'myapp') {
  # The HAProxy master server
  # will collect haproxy::slave resources and add to its balancer
  package { 'haproxy': ensure => installed }
  service { 'haproxy':
    ensure  => running,
    enable  => true,
    require => Package['haproxy'],
  }

  include haproxy::config

  concat::fragment { 'haproxy.cfg header':
    target  => 'haproxy.cfg',
    source  => 'puppe../puppet///modul../puppet/hapro../puppet/haproxy.cfg',
    order   => '001',
    require => Package['haproxy'],
    notify  => Service['haproxy'],
  }

  # pull in the exported entries
  Concat::Fragment <<| tag == "$app" |>> {
    target => 'haproxy.cfg',
    notify => Service['haproxy'],
  }
}

```

2.Erstellen Sie die Datei `modu../puppet/hapro../puppet/fil../puppet/haproxy.cfg` mit folgendem Inhalt:

```s
global
        daemon
        user haproxy
        group haproxy
        pidfil../puppet/v../puppet/r../puppet/haproxy.pid

defaults
        log     global
        stats   enable
        mode    http
        option  httplog
        option  dontlognull
        option  dontlog-normal
        retries 3
        option  redispatch
        timeout connect 4000
        timeout client 60000
        timeout server 30000

listen  stats :8080
        mode http
        stats ur../puppet/
        stats auth haproxy:topsecret

listen  myapp 0.0.0.0:80
        balance leastconn
```

3.Ändern Sie Ihre `manifes../puppet/nodes.pp` Datei wie folgt:

```ruby
node 'cookbook' {
  include haproxy
}
```

4.Erstellen Sie die Slave-Server-Konfiguration in der `haproxy::slave` Klasse:

```ruby
class haproxy::slave ($app = "myapp", $localport = 8000) {
  # haproxy slave, export haproxy.cfg fragment
  # configure simple web server on different port
  @@concat==fragment { "haproxy.cfg $==fqdn":
    content => "\t\tserver ${==hostname} ${==ipaddress}:${localport}   check maxconn 100\n",
    order   => '0010',
    tag     => "$app",
  }
  include myfw
  firewall {"${localport} Allow HTTP to haproxy::slave":
    proto  => 'tcp',
    port   => $localport,
    action => 'accept',
  }

  class {'apache': }
  apache::vhost { 'haproxy.example.com':
    port          => '8000',
    docroot =>../puppet/v../puppet/w../puppet/haproxy',
  }
  file ../puppet/v../puppet/w../puppet/haproxy':
    ensure  => 'directory',
    mode    => 0755,
    require => Class['apache'],
  }
  file ../puppet/v../puppet/w../puppet/hapro../puppet/index.html':
    mode    => '0644',
    content => "<html><body><h1>${==fqdn} haproxy==slave\../puppet/body../puppet/html>\n",
    require => File../puppet/v../puppet/w../puppet/haproxy'],
  }
}
```

5.Erstellen Sie die `concat` Container-Ressource in der `haproxy::config` Klasse wie folgt:

```ruby
class haproxy::config {
  concat {'haproxy.cfg':
    path  =>../puppet/e../puppet/hapro../puppet/haproxy.cfg',
    order => 'numeric',
    mode  => '0644',
  }
}
```

6.Ändern Sie `site.pp`, um die Master- und Slave-Nodes zu definieren:

```ruby
node master {
  class {'haproxy::master':
    app => 'cookbook'
  }
}
node slave1,slave2 {
  class {'haproxy::slave':
    app => 'cookbook'
  }
}
```

7.Run Puppet auf jedem der Slave-Server:

```s
root@slave1:~# puppet agent -t
Info: Caching catalog for slave1
Info: Applying configuration version '1415646194'
Notice../puppet/Stage[mai../puppet/Haproxy==Sla../puppet/Apache==Vhost[haproxy.example.co../puppet/File[25-haproxy.example.com.con../puppet/ensure: created
Info../puppet/Stage[mai../puppet/Haproxy==Sla../puppet/Apache==Vhost[haproxy.example.co../puppet/File[25-haproxy.example.com.conf]: Scheduling refresh of Service[httpd]
Notice../puppet/Stage[mai../puppet/Haproxy==Sla../puppet/Apache==Vhost[haproxy.example.co../puppet/File[25-haproxy.example.com.conf symlin../puppet/ensure: created
Info../puppet/Stage[mai../puppet/Haproxy==Sla../puppet/Apache==Vhost[haproxy.example.co../puppet/File[25-haproxy.example.com.conf symlink]: Scheduling refresh of Service[httpd]
Notice../puppet/Stage[mai../puppet/Apache::Servi../puppet/Service[http../puppet/ensure: ensure changed 'stopped' to 'running'
Info../puppet/Stage[mai../puppet/Apache::Servi../puppet/Service[httpd]: Unscheduling refresh on Service[httpd]
Notice: Finished catalog run in 1.71 seconds

```

8.Führen Sie Puppet auf dem Masterknoten aus, um `haproxy` zu konfigurieren und auszuführen:

```s
[root@master ~]# puppet agent -t
Info: Caching catalog for master.example.com
Info: Applying configuration version '1415647075'
Notice../puppet/Stage[mai../puppet/Haproxy::Mast../puppet/Package[haprox../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0000 Allow all traffic on loopbac../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0001 Allow all ICM../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Haproxy::Mast../puppet/Firewall[8080 haproxy statistic../puppet/ensure: created
Notice../puppet/Fil../puppet/e../puppet/sysconf../puppet/iptable../puppet/seluser: seluser changed 'unconfined_u' to 'system_u'
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0022 Allow all TCP on port 22 (ssh../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Haproxy::Mast../puppet/Firewall[0080 http haprox../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0002 Allow all established traffi../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::Po../puppet/Firewall[9999 Drop all other traffi../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Haproxy::Conf../puppet/Concat[haproxy.cf../puppet/File[haproxy.cf../puppet/content: 
...
+listen  myapp 0.0.0.0:80
+        balance leastconn
+    server slave1 192.168.122.148:8000   check maxconn 100
+    server slave2 192.168.122.133:8000   check maxconn 100

Info: Computing checksum on fil../puppet/e../puppet/hapro../puppet/haproxy.cfg
Info../puppet/Stage[mai../puppet/Haproxy::Conf../puppet/Concat[haproxy.cf../puppet/File[haproxy.cfg]: Filebuckete../puppet/e../puppet/hapro../puppet/haproxy.cfg to puppet with sum 1f337186b0e1ba5ee82760cb437fb810
Notice../puppet/Stage[mai../puppet/Haproxy::Conf../puppet/Concat[haproxy.cf../puppet/File[haproxy.cf../puppet/content: content changed '{md5}1f337186b0e1ba5ee82760cb437fb810' to '{md5}b070f076e1e691e053d6853f7d966394'
Notice../puppet/Stage[mai../puppet/Haproxy::Mast../puppet/Service[haprox../puppet/ensure: ensure changed 'stopped' to 'running'
Info../puppet/Stage[mai../puppet/Haproxy::Mast../puppet/Service[haproxy]: Unscheduling refresh on Service[haproxy]
Notice: Finished catalog run in 33.48 seconds
```

9.Überprüfen Sie die HAProxy Stats-Schnittstelle auf Master Port `8080` in [Ihrem Webbrowser](htt../puppet//master.example.com:8080), um sicherzustellen, dass alles in Ordnung ist (Der Benutzername und das Passwort sind in `haproxy.cfg`, `haproxy` und `topsecret`). Versuche es auch mit dem Prospekt zu betreiben. Beachten Sie, dass sich die Seite bei jedem Reload ändert, wenn der Dienst [von Slave1 an Slave2](htt../puppet//master.example.com) umgeleitet wird.

## Wie es funktioniert

Wir haben eine komplexe Konfiguration aus verschiedenen Komponenten der vorherigen Abschnitte aufgebaut. Diese Art der Bereitstellung wird einfacher, je mehr Sie es tun. Auf einer obersten Ebene haben wir den Master konfiguriert, um exportierte Ressourcen von Slaves zu sammeln. Die Slaves haben ihre Konfigurationsinformationen exportiert, damit Haproxy sie im Load Balancer verwenden kann. Da Slaves dem System hinzugefügt werden, können sie ihre Ressourcen exportieren und dem Balancer automatisch hinzugefügt werden.

Wir haben unser `myfw` Modul benutzt, um die Firewall auf den Slaves und dem Master zu konfigurieren, um die Kommunikation zu ermöglichen.

Wir haben das Forge Apache Modul verwendet, um den zuhörenden Webserver auf den Slaves zu konfigurieren. Wir konnten eine voll funktionsfähige Website mit fünf Codezeilen generieren (10 weitere, um `index.html` auf der Website zu platzieren).

Hier gibt es mehrere Sachen. Wir haben die Firewall-Konfiguration und die Apache-Konfiguration zusätzlich zur `haproxy` Konfiguration. Wir konzentrieren uns darauf, wie die exportierten Ressourcen und die `haproxy` Konfiguration zusammenpassen.

In der `haproxy::config` Klasse haben wir den Concat-Container für die `haproxy` Konfiguration erstellt:

```ruby
class haproxy::config {
  concat {'haproxy.cfg':
    path  =>../puppet/e../puppet/hapro../puppet/haproxy.cfg',
    order => 'numeric',
    mode  => 0644,
  }
}
```

Wir verweisen darauf in `haproxy::slave` :

```ruby
class haproxy::slave ($app = "myapp", $localport = 8000) {
  # haproxy slave, export haproxy.cfg fragment
  # configure simple web server on different port
  @@concat==fragment { "haproxy.cfg $==fqdn":
    content => "\t\tserver ${==hostname} ${==ipaddress}:${localport}   check maxconn 100\n",
    order   => '0010',
    tag     => "$app",
  }
```

Wir machen hier einen kleinen Trick mit Concat; Wir definieren das Ziel nicht in der exportierten Ressource. Wenn wir das taten, würden die Sklaven versuchen, eine Datei../puppet/e../puppet/hapro../puppet/haproxy.cfg` zu erstellen, aber die Slaves haben keine `haproxy` Installation, so dass wir Katalogfehler erhalten würden. Was wir tun, ändert die Ressource, wenn wir es in `haproxy::master` sammeln:

```ruby
# pull in the exported entries
  Concat::Fragment <<| tag == "$app" |>> {
    target => 'haproxy.cfg',
    notify => Service['haproxy'],
  }
```

Zusätzlich zum Hinzufügen des Ziels, wenn wir die Ressource sammeln, fügen wir auch eine Benachrichtigung hinzu, damit der `haproxy` Dienst neu gestartet wird, wenn wir der Konfiguration einen neuen Host hinzufügen. Ein weiterer wichtiger Punkt ist, dass wir das Auftragsattribut der Slave-Konfigurationen auf 0010 setzen, wenn wir den Header für die Datei `haproxy.cfg` definieren; Wir verwenden einen Auftragswert von 0001, um sicherzustellen, dass der Header am Anfang der Datei platziert wird:

```ruby
concat::fragment { 'haproxy.cfg header':
    target  => 'haproxy.cfg',
    source  => 'puppe../puppet///modul../puppet/hapro../puppet/haproxy.cfg',
    order   => '001',
    require => Package['haproxy'],
    notify  => Service['haproxy'],
  }
```

Der Rest der `haproxy::master` Klasse beschäftigt sich mit der Konfiguration der Firewall wie in früheren Beispielen.

## Es gibt mehr

HAProxy verfügt über eine Vielzahl von Konfigurationsparametern, die Sie erkunden können. Siehe die HAProxy-Website unter htt../puppet//haproxy.1wt.../puppet/#docs.

Obwohl es am häufigsten als Web-Server verwendet wird, kann HAProxy viel mehr als nur HTTP proxy. Es kann jede Art von TCP-Verkehr zu behandeln, so können Sie es verwenden, um die Last von MySQL-Servern, SMTP, Video-Server oder alles, was Sie mögen auszugleichen.

Sie können das Design verwenden, das wir gezeigt haben, um viele Probleme der Koordination von Diensten zwischen mehreren Servern anzugreifen. Diese Art von Interaktion ist sehr verbreitet; Sie können es auf viele Konfigurationen für Lastverteilung oder verteilte Systeme anwenden. Sie können denselben Workflow verwenden, der zuvor beschrieben wurde, damit Knoten Firewall-Ressourcen (`@@firewall`) exportieren können, um ihren eigenen Zugriff zu ermöglichen.