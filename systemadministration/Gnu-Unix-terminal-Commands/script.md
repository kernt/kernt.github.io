### [ script ]

Das erste um seine Eingaben im Linux-Terminal zu speichern und anschließend wiederzugeben ist `script`, welches meist schon vorinstalliert ist.  
Falls nicht, findet man dieses Tool im Paket `util-linux`.  
`script` speichert alle Eingaben als Klartext, Sollte kein Dateiname als Ziel angegeben werden wird `typescript` als Standardname verwendet.

##### Aufzeichnung starten

Um mit der Aufzeichnung zu beginnen, verwendet man:

`script --timing=<dateiname> <dateiname>`

**Timing Datei**  
Der Parameter `--timing` gibt an unter welchem Dateinamen die Zeitangaben gespeichert werden, welche bei der Aufzeichnung entstehen.

Eine solche Datei hat dann in etwa den folgenden Inhalt:

```
0.061325 156
2.185293 2
0.000062 2
0.000138 17
0.000664 156
1.461482 1
```

Hierbei gibt die erste Spalte an wie viel Zeit seit der letzten Eingabe vergangen ist und die zweite Spalte gibt an wie viel Zeichen dann ausgegeben wurden.

Beispiel:  
`script --timing=time.txt testfile`

Hierbei werden die Zeitangaben in der `time.txt` und die eigentlichen Terminal Ein/Ausgaben in der `testfile` gespeichert.

Sobald der Befehl ausgeführt wurde sollte folgende Meldung erscheinen:

`Skript gestartet, die Datei ist testfile`

Nun kann man regulär weiterarbeiten und dabei werden nun alle Eingaben, welche auch im Terminal sichtbar sind, aufgezeichnet.  
Sollten Passwörter eingegeben werden, so werden diese nicht mit gespeichert, da diese ja auch nicht im Terminal sichtbar erscheinen.
##### Aufnahme stoppen

Um die Aufnahme dann zu stoppen, reicht es ein einfaches `exit` einzugeben, alternativ funktioniert auch **Strg+D**.
Anschließend sollte folgende Meldung erscheinen:

`Skript wurde beendet, die Datei ist testfile`
##### Aufnahme abspielen

um sich nun das gespeicherte anzuschauen, verwendet man:

`scriptreplay --timing=time.txt testfile`

Nun wird die Aufnahme mit den entsprechenden Ein/Ausgaben noch einmal im Terminal ausgeben.

### [ sudo-io ]

##### Ein Script als Login-Shell

Damit die eingegebenen Befehle mitgeloggt werden, muss der Benutzer in eine extra Shell geworfen werden.  
Dies geschieht, indem man als Shell für den Login eines Remote-Benutzers ein vorbereitetes Shell-Script angibt.

Bsp.: `testuser:x:1001:1001:,,,:/home/testuser:/usr/local/bin/logshell.sh`

hierzu wird bei der Einrichtung des Benutzers per `--shell` das entsprechende Script als Login-Shell festgelegt.

Das Shell-Script sollte folgenden Inhalt haben:

```sh
#!/bin/bash
/usr/bin/sudo -u `whoami` /bin/bash -c "/bin/bash"
unset SUDO_COMMAND
```

hierbei kann man natürlich noch andere Befehle aufrufen, z.B. Ausgeben von Hinweisen oder ähnliches…
Diese Script sollte nach der Erstellung auch entsprechende Ausführungsrechte bekommen ![](https://techgoat.net/images/32.png) z.B. per `chmod +x /usr/local/bin/logshell.sh`
##### Eintrag in die sudoers

Die sauberste Variante ist das anlegen einer entpsrechenden Datei unter `/etc/sudoers.d`, welche folgenden Inhalt haben sollte:

```sh
%testuser ALL=(%testuser)   LOG_INPUT: LOG_OUTPUT: ALL
Defaults:%testuser env_keep+="SSH_AUTH_SOCK"
```

[ Hierbei sollte **testuser** natürlich durch den entsprechenden Gruppennamen/Benutzernamen ersetzt werden ]
##### Log

Das Log wird beim einloggen des Benutzers angelegt und die eingegebenen Daten werden pro Session gespeichert.  
Der Speicherort ist hierbei `/var/log/sudo-io`

Der Aufbau des logs sieht für jede Session in etwa wie folgt aus:

```
├── 00
│   └── 00
│       └── 01
│           ├── log
│           ├── stderr
│           ├── stdin
│           ├── stdout
│           ├── timing
│           ├── ttyin
│           └── ttyout
└── seq
```

##### Abspielen des Logs

um sich eine Session anzuschauen verwendet man den Befehl `sudoreplay <Session-ID>`.  
Die Session-ID setzt sich aus dem Namen des Ordners der entsprechenden Session zusammen.  
Im oben gezeigten Beispiel lautet der Aufruf z.B.: `sudoreplay 000001` (hierzu muss man nicht extra in das Log-Verzeichnis wechseln)
### [ ttyrec ]

Ähnlich wie `script` speichert auch ttyrec seine Informationen mit entsprechenden Zeitstempel in eine Datei.  
`ttyrec` findet sich in den meisten Repos zum nachinstallieren.

##### Aufnahme starten

Um eine Aufnahme zu starten verwendet man einfach nur `ttyrec`. Will man das in einer bestimmten Datei speichern, so gibt man `-a` an.  
Bsp.: `ttyrec -a tesfile`  
Mittels `ttyrec -e <Programmname>` kann ein Programm ausgeführt und gleichzeitig die Aufzeichnung gestartet werden.

##### Aufnahme stoppen

Ähnlich wie bei `script` wird die Aufnahme mittels `exit` gestoppt, alternativ funktioniert auch **Strg+D**.
##### Aufnahme abspielen

Der Befehl zum abspielen lautet: `ttyplay <dateiname>`

Es gibt auch weitere Tools, welche solche eine ttyrec-Session in ein .gif oder ähnliches umwandeln können.
### [ playitagainsam ]

Inspiriert durch die beiden obigen Tools und dem Python-Tool `playerpiano` entstand das Tool `playitagainsam`.  
Hierbei kann der Output von mehreren Terminals aufgezeichnet und ausgegeben werde.  
`playitagainsam` findet sich z.B. bei Fedora in den Standard-Repos.
##### Aufnahme starten

Um eine Aufnahme zu beginnen verwendet man:  
`pias record <dateiname>`

Der aufgezeichnete Output wird hierbei in eine JSON-Datei gespeichert.

Mit dem Parameter `--join` ist es möglich mehrere Terminal-Sessions parallel aufzuzeichnen. Das eignet sich gut für z.B.: Aufzeichnung Server-Client Funktionsweisen.

Bsp: `pias --join record <dateiname>`
##### Aufnahme stoppen

Auch hier wird die Aufnahme mittels `exit` gestoppt.
##### Aufnahme abspielen

Um das gespeicherte Abzuspielen verwendet man:  
`pias play <dateiname>`