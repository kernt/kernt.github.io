---
tags:
  - sicherheit
  - iptables
  - puppet
  - konfigurationsmanagement
puppet:
---
# Puppet management von iptabels

In diesem Kapitel werden wir beginnen, Dienste zu konfigurieren, die eine Kommunikation zwischen Hosts über das Netzwerk erfordern. Die meisten Linux-Distributionen werden standardmäßig auf eine Host-basierte Firewall, iptables laufen. Wenn Sie möchten, dass Ihre Hosts miteinander kommunizieren, haben Sie zwei Möglichkeiten: deaktivieren Sie iptables oder konfigurieren Sie iptables, um die Kommunikation zu ermöglichen.

Ich ziehe es vor, die iptables einzuschalten und den Zugriff zu konfigurieren. Keeping iptables ist nur eine weitere Schicht auf deiner Verteidigung über das Netzwerk. Iptables ist keine magische Kugel, die Ihr System sicher macht, aber es wird den Zugriff auf Dienste blockieren, die Sie nicht beabsichtigen, sich dem Netzwerk auszusetzen.

Das Konfigurieren von iptables ist eine komplizierte Aufgabe, die eine tiefe Kenntnis der Vernetzung erfordert. Das hier vorgestellte Beispiel ist eine Vereinfachung. Wenn Sie mit iptables nicht vertraut sind, schlage ich Ihnen vor, dass Sie iptables fortsetzen, bevor Sie fortfahren. Weitere Informationen finden Sie unter htt../puppet//wiki.centos.o../puppet/HowT../puppet/Netwo../puppet/IPTables oder http../puppet//help.ubuntu.c../puppet/communi../puppet/IptablesHowTo.

## Fertig werden

In den folgenden Beispielen verwenden wir das Puppet Labs Firewall Modul, um iptables zu konfigurieren. Vorbereiten, indem Sie das Modul in Ihr Git-Repository mit dem `puppet modul install`:

```s
t@mylaptop ~ $ puppet module install -i../puppet/pupp../puppet/modules puppetlabs-firewall
Notice: Preparing to install int../puppet/ho../puppet/thom../puppet/pupp../puppet/modules ...
Notice: Downloading from http../puppet//forgeapi.puppetlabs.com ...
/ho../puppet/thom../puppet/pupp../puppet/modules
└── puppetlabs-firewall (v1.2.0)
```

## Wie es geht

Um das Firewall-Modul zu konfigurieren, müssen wir einen Satz von Regeln erstellen, der vor allen anderen Regeln angewendet wird. Als einfaches Beispiel erstellen wir folgende Regeln:

* Erlaube allen Verkehr auf der Loopback (Lo) -Schnittstelle

* Erlaube allen ICMP-Verkehr

* Erlaube allen Verkehr, der Teil einer etablierten Verbindung ist (ESTABLISHED, RELATED)

* Erlaube alle TCP-Verkehr zu Port 22 (ssh)

Wir erstellen eine `myfw` (meine Firewall) Klasse, um das Firewallmodul zu konfigurieren. Wir werden dann die `myfw` Klasse an einen Knoten anwenden, um iptables auf diesem Knoten konfiguriert zu haben:

1.Erstellen Sie eine Klasse, um diese Regeln zu enthalten und nennen Sie es `myfw::pre` :

```ruby
class myfw::pre {
  Firewall {
    require => undef,
  }
  firewall { '0000 Allow all traffic on loopback':
    proto => 'all',
    iniface => 'lo',
    action => 'accept',
  }
  firewall { '0001 Allow all ICMP':
    proto => 'icmp',
    action => 'accept',
  }
  firewall { '0002 Allow all established traffic':
    proto => 'all',
    state => ['RELATED', 'ESTABLISHED'],
    action => 'accept',
  }
  firewall { '0022 Allow all TCP on port 22 (ssh)':
    proto => 'tcp',
    port => '22',
    action => 'accept',
  }
}
```

