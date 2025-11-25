---
tags:
  - pxe
  - networking
  - tftp
---
# Automatisierte Installation mit PXE-Boot und einer Preseed-Datei

Nun, da wir ein gespiegeltes Repository von Paketen haben, können wir es auch verwenden, um die Dateien zu bedienen, die unsere Hosts über das Netzwerk aufbauen. Das Erstellen von Bare-Metal-Servern über das Netzwerk hat viele Vorteile, so dass Sie einfach einen Bare-Metal-Server booten und über DHCP konfigurieren und ein Betriebssystem installieren können, ohne zusätzliche Interaktionen. Das PXE-Booten ermöglicht den Einsatz von diskless Clients, die booten und nur über das Netzwerk laufen können.

Dies ist ein sehr langes Rezept, aber es ist erforderlich. Obwohl es relativ einfach ist, eine PXE-Boot-Umgebung einzurichten, benötigt es mehrere Elemente. Im Laufe dieses Rezepts werden Sie drei Hauptkomponenten erstellen: einen Apache-Server, einen DHCP-Server (Dynamic Host Configuration Protocol) und einen TFTP-Server (Trivial File Transfer Protocol). Alle diese werden zusammenarbeiten, um die benötigten Dateien zu bedienen, damit ein Client booten und z.B Ubuntu installieren kann. Obwohl es hier mehrere Komponenten gibt, können sie alle bequem auf einem einzigen Server laufen.

