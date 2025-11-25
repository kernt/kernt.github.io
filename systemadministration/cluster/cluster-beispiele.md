---
tags:
  - cluster
  - ha
  - crm
---
# Cluster mit crm

**Ressourcen schwenken und Ursprungszustand wiederherstellen**

**Ressourcenverteilung ansehen, -r zeigt dabei auch gestoppte Ressourcen an**

`crm_mon -1r`

**zeigt die Clusterkonfiguration**

`crm configure show`

**Ressource migrieren**

`crm resource migrate <resource> <Zielsystem>`

 **Änderungen zusehen**
 
`crm_mon -r` 

**erst ausführen, wenn alle Ressourcen sauber migriert sind**

`crm resource unmigrate <resource>`

**host auf standby setzen und so sichergehen, dass dort keine Ressourcen mehr sind, oder hin verteilt werden, alle Ressourcen, die auf diesem Host verblieben sind, werden auf den anderen Clusterpartner migriert**

```sh
crm node standby <hostname>
systemctl stop pacemaker
```

**Updates aufspielen**

`systemctl start pacemaker`

**host Resource bereinigen, die gefailed ist**

`crm node online <hostname>`

**Ressourcenverteilung ansehen, -r zeigt dabei auch gestoppte Ressourcen an**

`crm_mon -1r`

**Konfiguration anzeigen und prüfen, wo die Ressourcen laufen sollten**

`crm configure show`

**gefailte Ressource nachstarten**

`crm resource cleanup <resource>`

**zusehen wie die Ressource neu gestartet wird**

`crm_mon -r`
