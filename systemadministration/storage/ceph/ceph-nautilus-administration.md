---
tags:
  - storage
  - ceph
---
# Ceph Administration

## Ceph Storage Administration

`mount.ceph cephip:6789:/ /cephfs -o name=admin,secretfile=/etc/ceph/admin.secret`

> Für Ceph älter als nautilus

LVM'S  auflisten

`ceph-volume lvm list`

Cluster zustand

`ceph health detail`

Log details ausgeben

`ceph crash info 2021-07-25_07:28:20.848217Z_6f504561-ff9e-4ed9-a784-14332acf3428`

PG Repairen

`ceph pg repair 5.8`

Logs Archiviren

`ceph crash archive-all`

OSD Suchen

`ceph osd find 71` 

Bluestore Reparatur

`ceph-bluestore-tool repair --path /var/lib/ceph/osd/ceph-71`