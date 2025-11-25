Softraid startet nach einem Recovery nicht mehr:

```sh
cat /proc/mdstat
Personalities : [raid10]
md0 : inactive nvme0n1p1[5]
      781280240 blocks super 1.2
 
unused devices: <none>
mdadm: superblock on /dev/nvme0n1 doesn't match others - assembly aborted
```

`mdadm --manage --set-faulty /dev/md0 /dev/nvme0n1`
`(Sollte einen ReSync anstossen)`

Wenn das nichts bringt:

`mdadm --stop /dev/md0`

Nun das Raid OHNE die fehlerhafte Platte neu starten

```
mdadm --assemble --verbose /dev/md0 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1 /dev/nvme4n1 /dev/nvme5n1
```


Die defekte Platte aus dem RAID Verbund löschen und wieder hinzufügen

```sh
mdadm --manage /dev/md0 -r /dev/nvme0n1
mdadm --manage /dev/md0 -a /dev/nvme0n1
```

Status prüfen

```sh
cat /proc/mdstat
 
Personalities : [raid10]
 
md0 : active raid10 nvme0n1[6](S) nvme1n1[0] nvme5n1[4] nvme4n1[3] nvme3n1[2] nvme2n1[1]
 
2343843072 blocks super 1.2 256K chunks 2 near-copies [6/5] [UUUUU_]
[>....................]  resync =  0.5% (13474624/2343843072) finish=320.1min speed=121325K/sec 
bitmap: 18/18 pages [72KB], 65536KB chunk
 
unused devices: <none>
# export NMON=d; nmon
```

Wenn dieses Kommando eine hohe Auslastung der Platten anzeigt, dann ist das Raid synced