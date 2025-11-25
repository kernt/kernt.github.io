---
tags:
  - networking
  - ipv6
  - gnu-tools
  - system-administration
---
# Einrichtung von ipv6

Default route einrichten

`ip route add default via fe80::1 dev eth0`

Routen Prüfen

`ip -6 addr show`

ipv6 Public setzen

`ip route add default via fe80::1 dev eth0`

Route Prüfen

`ip -6 route show`

Ausgabe: 
```sh
default via fe80::1 dev eth0 metric 1024 pref medium
```

*Änderungen permanent speichern*

Bei Centos

`echo "NETWORKING_IPV6 = "yes" >> /etc/sysconfig/network `

IP Konfiguration einfügen

`echo 'IPV6INIT = "yes"IPV6ADDR = "2001:db8==1/64"IPV6_DEFAULTGW = "fe80==1"IPV6_DEFAULTDEV = "eth0" ' >> /etc/sysconfig/network-scripts/ifcfg-eth0 `

* [ip tools zum berechnen](https://www.vultr.com/resources/ipv4-converter/)
# ipv6 Netze

Wenn man z.B die IP 192.168.4.194 im netz 192.168.4.0 als wie noch bei ipv4 im ipv6 nutzen möchte, muss _::ffff:${IPV4}_ die ipv4 anhängen.

Beispiel:

`::ffff:192.168.4.194`

