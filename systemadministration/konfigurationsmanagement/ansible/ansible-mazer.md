---
tags:
  - konfigurationsmanagement
  - ansible
---

# Ansible Mazer Installation

Die Installation wird mit [Python pip](../python-pip) durchgeführt.

`pip install mazer`

## Konfiguration

Zur Konfiguration wird die Datei `~/.ansible/mazer.yml` benutzt die sollte im Quellverzeichnis stehen. 

Hier eine Beispielkonfiguration für den Anfang

```sh
version: 1
server:
  ignore_certs: false
  url: https://galaxy-qa.ansible.com
content_path: ~/.ansible/content
options:
  local_tmp: ~/.ansible/tmp
  role_skeleton_ignore:
     - ^.git$
     - ^.*/.git_keep$
  role_skeleton_path: null
  verbosity: 0
```

Die Optionen bedeuten folgendes:

### Version

Die Konfiguration des benutzen Formats. Default ist 1

### Server

Gibt Galaxy Server verbindungs Informationen an hier ignoriere Zertifikate und eine url
Der wert für den  Galaxy Server wird hier gesetzt,und der wert von **ignore_certs** wird auf **true** der **false** gesetzt.

### content_path

Geben Sie einen Pfad zu einem Verzeichnis im lokalen Dateisystem an, in dem Ansible-Inhalte installiert werden. Der Standardwert ist `~/ .ansible/content`

### options

Verschiedene Konfigurationsoptionen werden hier festgelegt, einschließlich: local_tmp, role_skeleton, role_skeleton_path, verbosity.

### local_tmp

Pfad, den Mazer für temporären Arbeitsbereich verwenden kann, um beispielsweise Archivdateien zu erweitern.

### role_skeleton_path

Pfad zu einer Rollenstruktur, die mit dem Befehl init verwendet werden soll. Überschreibt die Standardrollenstruktur.

### role_skeleton_ignore

Liste der Dateinamensmuster, die beim Kopieren des Inhalts des Rollen-Skelettpfads ignoriert werden sollen.

### verbosity

Steuert den Standardwert der von Mazer zurückgegebenen Ausgabe.
