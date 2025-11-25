
**Werte für die Clients**

|Eintrag (ohne vorangestelltes `option`)|Parameter|Bedeutung|
|---|---|---|
|`routers`|Hostname oder IP-Adresse|Router bzw. Gateway ins Internet.|
|`domain-name-servers`|Hostnamen oder IP-Adressen|Domain-Nameserver.|
|`host-name`|Hostname|Rechnername des Clients.|
|`ntp-servers`|Hostname oder IP-Adressen|Timeserver für Zeitabgleich.|
|`netbios-node-type`|1, 2, 4 oder 8 (empfohlen)|Methode der Namensauflösung unter Windows. 1 steht für Broadcast, 2 für [Unicast](https://www.linux-community.de/ausgaben/linuxuser/2004/04/dhcp-server-fuers-lokale-netzwerk/2/4/#article_gunicast "Unicast\|An jeden Client wird eine Kopie der Datei vom Server geschickt. Solche Punkt-zu-Punkt-Verbindungen sind einfach zu realisieren, belasten den Server bei sehr vielen Empfängern aber stark."), 4 für Mixed Mode (zuerst Broadcast, dann Unicast versuchen) und 8 für einen Hybridmodus, bei dem zuerst Unicast, alternativ Broadcast zum Einsatz kommt.|
|`netbios-name-servers`|Hostname|WINS-Server zur Internet-Naming-Service-Auflösung unter Windows.|
|`domain-name`|Domainname|Name der Netzwerkdomain.|
|`nis-domain`|Domainname|Name der [NIS](https://www.linux-community.de/ausgaben/linuxuser/2004/04/dhcp-server-fuers-lokale-netzwerk/2/4/#article_gnis "NIS\|Der "Network Information Service" vereinfacht die Verteilung von Konfigurationen im Netzwerk. Ein zentraler NIS-Server stellt Informationen zu Login-Namen, Passwörtern und Home-Verzeichnissen, Gruppenzugehörigkeiten und Rechner-Namen zur Verfügung. Der NIS-Server ergänzt so die Einträge in beliebigen Konfigurationsdateien der Clients, z. B. /etc/passwd, /etc/groups oder /etc/hosts. Ob und zu welcher Konfigurationsdatei der NIS-Server zu befragen ist, regelt die Datei /etc/nsswitch.conf. Alternativ kann der DHCP-Server angeben, wer NIS-Server im Netzwerk ist.")-Domain.|
|`nis-servers`|Hostname oder IP-Adressen|Zuständiger NIS-Server.|
|`subnet-mask`|Netzwerkmaske|Netzwerkmaske des Netzsegments.|
**dhcpd-Settings**

| Eintrag              | Parameter                                    | Bedeutung                                                                                                                                                                                                                 |
| -------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `default-lease-time` | Zeit in Sekunden                             | Wie lange sind die vergebenen Daten gültig? Vor Ablauf dieser Zeit muss ein Client erneut um eine IP-Adresse bitten. Überschreitet er das Zeitfenster, kann diese spezielle IP-Adresse an andere Rechner vergeben werden. |
| `max-lease-time`     | Zeit in Sekunden                             | Wie lange sind vergebene Daten maximal gültig? Falls der Client eine besonders lange `default-lease-time` anfordert, legt dieser Parameter die Obergrenze fest.                                                           |
| `subnet`             | Netzwerkadresse                              | Netzsegment, für das die Konfiguration gilt (siehe Kasten 1).                                                                                                                                                             |
| `netmask`            | Netzwerkmaske                                | Maske des Netzsegments (siehe Kasten 1).                                                                                                                                                                                  |
| `range`              | kleinste und größte zu vergebende IP-Adresse | Bereich der vom DHCP-Server zu verwaltenden IP-Adressen.                                                                                                                                                                  |
| `fixed-address`      | IP-Adresse oder Hostname                     | Die einem bestimmten Client zugedachte feste Adresse.                                                                                                                                                                     |
| `filename`           | Dateiname                                    | Bootimage eines bestimmten Clients (siehe Abschnitt „DHCP für Anspruchsvolle“).                                                                                                                                           |
| `hardware ethernet`  | MAC-Adresse                                  | Hardware-Adresse eines Clients.                                                                                                                                                                                           |
_`/etc/dhcp/dhcpd.conf`_

```c
default-lease-time 600;
max-lease-time 7200;
subnet 192.168.4.0 netmask 255.255.255.0 {
	option subnet-mask 255.255.255.0;
	option broadcast-address $ip-address ;
    option ntp-servers ip-address_ [, ip-address... ];
	option routers 192.168.4.1;
	option bootfile-name  ;
    option log-servers ip-address [,ip-address_... ];
    option pop-server ip-address [, ip-address_... ];
    option smtp-server ip-address [, ip-address... ];
    option tftp-server-address ip-address [, _ip-address_... ];
	range 192.168.4.2 192.168.4.192;
	
}

subnet 192.168.7.0 netmask 255.255.255.0 {
	option subnet-mask 255.255.255.0;
	option routers 192.168.5.1;
	option domain-search "example.com";
	option domain-name-servers 192.168.1.1;
	option time-offset -18000;     # Eastern Standard Time
	range 192.168.5.2 192.168.7.254;
}

subnet6 2001:db8:0:1::/64 {
        range6 2001:db8:0:1::129 2001:db8:0:1::254;
        option dhcp6.name-servers fec0:0:0:1::1;
        option dhcp6.domain-search ".local";
}

host  {
	hardware ethernet 56:34:bd:2b:77:40;
	fixed-address 192.168.4.29;
	option host-name srv3;
}

host interface0 {
	hardware ethernet 56:34:bd:2b:77:40;
    option ip-forwarding True;
	option interface-mtu 1500;
	fixed-address 192.168.4.29;
}

host interface1 {
	hardware ethernet 56:34:bd:2b:77:40;
	option interface-mtu 1500;
    option ip-forwarding True;
	fixed-address 192.168.4.29;
}
```