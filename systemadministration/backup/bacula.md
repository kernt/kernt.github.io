---
tags:
  - backup
  - bacula
---
# Bacula Konfiguration

```sh
Pool {
  Name = Full
  Maximum Volumes = 4
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Volume Retention = 3 Months
  Catalog Files = yes
  UseVolumeOnce = no
}
Pool {
  Name = Inc
  Maximum Volumes = 2
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Volume Retention = 12 hours
  Catalog Files = yes
  UseVolumeOnce = no
}

JobDefs {
  Name = "Backup"
  Type = Backup
  Level = Incremental
  FileSet = "Full Set"
  Schedule = "Daily"
  Storage = Tape
  Messages = Standard
  Priority = 50
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
  Pool = Full
}

Job {
  Name = "Backup Offsite"
  JobDefs = "Backup"
  Client = bacula-fd
  Incremental Backup Pool = Inc
  Full Backup Pool = Full
  Priority = 15
}
```
## Ansible Bacula

https://gitlab.bacula.org/bacula-community-edition/ansible-collection-bacula-community/-/tree/main?ref_type=heads
https://github.com/bacula-web/bacula-web/tree/v9.5.1
https://www.bacula.lat/community/bacula-community-9-x-official-packages-installation-script/?lang=en
https://www.bacula.org/bacula-binary-package-download/?
https://www.bacula.org/documentation/documentation/

Binär Datein

https://www.bacula.org/packages/66a1f7db50a00/
https://www.bacula.org/packages/66a1f7db50a00/debs/15.0.2/dists/bookworm/main/binary-amd64/INSTALL

## Quellen

* [bacula](https://www.21x9.org/tag/bacula/)
