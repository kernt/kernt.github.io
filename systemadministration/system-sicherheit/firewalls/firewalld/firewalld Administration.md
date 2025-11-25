---
tags:
  - sicherheit
  - firewalld
---
# Administration mit firewalld

Thematische Auflistung der Befähle zur Administration von firewalld

## firewalld Ports

**TCP Port freigeben**

`firewall-cmd --permanent --zone=public --add-port=80/tcp`

**UDP Port freigeben**

`firewall-cmd --permanent --zone=public --add-port=123/udp`

## firewalld Änderungen anwenden

**Aktuelle Konfiguration schreiben**

`firewall-cmd --runtime-to-permanent`

**Änderungen anwenden**

`firewall-cmd --reload`

## firewalld Interfaces

**Interface wechseln**

`firewall-cmd --permanent --zone=trusted --change-interface=docker0`

## firewalld Services

**Mehere Services hinzufügen**

`firewall-cmd --zone=internal --add-service={http,https,dns} --permanent`

**Services auflisten**

`firewall-cmd --list-services`

**Verfügbare Services ausgeben**

`firewall-cmd --get-services`

**Infos zu einem Service**

`firewall-cmd --info-service _service_name_`
## firewalld Zonen

**default Zone ausgeben**

`firewall-cmd --get-active-zones`

**default Zone ändern**

`firewall-cmd --set-default-zone=_zone_`

**Services einer Zone ausgeben**

`firewall-cmd --zone=public --list-all`

**all zonen auflisten**

`firewall-cmd --get-zones`

**Active Zonen auflisten**

`firewall-cmd --get-active-zones`

***Infos zur Zone*

`firewall-cmd --info-zone=zone_name`

## firewalld NAT masquerade

```
firewall-cmd --new-policy NAT_int_to_ext --permanent
firewall-cmd --permanent --policy NAT_int_to_ext --add-ingress-zone internal
firewall-cmd --permanent --policy NAT_int_to_ext --add-egress-zone public
firewall-cmd --permanent --policy NAT_int_to_ext --set-target ACCEPT
```
## firewalld Rich rules

**rich rule hinzufüge**

`firewall-cmd [--zone=_zone_name_] [--permanent] --add-rich-rule='_rich_rule_'`

Beispiel erlaube alle verbindungen vom Netzwerk 192.168.1.0/34 zum NFS Service

`firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="nfs" accept'`

Erlauben von Verbindungen vom netz 192.168.2.3 zum port 1234/tcp

`firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.2.3" port port="1234" protocol="tcp" accept'`

`firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.2.3" port port="1234" protocol="tcp" accept'`

```
rule [family="rule family"]
    [ source [NOT] [address="address"] [mac="mac-address"] [ipset="ipset"] ]
    [ destination [NOT] address="address" ]
    [ element ]
    [ log [prefix="prefix text"] [level="log level"] [limit value="rate/duration"] ]
    [ audit ]
    [ action ]
```
# Firewalld Zonen

`public` Eingehend sind Verbindungen auf ssh, mdns und dhcpv6-client erlaubt.
Dies ist die Default-Zone.

`block`
Eingehende Verbindungen werden zurückgewiesen (reject).

`dmz`
Limitierter Zugriff auf das interne Netzwerk, eingehend ist nur SSH erlaubt.

`drop`
Eingehende Verbindungen werden verworfen (drop).

`external`
Nur ausgewählte eingehende Verbindungen sind erlaubt; Rechner wird zum Router, da Masquerading aktiviert ist.

`home`
Wie „work“ plus samba-client.

`internal`
Wie „home“.

`trusted`
Alles erlaubt.

`work`
Anderen Rechnern wird grundsätzlich vertraut; eingehend sind Verbindungen auf ssh, mdns, ipp-client und dhcpv6-client erlaubt.

https://wiki.archlinux.org/title/Firewalld

https://docs.linuxfabrik.ch/base/security/firewalld.html

https://www.redhat.com/sysadmin/beginners-guide-firewalld


## firewalld Rich rules

**example**

```sh
rule family="ipv4" source address="192.168.1.0/24" service name="nfs" accept
rule family="ipv4" source address="192.168.2.3" port port="1234" protocol="tcp" accept
rule family="ipv4" source address="62.171.183.4/32" service name="icmp" accept
rule family="ipv6" source address="" service name="icmp" accept
```

https://github.com/projectcalico/calico/issues/3425
rule protocol value="ipip" accept

https://docs.tigera.io/calico/latest/getting-started/kubernetes/requirements#network-requirements


`firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.2.3" port port="1234" protocol="tcp" accept'`

rule [family="rule family"]
    [ source [NOT] [address="address"] [mac="mac-address"] [ipset="ipset"] ]
    [ destination [NOT] address="address" ]
    [ element ]
    [ log [prefix="prefix text"] [level="log level"] [limit value="rate/duration"] ]
    [ audit ]
    [ action ]

rule [family="rule family"]
    [ source [NOT] [address="address"] [mac="mac-address"] [ipset="ipset"] ]
    [ destination [NOT] address="address" ]
    [ element ]
    [ log [prefix="prefix text"] [level="log level"] [limit value="rate/duration"] ]
    [ audit ]
    [ action ]

rule [family="rule family"]
    [ source [NOT] [address="address"] [mac="mac-address"] [ipset="ipset"] ]
    [ destination [NOT] address="address" ]
    [ element ]
    [ log [prefix="prefix text"] [level="log level"] [limit value="rate/duration"] ]
    [ audit ]
    [ action ]

`firewall-cmd --permanent --add-port=4789/udp`
