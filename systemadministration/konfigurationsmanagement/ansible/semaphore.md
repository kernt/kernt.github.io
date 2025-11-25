---
tags:
  - konfigurationsmanagement
  - ansible
  - semaphore
---
# Ansible semaphore

**semaphore** bezeichnet sich selbst einfach nur als „open source Ansible UI“ und erfüllt als diese auch ihren Zweck.

### Installation

Voraussetzung für die Installation von **semaphore** ist eine MySQL-Instanz.  
Ein **MySQL** oder **Mariadb** ab der Version 5.5 reicht da schon aus.

Ich verwende bei mir folgendes:

```
Name       : mariadb-server
Architektur : x86_64
Version    : 5.5.60
OS: CentOS Linux release 7.6.1810 
Ansible Version: ansible-playbook 2.7.8
```

Github-Seite des Projekts: [https://github.com/ansible-semaphore](https://github.com/ansible-semaphore/semaphore)

Ich installiere das auf der Github-Projektseite bereitgestlellte Release per `rpm`:

```
$ wget https://github.com/ansible-semaphore/semaphore/releases/download/v2.5.1/semaphore_2.5.1_linux_amd64.rpm
$ rpm -i semaphore_2.5.1_linux_amd64.rpm
```

### Einrichten von semaphore

Das Setup beginnt, wenn `semaphore` mit dem Parameter `-setup` gestartet wird.

Hier gitl es nun noch ein paar Standardwerte zu setzen:

```
$ semaphore -setup                                              
 Hello! You will now be guided through a setup to:

 1. Set up configuration for a MySQL/MariaDB database
 2. Set up a path for your playbooks (auto-created)
 3. Run database Migrations
 4. Set up initial semaphore user & password

 > DB Hostname (default 127.0.0.1:3306): 127.0.0.1:3306
 > DB User (default root): root
 > DB Password: toor
 > DB Name (default semaphore): semaphore
 > Playbook path (default /tmp/semaphore): /home/rasputin/playbooks
 > Web root URL (optional, example http://localhost:8010/): http://192.168.2.100:8010/
 > Enable email alerts (y/n, default n): n
 > Enable telegram alerts (y/n, default n): n
 > Enable LDAP authentication (y/n, default n): n

 Generated configuration:
 {
        "mysql": {
                "host": "127.0.0.1:3306",
                "user": "root",
                "pass": "toor",
                "name": "semaphore"
        },
        "port": "",
        "tmp_path": "/home/rasputin/playbooks",
        "cookie_hash": "PL1WhZJJZ7KKRcLHAd8cebx82+LPNAsTPE79tNGQ32o=",
        "cookie_encryption": "tT2hodfYqckUNsC9IeF/M0E/W6NzyR/ty7+gBA37K8o=",
        "email_sender": "",
        "email_host": "",
        "email_port": "",
        "web_host": "http://192.168.2.100:8010/",
        "ldap_binddn": "",
        "ldap_bindpassword": "",
        "ldap_server": "",
        "ldap_searchdn": "",
        "ldap_searchfilter": "",
        "ldap_mappings": {
                "dn": "",
                "mail": "",
                "uid": "",
                "cn": ""
        },
        "telegram_chat": "",
        "telegram_token": "",
        "concurrency_mode": "",
        "max_parallel_tasks": 0,
        "email_alert": false,
        "telegram_alert": false,
        "ldap_enable": false,
        "ldap_enable": false,
        "ldap_needtls": false
 }

 > Is this correct? (yes/no): yes
 > Config output directory (default /root): /root
 Running: mkdir -p /root..
 Configuration written to /root/config.json..
 Pinging db..

 Running DB Migrations..
Checking DB migrations
Creating migrations table
Executing migration v0.0.0 (at 2019-03-13 13:56:17.135736098 +0100 CET m=+214.712393508)...
 [11/11]
Executing migration v1.0.0 (at 2019-03-13 13:56:17.200166993 +0100 CET m=+214.776824388)...
 [7/7]
Executing migration v1.1.0 (at 2019-03-13 13:56:17.355522169 +0100 CET m=+214.932179538)...
 [1/1]
Executing migration v1.2.0 (at 2019-03-13 13:56:17.367823076 +0100 CET m=+214.944480449)...
 [1/1]
Executing migration v1.3.0 (at 2019-03-13 13:56:17.374806985 +0100 CET m=+214.951464365)...
 [3/3]
Executing migration v1.4.0 (at 2019-03-13 13:56:17.42232979 +0100 CET m=+214.998987190)...
 [2/2]
Executing migration v1.5.0 (at 2019-03-13 13:56:17.461199709 +0100 CET m=+215.037857148)...
 [1/1]
Executing migration v0.1.0 (at 2019-03-13 13:56:17.48707641 +0100 CET m=+215.063733781)...
 [6/6]
Executing migration v1.6.0 (at 2019-03-13 13:56:17.530711262 +0100 CET m=+215.107368677)...
 [4/4]
Executing migration v1.7.0 (at 2019-03-13 13:56:17.587992416 +0100 CET m=+215.164649733)...
 [1/1]
Executing migration v1.8.0 (at 2019-03-13 13:56:17.598386373 +0100 CET m=+215.175043678)...
 [2/2]
Executing migration v1.9.0 (at 2019-03-13 13:56:17.613534144 +0100 CET m=+215.190191487)...
 [2/2]
Executing migration v2.2.1 (at 2019-03-13 13:56:17.625215558 +0100 CET m=+215.201872883)...
 [2/2]
Executing migration v2.3.0 (at 2019-03-13 13:56:17.652725557 +0100 CET m=+215.229382919)...
 [3/3]
Executing migration v2.3.1 (at 2019-03-13 13:56:17.687516679 +0100 CET m=+215.264174043)...
 [1/1]
Executing migration v2.3.2 (at 2019-03-13 13:56:17.704184137 +0100 CET m=+215.280841529)...
 [1/1]
Executing migration v2.4.0 (at 2019-03-13 13:56:17.755708707 +0100 CET m=+215.332366024)...
 [1/1]
Executing migration v2.5.0 (at 2019-03-13 13:56:17.765108948 +0100 CET m=+215.341766252)...
 [1/1]
Migrations Finished
```

Ganz am Ende des Einrichtens von **semaphore** bekommt man noch einmal eine Zusammenfassung angezeigt und auch wie **semaphore** zu starten ist.

```
> Username: rasputin
 > Email: spam@techgoat.net
WARN[0226] sql: no rows in result set                    level=Warn
 > Your name: rasputin
 > Password: toor

 You are all setup rasputin!
 Re-launch this program pointing to the configuration file

./semaphore -config /root/config.json

 To run as daemon:

nohup ./semaphore -config /root/config.json &

 You can login with 1337@techgoat.net or rasputin.

Damit ist *semaphore* nun erfolgreich eingerichtet und kann gestartet werden.
```

### semaphore verwenden

Ich starte **semaphore** nicht im Hintergrund um zu schauen was genau passiert.

```
$ semaphore -config /root/config.json
Using config file: /root/config.json
Semaphore v2.5.1
Port :3000
MySQL root@127.0.0.1:3306 semaphore
Tmp Path (projects home) /home/rasputin/playbooks
Checking DB migrations
 403  14:00:22      22.378µs |   GET     /api/user
 204  14:00:28  457.934978ms |   POST    /api/auth/login
 200  14:00:28   92.243091ms |   GET     /api/user
 200  14:00:28    7.773465ms |   GET     /api/projects
 200  14:00:28    8.431954ms |   GET     /api/events/last
...
```

Nun lauscht **semaphore** standardmäßig auf Port 3000.

`tcp LISTEN 0 128 :::3000 :::* users:(("semaphore",pid=18334,fd=3))`

Im Browser sollte man nun beim aufrufen der der entsprechenden IP mit Portangabe (Bsp.: `http://192.168.2.100:3000`) eine Login-Maske erscheinen.

Hier kann man sich nun mit den eben gesetzten Zugangsdaten anmelden und kommt so auf das Dashboard, wo einem die Übersicht über die letzten Events und eine Liste der eingerichteten Projekte erwartet.

##### Ein Projekt einrichten

Wir starten mit einem Klick auf das `+` bei der Projektliste.  
Hier vergibt man einen entsprechenden Namen für das Projekt und klickt auf **create**.

Das neue Projekt erscheint nun in der Liste und durch Klick auf den Namen gelangt man zu den Einzelheiten.  
Als erstes kann sollte man die benötigten Keys einrichten. Seien es SSH-Keys oder Keys für AWS, Google und Co.  
Ich habe hier einen SSH_Key eingetragen, welcher auch auf github entsprechend hinterlegt ist.  
Natürlich sollte man mit dem hinterlegten Keys auch auf die entsprechenden Systeme, welche etwas später im `inventory` festgelegt sind.  
Hierzu klickt man auf `Key Store` —> `create key` und trägt die entsprechenden Infos ein.

Anschließend richtet man sich ein `Playbook Repository` ein, indem man auf `Playbook Repositories` —> `create repository` klickt und neben einen eindeutigen Namen trägt man auch die URL zu dem Repo mit den Playbooks ein.  
Hier gibt man auch den eben eingetragenen SSH-Key an und klickt abschließend auf `create`.

Nun legt man unter `Inventory` noch ein entsprechendes `inventory`-file an, es kann auch ein vorhandenes verwendet werden.  
Dabei handelt es sich quasi um die Zielsysteme, welche in frei wählbare Gruppen aufgeteilt werden können.  
Dies geschieht mit Klicks auf `Inventory` —> `create Inventory`.  
Type **Static** lässt einem eine feste Liste anlegen, während bei Type **File** ein vorhandenes `inventory` verwendet werden kann.  
Abschließend klickt man auf `create` um den Schritt abzuschließen.

Nun geht es an das einrichten des eigentlichen Tasks…

Hierfür klicket man auf `Task Templates` —> `new template` und trägt hier einen Alias ein, gibt den Namen des Playbooks aus dem Repo an welches ausgeführt werden soll und wählt noch die vorher eingerichteten `SSH-Keys`, `Playbook Repoisitory` und `Inventory` aus.  
Zusätzlich lassen sich im unteren Textfeld `Extra CLI Arguments` weitere Parameter und Argumente eintragen.

Um nun das im `Task Template` hinterlegte Ansible-Playbook aus dem entsprechenden github-repo zu starten, klickt man im `Task Template` bei entsprechenden Task auf `run`.

Hier hat man nun noch einmal die Möglichkeit ein anderes Playbook oder auch noch weitere Enviroment Variablen anzugeben und entwender erst einen `dry run` (trestlauf, ohne wirkliche ausführung der Aktionen@ durchführen oder gleich richtig, per klick auf `run!` zu starten.  
Zusätzlich kann man den erweiterten Debug-Modus (`-vvvv`) aktivieren um eine möglichst ausführliche Ausgabe zu erzeugen.

Im nachfolgenden Fenster erscheint nun die Live-Ausgabe des Logs mit Zeitstempel:

