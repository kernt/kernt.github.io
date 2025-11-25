---
tags:
  - netzwerk
  - bound
---
# Read Hat Netzwerk-Bonding konfigurieren

|Bonding-Modus|Konfiguration am Switch|
|:--|:--|
|`0`-`balance-rr`|Erfordert die Aktivierung des statischen EtherChannels, nicht die Aushandlung des Link Aggregation Control Protocol (LACP).|
|`1`-`active-backup`|Am Switch ist keine Konfiguration erforderlich.|
|`2`-`balance-xor`|Erfordert die Aktivierung des statischen EtherChannels, nicht LACP-ausgehandelt.|
|`3`-`broadcast`|Erfordert die Aktivierung des statischen EtherChannels, nicht LACP-ausgehandelt.|
|`4`-`802.3ad`|Erfordert die Aktivierung des LACP-ausgehandelten EtherChannels.|
|`5`-`balance-tlb`|Am Switch ist keine Konfiguration erforderlich.|
|`6`-`balance-alb`|Am Switch ist keine Konfiguration erforderlich.|

bound0 Konfiguration ausgeben

`cat /proc/net/bonding/bond0`

**Konfigurieren Sie eine Slave-Schnittstelle mit `eth0`**

~]# vi /etc/sysconfig/network-scripts/ifcfg-eth0
NAME=bond0-slave0
DEVICE=eth0 
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=no

> Die Verwendung der NAME-Direktive ist optional. Es dient zur Anzeige über eine GUI-Schnittstelle wie **nm-connection-editor** und **nm-applet** .

**Konfigurieren Sie eine Slave-Schnittstelle mit `eth1`**

~]# vi /etc/sysconfig/network-scripts/ifcfg-eth1
NAME=bond0-slave1
DEVICE=eth1
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=no

>Die Verwendung der NAME-Direktive ist optional. Es dient zur Anzeige über eine GUI-Schnittstelle wie **nm-connection-editor** und **nm-applet** .

**Konfigurieren Sie eine Channel-Bonding-Schnittstelle `ifcfg-bond0`**

~]# vi /etc/sysconfig/network-scripts/ifcfg-bond0
NAME=bond0
DEVICE=bond0
BONDING_MASTER=yes
TYPE=Bond
IPADDR=192.168.100.100
NETMASK=255.255.255.0
ONBOOT=yes
BOOTPROTO=none
BONDING_OPTS="mode=active-backup miimon=100"
NM_CONTROLLED=no

>Die Verwendung der NAME-Direktive ist optional. Es dient zur Anzeige über eine GUI-Schnittstelle wie **nm-connection-editor** und **nm-applet** . In diesem Beispiel wird MII für die Linküberwachung verwendet.

**Überprüfen Sie den Status der Schnittstellen auf dem Server**

`ip addr`

## Konflikte mit Schnittstellen lösen


**alle `ifcfg`Dateien aufzulisten, die möglicherweise einen Konflikt verursachen**

```sh
grep -r "ONBOOT=yes" /etc/sysconfig/network-scripts/ | cut -f1 -d":" | xargs grep -E "IPADDR|SLAVE"
/etc/sysconfig/network-scripts/ifcfg-lo:IPADDR=127.0.0.1
```

**Bindung auf dem Server Prüfen**

```sh
ifup /etc/sysconfig/network-scripts/ifcfg-bond0
```

**Details zur Bindungskonfiguration**

```sh
cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.6.0 (September 26, 2009)

Bonding Mode: transmit load balancing
Primary Slave: None
Currently Active Slave: eth0
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth0
MII Status: up
Speed: 100 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 52:54:00:19:28:fe
Slave queue ID: 0

Slave Interface: eth1
MII Status: up
Speed: 100 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 52:54:00:f6:63:9a
Slave queue ID: 0
```

VLAN-Schnittstellendatei `bond0.192`

```
vi /etc/sysconfig/network-scripts/ifcfg-bond0.192
DEVICE=bond0.192
NAME=bond0.192
BOOTPROTO=none
ONPARENT=yes
IPADDR=192.168.10.1
NETMASK=255.255.255.0
VLAN=yes
NM_CONTROLLED=no
```

Rufen Sie die VLAN-Schnittstelle

ifup /etc/sysconfig/network-scripts/ifcfg-bond0.192
Determining if ip address 192.168.10.1 is already in use for device bond0.192...

`ethtool --identify ifname integer`

> Dabei ist _Ganzzahl_ die Häufigkeit, mit der die LED an der Netzwerkschnittstelle blinken soll.

> - Das Bonding-Modul unterstützt nicht `STP`. Erwägen Sie daher, das Senden von BPDU-Paketen vom Netzwerk-Switch zu deaktivieren.

## nmcli tool

**Änderungen Monitoren**

`nmcli m`

/etc/sysconfig/network-scripts/readme-ifcfg-rh.txt

NetworkManager stores new network profiles in keyfile format in the
/etc/NetworkManager/system-connections/ directory. 
Previously, NetworkManager stored network profiles in ifcfg format
in this directory (/etc/sysconfig/network-scripts/). However, the ifcfg 
format is deprecated. By default, NetworkManager no longer creates 
new profiles in this format.
Connection profiles in keyfile format have many benefits. For example,
this format is INI file-based and can easily be parsed and generated.
Each section in NetworkManager keyfiles corresponds to a NetworkManager
setting name as described in the nm-settings(5) and nm-settings-keyfile(5)
man pages. Each key-value-pair in a section is one of the properties
listed in the settings specification of the man page.
If you still use network profiles in ifcfg format, consider migrating
them to keyfile format. To migrate all profiles at once, enter: 

nmcli connection migrate                                                                                                         

This command migrates all profiles from ifcfg format to keyfile                                               
format and stores them in /etc/NetworkManager/system-connections/. 
Alternatively, to migrate only a specific profile, enter:

nmcli connection migrate 

For further details, see:                                                                                                          

* nm-settings-keyfile(5)                                                                                                           

* nmcli(1)