---
tags:
  - ip
  - konsole
  - bash
  - gnu-tools
  - system-administration
  - networking
---
# linux Kommando ip

| Zweck | iproute2 Kommando | iproute2 Kommando | Kurzversion | net-tools Kommando| |
| :----: | :----: | :----: | :----: | :----: | :----: |
|Linkstatus anzeigen |ip link show |ip l |ifconfig| |
|Linkstatus inkl. Statistik(RX/TX bytes, errors, ...) anzeigen | ip -statistics link show |ip -s l |ifconfig| |
|IP Adresse anzeigen |ip addr show |ip a |ifconfig -a| |
|IP Adresse setzen |ip addr add IP/NETMASK dev DEVICE |ip a a IP/NETMASK dev DEVICE |ifconfig DEVICE IP/NETMASK| |
|IP Adresse entfernen |	ip addr del IP/NETMASK dev DEVICE|ip a d IP/NETMASK dev DEVICE| |
|IP Adressen entfernen | ip addr flush dev DEVICE |ip a f dev DEVICE| |
|Routingtabelle anzeigen | ip route show |ip r |route -n| |
|Standardgateway setzen | ip route add default via IP |ip r a default via IP | route add default gw IP DEVICE| |
|ARP-Cache anzeigen |ip neigh show |ip n |arp -na| |

**Infos zu eth0**

`ip addr show dev eth0`

**Routing Tabelle**

`ip route show`

**Statistiken Anzeigen**

`ip -s link show eth0`

**Links Anzeigen**

`ip link show`

**Device Entfernen**

`ip link delete eth0.100`

**Vlan device erstellen**

`ip link add link eth0 name eth0.100 type vlan id 100`

**IP hinzufügen**

`ip addr add 192.168.100.1/24 brd 192.168.100.255 dev eth0.100`

**IP up setzen**

`ip link set dev eth0.100 down`