2.Wenn der Verkehr nicht mit einer der vorherigen Regeln übereinstimmt, wollen wir eine letzte Regel, die den Verkehr fallen lässt. Erstellen Sie die Klasse `myfw::post`, um die Standard-Drop-Regel zu enthalten:

```ruby
class myfw::post {
  firewall { '9999 Drop all other traffic':
    proto  => 'all',
    action => 'drop',
    before => undef,
  } 
}
```

3.Erstellen Sie eine `myfw` Klasse, die `myfw==pre` und `myfw==post` enthält, um die Firewall zu konfigurieren:

```ruby
class myfw {
  include firewall
  # our rulesets
  include myfw::post
  include myfw::pre

  # clear all the rules
  resources { "firewall":
    purge => true
  }

  # resource defaults
  Firewall {
    before => Class['myfw::post'],
    require => Class['myfw::pre'],
  }
}
```

4.Bringt die `myfw` Klasse zu einer Knotendefinition an. Ich mache das zu meinem Kochbuchknoten:

```ruby
node cookbook {
  include myfw
}
```

5.Führen Sie Puppet auf Kochbuch, um zu sehen, ob die Firewall-Regeln angewendet wurden:

```s
[root@cookbook ~]# puppet agent -t
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1415512948'
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[000 Allow all traffic on loopbac../puppet/ensure: created
Notice../puppet/Fil../puppet/e../puppet/sysconf../puppet/iptable../puppet/seluser: seluser changed 'unconfined_u' to 'system_u'
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0001 Allow all ICM../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0022 Allow all TCP on port 22 (ssh../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::P../puppet/Firewall[0002 Allow all established traffi../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Myfw::Po../puppet/Firewall[9999 Drop all other traffi../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/My../puppet/Firewall[9003 49bcd611c61bdd18b235cea46ef04fa../puppet/ensure: removed
Notice: Finished catalog run in 15.65 seconds
```

6.Überprüfen Sie die neuen Regeln mit `iptables-save`:

```s
# Generated by iptables-save v1.4.7 on Sun Nov  9 01:18:30 2014
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [74:35767]
-A INPUT -i lo -m comment --comment "0000 Allow all traffic on loopback" -j ACCEPT 
-A INPUT -p icmp -m comment --comment "0001 Allow all ICMP" -j ACCEPT 
-A INPUT -m comment --comment "0002 Allow all established traffic" -m state --state RELATED,ESTABLISHED -j ACCEPT 
-A INPUT -p tcp -m multiport --ports 22 -m comment --comment "022 Allow all TCP on port 22 (ssh)" -j ACCEPT 
-A INPUT -m comment --comment "9999 Drop all other traffic" -j DROP 
COMMIT
# Completed on Sun Nov  9 01:18:30 2014
```

## Wie es funktioniert

Dies ist ein großartiges Beispiel dafür, wie man Metaparameter einsetzt, um eine komplexe Bestellung mit wenig Aufwand zu erreichen. Unser `myfw` Modul erreicht folgende Konfiguration:

