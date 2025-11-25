---
tags:
  - tftpd
---
# tftp Installation

|Distribution|Packetname|Info|
| :---: | :---: | :---: |
|Debian|tftpd-hpa|Server|

Installation Übersicht zu den Linux Distributionen

*[Debain/Ubuntu](../tftpd-install-debian)

Nach Jeder Änderung muss der tftp Server mit :

`/etc/init.d/tftpd-hpa restart`

oder

`systemctl restart tftpd-hpa`

neu gestartet werden.
## Test für den Server

**Test ob der Server Läuft**
`pgrep -lf tftpd`
## Konfiguration

### Boot Images

Debian Boot Image erstellen

```sh
VERSION=8 # jessie, 7.0 for wheezy
ARCH=amd64 # or any other release architecture
apt-get install debian-installer-$VERSION-netboot-$ARCH
```
### Menu Konfigurieren

Ubuntu Einträge

```sh
  menu label Ubuntu 16.04 i386 Preseed
  kernel /ubuntu16.04/i386/linux
  append initrd=/ubuntu16.04/i386/initrd.gz vga=normal ramdisk_size=16384 root=/dev/ram rw preseed/url=ftp://mirror.home.lan/pub/linux/preseed/xenial.seed debian-installer/locale=de_DE keyboard-configuration/layoutcode=de localechooser/translation/warn-light=true localechooser/translation/warn-severe=true netcfg/choose_interface=auto netcfg/get_hostname=ubuntu --

label ubuntu16.04_amd64_ps
  menu label Ubuntu 16.04 amd64 Preseed
  kernel /ubuntu16.04/amd64/linux
  append initrd=/ubuntu16.04/amd64/initrd.gz vga=normal ramdisk_size=16384 root=/dev/ram rw preseed/url=ftp://mirror.home.lan/pub/linux/preseed/xenial.seed debian-installer/locale=de_DE keyboard-configuration/layoutcode=de localechooser/translation/warn-light=true localechooser/translation/warn-severe=true netcfg/choose_interface=auto netcfg/get_hostname=ubuntu --
```

## Fehlerbehebung

Um die Logs durch kann entweder

`tail -f /var/log/syslog`

oder systemd systemen

 `journalctl -fu tftpd-hpa`

benutzt werden.

Source:

* [gtkbd](http://www.gtkdb.de/index_34_2792.html)
* [Debian tftp Wiki](https://wiki.debian.org/PXEBootInstall?action=show&redirect=DebianInstaller%2FNetbootPXE)
