---
tags:
  - storage
  - vdo
---
# vdo

VDO arbeitet wie folgt:

1. Leere Datenblöcke (Zero-Blocks) filtern.
2. Duplikate eliminieren.
3. Datenblöcke per LZO komprimieren.

Installation:

`dnf -y install vdo`

Konfiguration:

```sh
vdo create --name vdo0 --device /dev/sdb1 --vdoLogicalSize 10T
udevadm settle
vdo status --name vdo0
vdostats --human-readable
```

Dateisystem erstellen, in LVM einbinden etc.

Mount, falls Dateisystem erstellt wurde:

```sh
blkid /dev/mapper/vdo0
mkdir -p /mnt/vdo0
```

/etc/fstab[Link zu diesem Quellcode](https://docs.linuxfabrik.ch/base/filesystem/vdo.html#id1 "Link zu diesem Quellcode")

```sh
# delay mounting the file system until systemd starts the vdo.service
# see `man systemd.mount`
UUID="..." /mnt/vdo0 xfs defaults,x-systemd.requires=vdo.service 0 0
```

[vdo](https://docs.linuxfabrik.ch/base/filesystem/vdo.html)