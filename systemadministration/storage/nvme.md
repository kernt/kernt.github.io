---
tags:
  - nvme
  - sfdisk
  - mdadm
---

**NVMEs finden**
```
cat /proc/mdstat
```

**Raid Controller config anzeigen**

`ssacli ctrl all show config`
# wenn direkt angebunden hilft nur df -h
 
```sh
fdisk -l
dmidecode  | grep -i nvm
 
cat /sys/devices/pci0000:36/0000:36:01.0/0000:38:00.0/nvme/nvme3/{serial,model}
 
for i in `ls -d /sys/devices/*/*/*/nvme/nvme?`
     do 
        echo "$i"
        cat $i/{serial,model} ; echo ""
      done
```
# Softraid einrichten

Ermitteln der NVMe´s

`sfdisk -s 2> /dev/null | egrep -i "nvme" | awk -F ":" '{ print $1 }'`

Softraid erzeugen:

> !!! Achtung ! Sinnvoll wäre bei 2 Platten eher ein RAID 1 (HIER RAID 10 als Beispiel), aber wir hatten unter RHEL 6.x Probleme damit.
   Das einzige Raid, das funktionierte und danach noch schnell genug war, war ein RAID 10 !!!

`mdadm --create /dev/md0 --run --auto=yes --level=10 --spare-devices=0 --raid-devices=2 --chunk=256 /dev/nvme0n1 /dev/nvme1n1`

mdadm.conf schreiben

```sh
cat <<- 'EOF' | tee /etc/mdadm.conf
DEVICE partitions
MAILADDR root
EOF
```

`mdadm --detail --brief /dev/md0 >> /etc/mdadm.conf`

**Zustand Prüfen**

`cat /proc/mdstat`

**Filesystem XFS erzeugen:**

```
mkfs.xfs /dev/md0
```

**Einmalig (!!!) einen Eintrag in /etc/fstab erstellen**

`echo "/dev/md0 /mysql xfs defaults 1 2" >> /etc/fstab`

Auf Softraid Initialisierung warten (am Besten in ein kleines Script pasten und ausführen...)

**Waiting for softraid completion**

```sh
mdadm --grow --bitmap=/tmp/bitmap-md0.bin /dev/md0 # Alternativ: mdadm --grow --bitmap=internal /dev/md0 --> slows down raid-build
  while [ "$(cat /proc/mdstat | grep active)" ] && [ "$(mdadm -D /dev/md0 | grep "State :" | grep -i "degraded")" ]
   do
     clear; echo "WAITING FOR SOFTRAID INIT TO BE FINISHED:"
     echo "========================================="
     cat /proc/mdstat
     sleep 60
   done
    mdadm --grow --bitmap=none /dev/md0
fi
```

Hiermit kann die Initialisierung des Softraid u.U. beschleunigt werden:

`sysctl -w vm.dirty_background_ratio=5`
`sysctl -w vm.dirty_ratio=10`
`sysctl -w dev.raid.speed_limit_min=50000`
`sysctl -w dev.raid.speed_limit_max=2000000 # (4-5 drives. For bigger arrays use 5000000)`
`blockdev --setra 65536 /dev/md0`

```
ACTIVEDISKS=("$(cat /proc/mdstat | grep md0 | sed 's/.*raid5 \(.*\)/\1/g' | sed 's/\[[0-9]\]//g')")
for activedisk in ${ACTIVEDISKS[@]}; do blockdev --setra 65536 /dev/${activedisk}; done
```

`echo 32768 > /sys/block/md0/md/stripe_cache_size`

```
mdadm --grow --bitmap=internal /dev/md0
```

Nach erfolgter Initialisierung das hier ausführen:

`mdadm --grow --bitmap=none /dev/md0'`
# Softraid löschen (im Fallback Fall)

Ermitteln der NVMe´s

`sfdisk -s 2> /dev/null | egrep -i "nvme" | awk -F ":" '{ print $1 }'`

Aktive soft-raided NVME´s listen:

`awk '{ print $4 }' /dev/md/md-device-map | tr -d " "`

Für jede aktive NVMe:

`mdadm --grow --bitmap=none ${activemd}`

Alle Partitionen auf allen NVMe´s löschen:

```
for part in {4..1}; do mdadm --stop ${activemd}p${part}; parted -s ${activemd} rm $part; done
```

Superblock löschen:

`mdadm --zero-superblock ${activemd}`

Softraid Umounten

`umount -fl ${activemd}`

Softraid stoppen

`mdadm --stop ${activemd}`

Softraid mit Nullen löschen

`dd if=/dev/zero of=/dev/md0 bs=1M count=32`

Softraid löschen

`mdadm --remove ${activemd}`

Superblock auf jeder NVMe löschen

`mdadm --zero-superblock ${nvmedrive}`

Softraid Config löschen

`rm -f /etc/mdadm.conf`