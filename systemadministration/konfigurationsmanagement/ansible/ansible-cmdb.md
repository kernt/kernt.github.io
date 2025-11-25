---
tags:
  - konfigurationsmanagement
  - ansible
  - ansible-builder
---

# ansible-cmd installation und Konfiguration

**Install ansible-cmdb**

`pipx ansible-cmdb`

### Verwenden von ansible-cmdb

##### Abrufen der Informationen von den verschiedenen Hosts

Die entsprechenden Informationen zu den einzelnen Hosts sammelt **Ansible** mithilfe des `setup` Modules und der Option `--tree`  
Auszug aus `ansible --help`:  
`-t TREE, --tree=TREE log output to this directory`

Der entsprechende Aufruf lautet:  
`ansible -m setup -i hosts --tree out/ all`

Hierbei ist zu beachten, dass nach der Option `--tree` die Angabe eines Verzeichnisses folgen muss. In diesem wird dann anschließend pro Host eine json-Datei erstellt und mit den entsprechenden facts befüllt.

```
$ ls out/
code01  gitlab2  monitoring  testserver01
```

##### Erstellen des CMDB-Dashboards

Nun kommt `ansible-cmdb` ins Spiel…

Die allgemeine Syntax lautet hierbei:  
`ansible-cmdb /Pfad/zu/den/json-Dateien > /Pfad/zur/Output.html`

Bsp.:  
`ansible-cmdb out/ > /var/www/cmdb.html`

Im Browser seiner Wahl wird beim aufrufen der erstellten **Output.html** das erstellte Dashboard angezeigt.

Wenn man die Prozedur des Abrufens der Daten, z.B. per Cron, Systemd-timers, etc. entsprechend automatisiert, erhält eine aktuelle Systemübersicht der entsprechenden Infrastruktur, welche sich mit jedem Webserver ausgeben lässt.