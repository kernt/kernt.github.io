---
tags:
  - monitoring
  - icinga2
---
# Icinga2 

Übersicht

* [Icinga2 icingaweb2](../icingaweb2)
* [Icinga2 api](../monitorring/icinga2/icinga2-api)

## Icinga2 grundlegende Struktur

*/etc/icinga2*

```sh
conf.d
constants.conf
features-available
features-enabled
icinga2.conf
pki
scripts
zones.conf
zones.d
```

*/etc/icinga2/conf.d*

```sh
app.conf
apt.conf
commands.conf
downtimes.conf
groups.conf
hosts.conf
notifications.conf
services.conf
templates.conf
timeperiods.conf
users.conf
```

*/etc/icinga2/*

```
README 
```

Die README zeit zu https://icinga.com/docs/icinga2/latest/doc/06-distributed-monitoring/
und schreit zusammengefast das man hier die Verteielte konfiguration via agends gemacht wird.
Die dann dort liegende Konfiguration wird dann auch verteilt.
Checks die nur local Sinn machen können dann unter /etc/icinga2/conf.d bleiben.

Der praktische Weg ist es ein verzeichnis zu erstellen und es extra zu includieren.
Das sollte man machen damit man z.B es ein git repository bringen kann ohne alles darin zu haben.
Zum Beispiel kann man den Kunden 

## Service Monitorring

In der Datei `constants.conf` im Verzeichnis **/etc/icinga2** werden die Plugins zum Monitorring definiert.
Auf einem [Ubuntu OS](../ubuntu) kann man die plugins unter **/usr/lib/nagios/plugins** finden

In der Documentation von [icinga2](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/service-monitoring) wird leider von einer anderen Einrichtung ausgegangen :
```
# su - icinga -s /bin/bash
$ /opt/monitoring/plugins/check_snmp_int.pl --help
```

Hier wird angenomen , dass wir eigene Plugins unter `/opt/monitoring/plugins/` verwenden was wir aber noch nicht am Anfang nutzen und es für die Meisten aufgaben unter `/usr/lib/nagios/plugins` eine lösung vorhanden ist . 

Folgede Plugins kann man unter `/usr/lib/nagios/plugins` bereits finden .

```sh
check_apt      check_disk_smb  check_hpjd          check_ldap         check_nagios    check_overcr     check_simap  check_udp
check_breeze   check_dns       check_http          check_ldaps        check_nntp      check_pgsql      check_smtp   check_ups
check_by_ssh   check_dummy     check_icmp          check_load         check_nntps     check_ping       check_snmp   check_users
check_clamd    check_file_age  check_ide_smart     check_log          check_nt        check_pop        check_spop   check_wave
check_cluster  check_flexlm    check_ifoperstatus  check_mailq        check_ntp       check_procs      check_ssh    negate
check_dbi      check_fping     check_ifstatus      check_mrtg         check_ntp_peer  check_real       check_ssmtp  urlize
check_dhcp     check_ftp       check_imap          check_mrtgtraf     check_ntp_time  check_rpc        check_swap   utils.pm
check_dig      check_game      check_ircd          check_mysql        check_nwstat    check_rta_multi  check_tcp    utils.sh
check_disk     check_host      check_jabber        check_mysql_query  check_oracle    check_sensors    check_time
```

Hier ein funktionierendes Beispiel :

```sh
su - icinga -s /bin/bash

/usr/lib/nagios/plugins/check_dns -H 192.168.4.14
DNS OK: 0,087 seconds response time. 192.168.4.14 returns rp4.example.com.|time=0,087306s;;;0,000000
```

#### Plugins Testen und Benutzen 

### Zonen
Zonen dienen unter [ICINGA2](../icinga2) als logische Unterscheidung der Zuständigkeiten bei einem Cluster.
Default wird der Hostname als ZoneName gesetzt in der Datei `constants.conf` 

Quellen: 
[Service Monitoring](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/service-monitoring)
[](https://docs.icinga.com/icinga2/latest/doc/module/icinga2/chapter/service-monitoring#service-monitoring-dns)
[]()

## Erweiterte Konfiguration

Hier sind einige Erweiterung der Standard Einrichtungen

 [Advanced Nagios Plugins Collection](https://github.com/HariSekhon/nagios-plugins)
 
Es gibt Erweiterungen der Nagios Plugins die auch für [Icinga2](../icinga2) genutzt werden können.
Einrichtung

```sh
git clone https://github.com/HariSekhon/nagios-plugins.git && cd nagios-plugins

make && make clean 

```

Dann kann ein neues verzeichnis erstellen

```
mkdir -p /opt/monitoring/plugins && cp {*.pl|*.py} 
```
und alle plugins hierhin Kopieren 

# Icinga2 Administration

Daly Operationen zum [Icinga2](../icinga2) Monitorring

## Upgrades

Vor einem Update/Upgrade muss immer geprüft werden op auch ein  Major [Icinga2](../icinga2) vorgenommen wird.

## Ubuntu/Debian mit mysql db

1.  Aktuelle Version von icinga2 Notieren `icinga2 version $( icinga2 --version | grep "^icinga2" | cut -d":" -f  2 | sed 's\)\\g' ) `
2. Nach dem Distribution Upgrade die Versionen vergleichen für jedes Minor Update muss je ein Schema Update gemacht werden .
3. Schema upgrade durchführen hier Zielversion 2.6.0 `mysql -u root -p icinga < /usr/share/icinga2-ido-mysql/schema/upgrade/2.6.0.sql`

`-p icinga` ist die Zieldatenbank PW ist im 

**INFO**
Für `MySQL Server` sind unter `/usr/share/icinga2-ido-mysql/schema/upgrade/` alle Dateien für ein upgrade zu finden 

> Es kommt vor , dass nicht alle notwendigen sql Updates der Datenbank vom Paketmanager durchgeführt werden. Um zu nachzuvollziehen wo der Paketmanager nicht richtig gearbeitet hat, sollte man sich also die Versionen vorher notieren!  

