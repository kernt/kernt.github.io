
Install

`sudo apt-get install gddrescue`

**Dest image erstellen**

`sudo dd if=/dev/zero of=/path/to/destination/image bs=1M count=0 seek=SIZE`

**Defecttes /dev/sdX größe ermitteln**

sudo blockdev --getsize64 /dev/sdX

**Kopieren der Partition zu einem Image File Ausgabe Datei ist backup.img**

`sudo ddrescue -d /dev/sdX backup.img backup.logfile`

