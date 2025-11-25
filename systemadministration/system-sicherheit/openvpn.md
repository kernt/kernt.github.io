---
tags:
  - sicherheit
  - openvpn
---
# OpenVPN

## TUN und TAP

### Bridging (TAP-Device)

|Vorteile|Nachteile|
|---|---|
|- Verhält sich wie ein echter Netzwerkadapter<br>- Beliebige Netzwerkprotokolle<br>- Client transparent im Zielnetz<br>- Broadcasts und Wake-On-LAN|- Ineffizient<br>- Hoher Broadcast-Overhead am VPN-Tunnel<br>- Schlechte Skalierbarkeit|

Routing (TUN-Device)

| Vorteile                                                                                                          | Nachteile                             |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| - Weniger Traffic-Overhead<br>- Geringere Bandbreitenbelastung, weil kein Ethernet-Layer<br>- Gute Skalierbarkeit | - Nur IP-Pakete<br>- Keine Broadcasts |

# OpenVPN Server Setup on Linux

**Public Key Infrastructure Setup**

Create a directory to store logs

`mkdir /var/log/openvpn`

Create a separate directory to keep scripts, certificates and keys to ensure that any changes to the scripts will not be lost when the OpenVPN package is updated

`mkdir /etc/openvpn/easy-rsa`

Copy all the content from the examples directory

`cp -R /usr/share/doc/openvpn/examples/easy-rsa/2.0/* /etc/openvpn/easy-rsa/`

Check OpenSSL Version

`openssl version`

and

`dpkg -s openssl | grep -i version`

OpenVPN Server Configuration

We need to copy the default server configuration file first

`cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn`

Unzip it

`unzip /etc/openvpn/server.conf.gz`

Open and modify the server’s configuration file so that it looks something like this

```ini
# cat /etc/openvpn/server.conf
[[listen]] on IPv4
local 0.0.0.0

[[we]] use a non-default port
port 11194

[[UDP]] protocol chosen for better protection against DoS attacks and port scanning
proto udp

[[using]] routed IP tunnel
dev tun

[[full]] paths to keys and certificates
ca /etc/openvpn/ca.crt
cert /etc/openvpn/deb-server.crt
key /etc/openvpn/deb-server.key
dh /etc/openvpn/dh1024.pem

[[set]] OpenVPN subnet
server 10.26.0.0 255.255.255.0

[[maintain]] a record of client-to-virtual-IP-address
ifconfig-pool-persist ipp.txt

[[ping]] every 10 seconds, assume that remote peer is down if no ping received during 60
keepalive 10 60

[[cryptographic]] cipher, must be the same (copied) on the client config file as well
cipher AES-256-CBC

[[enable]] compression on VPN link
comp-lzo

max-clients 10

[[downgrade]] daemon privileges (non-Windows only)
user nobody
group nogroup

[[try]] to preserve some state across restarts
persist-key
persist-tun

[[log]] files
status /var/log/openvpn/openvpn-status.log
log /var/log/openvpn/openvpn.log
log-append /var/log/openvpn/openvpn.log

[[log]] file verbosity
verb 3
```


**Start client Service**

