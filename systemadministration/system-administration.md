---
title: System Administration
tags:
  - Linux
  - systemctl
  - journalctl
  - system-administration
---
# System Administration

## Path-Units

Mit Hilfe von Systemd Path-Units können Dateien oder Verzeichnisse auf Änderungen hin überwacht werden. Tritt ein definiertes Ergebnis wie z.B. das Anlegen einer Datei ein, wird eine Service-Unit ausgeführt.
# System Administrations Tools

Benutzerverwaltung

* [adduser  Hinzufügen eines Benutzers](../systemadministration/adduser)
* [chsh Änderung der Standard-Shell des Benutzers ("change shell")](chsh.md)
* [deluser Löschung eines Benutzers ("delete user")](deluser.md)
* [groupdel Löschung einer Gruppe ("delete group")](groupdel.md)
* [groupmod Bearbeitung einer Gruppe ("modify group")](groupmod.md)
* [newgrp Änderung der Gruppe des aktuellen Benutzers ("new group")](newgrp.md)
* [passwd Änderung des Passworts eines Benutzers ("password")](passwd.md)
* [usermod Bearbeitung eines Benutzerkontos ("modify user")](../usermod)
* [chfn erweiterte Benutzerinformationen anpassen](chfn.md)
## Prozesssteuerung
* [ps](ps.md)
## Service Management

* [Systemd](systemd.md)

### Grundkommandos

* [cat Verknüpfung von Dateien ("concatenate")](cat.md)
* [cd Wechsel des Arbeitsverzeichnisses ("change directory")](./cd)
* [cp Kopie von Dateien oder Verzeichnissen ("copy")](cp.md)
* [date Anzeige von Datum und Zeit](date.md)
* [echo Anzeige eines Textes](echo.md)
* [exit Ende der Sitzung](exit.md)
* [info Anzeige einer Hilfe-Datei](info.md)
* [ln Link zu einer Datei oder einem Verzeichnis ("link")](link.md)
* [ls Auflistung von Dateien ("list")](ls.md)
* [man Ausgabe der Handbuchseite zu einem Befehl oder einer Anwendung ("manual")](man.md)
* [mkdir Erzeugung von Verzeichnissen ("make directory")](..(mkdir))
* [mmv Multiple move (Datei-Mehrfachoperationen mit Hilfe von Wildcard-Mustern)](mmv.md)
* [mv Kopieren einer Datei und Löschen der Ursprungsdatei ("move"); mv im aktuellen Verzeichnis ausgeführt: Umbenennung einer Datei](mv.md)
* [pwd Anzeige des aktuellen Verzeichnisses ("print working directory")](pwd.md)
* [rm Löschen von Dateien und Verzeichnisse ("remove")](rm.md)
* [rmdir Löschen eines leeren Verzeichnisses ("remove directory")](rmdir.md)
* [sudo Root-Rechte für den Benutzer ("substitute user do")](sudo.md)
* [touch Änderung der Zugriffs- und Änderungszeitstempel einer Datei oder eines Verzeichnisses (auch: Erstellen von Dateien)](touch.md)
* [unlink Löschen einer Datei](unlink.md)

### Netzwerk

Hosts in `/etc/hosts` ausgeben

`echo -e "$(hostname -i)\$(hostname)" | sudo tee -a /etc/hosts`

* [dig Namensauflösung (DNS)](../dig)
* [iwconfig Werkzeug für WLAN-Schnittstellen](../iwconfig)
* [ifconfig Anzeigen und Konfiguration von Netzwerkgeräten](ifconfig.md)
* [ip Anzeigen und Konfiguration von Netzwerkgeräten. Nachfolger von ifconfig](ip.md)
* [iw der Nachfolger von iwconfig](../iw)
* [netstat Auflistung offener Ports und bestehender Netzwerkverbindungen ("network statistics")](../netstat)
* [ping Prüfen der Erreichbarkeit anderer Rechner über ein Netzwerk](../ping)
* [route 🇩🇪 Anzeige und Änderung der Route (Routingtabelle)](../route)
* [ss der Nachfolger von netstat ("socket statistics")](ss.md)
* [traceroute Routenverfolgung und Verbindungsanalyse](../traceroute)
* [nc nmap ](nmap.md)

### Dateiwerkzeuge

* [basename Rückgabe des Dateinamens](basename.md)
* [lsof Anzeige offener Dateien ("list open files")](lsof.md)

### Storage

Neue Images Scannen

```sh
for host in $(ls -1d /sys/class/scsi_host/*)
	do echo "- - -" > ${host}/scan
done
for device in $(ls -1d /sys/class/scsi_disk/*)
	do echo "1" > ${device}/device/rescan 
done
```

### Systemüberwachung

Pager

* [less](../system-administration-pager-less)

**Weitere Nützliche Befehle**

* [Pager](../system-administration-pager)

## Packet Management

**RPM**

* [RPM](../rpm)
* [YUM](../yum)
* [Repositories](repositories.md)

`APT und DEP`

* [APT](../apt)
* [DEP](../dep)

**Repository bei apt**

```sh
wget -O - 'http://repo.proxysql.com/ProxySQL/repo_pub_key' | apt-key add -
echo deb http://repo.proxysql.com/ProxySQL/proxysql-1.4.x/$(lsb_release -sc)/ ./ \
| tee /etc/apt/sources.list.d/proxysql.list
```


**Große Datei erzeugen**

`truncate  -s 99999G bigfile`

## Doku

https://homelabtopia.com/category/informatik/linux/page/2/