Eine Alternative zu diesem Rezept ist das Projekt (https://cobbler.github.io). Cobbler bietet die meisten dieser Elemente from the Box und fügt eine leistungsfähige Management-Schicht auf die Obersteebene ein; Allerdings ist es sehr umfangreich, wie es funktioniert und muss ausgewertet werden, um zu sehen, wie es in Ihre Umgebung passt, aber es lohnt sich, es zu testen.

Es lohnt sich zu bedenken, dass dieses Rezept für Bare-Metal-Server-Installationen konzipiert ist, und im Allgemeinen ist es nicht der beste Weg, um virtualisierte oder Cloud-basierte Server zu verwalten. In solchen Fällen bietet der Hypervisor oder Provider fast sicher eine bessere und optimiertere Installationsmethode für die Plattform an.

### Fertig werden

Um diesem Rezept zu folgen, empfiehlt es sich, einen Host mit einer sauberen Installation von Ubuntu 14.04 zu haben. Idealerweise sollte dieser Host mindestens 20 GB oder mehr Festplatte haben, denn zumindest muss er das Ubuntu-Setup-Medium enthalten.

### Wie es geht…

### Lassen Sie uns eine PXE-Boot-Umgebung einrichten:

1. Die erste Komponente, die wir konfigurieren werden, ist der TFTP-Server. Dies ist eine abgespeckte Version von FTP. TFTP eignet sich hervorragend für das Booten von Netzwerken, wo Sie einen unidirektionalen Datenfluss haben, der einfach und schnell ausgeliefert werden muss. Wir werden den TFTP Server benutzen, der mit Ubuntu 14.04 mitgeliefert wird. Um es zu installieren, geben Sie den folgenden Befehl ein:

`$ sudo apt-get install tftpd-hpa`

Dadurch werden die Pakete und deren Abhängigkeiten installiert.

2. Als nächstes müssen wir unseren TFTP-Server konfigurieren. Verwenden Sie Ihren bevorzugten Editor, bearbeiten Sie die TFTP-Konfigurationsdatei unter `/etc/default/tftpd-hpa`. Standardmäßig sollte es dem ähneln:

```s
# /etc/default/tftpd-hpa
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS="[::]:69"
TFTP_OPTIONS="--secure"
```

Sie müssen dies ändern, damit es als Daemon laufen kann. Passen Sie die Datei an, um die folgende Zeile hinzuzufügen:

```s
# /etc/default/tftpd-hpa
RUN_DAEMON="yes"
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS="[::]:69"
TFTP_OPTIONS="--secure"
```

3. Damit kann der Prozess in einem Deamon Modus gestartet werden kann. Beachten Sie auch das TFTP-Verzeichnis. Wenn Sie gewählt haben, um Ihr Installationsmedium an einem anderen Ort zu speichern, müssen Sie dieses Verzeichnis ändern. Schließlich starten Sie den TFTP-Server mit folgendem Befehl:

`sudo service tftpd start`

4.Nun, da wir unseren TFTP-Server konfiguriert haben, müssen wir ihm einige Daten geben, um zu das System mit daten bedienen zu können. In diesem Fall werden wir die Ubuntu-Installationsdateien in unser TFTP-Verzeichnis kopieren, damit sie den Clients PXE-Booten mit diesem Server dienen können. Wenn du noch nicht kommst, lade die Ubuntu 14.04 installiere ISO von Ubuntu auf deinen TFTP Server; Sie können es herunterladen von: http://www.ubuntu.com/download/server. Sobald Sie es heruntergeladen haben, gehen Sie vor und mounteSie es auf das `mnt`-Verzeichnis mit dem folgenden Befehl:

`sudo mount -o loop <location of ISO> /mnt`

5. Sobald die ISO montiert ist, können Sie sie in das TFTP-Stammverzeichnis kopieren. Du brauchst eigentlich nicht das ganze ISO-Image, nur den Inhalt des `netboot`-Verzeichnisses. Kopiere es mit folgendem Befehl:
`cp -r /mnt/install/netboot/* /var/lib/tftpboot/`

Beachten Sie, dass ich es an den Standardspeicherort für den TFTP-Server kopiere. Dies ist konfigurierbar, wenn Sie das ISO-Image auf dem zentralen Speicher wie einem NFS-Server behalten möchten.

6. Schließlich müssen wir eine kleine Bearbeitung zu den Dateien machen, die wir kopiert haben, um unsere Clients von unserer `PreSeed` Datei zu booten. Öffnen Sie die folgende Konfigurationsdatei in Ihrem bevorzugten Editor:

`/var/lib/tftpboot/pxelinux.cfg/default`

Fügen Sie folgendes ein:

```sh
label linux
        kernel ubuntu-installer/amd64/linux
        append preseed/url=http://<<NAME OF BOOT SERVER>>/ks.cfg vga=normal initrd=ubuntu-installer/amd64/initrd.gz ramdisk_size=16432 root=/dev/rd/0 rw  --
```

Es gibt ein paar Dinge über die vorherige Konfiguration zu beachten. Erstens basiert es auf der 64-Bit-Installation von Ubuntu, so dass sich Ihre Architektur unterscheiden kann. Zweitens, beachten Sie die Zeile, die liest:
`pressed/url=http:// <<NAME OF BOOT SERVER>>//ks.cfg`

Dies sollte die IP-Adresse (oder noch besser, DNS-Name) des Servers, die Sie als PXE-Boot-Server konfiguriert haben, widerspiegeln.

7. Als nächstes müssen wir einen DHCP-Server konfigurieren, um unsere neu gestarteten Clients mit einigen grundlegenden Netzwerkinformationen zu versorgen. Sie können diesen Abschnitt überspringen, wenn Sie bereits einen DHCP-Server haben und gehen Sie direkt zum nächsten Abschnitt. Allerdings müssen Sie Ihren DHCP-Server so konfigurieren, dass er auf die Clients zeigt, die auf Ihren PXE-Server booten.

Wenn Sie nicht sicher sind, ob Sie einen DHCP-Server haben oder nicht, konsultieren Sie die Personen, die Ihr Netzwerk verwalten. Nichts ist mehr garantiert, um Ihren Netzwerkadministrator zu hacken, als einen DHCP-Server zu erstellen, wenn sie bereits einen haben. Im besten Fall wird es nichts tun; Im schlimmsten Fall kann es ernsthafte Probleme in Ihrem Netzwerk verursachen und sogar Produktionsprobleme verursachen. Wenn im Zweifel, fragen Sie.

Wenn du noch keinen DHCP-Server hast, dann ist es ziemlich einfach zu installieren und zu konfigurieren. Zuerst installieren wir die benötigten Pakete für den DHCP-Server, der mit Ubuntu mit folgendem Befehl versendet wird:
`sudo apt-get install isc-dhcp-server`

8. Als nächstes konfigurieren wir unseren neu installierten DHCP-Server. Ich benutze den IP-Bereich, den ich in meinem Testlabor als Beispiel (10.0.1.0) verwende, aber gehe vor und ändere die Beispiele nach deinem Setup. Öffnen Sie die folgende Konfigurationsdatei mit Ihrem bevorzugten Editor:
`/etc/dhcp/dhcpd.conf`

Die ersten Optionen, die wir einstellen müssen, sind unsere Domain- und Nameserver. Standardmäßig sollte die Konfiguration wie folgt aussehen:

```sh
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;
```

Wir müssen das ändern, um unserem Setup zu entsprechen. In meinem Fall sieht es so aus:

```sh
option domain-name "stunthamster.com";
option domain-name-servers ns1.stunthamster.com, ns2.stunthamster.com;
```

9. Ändern Sie sie, um Ihre eigenen Domain- und Nameserver zu entsprechen. Als nächstes müssen wir das den autorisierenden DHCP-Server für dieses Netzwerk machen. Suchen Sie die Zeile, die liest:

`authoritative`

Stellen Sie sicher, dass es unkommentiert ist. Damit wird sichergestellt, dass der DHCP-Server zum Verwalten des Netzwerkbereichs verwendet wird und die Kunden Leasingstücke anmutig aufgeben und so weiter.

10. Schließlich können wir die DHCP-Konfiguration für unser Netzwerk erstellen. Dies sollte am unteren Rand der Konfigurationsdatei hinzugefügt werden. Noch einmal ist das folgende Beispiel für mein Netzwerk. Sie sollten die Werte für Ihren eigenen IP-Bereich ersetzen:

```s
subnet 10.0.1.0 netmask 255.255.255.0 {
 range 10.0.1.20 10.0.1.200;
 option domain-name-servers ns1.stunthamster.com;
 option domain-name "stunthamster.com";
 option routers 10.0.1.1;
 option broadcast-address 10.0.1.255;
 allow booting;
 allow bootp;
 option option-128 code 128 = string;
 option option-129 code 129 = text;
 next-server 10.0.1.11;
 filename "pxelinux.0";
 default-lease-time 600;
 max-lease-time 7200;
 }
```

Notieren Sie sich die `next-server` Option: Dies sagt dem Client, wo Ihr TFTP-Server ist und sollte auf Ihren Server passen.

Obwohl dein nächster Server (TFTP) der gleiche sein kann wie dein DHCP-Server, und in diesem Beispiel ist es besser, ihn in der Produktion zu trennen. Obwohl sie in den letzten Jahren besser geworden sind, sind TFTP-Server immer noch als unsicher und es ist besser, sicher zu spielen und TFTP auf seinem eigenen Server zu verlassen.

11. Sobald Sie mit Ihren Einstellungen zufrieden sind, speichern Sie die Konfiguration und starten Sie den DHCP-Server mit dem folgenden Befehl neu

`sudo service isc-dhcp-server restart`

Für unsere nächste Aufgabe gehen wir weiter und konfigurieren unseren Nginx Server. Wir verwenden Nginx, um sowohl das Installationsmedium als auch die vorgewählte Konfiguration über http zu hosten. Im Wesentlichen verbindet sich der Client mit dem Server, der in der Kernel-Konfiguration angegeben ist, um sein Installationsmedium herunterzuladen und Anweisungen vorzubereiten, sobald es das PXE-Boot-Tool zum Starten des Kernels verwendet hat.

Obwohl ich Nginx verwende, kannst du jeden beliebigen HTTP-Server deiner Wahl verwenden, zum Beispiel Apache. Nginx ist mein bevorzugter Server in diesen Fällen, da es klein ist, einfach zu konfigurieren und sehr performant bei der Bereitstellung von statischen Assets:

1. Zuerst installieren wir nginx mit folgendem Befehl:

`sudo apt-get install nginx`

2. Als nächstes müssen wir es konfigurieren, um das Installationsmedium zu bedienen, das wir im vorherigen Schritt kopiert haben. Öffnen Sie mit Ihrem Editor die folgende Konfigurationsdatei:

`/etc/nginx/sites-available/default`

Standardmäßig ähnelt die Konfiguration wie das folgende Code-Snippet (ich habe Kommentare für Klarheit entfernt):

```s
server {
         listen 80 default_server;
         listen [::]:80 default_server ipv6only=on;
         root /usr/share/nginx/html;
         index index.html index.htm;
         server_name localhost;
         location /
         {
           try_files $uri $uri/ =404;
         }
       }
```

Ändern Sie es wie folgt:

```s
server {
         listen 80 <<BOOT SERVERNAME>>;
         listen [::]:80 <<BOOT SERVERNAME>>ipv6only=on;
         root /var/lib/tftpboot/;
         index;
         server_name <<BOOT SERVERNAME>>;
         location /
         {
           try_files $uri $uri/ =404;
         }
       }
```

Ersetzen Sie die Zeile, die` << BOOT SERVERNAME >>` im vorherigen Beispiel mit dem DNS-Namen Ihres Boot-Servers liest.

3. Diese Konfiguration dient dem Inhalt Ihres TFTP-Verzeichnisses und ermöglicht es Ihren Clients, die Ubuntu-Installationsdateien herunterzuladen. Denken Sie daran, dass diese Konfiguration keine Sicherheit hat und ermöglicht es den Menschen, den Inhalt des Verzeichnisses zu durchsuchen. So stellen Sie sicher, dass Sie in diesem Verzeichnis keine sensible Art platzieren!

4. Schließlich können wir die `Preseed`-Datei konfigurieren. Die `Preseed`-Datei ist im Wesentlichen eine Datei, die die Antworten auf die Fragen enthält, die der interaktive Installateur von Ubuntu darstellen wird, so dass völlig unbeaufsichtigte Installationen möglich sind. Schauen wir uns eine `Preseed`-Datei an und konstruieren sie in Stufen. Erstellen Sie die folgende Datei in Ihrem Editor:

`/var/lib/tftpboot/ks.cfg`

5. Zuerst wollen wir unseren Installateur darauf hinweisen, das lokale Repository zu verwenden, das wir im vorherigen Rezept erstellt haben:

`d-i apt-setup/use_mirror boolean true`
`choose-mirror-bin | mirror/http/hostname string <HOSTNAME OF MIRROR>`

Ändern Sie das vorhergehende Beispiel, um Ihren lokalen Spiegel zu reflektieren.

Sie müssen diese Option nicht unbedingt einstellen. Wenn es unberührt bleibt, benutzt Ubuntu das offizielle Repository, um die Installation durchzuführen. Allerdings, wie in der ersten Rezept in diesem Kapitel erwähnt, Gebäude etwas mehr als eine Handvoll von Servern ist viel schneller mit einem lokalen Spiegel.

6. Lassen Sie uns mit einigen grundlegenden Einstellungen, welche Sprache zu verwenden, was den Hostnamen zu setzen, unser Gebietsschema für die Zwecke der Tastatur, und die Einstellung der Zeitzone des Servers, den wir bauen. Wir können dies mit dem folgenden Code-Snippet tun:

```sh
d-i debian-installer/locale string en_UK.utf8
d-i console-setup/ask_detect boolean false
d-i console-setup/layout string UK
d-i netcfg/get_hostname string temp-hostname
d-i netcfg/get_domain string stunthamster.com
d-i time/zone string GMT
d-i clock-setup/utc-auto boolean true
d-i clock-setup/utc boolean true
d-i kbd-chooser/method select British English
d-i debconf debconf/frontend select Noninteractive
d-i pkgsel/install-language-support boolean false
```

7. Als nächstes müssen wir dem Installateur mitteilen, wie die Festplatten auf unserem Host konfiguriert werden. Das folgende Snippet nimmt einen einzelnen Festplattenhost an und entfernt alle vorhandenen Partitionen. Ich habe auch den Partitionsmanager angewiesen, die Gesamtheit der Festplatte zu verwenden und ein **Logical Volume Manager (LVM)** Gerät einzurichten:

```sh
d-i partman-auto/method string lvm
d-i partman-auto/purge_lvm_from_device boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-auto/choose_recipe select atomic
d-i partman/confirm_write_new_label boolean true
d-i partman/confirm_nooverwrite boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
preseed partman-lvm/confirm_nooverwrite boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true
d-i partman-auto-lvm/guided_size string max
```

1. Der nächste Satz von Antworten beschäftigt sich mit der Benutzerverwaltung:

Zuerst müssen wir unseren Standardbenutzer konfigurieren. Standardmäßig erlaubt Ihnen Ubuntu nicht, sich direkt als `root`-Benutzer anzumelden (eine unglaublich gute Übung!), Sondern ermöglicht es Ihnen, einen Benutzer zu erstellen, der für Administrationszwecke verwendet werden soll. Das folgende Snippet wird einen Benutzer von `adminuser`mit einem `passwort`des Passwortes erstellen. Ändern Sie diese Werte entsprechend Ihrem eigenen Setup.

Im folgenden Beispiel wird ein verschlüsseltes Passwort verwendet. Dies stellt sicher, dass die Leute das Passwort für Ihren Standardbenutzer nicht sehen können, indem Sie einfach Ihr TFTP-Repository durchsuchen. Um das verschlüsselte Passwort zu erstellen, kannst du den Befehl `mkpasswd -m sha-512` in einer Linux-Befehlszeile verwenden:

```sh
d-i passwd/user-fullname string adminuser
d-i passwd/username string changeme
d-i passwd/user-password-crypted password <<CRYPTED_PASSWORD>>
d-i user-setup/encrypt-home boolean false
```

2. Schließlich sagen wir dem Installateur, welche Pakete als Teil der Basisinstallation installiert werden sollen. Im Allgemeinen möchten Sie diese Pakete auf diejenigen beschränken, die Sie benötigen, um Ihr Konfigurationsmanagement-Tool auszuführen und sonst nichts. Dies hält die Basisinstallation klein und sorgt auch dafür, dass Sie Pakete über Ihr Konfigurationsmanagement-Tool verwalten. Das folgende Snippet installiert einen Openssh-Server, damit Sie sich bei der Erstellung des Servers anmelden und die automatischen Updates deaktivieren können. Vielleicht möchten Sie dies an, aber ich ziehe es vor, es zu verlassen, damit ich weiß, dass nur die Pakete, die ich explizit installieren, auf die Server gedrückt werden, die ich baue.

```sh
d-i pkgsel/include string openssh-server
d-i pkgsel/upgrade select full-upgrade
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true
d-i finish-install/reboot_in_progress note
d-i pkgsel/update-policy select none
```

Sobald Sie mit Ihrer Konfiguration zufrieden sind, speichern Sie die Datei.

3. Es ist ein langer Slog, aber wir sind bereit, unseren ersten Client von unserem glänzenden neuen Build-Server zu bauen. Um dies zu tun, stellen Sie sicher, dass Ihr Client mit demselben Netzwerk wie Ihr `PreSeed`-Server verbunden ist und konfigurieren Sie Ihre Client-Boot-Reihenfolge, um `PXE-Boot` zuerst auszuwählen und neu zu starten.

Obwohl seine seltene, einige Kunden sind nicht in der Lage zu verwenden PXE zu booten; Dies ist vor allem bei älteren Hardware vorrangig. In solchen Fällen können Sie Ihre Preseed-Datei weiterhin verwenden, aber Sie müssen ein benutzerdefiniertes Boot-Medium erstellen, um Ihren recalcitrant-Client zu booten. Sie finden Anleitungen zum Erstellen dieses unter https://help.ubuntu.com/community/LiveCDCustomization.

Wenn alles gut geht, sollten Sie mit einem Bildschirm, der schnell Reißverschlüsse durch die Ubuntu-Installations-Bildschirme, alle ohne Sie brauchen, um einen Finger zu heben und Sie sollten in der Lage sein, sich in Ihrem frisch gebauten Server mit den Anmeldeinformationen, die Sie in Ihrem Preseed-Datei Wenn es fertig ist.
### Siehe auch

Wir haben in diesem Rezept viel Boden gedeckt, und ich empfehle Ihnen sehr, die folgende Dokumentation zu lesen, um ein tieferes Verständnis dafür zu erhalten, wie jede Komponente konfiguriert ist und auch die verfügbaren Optionen zu untersuchen:
DHCP help:

https://help.ubuntu.com/community/isc-dhcp-server

Official Ubuntu Preseed documentation:

https://help.ubuntu.com/14.04/installation-guide/amd64/apb.html

Example Preseed:

https://help.ubuntu.com/lts/installation-guide/example-preseed.txt