![iptabe-class](http../puppet//www.packtpub.c../puppet/graphi../puppet/97817882976../puppet/graphi../puppet/4882OS_08_01.jpg)

Alle Regeln in der `myfw==pre` Klasse sind garantiert, um vor irgendwelchen anderen Firewall-Regeln zu kommen, die wir definieren. Die Regeln in `myfw==post` sind garantiert nach irgendwelchen anderen Firewall-Regeln zu kommen. Also, wir haben die Regeln in `myfw==pre` first, dann alle anderen Regeln, gefolgt von den Regeln in `myfw==post`.

Unsere Definition für die myfw-Klasse setzt diese Abhängigkeit mit Ressourcenvorgaben ein:

```ruby
  # resource defaults
  Firewall {
    before => Class['myfw::post'],
    require => Class['myfw::pre'],
  }
```

Diese Vorgaben sagen zuerst Puppet, dass jede Firewall-Ressource vor irgendetwas in der `myfw==post` Klasse ausgeführt werden soll. Zweitens erzählen sie der Puppe, dass jede Firewall-Ressource verlangt, dass die Ressourcen in `myfw==pre` bereits ausgeführt werden.

Als wir die `myfw==pre` Klasse definiert haben, haben wir die Require-Anweisung in einem Ressourcen-Standard für Firewall-Ressourcen entfernt. Damit wird sichergestellt, dass sich die Ressourcen innerhalb der `myfw==pre` class nicht vor der Ausführung erfordern (Puppet wird beschweren, dass wir eine zyklische Abhängigkeit anders erstellt haben):

```ruby
Firewall {
    require => undef,
  }
```

Wir verwenden den gleichen Trick in unserem `myfw::post` Definition. In diesem Fall haben wir nur eine einzige Regel in der Post-Klasse, also entfernen wir einfach die `before` Anforderung:

```ruby
firewall { '9999 Drop all other traffic':
    proto  => 'all',
    action => 'drop',
    before => undef,
  }
```

Schließlich schließen wir eine Regel ein, um alle vorhandenen iptables Regeln auf dem System zu löschen. Wir tun dies, um sicherzustellen, dass wir eine konsistente Reihe von Regeln haben Nur Regeln, die in der Puppe definiert sind, bleiben bestehen:

```ruby
# clear all the rules
resources { "firewall":
  purge => true
}
```

## Es gibt mehr

Wie wir angedeutet haben, können wir nun Firewall-Ressourcen in unseren Manifests definieren und sie nach der Initialisierungsregeln (`myfw==pre`) auf die iptables-Konfiguration angewendet haben, aber vor dem endgültigen Drop (`myfw==post`). Um beispielsweise den http-Verkehr auf unserem Kochbuch zuzulassen, ändern Sie die Knotendefinition wie folgt:

```ruby
  include myfw
  firewall {'0080 Allow HTTP':
    proto  => 'tcp',
    action => 'accept',
    port  => 80,
  }
```

Puppet run auf dem cookbook node:

```s
[root@cookbook ~]# puppet agent -t
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1415515392'
Notice../puppet/Fil../puppet/e../puppet/sysconf../puppet/iptable../puppet/seluser: seluser changed 'unconfined_u' to 'system_u'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Firewall[0080 Allow HTT../puppet/ensure: created
Notice: Finished catalog run in 2.74 seconds
```

Vergewissern Sie sich, dass die neue Regel nach der letzten myfw :: pre-Regel hinzugefügt wurde (Port 22, ssh):

```s
[root@cookbook ~]# iptables-save
# Generated by iptables-save v1.4.7 on Sun Nov  9 01:46:38 2014
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [41:26340]
-A INPUT -i lo -m comment --comment "0000 Allow all traffic on loopback" -j ACCEPT 
-A INPUT -p icmp -m comment --comment "0001 Allow all ICMP" -j ACCEPT 
-A INPUT -m comment --comment "0002 Allow all established traffic" -m state --state RELATED,ESTABLISHED -j ACCEPT 
-A INPUT -p tcp -m multiport --ports 22 -m comment --comment "0022 Allow all TCP on port 22 (ssh)" -j ACCEPT 
-A INPUT -p tcp -m multiport --ports 80 -m comment --comment "0080 Allow HTTP" -j ACCEPT 
-A INPUT -m comment --comment "9999 Drop all other traffic" -j DROP 
COMMIT
# Completed on Sun Nov  9 01:46:38 2014
```

### Tip

Das Puppet Labs Firewall Modul hat einen eingebauten Begriff der Ordnung, alle unsere Firewall Resource Titel beginnen mit einer Nummer. Dies ist eine Voraussetzung. Das Modul versucht, Ressourcen auf der Grundlage des Titels zu bestellen. Sie sollten dies bei der Benennung Ihrer Firewall-Ressourcen beachten.

Im nächsten Abschnitt verwenden wir unser Firewall-Modul, um sicherzustellen, dass zwei Knoten nach Bedarf kommunizieren können.