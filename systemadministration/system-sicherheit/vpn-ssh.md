---
tags:
  - vpn
  - ssh
---
# VPN SSH

Wichtige Einstellungen in der sshd_conf file

```s
PermitRootLogin yes
PermitTunnel yes
```

Daten zum beispiel

10.0.1.1 (server)
10.0.1.2 (client)

Um die Netze zu verbinden bietet sich ein vlan an.
auf dem client solle man dann eine vlan mit der Adresse 10.0.1.2 einrichten.

ToDo : Link für VLAN einrichtung

## Einstellungen

Auf dem Linux Server

```s
ssh -w5:5 root@hserver
ifconfig tun5 10.0.1.1 netmask 255.255.255.252
```

tun device einrichten

Auf den Client Linux system ist folgendes einzurichten

```s
ifconfig tun5 10.0.1.2 netmask 255.255.255.252
ifconfig tun5 10.0.1.2 10.0.1.1
```

Einstellungen bei zwei Netzwerken

Beispiel Daten
Netzwerke:

* 192.168.51.0/24   (netA)
* 192.168.16.0/24   (netB)
* 10.0.1.2 (client)
* 10.0.1.1 (server)

Gateways

* 192.168.51.0

Verbindung von gateA zu gateB

Konfiguration für gateB

```s
gateA># ssh -w5:5 root@gateB
gateB># ifconfig tun5 10.0.1.1 netmask 255.255.255.252 # Executed on the gateB shell
gateB># route add -net 192.168.51.0 netmask 255.255.255.0 dev tun5
gateB># echo 1 > /proc/sys/net/ipv4/ip_forward        # Only needed if not default gw
gateB># iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Konfiguration für gateA

```s
gateA># ifconfig tun5 10.0.1.2 netmask 255.255.255.252
gateA># route add -net 192.168.16.0 netmask 255.255.255.0 dev tun5
gateA># echo 1 > /proc/sys/net/ipv4/ip_forward
gateA># iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Source

* http://sleepyhead.de/howto/?href=vpn