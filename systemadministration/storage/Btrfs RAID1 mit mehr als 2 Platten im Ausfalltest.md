---
tags:
  - btrfs
---
Nachdem ich den Testaufbau eingerichtet hatte, wollte ich noch testen ob die Daten im Pool wirklich einen Festplattenausfall überstehen.

Für den Aufbau wurde Ubuntu 16.04.04 genutzt mit HWE Kernel.  
Die Btrfs Version ist die 4.4, welche der aktuellen aus den Repositorys entspricht.  
Die Festplatten sind 3× 32GiB virtuelle Disks im Format RAW unter Proxmox.

Dafür habe ich den Pool mit Bildern in JPEG sowie RAW gefüllt. Später wurde der Datenbestand um eine .iso erweiter.
#### Hash erstellen

Für jede Datei wird nun ein MD5 Hash gezogen um nach dem Ausfall zu überprüfen, ob sich die Datei geändert hat.  
Die Datei mit den Hashwerten, wird unter /tmp gespeichert, damit diese sich nich selbst ein Hash verpasst. Da dieser im Anschluss in die Datei geschrieben wird und der Hash für die Datei nichtmehr passt.

`find . -type f -print0 | xargs -0 md5sum > /tmp/MD5SUM`
#### Ausfall herbeiführen

```
root@testsystem:~# btrfs fi show
Label: 'storage'  uuid: 2501b7ad-0bdb-427a-a969-8d07fbe67070
        Total devices 3 FS bytes used 7.53GiB
        devid    1 size 32.00GiB used 6.00GiB path /dev/vdb
        devid    2 size 32.00GiB used 6.01GiB path /dev/vdc
        devid    3 size 32.00GiB used 6.01GiB path /dev/vdd
```

Die Platte habe ich im laufenden Betrieb, in Proxmox von der VM detatcht.

Der Ausfall ist auch sofort sichtbar.

```
root@testsystem:/mnt# btrfs fi show
Label: 'storage'  uuid: 2501b7ad-0bdb-427a-a969-8d07fbe67070
        Total devices 3 FS bytes used 7.53GiB
        devid    1 size 32.00GiB used 6.00GiB path /dev/vdb
        devid    3 size 32.00GiB used 6.01GiB path /dev/vdd
        *** Some devices missing
```

Anscheinend werden fehlende Devices aber nicht in jedem Fenster angezeigt.

```
root@testsystem:/mnt# btrfs fi usage /mnt
Overall:
    Device size:                  96.00GiB
    Device allocated:             18.02GiB
    Device unallocated:           77.98GiB
    Device missing:               32.00GiB
    Used:                         15.06GiB
    Free (estimated):             39.47GiB      (min: 39.47GiB)
    Data ratio:                       2.00
    Metadata ratio:                   2.00
    Global reserve:               16.00MiB      (used: 0.00B)
Data,RAID1: Size:8.00GiB, Used:7.52GiB
   /dev/vdb        5.00GiB
   /dev/vdc        6.00GiB
   /dev/vdd        5.00GiB
Metadata,RAID1: Size:1.00GiB, Used:8.69MiB
   /dev/vdb        1.00GiB
   /dev/vdd        1.00GiB
System,RAID1: Size:8.00MiB, Used:16.00KiB
   /dev/vdc        8.00MiB
   /dev/vdd        8.00MiB
Unallocated:
   /dev/vdb       26.00GiB
   /dev/vdc       25.99GiB
   /dev/vdd       25.99GiB
```

Um den Fall noch realistischer zu gestalten, kommen zu dem Pool aber noch Dateien hinzu. Im produktiven Einsatz kommen auch neue Dateien hinzu, ehe die defekte Platte ausgetauscht werden konnte.

Hier wurde ein Hash erzeugt, bevor die Datei auf den Pool geladen wurde.

`md5sum <Datei> > MD5SUM`

Nach der Übertragung der erste Vergleich.

```
vorher: 6cb5a4144f81cc6308a4e7c64c2d3eeb
nachher: 6cb5a4144f81cc6308a4e7c64c2d3eeb
```

Das Speichern von einer Datei auf dem degraded Pool sorgt schonmal für keine Schwierigkeiten.

Um die Sache noch etwas interessanter zu gestalten wurde die Maschine noch hart ausgeschalten um zb. einen Fehler in der Stromversorgung zu simulieren.

Gleichzeitig nutze ich die Downtime um der VM die neue Platte mit zugeben.
#### Plattenaustausch

Nachdem die Maschine wieder gestartet ist, wurde die fehlende Platte durch eine neue ersetzt. Diese besitzt eine Größe von 60GiB, um den Fall zu testen, dass keine kleineren Festplatten mehr greifbar sind.

Durch den Neustart der Maschine ist die Platte, welche ehemals `vdd` war, nun `vdc`. Die nach dem Start eingehangene 60GiB Festplatte wird nun als `vdd` adressiert.

Btrfs scheint dieses Wechselspiel nicht zu stören. Da die Platten über eine Sub-UUID, welche von Btrfs vergeben wird, adressiert werden.

```
/dev/vdb: LABEL="storage" UUID="2501b7ad-0bdb-427a-a969-8d07fbe67070" UUID_SUB="7303d213-b68c-4d03-be31-1cf10e23522e" TYPE="btrfs"
/dev/vdc: LABEL="storage" UUID="2501b7ad-0bdb-427a-a969-8d07fbe67070" UUID_SUB="ed217b2f-55a5-4aa6-bbba-83458f328ef9" TYPE="btrfs"
```

```
root@testsystem:/mnt# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vdd                       252:48   0   60G  0 disk
vdb                       252:16   0   32G  0 disk /mnt
sr0                        11:0    1  829M  0 rom
vdc                       252:32   0   32G  0 disk
vda                       252:0    0   32G  0 disk
├─vda2                    252:2    0    1K  0 part
├─vda5                    252:5    0 31.5G  0 part
│ ├─testsystem--vg-swap_1 253:1    0    2G  0 lvm  [SWAP]
│ └─testsystem--vg-root   253:0    0 29.5G  0 lvm  /
└─vda1                    252:1    0  487M  0 part /boot
```

Die fehlende Festplatte konnte nun einfach [ausgetauscht](https://techgoat.net/index.php?id=91) werden.
Die Übersicht sieht hiernach auch wieder sauber aus. Wenn auch nicht ganz synchron verteilt.

```
root@testsystem:~# btrfs fi show
Label: 'storage'  uuid: 2501b7ad-0bdb-427a-a969-8d07fbe67070
        Total devices 3 FS bytes used 8.04GiB
        devid    1 size 32.00GiB used 6.03GiB path /dev/vdb
        devid    2 size 32.00GiB used 7.03GiB path /dev/vdc
        devid    3 size 32.00GiB used 7.00GiB path /dev/vdd
```
#### Hash Auswertung

Die beiden Dateien vor und nach dem Test sind bis darauf, dass die eine Datei hinzugekommen ist, identisch.  
Da sich die Hashwerte nicht verändert haben, dürften die Daten auch nichts an Intigrität verloren haben.