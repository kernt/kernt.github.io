---
tags:
  - lvm
  - system-administration
  - storage
  - gnu-tools
---
# LVM

Physical volume (PV)
Volume group (VG)
Logical volume (LV)
Physical extent (PE)

- pvchange = Attribute ändern
- pvck = Metadaten Prüfen
- pvcreate = Erstellen einer PV
## Neue PV anlegen einer VG hinzufügen, LV VG zuordnen

```sh
pvcreate /dev/sdb1
vgcreate diskpool /dev/sdb1
lvcreate -L 10GB -n testLV diskpool
```
## Physikalische Platten verwalten

**Neue Festplatte hinzufügen**

`pvcreate /dev/sda`

**Volume reduzieren**

`pvresize --setphysicalvolumesize 40G /dev/sda1`

**Ifos zu pv anzeigen**

`pvdisplay -v -m`

**Freie Segmente verschieben**

`pvmove --alloc anywhere /dev/sdd1:307201-399668 /dev/sdd1:0-92467`

**PV zu vg hinzufügen**
```sh
pvcreate /dev/sdb1
vgextend MyVolGroup /dev/sdb1
```

**Daten verschieben**
`pvmove /dev/sdb1 /dev/sdf1`

**PV von der vg verschieben**
`vgreduce MyVolGroup /dev/sdb1`

**faild disk los werden**

`vgreduce --removemissing --force MyVolGroup`
## Volume Groups verwalten 

**Neue Volume Gruppe anlegen**
`vgcreate MyVolGroup /dev/sdb1`

**Volume Group vergrößern**
`vgextend $VG /dev/sda3`

**Aktiviren einer volume groupe**

`vgchange -a y MyVolGroup`

**Deactiviren einer volume group**

`vgchange -a n MyVolGroup`

**Umbenenen einer volume group**

`vgrename MyVolGroup my_volume_group`

## Logische Laufwerke verwalten

**Vergrößern des LV**
`lvextend -L 1G /dev/mapper/deviceN`

> ACHTUNG 1G ist die Größe das Gerät insgesamt hat!

**To create a LV `homevol` in a VG `MyVolGroup` with 300 GiB of capacity, run:**

`lvcreate -L 300G MyVolGroup -n homevol`

**or, to create a LV `homevol` in a VG `MyVolGroup` with the rest of capacity, run:**

`lvcreate -l 100%FREE MyVolGroup -n homevol`

**lv umbenennen**

`lvrename MyVolGroup old_vol new_vol`

**Extend the logical volume `mediavol` in `MyVolGroup` by 10 GiB and resize its file system _all at once_:**

`lvresize -L +10G --resizefs MyVolGroup/mediavol`

**Set the size of logical volume `mediavol` in `MyVolGroup` to 15 GiB and resize its file system _all at once_:**

`lvresize -L 15G --resizefs MyVolGroup/mediavol`

**If you want to fill all the free space on a volume group, use the following command:**

```sh
lvresize -l +100%FREE --resizefs MyVolGroup/mediavol
lvcreate -L 300G MyVolGroup -n homevol /dev/sda1
```

**Vergrößern des LV auf die maximale Größe**

`lvextend -l +100%FREE /dev/mapper/deviceN`


## LVM Snapshots

**Snapshot erstellen**

`lvcreate --size 100M --snapshot --name snap01vol /dev/MyVolGroup/lvol`

> Hier können 100M an daten verändert werde bevor das Snapshot Volume voll ist

**Anderungen mit übernehmen**

`lvconvert --merge /dev/MyVolGroup/snap01vol`

## LVM Cache pool verwenden

Zur Beschleunigung kann 

