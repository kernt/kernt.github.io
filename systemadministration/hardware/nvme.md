---
tags:
  - nvme
  - hardware
---
# NVME Firmware

Die HGST NVME Decives in den Nodes können mit neuer Firmware bestückt werden, sofern diese vorliegt.
Ausgeliefert wurden die Devices mit FW110, z.Zt. läuft FW114, die wir über EuroStor erhalten haben.

Im Homeverzeichnis von root findet man auf jeder Node einen Ordner nvme_firmware.
Darin findet man alles was man zum Update der Firmware benötigt.

    Update Tool (dm-1.7.0-Linux)
    FW Image KNGND114.bin (enthalten in dem .7z-File)
    Eine Readme, die diesen Artikel nochmal in die drei kurzen Befehle zusammenfasst.

Device Scan

`root@ceph-1:~ ./nvme_firmware/dm-1.7.0-Linux/bin/dm-cli scan`

Das dient nur dazu damit das dm Tool von HGST die Devices kennt, denn sonst funktionieren die weiteren Funktionen von dem Tool nicht.
Aktuelle Firmware auf dem Device anzeigen

```sh
root@ceph-1:~ ./nvme_firmware/dm-1.7.0-Linux/bin/dm-cli manage-firmware -p /dev/nvme0 -l
[/dev/nvme0]
  Product Name                 = Ultrastar
  Device Type                  = NVMe Controller
  Device Path                  = /dev/nvme0
  UID                          = 1C58SDM00007EC41HUSMR7616BHP3010023
  Alias                        = @nvme1
  Running Firmware Version     = KNGND114
  Running Firmware Source Slot = 2
  Slot Info =
    Slot Number Read Only Next Running Firmware Slot Firmware Version
    1           true      false                      KNGND110        
    2           false     true                       KNGND114        
    3           false     false                      KNGND110        
    4           false     false                      KNGND110        
    5           false     false                      KNGND110        

Results for manage-firmware: Operation succeeded.
```

Das gibt einem Informationen darüber welche Firmware auf dem Device z.Zt. geladen ist und eine Liste mit den Firmwareversionen, die auf dem Device installiert sind.
Man könnte insgesamt 5 verschiedene Versionen auf einem Device installieren (Slot 1 5), wobei Slot 1 read only mit der Auslieferungsversion ist.

Wie man sieht ist in Slot 2 FW v114 installiert und diese läuft z.Zt. auch.

Firmware aktualisieren
```
# Synopsis: dm-cli manage-firmware -p /dev/nvme[0/1] -f firmware_image.bin -s FW_SLOT_NUM --activate --load

# Bsp.
root@ceph-1:~ ./nvme_firmware/dm-1.7.0-Linux/bin/dm-cli manage-firmware -p /dev/nvme0 -f nvme_firmware/KNGND114.bin -s 2 --activate --load
```

Nachdem die Firmware geflasht wurde muss der Host einmal neu gestartet werden, sprich das übliche Verfahren um ne Ceph-node zu reboot (noout,norebalance etc.)

Log initialisieren
`nvme smart-log /dev/nvme0n1`

Log SMART Status ausgeben
`nvme smart-log /dev/nvme0n1`
 
Den Error log ausgeben
`nvme error-log /dev/nvme0n1`

