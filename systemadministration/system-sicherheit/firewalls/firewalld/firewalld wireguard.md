Die folgenden Befehle setzen die benötigten Firewallregeln:

```shell
# Schnittstelle zum Internet (hier: eth0) zur Zone "public" hinzufügen
sudo firewall-cmd --permanent --zone=public --add-interface=eth0
# Eingehende Verbindungen auf dem eingestellten Wireguard-Port erlauben
sudo firewall-cmd --permanent --zone=public --add-port=54321/udp

# Wireguard Schnittstelle zu neuer Zone "wg-inet" hinzufügen
sudo firewall-cmd --permanent --new-zone=wg-inet
sudo firewall-cmd --permanent --zone=wg-inet --add-interface=wg0
# eingehenden Traffic aus dem Wireguard Tunnel erlauben
sudo firewall-cmd --permanent --zone=wg-inet --set-target=ACCEPT

# Traffic zwischen beiden Zonen durch Policy explizit erlauben
sudo firewall-cmd --permanent --new-policy=wg-inet-to-public
sudo firewall-cmd --reload
sudo firewall-cmd --permanent --policy=wg-inet-to-public --add-ingress-zone=wg-inet
sudo firewall-cmd --permanent --policy=wg-inet-to-public --add-egress-zone=public
sudo firewall-cmd --permanent --policy=wg-inet-to-public --set-target=ACCEPT

# NAT für IPv4 aktivieren (implizit auch IP-Forwarding generell)
# Dieser Teil ersetzt die PostUp / PostDown Befehle!
sudo firewall-cmd --permanent --zone=public --add-masquerade

# alle Regeln übernehmen
sudo firewall-cmd --reload
```

Durch diese Befehle werden permanente Firewallregeln definiert, die als Dateien in `/etc/firewalld` gespeichert sind. Der Inhalt sollte nun wie folgt aussehen:

**/etc/firewalld/zones/public.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <port port="54321" protocol="udp"/>
  <masquerade/>
  <interface name="eth0"/>
  <forward/>
</zone>
```

**/etc/firewalld/zones/wg-inet.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone target="ACCEPT">
  <interface name="wg-inet"/>
  <forward/>
</zone>
```

**/etc/firewalld/policies/wg-inet-to-public.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<policy target="ACCEPT">
  <ingress-zone name="wg-inet"/>
  <egress-zone name="public"/>
</policy>
```
### 4. ndppd (Neighbor Discovery Protocol Proxy Daemon)

Damit nun die Wireguard Client Peers die (öffentlichen) IPv6-Adressen für den Internetzugriff nutzen können, ist als “Geheimzutat” die Software `ndppd` notwendig. Diese kann mit `sudo apt install ndppd` einfach installiert werden.

Die sogenannten “Neighbor Solicitation” Pakete des Neighbor Discovery Protocol (NDP), sind das IPv6-Äquivalent zu ARP in IPv4-Netzen. Das Internet-Gateway des Servers sendet diese Nachrichten an alle Hosts im Netz, um die MAC-Adresse zu einer Adresse im Netz herauszufinden. Da diese Adressen allerdings nicht auf der Internet-Schnittstelle des Servers liegen, sondern ein einem Ende des Wireguard-Tunnels, werden diese Pakete vom Netzwerkstack des Servers ignoriert, sodass das Gateways die Pakete nicht zustellen kann.

Mit dem `ndppd` kann das Problem gelöst werden. Dieses prüft für alle eingehenden Neighbor Solicitation Nachrichten, ob die angefragte IPv6-Adresse in einen konfigurierten Bereich fällt und leitet diese an eine definierte Schnittstelle weiter, also in diesem Fall die Wireguard-Schnittstelle.

Dazu muss entsprechend der Dokumentation (`man 5 ndppd.conf`) die Konfigurationsdatei `/etc/ndppd.conf` erstellt und wie folgt mit Inhalt gefüllt werden:

```text
proxy eth0 {
    rule 2001:db8::cafe:0/112 {
        iface wg0
    }
}
```

Dem Beispiel anzupassen sind die Internet-Schnittstelle (hier `eth0`) und vor allem der (möglichst kleine!) Teil des IPv6-Netzes (hinter `rule`), das den Wireguard Client Peers zur Verfügung gestellt werden soll. Der Teil `iface wg0` teilt `ndppd` mit, an welche Schnittstelle die entsprechenden Neighbor Solicitations weitergeleitet werden sollen. Diese werden dann von den Client Peers beantwortet. Alternativ kann stattdessen das Keyword `static` angegeben werden, sodass die entsprechenden Neighbor Solicitations sofort beantwortet werden sollen und **nicht** an die Peers über die Wireguard-Schnittstelle weitergeleitet werden. Dadurch erscheinen diese Adressen immer erreichbar zu sein. Diese Einstellung ist gegenenfalls zuverlässiger, versorgt aber das Gateway mit potentiell falschen Informationen.

