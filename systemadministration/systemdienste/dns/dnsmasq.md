---
tags:
  - dnsmasq
  - dns
  - dig
  - ss
---
**/etc/dnsmasq.conf**

```
port=53
domain-needed
bogus-priv
listen-address=127.0.0.1,your-server-ip
expand-hosts
domain=dns-example.com
cache-size=1000
server=/google.com/8.8.8.8 # leitet alle DNS-Anfragen für google.com-Seiten an den Google Nameserver weiter
server=/de/213.73.91.35 # leitet alle DNS-Anfragen für .de-Seiten an den Nameserver des Berliner CCC weiter
address=/doubleclick.net/127.0.0.1
address=/google-analytics.com/127.0.0.1
```

`dnsmasq --test`

`ss -alnp | grep -i :53`

`dig host1.dns-example.com +short`

# DNS/DHCP Server auf der Basis dnsmasq

Am einfachsten kann man ein DNS/DHCP Server mit dnsmasq aufsetzen.

Dieser ist in den Repos meist einfach erhalten und kann dementsprechend zb. mit `apt install dnsmasq` installiert werden.

Es gibt nur eine Configdatei, welche auch schon einige Beispiele bereithält. Diese ist unter `/etc/dnsmasq.conf` zu finden.

#### DNS

Dnsmasq schaut in der `/etc/hosts` nach ihm bekannten Hostnames und deren IP Adresse.  
Sollen noch die DHCP Leases aufgelöst werden, muss noch folgende Zeile auskommentiert werden `addn-hosts=<Pfad zur Leasedatei>`. Standardmäßig liegen die Leases unter `/var/lib/misc/dnsmasq.leases`.

`addn-hosts=/var/lib/misc/dnsmasq.leases`

Soll der dnsmasq als Cache fungieren, muss er noch nachgelagerte DNS kennen um diese abfragen zu können. Diese werden unter `/etc/resolv.conf` hinterlegt.

Um Probleme bei der `/etc/resolv.conf`, hervorgerufen zb. durch Netplan oder resolvconf, zu vermeiden, bietet es sich an mit `resolv-file=<Pfad zur Datei>` eine Datei anzugeben welche die Nameserver enthält. Der Aufbau der Datei ist gleich der normalen `/etc/resolv.conf`.  
Hierbei ist zu beachten, dass die Datei unter `/etc` liegen muss, da sich der dnsmasq sonst weigert die Datei zu lesen.

#### DHCP

Um den IP Bereich für das DHCP einzurichten, muss die folgende Zeile auskommentiert werden:

```
dhcp-range=<erste IP des Bereiches>,<letze IP des Bereiches>,<Leasetime, zb 2h>
```

Für statische Leases:

```
dhcp-host=<MAC Adresse>,<IP die vergeben werden soll>
```

Der DHCP kann mehrere Optionen mitgeben, diese können zb. das Standardgateway oder der bevorzugte DNS Server sein.

```
dhcp-option=<DHCP Option>,<Wert>
```

Interessant sind hier vor allem die Option 3 und 6. Die Option 3 setzt die Router im Subnet, wird nur einer mitgegeben, wird dieser als Standardgateway ausgewählt. Option 6 übergibt die DNS Server welche genutzt werden sollen.  
Bei beiden Optionen entspricht der Wert, der IP.

Beispiel:

```
dhcp-option=3,192.168.1.1
```