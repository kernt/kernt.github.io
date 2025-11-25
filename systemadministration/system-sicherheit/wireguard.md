---
tags:
  - sicherheit
  - networking
  - wireguard
  - vpn
---
# Wireguard

WireGuard ist VPN welches sich dann ähnlich aufbaut wie ssh
## Server Einrichten

_Hier wird von folgender Konfiguration ausgegangen:_

Die Public IP des Server lautet: _144.123.56.78_
Im VPN wird folgendes Netz verwendet _192.168.200.0/24_
Der Server bekommt im VPN die IP _192.168.200.1_ und lauscht auf dem Port _51820_ .
Interface für die WireGuard ist _wg0_ .

General ist eine IP Weiterleitung notwendig um in andere netze zu kommen

Zunächst muss sichergestellt werden, dass IP-Forwarding für sowohl IPv4 als auch IPv6 im Kernel aktiviert ist. Das ist nötig, damit die Pakete von der Wireguard- zu der für den Internetzugang verwendete Netzwerkschnittstelle (und umgekehrt) weitergeleitet werden können. Prüfen lässt sich das durch das Lesen der beiden entsprechenden sysfs-Dateien, die beide den Wert “1” enthalten müssen.

```bash
cat /proc/sys/net/ipv4/ip_forward
1
cat /proc/sys/net/ipv6/conf/all/forwarding
1
```

Falls einer oder beide Werte auf “0” stehen sollte, kann das Forwarding in `/etc/sysctl.conf` **oder besser** in einer `*.conf` Datei im Verzeichnis `/etc/sysctl.d/` permantent aktiviert werden:

```ini
# /etc/sysctl.d/90-ip-fwd.conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

Zum Übernehmen kann der Befehl `sysctl --system` ausgeführt werden oder das System neugestartet werden. Danach sollten die beiden sysfs-Dateien den Wert “1” enthalten.

```sh
# IP forwarding
sysctl -w net.ipv4.ip_forward=1; sysctl -w net.ipv6.ip_forward=1; sysctl -p
```

Für den Server muss wie folgenden Schlüssel angelegt werden

`wg genkey | tee server_private.key | wg pubkey > server_public.key`

Server Konfiguration in _/etc/wireguard/wg0.conf_

```sh
[Interface]
Address = 192.168.200.1/24
ListenPort = 51820

PrivateKey = <Server Private-Key>
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Cleint1
[Peer]
PublicKey = <Client Public-Key>
AllowedIPs = 192.168.200.2/32

# Cleint2
[Peer]
PublicKey = <Client Public-Key>
AllowedIPs = 192.168.200.3/32
```

Aktivieren  das soeben konfigurierte WireGuard-Interface

`wg-quick up wg0`

WireGuard-Interface nach dem Hochfahren Ihres Servers automatisch startet

`systemctl enable wg-quick@wg0`

`systemctl start wg-quick@wg0` 
## Konfiguration eines Clients

_Hier wird von folgender Konfiguration ausgegangen:_

Die Public IP des Server lautet: _144.123.56.78_
Im VPN wird folgendes Netz verwendet _192.168.200.0/24_
Der Server bekommt im VPN die IP _192.168.200.1_ und lauscht auf dem Port _51820_ .
Interface für die WireGuard ist _wg0_ .

General ist eine IP Weiterleitung notwendig um in andere netze zu kommen

```sh
# IP forwarding
sysctl -w net.ipv4.ip_forward=1; sysctl -w net.ipv6.ip_forward=1; sysctl -p
```

Für jeden Cleint müss wie folgt schlüssel angelegt werden

`wg genkey | tee client1_private.key | wg pubkey > client1_public.key`

Client Konfiguration in _/etc/wireguard/wg0.conf_

```sh
[Interface]
PrivateKey = <Client Private-Key>
Address = 192.168.200.2
DNS = 1.1.1.1

# cleint1
[Peer]
PublicKey = <Server Public-Key>
Endpoint = Server-IP:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

Aktivieren  das soeben konfigurierte WireGuard-Interface

`wg-quick up wg0`

WireGuard-Interface nach dem Hochfahren Ihres Clients automatisch startet

`systemctl enable wg-quick@wg0`


Quellen:

https://www.procustodibus.com/blog/2021/05/wireguard-ufw/

https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04

https://dev.to/tangramvision/exploring-ansible-via-setting-up-a-wireguard-vpn-3389

