# PowerDNS dnsdist

dnsdist ist ein DNS-Loadbalancer.

Über dnsdist werden die DNS Anfragen in Abhängigkeit von der Quelle an unterschiedliche Ziele weitergeleitet.

Dies wird beispielsweise benutzt um einen Internen und Externen DNS Server abzubilden. Wie der Hidden Primary beim ISC Bind Server

Der Interne liefert biespielsweise die internen IP's der Systeme  zurück oder enthält Systeme die aus dem Internet nicht erreichbar sind.

Mit PowerDNSAdmin gibt dann auch die Oberfläche ist per http über den Konfigurierten Port zu erreichen.

Die Replikation erfolgt vom Master zu den MySQL Slaves.

# PowerDNS dnsdist Konfiguration

Pro Instanz wird ein Authoritativer und optional ein Recursiver Nameserver gestartet.

## dnsdist Konfigurations Optionen Beschreibung 

*powerdns_mysql_db: powerdns*

powerdns_mysql_db	String	Name der MySQL DB für PowerDNS

*mysql_pass: randompassword*

Passwort für den MySQL Benutzer

*mysql_user: powerdns*

mysql_user	String	MySQL Benutzer für die DB

_mysqlmaster	Boolean_
Nur das erste System ist ein MySQL Master. Nur auf diesem System läuft ein Admininterface (Die alle anderen Systeme replizieren)

_powerdns_master: True_

powerdns_master	Boolean	Wird das System ein DNS Master

_apikey: randompassword_
apikey	String	API Key für das Webinterface von PowerDNS (Optional, wird nur auf dem MySQL Master benötigt)

port	Integer	Port für den Autoriativen Nameserver	 port: 5063
recursor	Boolean	Recursor für diese Instanz bereitstellen	recursor: True
recursor_port	Integer	Port für den Recurser	recursor_port: 5073
catch_iprange_recursor	List	Anfragen von diesen IP's/Bereichen an den recursor dieser

Instanz schicken	      catch_iprange_recursor:
        - 62.156.190.133/32
        - 62.156.190.134/32
        - 192.168.253.25/32

*catch_all	Boolean*

Alle DNS Anfragen die nicht über Regeln abgefangen werden, werden an diese Instanz weitergeleitet.

*catch_all: True*

Es darf muss genau eine Instanz als Catchall definiert werden!

*allow_axfr_ips*

```yaml
  allow_axfr_ips:
    - 194.25.0.125
    - 194.246.96.0/24
```

List IP-Adressen/Bereiche die einen Zonentransfer durchführen dürfen
(Wird nur benötigt, wenn weitere DNS Server existieren die nicht durch uns betrieben werden, die aber diese Zonen bereistellt)

*webserver_ip: 172.24.28.125*

webserver_ip	String	PowerDNS API wird unter dieser IP Adresse bereitgestellt (Benötigt für Admininterface, darf nicht localhost sein)

*webserver_port: 8391*

webserver_port	Integer	PowerDNS API wird unter Port bereitgestellt (Benötigt für Admininterface)

*admininterface: True*

admininterface	Boolean	Admininterface (Http) für diese Instanz bereitstellen (Nur auf dem MySQL Master)

*admin_port: 8393*

admin_port	Integer	Port auf dem das Admininterface hört, ist über jede IP des Systems erreichbar.

*mysql_backup_dir: /srv/mysql_backup/internal*

mysql_backup_dir	String	Pfad in dem die DB Backups für die Datenbank erstellt werden


# dnsdist Beispielkonfiguration

PowerDNS/MySQL Master mit Admin Interface

Zwei Instanzen, external ist die "Catchall" Quelle erweitern
PowerDNS/MySQL Slave

Zwei Instanzen, external ist die "Catchall" Quelle erweitern
Deployment