Konvertieren einer schnellen geräts (`/dev/_fastdisk_`) zu PV und fügen Sie Ihre bestehende VG zu (`MyVolGroup`

vgextend MyVolGroup /dev/ _schnelles Diskette_

Erstellen Sie einen Cache-Pool mit automatischen Metadaten`/dev/_fastdisk_`und konvertieren Sie das bestehende LV`MyVolGroup/rootvol`zu einem abgespeicherten Band, alles in einem Schritt:

`lvcreate --type cache --cachemode writethrough -l 100%FREE -n root_cachepool MyVolGroup/rootvol /dev/fastdisk`

Cachemode hat zwei Möglichkeiten:

- `writethrough`sorgt dafür, dass alle geschriebenen Daten sowohl im Cache-Pool LV als auch auf dem Ursprung LV gespeichert werden. Der Verlust eines Geräts, das in diesem Fall mit dem Cache-Pool LV verbunden ist, würde nicht den Verlust von Daten bedeuten.
- `writeback`sorgt für eine bessere Leistung, aber auf Kosten eines höheren Datenverlustrisikos für den Fall, dass der für Cache verwendete Laufwerk ausschlägt.

**cache entvernen**

`lvconvert --uncache MyVolGroup/rootvol`

## LVM Raid erstellen

physische Volume erstellen.

`pvcreate /dev/sda2 /dev/sdb2`

Erstellen  Volumengruppe auf den physikalischen Volumen

`vgcreate MyVolGroup /dev/sda2 /dev/sdb2`

Erstellen Sie logische Volums `lvcreate --type _raidlevel_`

```sh
lvcreate -- RaidLevel [OPTIONS] -n Name -L Size VG [PVs]
# raid0
lvcreate -n myraid1 -i 2 -I 64 -L 70G VolGroup00 /dev/nvme1n1p1 /dev/nvmme0n1p1
# raid1
lvcreate -- raid1 -- s. s. s. s. 1 -L 20G -n myraid1 MyVolGroup /dev/sda2 /dev/sdb2
# rail10
lvcreate -n myraid1vol -L 100G -- typ raid10 -m 1 -i 2 MyVolGroup /dev/sdd1 /dev/sdc1 /dev/sdb1 /dev/sda5
```

**Bestehende verbänden in Raid konvertieren**

`lvconvert -- raid10 /dev/vg01/lv01`
`lvconvert -- raid10 /dev/vg01/lv01 /dev/sda1 /dev/sdb2 /dev/nvm0n1p1 ...`

**You can keep track of the progress of conversion with**

`watch lvs -o name,vg_name,copy_percent`

## lvm thin Provisioning

**ThinPool erstellen**

`lvcreate --type thin-pool -n MyThinPool -l 95%FREE MyVolGroup`

**thin snapshot**

`lvcreate -n GenericRoot -V 30G --thinpool MyThinPool MyVolGroup`

**thin snapshot für jede VPS**

`lvcreate -s MyVolGroup/GenericRoot -n SomeClientsRoot`


## Auto Extending des Thin Pools

Configure Auto Extending of the Thin Pool (Configure Over-Provisioning protection)

[](https://github.com/kubernetes-projects/lvm-localpv/blob/develop/docs/thin_provision.md#configure-auto-extending-of-the-thin-pool-configure-over-provisioning-protection)

- Since we automatically create thin pool as part of first thin volume provisioning, we need to enable the monitoring using `lvchange` command on the all thin pools across the nodes to use the auto extend threshold feature.
    
    To Ensure monitoring of the logical volume is enabled.
    
    ```
    $ lvs -o+seg_monitor
     LV             VG    Attr       LSize Pool           Origin Data%  Meta%  Move Log Cpy%Sync Convert Monitor
     lvmvg_thinpool lvmvg twi-aotz-- 2.07g                       0.00   11.52                            not monitored
    ```
    

If the output in the Monitor column reports, as above, that the volume is not monitored, then monitoring needs to be explicitly enabled. Without this step, automatic extension of the logical thin pool will not occur, regardless of any settings in the `lvm.conf`.

```
$ lvchange --monitor y lvmvg/lvmvg_thinpool
```

Double-check that monitoring is now enabled by running the `lvs -o+seg_monitor` command a second time. The Monitor column should now report the logical volume is being monitored.

```
$ lvs -o+seg_monitor
LV             VG    Attr       LSize Pool           Origin Data%  Meta%  Move Log Cpy%Sync Convert Monitor
lvmvg_thinpool lvmvg twi-aotz-- 2.07g                       0.00   11.52                            monitored
```

Editing the settings in the `/etc/lvm/lvm.conf` can allow auto growth of the thin pool when required. By default, the threshold is 100% which means that the pool will not grow. If we set this to, 75%, the Thin Pool will autoextend when the pool is 75% full. It will increase by the default percentage of 20% if the value is not changed. We can see these settings using the command grep against the file.

```
$ grep -E ‘^\s*thin_pool_auto’ /etc/lvm/lvm.conf 
thin_pool_autoextend_threshold = 100
thin_pool_autoextend_percent = 20
```

### DRBD und LVM

Für den DRBD muss swingend der cache vom lvm deaktivierte werden da sonst zu inkonsistentes mit drbd kommt!
Zuerst muss die `lvm.conf` angepasst werden

```sh
...
devices {
...
 write_cache_state=0
...
}
...
```

Prüfen des Vorgangs ist mit folgender abfrage möglich :

```sh
sudo lvm dumpconfig | grep write_cache
write_cache_state=0
```


Zusammenfassung der Schritte

    Vergrößern des Festplattenspeichers (auf physikalischer oder VMware Ebene)
    Neustarten der Maschine, damit der zusätzliche Festplattenspeicher erkannt wird
    Erstellen einer weiteren Partition z. B. mittels cfdisk
    Partitionstabelle neu Einlesen, z. B. per Reboot oder per Kommando partprobe
    Initialisieren einer neuen PV mittels pvcreate
    Vergrößern der VG mittels vgextend
    Vergrößern des LV mittels lvextend
    Vergrößern des Dateisystems z. B. mittels resize2fs
