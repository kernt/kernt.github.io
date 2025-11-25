---
tags:
  - backup
  - borg
---
# Borg Backup Inststallation

```sh
apt install borgbackup
```
# Borg Backup Administration

**Initialisieren eines neuen Backup repo**

`borg init -e repokey /opt/backup/borg/repo`

**Erstellen von einem  Backup Archive**

`borg create /opt/backup/borg/repo::Saturday1 ~/Documents`

**Now doing another backup, just to show off the great deduplication**

`borg create -v --stats /opt/backup/borg/repo::Saturday2 ~/Documents`

**Borg Backup mounten**

Beispiel annahmen

Disk

```bsh
sdb                   8:16   0   4,5T  0 disk
├─sdb1                8:17   0   2,5T  0 part /mnt/sdb1
├─sdb2                8:18   0     1T  0 part /mnt/sdb2
└─sdb3                8:19   0     1T  0 part /mnt/sdb3

```
Pfade:
- /mnt/sdb1/arbeitsrepo
- /mnt/sdb1/repo

**Fstab Eintrag**

 /mnt/sdb1/arbeitsrepo /mnt/backup/arbeitsrepo fuse.borgfs defaults,noauto,user 0 0`
 /mnt/sdb1/repo /mnt/backup/repo fuse.borgfs defaults,noauto,user 0 0

**Einträge extraieren**

`borg extract /media/peter/HD_Backup/borgbackups::MeineMedien home/peter/Videos`

**Repos Mounten**

`borg mount /mnt/sdb1/repo /mnt/backup/repo/`

[Borg Stable](https://borgbackup.readthedocs.io/en/1.2-maint/)
