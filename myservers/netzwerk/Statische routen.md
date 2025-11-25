
# Routen für das Wireguard VPN

IPc4Adresse: 192.168.11.1

Gateway: 192.168.11.1, 192.168.8.1
## Windows Wiregurad Routen

`route ADD 192.168.11.0 MASK 255.255.255.0 192.168.8.1 METRIC 3`

ping 192.168.11.1
## Linux Wireguard routen

```sh
ip route add 192.168.11.0/24 via 192.168.8.1
```

> Auf dem Cliens muss die wg0.conf erweitert werden:
```sh
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```
# Routen für OpenVPN VPN

IPv4-Subnetz: 192.168.10.0/24

