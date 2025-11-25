---
tags:
  - storage
  - ceph
---
# Ceph Uebersicht

* [ceph und kubernetes](../ceph-kubernetes)
* [ceph fehlerbehebung](../ceph-fehlerbehebung)
## Ceph Administration

Ceph Storage Administration

`mount.ceph cephip:6789:/ /cephfs -o name=admin,secretfile=/etc/ceph/admin.secret`

> Für Ceph älter als nautilus 