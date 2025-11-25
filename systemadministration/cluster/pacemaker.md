---
tags:
  - pacemaker
  - cluster
  - ha
---
# Pacemaker Installation und Konfiguration

* [Drbd](../drbd)

**Pacemaker wird mit und seinen Abhängigkeiten installiert**

```sh
yum install -y corosync pacemaker pcs resource-agents pacemaker cman pcs
```
**Alle Services Aktivieren**

```sh
systemctl enable corosync.service
systemctl enable pacemaker.service
systemctl enable pcsd.service
```
## Cluster Ressourcen anlegen

### Cluster drbd Ressourcen

Wichtig ist zu prüfen , dass der drbd Service nicht Läuft

`systemctl disable drbd`

und das , dass drbd Drive nicht gemountet ist.

### Virtuelle IP einrichten

`pcs resource create VirtualIP ocf:heartbeat:IPaddr2 ip=192.168.4.140 cidr_netmask=24 nic=eth0 op monitor interval=30s`
### drbd Ressource bekannt machen und mit pcs/cib erstellen

`pcs resource create p_drbd0 ocf:linbit:drbd params drbd_resource="drbd0"`
`pcs resource master ms_drbd0 p_drbd0 master-max=1 master-node-max=1 clone-max=2 clone-node-max=1 notify=true`
`pcs resource create fs_drbd0 Filesystem device=/dev/drbd0 directory=/var/drbd fstype=ext4`
### Stopping Cluster Services

`pcs cluster stop [--all] [node] [...]`

Mit `--all` wird auf allen nodes
### Zugangsdaten auf die Nodes einrichten

Username : hacluster

hacluster ist ein normaler localer user password kann also mit

`passwd hacluster`

geändert werden

Zugang  für einen neuen node anlegen :

`pcs cluster auth testnode1`

User ist dann der hacluster

Version mit der getestet wurde .

* 1.1.15-11.el7_3.2 ``` $()```

## Allemeine Informationen zum Management

Unter `https://$NODE.example.com:2224`  kann die man die  jeweilige node via web Interface erreichen. 

## Firewall Einrichtung

Die Ports müssen auf allen nodes freigegeben sein :

```sh
firewall-cmd --zone=trusted --add-source=10.0.0.0/24 --permanent
firewall-cmd --reload
```

Hier muss das Netzwerk `10.0.0.0/24`  entsprechend angepasst werden
## Szenarios

### Einen Neuen Node Hinzufügen

1. Installation und Konfiguration
2. hacluster Password einrichten
3. Firewall Einrichtung
4. Alle Services Aktivieren
5. `pcs cluster setup --start --name testcluster $NEWNODE --transport udpu` auf dem Aktiven Node durchführen
### Neuen Cluster einrichten

1.Installation und Konfiguration
2.hacluster Password einrichten
3.Firewall Einrichtung
4.Alle Services Aktivieren
###### Quellen

* [Clusters_from_Scratch](http://clusterlabs.org/doc/en-US/Pacemaker/1.1/html/Clusters_from_Scratch/ch07.html)
* [Manpage crmsh](http://crmsh.github.io/man-3/)
* [configuring-nfs-ha-using-redhat-cluster-pacemaker-rhel-7](http://www.unixarena.com/2016/08/configuring-nfs-ha-using-redhat-cluster-pacemaker-rhel-7.html)

