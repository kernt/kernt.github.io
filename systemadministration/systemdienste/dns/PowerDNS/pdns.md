---
tags:
  - pdns
  - dns
---
# PDNS Allgemein

Ist in zwei Komponenten aufgeteilt. Autoritativer Namenserver und Rekursiver Namenserver

**PowerDNS Authoritative Nameserver**

Löst die lokal gehosteten DNS Zonen auf.

**PowerDNS Recursor**

Löst alle DNS Anfragen rekursiv auf, außer es handelt sich um eine lokal vorhandene DNS Zone. In diesem Fall erfolgt die Weiterleitung an lokalen Autoriativen Nameserver.

**PowerDNS-Admin**

Stellt ein Weboberfläche für die Verwaltung von PowerDNS zur Verfügung.

PowerDNS-Admin wird über einen Docker Container bereitgestellt. Die Admin Oberfläche wird nur auf dem MySQL Master bereit gestellt, da der Slave sich vollständig repliziert.
# Mysql Datenbank pdns Einrichtung

```sql
Datenbank: ``
User:``
```

```sql
GRANT ALL ON pdns.* TO 'pdns'@'localhost' IDENTIFIED BY 'g7KShcbcv4UZQWBk';
GRANT ALL ON pdns.* TO 'pdns'@'tobkern-desktop.example.com' IDENTIFIED BY 'g7KShcbcv4UZQWBk';
```

**Installation Skript**

```sh
# https://partial.solutions/2015/foreman-and-powerdns.html
DATABASE=pdns
USERNAME=pdns
PASSWORD=g7KShcbcv4UZQWBk
ZONE=example.com

sudo dnf install pdns pdns-backend-mysql

sudo mysql <<EOF
CREATE DATABASE ${DATABASE};
CREATE USER '${USERNAME}'@'localhost' IDENTIFIED BY '${PASSWORD}';
GRANT ALL PRIVILEGES ON ${DATABASE}.* TO '${USERNAME}'@'localhost';
EOF

sudo mysql $DATABASE < /usr/share/doc/pdns-backend-mysql/schema.mysql.sql

sudo tee /etc/pdns/pdns.conf > /dev/null <<EOF
setuid=pdns
setgid=pdns
launch=gmysql
gmysql-dnssec=true
gmysql-host=localhost
gmysql-user=$USERNAME
gmysql-password=$PASSWORD
gmysql-dbname=$DATABASE
EOF

sudo systemctl start pdns
sudo systemctl enable pdns

sudo mysql <<EOF
INSERT INTO ${DATABASE}.domains (name, type) VALUES ('${ZONE}', 'master');
INSERT INTO ${DATABASE}.records (domain_id, name, type, content) VALUES (LAST_INSERT_ID(), '${ZONE}', 'SOA', 'ns1.${ZONE} hostmaster.${ZONE}. 0 3600 1800 1209600 3600');
EOF

sudo pdnssec rectify-zone $ZONE
```

**Prüfen der PowerDNS Insdanze**

```sh
dig @localhost SOA example.com
```
# PowerDNS Backup & Recovery

Das Backup wird nur auf dem MySQL Master erstellt.
Das Backup wird automatisiert über den Einträg im crontab

```sh
# Lines below here are managed by Salt, do not edit
# SALT_CRON_IDENTIFIER:powerdns-mysql-backup-mirapoint
46 3 * * * /usr/local/sbin/mysql_backup-mirapoint.sh > /dev/null 2>&1
# SALT_CRON_IDENTIFIER:powerdns-mysql-backup-external
32 3 * * * /usr/local/sbin/mysql_backup-external.sh > /dev/null 2>&1
```

Die Backups werden ein mal täglich gezogen und in den Verzeichnissen entsprechend der Konfiguration abgelegt.

# PowerDNS CLI Cheatsheet

**ZONE anlegen**

`pdnsutil --config-name <instanz> create-zone example.com`

**Daten zur zone hinzufügen root A-record und AAAA for IPv6**

```
pdnsutil --config-name <instanz> add-record example.com @ A 192.168.1.2
pdnsutil --config-name <instanz>add-record example.com @ AAAA 2a01:4f9:c010:30f4::1
```
 
**Daten zur zone hinzufügen Beispiel  subdomain (www)**

`pdnsutil --config-name <instanz> add-record example.com www A 192.168.1.2`

**Daten zur zone hinzufügen NS Server für eine Domain**

`pdnsutil --config-name <instanz> add-record example.com @ NS ns1.example.com`

**Daten zur zone hinzufügen**

```sh
pdnsutil --config-name <instanz> add-record example.com @ MX "10 example.com"
pdnsutil --config-name <instanz> add-record example.com 3600 TXT "google-site-verification=example-id"
```

**MANAGE ZONES // check the information provided**

`pdnsutil --config-name <instanz> list-zone example.com`

**MANAGE ZONES // modify the zone (if changes do not miss update serial -> UPDATE SERIAL)**

`pdnsutil --config-name <instanz> edit-zone example.com`

**MANAGE ZONES**

```
pdnsutil --config-name <instanz> check-all-zones
pdnsutil --config-name <instanz> show-zone example.com
```

**DELETE A SPECIFIC ZONE**

`pdnsutil --config-name <instanz> delete-zone example.com`

**UPDATE SERIAL**

`pdnsutil --config-name <instanz> increase-serial example.com`

**UPDATE THE CONNECTED DNS SERVERS**

`pdns_control --config-name <instanz> notify example.com`

# Power DNS Server einrichten unter Debian Linux

**Packete installieren**

```sh
sudo apt install pdns-server pdns-backend-mysql
```

#  PowerDNS Admin Web gui

```
...
api=yes
...
api-key=$(openssl rand -base64 16)
```

```
# List zones
curl -H 'X-API-Key: changeme' http://127.0.0.1:8081/api/v1/servers/localhost/zones | jq .

# Create new zone "example.org" with nameservers ns1.example.org, ns2.example.org
curl -X POST --data '{"name":"example.org.", "kind": "Native", "masters": [], "nameservers": ["ns1.example.org.", "ns2.example.org."]}' -v -H 'X-API-Key: changeme' http://127.0.0.1:8081/api/v1/servers/localhost/zones | jq .

# Show the new zone
curl -H 'X-API-Key: changeme' http://127.0.0.1:8081/api/v1/servers/localhost/zones/example.org. | jq .

# Add a new record to the new zone (would replace any existing test.example.org/A records)
curl -X PATCH --data '{"rrsets": [ {"name": "test.example.org.", "type": "A", "ttl": 86400, "changetype": "REPLACE", "records": [ {"content": "192.0.5.4", "disabled": false } ] } ] }' -H 'X-API-Key: changeme' http://127.0.0.1:8081/api/v1/servers/localhost/zones/example.org. | jq .

# Combined replacement of multiple RRsets
curl -X PATCH --data '{"rrsets": [
  {"name": "test1.example.org.",
   "type": "A",
   "ttl": 86400,
   "changetype": "REPLACE",
   "records": [ {"content": "192.0.2.5", "disabled": false} ]
  },
  {"name": "test2.example.org.",
   "type": "AAAA",
   "ttl": 86400,
   "changetype": "REPLACE",
   "records": [ {"content": "2001:db8::6", "disabled": false} ]
  }
  ] }' -H 'X-API-Key: changeme' http://127.0.0.1:8081/api/v1/servers/localhost/zones/example.org. | jq .
```