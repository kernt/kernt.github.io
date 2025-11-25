---
tags:
  - konfigurationsmanagement
  - deployment
  - foreman
---
# Foreman

* [Foreman discovery image](../foreman-discovery-image)

## Administration 
* [Foreman Installation](../foreman-installation)
* [Foreman Plugins](../foreman-plugins)
* [Foreman Smart Proxy](../foreman-add-smart-proxy)
* [Foreman Backups](../foreman-backup)
* [Foreman Restore](../foreman-restore)

**Allgemeine Informationen**

Für Ubuntu/Debian werden nicht alle features unterstützt.

Install Software Collection

```sh
yum install centos-release-scl
```

### Base Install script

```sh
#!/bin/bash

echo "Install epel Repo"
su -c 'rpm -Uvh http://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-8.noarch.rpm'

echo "Install foreman Repo"
http://yum.theforeman.org/releases/1.14/el7/x86_64/foreman-release.rpm

echo "Install Software collection"
yum install centos-release-scl

echo "$(hostname -I) $(hostname -f)" > /etc/hosts

echo "Install foreman"
yum -y install foreman-installer

echo "Puppet NTP Module"
puppet module install -i /etc/puppet/environments/production/modules saz/ntp

echo "Configore foreman"
foreman-rake db:migrate
foreman-rake db:seed

echo "deactivate selinux !!!!"
# sollte noch eine Seite für selinux ins wiki kommen mit einem Eintrag für die selinux policy
# Qulle: https://github.com/theforeman/foreman-selinux
setenforce 0

```
Quellen zur Installation
* [how-to-install-foreman-with-puppet-in-centos-and-ubuntu](https://www.unixmen.com/how-to-install-foreman-with-puppet-in-centos-and-ubuntu/)

### Provision KVM VM ohne  DHCP

Hier mit CentOS in Version

Mit dem virt-manager kann man die Erstellung der VM so einrichten, dass man direkt den Kernel lädt und ihm eine Antwortdatei über die Kernel Argumente übergibt.
Dadurch kann man vollautomatisch ein Grundsystem einrichten. 

Um es durchzuführen sind folgende schritte notwendig: 

Um sich besser orientieren zu können hab ich die vom Assistenten benutzten schritte als Überschrift übernommen.

**Schritt 1 von 5**
im Virtual Machine Manager Rechtsklick auf die Verbindung, dort wählt man Neu und nun kann man wählen wie das Beitreibsystem gestartet werden soll.
Da diese Optionen wichtig sind auch im Detail zu verstehen nun erst mal alle Möglichkeiten  noch genau erklärt.

1. Lokales Installationsmedium (ISO-Abbild oder CDROM)
  Unter **CD-Rom oder DVD benutzen** wird das Gerät vom Wirt Computer benutzt also ohne dort jemanden zu haben der einem die Disk ins Laufwerk schiebt kaum eine Möglichkeit bei  in der Praxis.

Mit **ISO-Abbild benutzen** kann man ein iso auswählen zu dem es einen [Speicherdatenträger](../virt-manager-speicherdatentraeger) gibt.
wenn keine Auswahl Möglichkeiten  über ISO-Abbild benutzen -> Durchsuchen gibt weiter unter [Speicherdatenträger](../virt-manager-speicherdatentraeger)

2. Installation vom Netzwerk (HTTP, FTP oder NFS)
Hier kann es in der Praxis am einfachsten sein seine VM's aufzurollen, Voraussetzung ist eine URL die mit allen notwendigen Installationsdateien lesbar ist.

**Schritt 2 von 5**

Hier kann man nun seine URL angeben und bei URL Optionen die Kernel Optionen 

```sh
ks=http://srv2.example.com/~tobkern/iso/unattended/anaconda/anaconda-ks.cfg ksdevice=bootif network kssendmac ip=192.168.4.52 netmask=255.255.255.0 gateway=192.168.4.1 dns=192.168.4.14
```

**Schritt 3 von 5**

Hier einfach mal weiter die Speicher- und CPU-Einstellungen können auch noch nachträglich angepasst werden, wer genau weiß was er benötigt kann hier natürlich gleich alles eintragen.

**Schritt 4 von 5**

Für unser Beispiel soll ein vm mit einem 12GB Image reichen.

**Schritt 5 von 5**

Wichtig ist hier den Hacken bei **Konfiguration bearbeiten vor der Installation** zu setzen !
Als Name  sollte man ändern um die vm später auch wieder zu finden Z.b.  cent7-gold da wir hier gerade ein [gold Image](../gold-image) erstellen.

### Netzwerk Konfiguration

Der foreman-installer wird sich (weil da Python die Konformität Prüft) ohne eine RFC konforme [FQDN](https://de.wikipedia.org/wiki/Fully-Qualified_Host_Name) sich weigern eine Installation durchzuführen, daher muss noch ein passender Eintrag local in der /etc/hosts eingerichtet werden.

**Quellen**

* [foreman-on-centos-7-or-rhel-7](https://syslint.com/blog/tutorial/how-to-install-and-configure-foreman-on-centos-7-or-rhel-7/)
* [googel paste foreman ](https://groups.google.com/forum/#!topic/foreman-users/6xFo8mzDOF0)

### Firewall für Foreman einrichten

```sh
# source : https://www.unixmen.com/how-to-install-foreman-with-puppet-in-centos-and-ubuntu/
firewall-cmd --permanent --add-port=53/tcp
firewall-cmd --permanent --add-port=67-69/udp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=3000/tcp
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --permanent --add-port=5910-5930/tcp
firewall-cmd --permanent --add-port=5432/tcp
firewall-cmd --permanent --add-port=8140/tcp
firewall-cmd --permanent --add-port=8443/tcp

[[Restart]] Firewall to take effect the changes.

firewall-cmd --reload

```

### Fehler

`The environment must be purely alphanumeric, not ''`
Lösung:
[6091-the-environment-must-be-purely-alphanumeric](https://ask.puppet.com/question/6091/the-environment-must-be-purely-alphanumeric/)

### Puppet Module für Foreman
* [theforeman/foreman](https://forge.puppet.com/theforeman/foreman)

## Erweiterte Einstellungen


### libvirt Einrichten
Um sich mit dem Node Sicher zu verbinden müssen folgende schritte vorgenommen werden.

1. Rechte Überprüfen
```
root# mkdir /usr/share/foreman/.ssh
root# chmod 700 /usr/share/foreman/.ssh
root# chown foreman:foreman /usr/share/foreman/.ssh
```
2. Auf dem foreman Server den foreman user eine ssh verbindung einrichten.
```
root# su foreman -s /bin/bash
foreman$ ssh-keygen
foreman$ ssh-copy-id root@hostname.com
foreman$ ssh root@hostname.com
exit
```
3.  In der Weboberfläche die Resoource einrichten

### Foreman Konfigurationen
* [homent4](../foreman-examples)

### Foreman CI Quellen

* [Jenkins ci.theforeman.org](http://ci.theforeman.org/)

**Quellen:**
[foreman-host-builder](https://github.com/xnaveira/foreman-host-builder)
[Foreman API version 1.15](https://theforeman.org/api/1.15/index.html)