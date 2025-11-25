---
tags:
  - sicherheit
  - aide
---

Einleitung
Ein Intrusion Detection System (englisch intrusion „Eindringen“, IDS) bzw. Angriffserkennungssystem ist ein System zur Erkennung von Angriffen, die gegen ein Computersystem oder Rechnernetz gerichtet sind. Das IDS kann eine Firewall ergänzen oder auch direkt auf dem zu überwachenden Computersystem laufen und so die Sicherheit von Netzwerken und Computersystemen erhöhen. Erkannte Angriffe werden meistens in Log-Dateien gesammelt und Benutzern oder Administratoren mitgeteilt; hier grenzt sich der Begriff von Intrusion Prevention System (englisch prevention „Verhindern“, IPS) ab, welches ein System beschreibt, das Angriffe automatisiert und aktiv verhindert.

Man unterscheidet drei Arten von IDS:

Host-basierte IDS (HIDS)
Netzwerk-basierte IDS (NIDS)
Hybride IDS
Advanced Intrusion Detection Environment (AIDE)
Aide ist ein Host-basiertes IDS (HIDS) und wurde wurde ursprünglich von Rami Lehti und Pablo Virolainen als freie Alternative für das kommerzielle Tripwire entwickelt.

Das Programm kann als kostengünstige Baseline-Steuerung und als Rootkit-Erkennungssystem eingesetzt werden.

Aide erstellt eine Momentaufnahme (Schnappschuss) des Systemzustands, erfasst dabei Prüfsummen, Änderungszeitpunkte und weitere Merkmale, die sich auf Dateien beziehen, die vorher vom Administrator festgelegt wurden. Dieser Schnappschuss wird verwendet, um eine Datenbank aufzubauen, die wahlweise zur Sicherung auf einem externen Datenträger gesichert und wiederhergestellt werden kann.

Wenn ein Administrator einen Integritätstest durchführen will, legt er die vorher erstellte Datenbank auf einem erreichbaren Datenträger bereit und weist AIDE an, den Zustand in der Datenbank mit dem Zustand im gerade laufenden System zu vergleichen. Sollten sich dabei Veränderungen zeigen, wird AIDE diese entdecken und dem Administrator berichten. Alternativ kann AIDE auch so konfiguriert werden, dass es automatisch zu bestimmten Zeitpunkten läuft und täglich berichtet, welche Veränderungen sich ergeben haben. Üblicherweise werden dafür zeitgesteuerte Dienste wie cron benutzt.

Installation
[root@fravm009104 ~]# yum install aide
Konfiguration
[root@fravm009104 ~]# vi /etc/aide.conf
Initialisierung
[root@fravm009104 ~]# aide --init
 
AIDE, version 0.15.1
 
### AIDE database at /var/lib/aide/aide.db.new.gz initialized.
Aktivierung
Damit die zuvor generiert Datenbank für die Überprüfung auf Änderungen genutzt werden kann, muss sie initial umbenannt werden:

[root@fravm009104 ~]# mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
Integritätsprüfung
Die Integritätsprüfung kann nach der Aktivierung mittels des folgenden Befehls erfolgen, den man ebenfalls täglich per Cronjob ausführen kann. Exemplarisch wurde vor dem Ausführen ein Kommentar in der "/etc/ssh/sshd_config" hinzugefügt:  

[root@fravm009104 ~]# echo "#Test" >> /etc/ssh/sshd_config
[root@fravm009104 ~]# aide --check
 
AIDE 0.15.1 found differences between database and filesystem!!
Start timestamp: 2022-08-03 08:50:03
 
Summary:
  Total number of files:        116398
  Added files:                  0
  Removed files:                0
  Changed files:                1
 
---------------------------------------------------
Changed files:
---------------------------------------------------
 
changed: /etc/ssh/sshd_config
 
---------------------------------------------------
Detailed information about changes:
---------------------------------------------------
 
File: /etc/ssh/sshd_config
 SHA256   : Lrbi94vw+sBm46TIr0DLIJJEhpKkR03P , qKpgxy7lIdXhL+qhJcC7mD2voTtfRvoo
Cronjob:

[root@fravm009104 ~]# vi /etc/crontab
05 4 * * * root /usr/sbin/aide --check

Monitoring
Die AIDE-Log-Dateien des täglich ausgeführten Cronjobs können per "check_logfiles" auf Findings überprüft werden und so über das Icinga2-Monitoring detektiert werden. Im Fall von Findings würde dann ein Alert generiert werden. 

Vorschlag
Der folgende Vorschlag (s. Mail im Anhang) wurde bereits vor dem ISAE 3402 Audit und zuletzt am 3.8.2022 unterbreitet, nachdem die Änderung der /etc/ssh/sshd_config auf den DB-Servern der DoItNow-Plattform - frasv53151[4,5] - am 12.1.2022 nicht aufgefallen war:

Für die Lösung des Problems hinsichtlich der Überprüfung der System-Integrität, die derzeit nicht geprüft wird, kommen folgende Ansätze infrage:

Integration von SIEM im zLogging und Einrichtung eines entsprechenden Alertings -> Ist gemäß @Ehmig, Björn kurzfristig nicht möglich, da noch Hardwareressourcen und Lizenzen fehlen. Darüber hinaus ist fraglich, ob darüber eine komplette Integritätsprüfung überhaupt abgebildet werden kann.
Taste-OS: Z.Z. gibt es keine Möglichkeit ein Alerting für Änderung des Compliance-Levels einzurichten, was evtl. in ferner Zukunft möglich sein könnte.
OSSec ist ein zentral gesteuertes HIDS (Host-basierte IDS), was überschneidende Funktionalitäten mit der SIEM & zLogging Plattform hat und ebenfalls Features besitzt, die eine Lizenzierung erfordern -> https://www.ossec.net/
AIDE (Advanced Intrusion Detection Environment) ist ein HIDS, dass für alle Distributionen frei zur Verfügung steht, schnell implementiert und durch Icinga überwacht werden kann.
