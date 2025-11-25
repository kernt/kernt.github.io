---
tags:
  - bash
  - konsole
  - system-administration
  - gnu-tools
  - systemctl
---
# Systemd

## systemctl

**Abhängigkeiten Anzeigen**

`systemctl list-dependencies $service`

**Units im Memory**

`systemctl list-units`

**Clean runtime**

`systemctl clean $UNIT`

**list jobs**

`systemctl list-jobs`

**list containers**

`systemctl list-machines`

**Create a new systemd unit name after-cbz_efs-mount as follows**

`sudo systemctl edit --force --full after-cbz_efs-mount`
## journalctl

**Boot vorgänge auflisten**

`journalctl --list-boots`

**filter nach datum**

`journalctl --since "2016-07-01" --until "2 minutes ago"`

**Rechte Prüfen**

`usermod -aG systemd-journal BENUTZERNAME`

**Filter nach Datum und Fehler Level**

`journalctl -p err -b --since "2019-01-11"`

**System starten ohne GUI**

`systemctl set-default -f multi-user.target`

**If you are unsure whether the service has the functionality to reload its configuration**

```sh
systemctl reload-or-restart application.service
```

```sh
systemctl list-dependencies sshd.service
```

https://unix.stackexchange.com/questions/348450/confused-by-execstartpre-entries-in-systemd-unit-file
## Path-Units

Mit Hilfe von systemd Path-Units können Dateien oder Verzeichnisse auf Änderungen hin überwacht werden. Tritt ein definiertes Ergebnis wie z.B. das Anlegen einer Datei ein, wird eine Service-Unit ausgeführt.

[systemd Timers](https://wiki.archlinux.org/title/systemd/Timers)

`systemctl list-unitfiles`
# [Systemd] Einem Service erlauben einen privilegierten Port zu nutzen

**Datum** 25.06.2018

Manchmal möchte man einem Prozess oder Tool, welches nicht als **root** Benutzer läuft, erlauben einen Port unterhalb von 1024 zu nutzen.  
Ein Beispiel dafür findet sich in dem [Techgoat-Artikel zu Caddy](https://techgoat.net/index.php?id=29), wo dem Webserver **Caddy** mithilfe des Befehls `sudo setcap cap_net_bind_service=+ep $(which caddy)` gestattet wurde den Port 80 bzw. 443 zu öffnen ohne als **root** Benutzer gestartet worden zu sein.

Dieser Schritt mit der Freigabe kann auch direkt in der entsprechenden Systemd `.service` Datei hinterlegt werden.

### Systemd-Service Datei bearbeiten

Grundsätzlich lässt sich ein servive-Script von Systemd mit `systemctl edit <servicename>.service` bearbeiten.  
Hierbei wird ein drop-in script erzeugt, welche unter `/etc/systemd/system/<servicename>.service.d/override.conf` zu finden ist und die entsprechende .service-Datei erweitert.  
Mit der Option `--full` kann man auch direkt die entsprechende .service-Datei bearbeiten.

Um nun einem unprivilegierten Prozess zu erlauben einen Port unter 1024 zu verwenden fügt man folgende Zeile(n) in das entsprechende .service-Script ein:

```
[Service]
AmbientCapabilities=CAP_NET_BIND_SERVICE
```

**Anmerkung:** Dies funktioniert allerdings erst ab systemd v229 oder später