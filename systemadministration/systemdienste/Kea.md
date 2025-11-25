Kea ist ein Open-Source-Programmpaket für die IP-Adressvergabe über das Dynamic Host Configuration Protocol. Es wurde entwickelt, um den klassischen DHCP ISC Server zu ersetzen. Es unterstützt DHCPv4 und DHCPv6, inklusive deren entsprechenden Erweiterungen und dynamische DNS Aktualisierung. Als Repository stehen aktuell 4 Datasources zur Verfügung Flatfile auch memfile, MySQL Datenbank oder MariaDB Datenbank, PostgreSQL Datenbank oder Casandra Datenbank. 

Kea setzt sich aus folgenden Komponenten zusammen

- **keactrl:** verantwortlich für den Start, Stopp, Umkonfiguration und Statusmeldungen des Kea-Server
- **kea-dhcp4:** Ist der DHCPv4 Server Prozess. Dieser Prozess antwortet auf DHCPv4 Anfragen von Clients
- **kea-dhcp6:** Ist der DHCPv6 Server Prozess. Dieser Prozess antwortet auf DHCPv6 Anfragen von Clients
- **kea-dhcp-ddns:** Der DHCP Dynamic DNS Prozess. Dieser Prozess dient als Mittler zwischen den DHCP Servern und externen. Er erhält Namesaktualisierungen oder Löschungen und sendet diese an die DNS Server.
- **kea-admin:** Werkzeug für die Datenbankverwaltung.
- **kea-lfc:** Dieser Prozess entfernt redundante Informationen.
- **kea-ctrl-agent:** Dieser Prozess stellt REST-Schnittstelle für die Serververwaltung zur Verfügung.
- **kea-netconf:** Ein Agentd, welcher eine YANG/NETCONF Schnittstelle für die Konfiguration bereitstellt.
- **kea-shell:** Einfacher Text Client, der die REST Schnittstelle zur Verbindung mit dem Kea Control Agent nutzt.
- **perfdhcp:** DHCP benchmarking Werkzeug, welches mehrere Clients simuliert um die Performanz zu messen.

https://web-wilke.de/install-and-run-kea-dhcp-with-stork-on-debian-11/
https://www.isc.org/kea/
https://www.isc.org/stork/