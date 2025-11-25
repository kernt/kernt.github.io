---
tags:
  - Linux
  - sshuttle
---
`sudo sshuttle -r tobkern@ 172.16.5.0/23 -v`

`sudo sshuttle -r User@Server:Port 0/0`

Die Portnummer lasse ich weg, wenn es sich um den SSH-Standardport 22 handelt. Die »0/0« bedeutet, dass Linux alle Verbindungen in den Tunnel lenken soll. Das bringt aber mit sich, dass ich dann andere Geräte im lokalen Netz nicht mehr erreiche. Um das lokale LAN weiterhin sichtbar zu halten, definiere ich es mit dem Parameter »-x« als Ausnahme:

`sshuttle --dns  -r  admin@144.91.77.182 0/0 -x 192.168.4.0/21`

Hier ist »–dns« mit dabei.
Damit laufen DNS-Abfragen auch durch den Tunnel, was per se nicht geschieht.
Das ist Sshuttle’s Achillesferse: Es transportiert nur TCP!
ICMP und UDP passen, abgesehen von DNS, nicht durch den Tunnel.

**Kubernetes calico**

`sudo sshuttle -r --dns tobkern@144.91.77.182 0/0 CalicoNet` 

**Route All Traffic Through SSH**: Securely route all network traffic through a remote SSH server.

`$ sshuttle -r user@remote-server 0/0`

**Include DNS Traffic**: Prevent DNS leaks by routing DNS queries through the remote server.

```sh
sshuttle -r user@remote-server --dns 0/0
```

**Route Specific Traffic Only**: Limit routing to a specific subnet, such as internal corporate resources.

```sh
sshuttle -r user@remote-server 10.1.2.0/24
```

**Bind to a Specific Network Interface**: Control which network interface SSHuttle listens on.

`sshuttle --listen 10.1.2.99 -r user@remote-server 0/0`

**Run as a Daemon**: Keep SSHuttle running in the background.

$ sshuttle -r user@remote-server 0/0 -D

The `-D` option runs SSHuttle as a daemon (background process). Combine this with logging for monitoring:

$ sshuttle -r user@remote-server 0/0 -D --logfile /path/to/logfile.log