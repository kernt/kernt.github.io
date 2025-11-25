---
tags:
  - jenkins
  - jenkins-slave
---
# User für Slave zugang vorbereiten

User jenkins suchen

`cat /etc/passwd | grep`

wenn vorhanden änderung von

`*:/bin/false`

zu

`*:/bin/bash`

durchfüren

dem _jenkins_ Benutzer ein password wie folgt vergeben:

`passwd jenkins`

zum Benutzer _jenkins_ werden

`su jenkins`

ausführenden Benutzer überprüfen

`whoami`

Verzeichnis wechsel durchführen.

`cd ~`

und Prüfen mit

`pwd`

ssh schlüßel erstellen

`ssh-keygen`

ssh Verbindung mit dem Zielsystem also dem slave durchführen

`ssh user@slavehost`

auf dem slave system den Benutzer _jenkins_ anlegen

`useradd jenkins`

Identischens Password für _jenkins_ Benutzer wie auf dem MAster system vergeben.

Sudo file bearbeiten

`visudo`

dort folgenden Eintrag hinzufügen

`jenkins ALL=(ALL) NOPASSWD: ALL`

Vom Master zum Slave den Pub Key austauschen

`ssh-copy-id jenkins@$IP`

In Jenkins unter _Manage-Jenkins-> Nodes_ den Node mit Agent erstellen in der Konfiguration muss unter _Launch method_  _Lauch agent via ssh_ gewählt werden und unter dem reiter der nach der Auswahl auf geht muss ein benutzer mit _add_ angelegt werden Unter _Domain_ die Global wählen, unter _Kind_ dann _SSH Username with private key_ dann noch _Enter directly_ und dort den Priv Key vom benutzer jenkins vom  "master" angeben. die ID in Beschreibung kann nach belieben eingetragen werden, sollte aber wieder zu zu orden sein.
