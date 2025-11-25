---
tags:
  - bash
  - konsole
  - lsof
  - gnu-tools
  - system-administration
---
# lsof Primär Anzeige offener Dateien ("list open files")

**Prozesse gerade alle die Bash benutzen**

`lsof /bin/bash`

**alle offenen Dateien anzeigen die nicht von _root_ Geöffnet sind anzeigen:**

`sudo lsof -u ^root`

**Alle Netzwerkport des benutzer _www-data_ ausgeben**

`sudo lsof -a -i -u www-data`

**Netzwerk addressen suchen ipv4**

`sudo lsof -i4 `

**Netzwerk addressen suchen ipv4**

`sudo lsof -i6 `

**nfs**

`sudo -N `

**TCP Datein im status LISTEN (CLOSED, IDLE, BOUND, LISTEN, ESTABLISHED,SYN_SENT, SYN_RCDV, ESTABLISHED, CLOSE_WAIT, FIN_WAIT1, CLOSING, LAST_ACK, FIN_WAIT_2, and TIME_WAIT)**

`sudo lsof -iTCP -sTCP:LISTEN`

**Nach Blocks und Timeouts suchen.**

`sudo lsof -S `

**TCP informationen**

`sudo lsof -T `


**lsof auf ein bestimmtes Verzeichnis einschränken**

`lsof +D /usr/lib/locale`

**Auflistung aller Internet- und x.25 (HP-UX) Netzwerkdateien**

`lsof -i`

**offene Dateien auflistet, basierend auf dem Portbereich**

`lsof -i :1-1024`

**Auflistung der offenen Dateien nach Verbindungsart (TCP oder UDP)**

`lsof -i udp`

**lsof list Parent PID von Prozessen mit PPID**

`lsof -R`

**PPID-Informationen für eine bestimmte PID**

`lsof -p [PID] -R`

