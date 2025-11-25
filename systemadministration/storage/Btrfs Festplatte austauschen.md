Es können funktionierende oder auch schon komplett fehlende Festplatten ausgetauscht werden.

Ein Pool welcher degraded ist, lässt sich nicht einfach so mounten. Hierbei muss zusätzlich eine Option mitgegeben werden.

`mount -o degraded <Poolpfad> <Mountziel>`

Zum austauschen benötigt man die DeviceID, der zu tauschenden Platte.  
Sollte diese fehlen, muss man schauen welche ID fehlt.

```
root@testsystem:/mnt# btrfs fi show
Label: 'storage'  uuid: 2501b7ad-0bdb-427a-a969-8d07fbe67070
        Total devices 3 FS bytes used 7.53GiB
        devid    1 size 32.00GiB used 6.00GiB path /dev/vdb
        devid    3 size 32.00GiB used 6.01GiB path /dev/vdd
        *** Some devices missing
```

`btrfs replace start <devid, in diesem Fall die 2> <Pfad zur Platte> <Mountpoint des Pools>`

Anschließend kann man den Status des Rebuild anzeigen.

```
root@testsystem:/mnt# btrfs replace status /mnt/
6.2% done, 0 write errs, 0 uncorr. read errs
```

Hier werden auch weitere Informationen, nach dem Abschluss angezeigt.

```
root@testsystem:/mnt# btrfs replace status /mnt/
Started on 11.Jun 01:48:17, finished on 11.Jun 01:50:23, 0 write errs, 0 uncorr. read errs
```

Anschließend sieht die Übersicht wieder unauffällig aus.

```
root@testsystem:~# btrfs fi show
Label: 'storage'  uuid: 2501b7ad-0bdb-427a-a969-8d07fbe67070
        Total devices 3 FS bytes used 8.04GiB
        devid    1 size 32.00GiB used 7.03GiB path /dev/vdb
        devid    2 size 32.00GiB used 6.00GiB path /dev/vdc
        devid    3 size 32.00GiB used 7.04GiB path /dev/vdd
```