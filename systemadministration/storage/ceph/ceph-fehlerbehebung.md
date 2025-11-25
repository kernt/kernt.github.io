---
tags:
  - storage
  - ceph
---
# Ceph Fehler Behebung

## Kommandos zur fehlerbehebung

**Cluster Übersicht**

`ceph -s`

**Logs ausgeben**

`ceph crash ls`

**Filesystem status**

`ceph fs Status`

**Inkonsistente pg ausgeben**

`ceph pg ls inconsistent`

**Details zu PG Fehlern**

`ceph health detail`

**Inkonsistente pg ausgeben in json und verarbeiten**

`ceph pg ls inconsistent -f json | jq -r .pg_stats[].pgid`

>  jq muss installiert sein

**PG id Reparatur**

`ceph pg repair`

**Fehler ins Archiv verschieben**

`ceph crash archive-all`

## Allgemeines zur Fehlerbehebung

**Fehler Analyse**

`ceph versions`

`ceph features`

`ceph fs status`

**Fehler**: [WRN]overall HEALTH_WARN 1 clients failing to respond to cache pressure

**Quellen**: http://lists.ceph.com/pipermail/ceph-users-ceph.com/2018-August/028884.html

**Behebung**: 
 ceph daemon mds.<MDS> config show | grep mds_cache_memory_limit 
überprüfen normal kann  man Ram hinzufügen, sonst warten bis Clients 
wider inodes freigeben

**Ursache**: Clients haben die inodes im Zugriff, insgesamt soviel das der Ram nicht ausreicht

**Manuelle Reparatur**

`ceph-bluestore-tool repair --path` `/var/lib/ceph/osd/ceph-74`

ceph pg repair ID 

Nach Prüfung der Logs kann man dann die Fehler bestätigen mit:  

ceph crash archive-all  

so bleiben die auch bestehen. 

--flush-all geht auch dann sind die aber weg!
