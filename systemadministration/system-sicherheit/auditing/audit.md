---
tags:
  - sicherheit
  - sicherheits-tools
  - audit
---
# Audit

### Binaries

- **auditd**: daemon to capture events and store them (log file)
- **auditctl**: client tool to configure auditd
- **audispd**: daemon to multiplex events
- **aureport**: reporting tool which reads from log file (auditd.log)
- **ausearch**: event viewer (auditd.log)
- **autrace**: using audit component in kernel to trace binaries
- **aulast**: similar to last, but instead using audit framework
- **aulastlog**: similar to lastlog, also using audit framework instead
- **ausyscall**: map syscall ID and name
- **auvirt**: displaying audit information regarding virtual machines
### Files

- audit.rules: used by auditctl to read what rules need to be used
- auditd.conf: configuration file of auditd
# Installation

### Debian / Ubuntu

`apt install auditd audispd-plugins`

### Fedora / Red Hat

Usually already installed (package: **audit** and **audit-libs**)
### audit.rules

`auditctl -a exit,always -F path=/etc/passwd -F perm=wa`

By defining the path option, we instruct the audit framework what directory or file to watch for. The permissions determine what kind of access will trigger an event. Although these look similar to file permissions, note that there is a important difference between the two. The four options are:

- **r** = read
- **w** = write
- **x** = execute
- **a** = attribute change
## Audit Konfigurationen

2.1 Protokollierung (auditing) muss beim Start durch Festlegen eines Kernelparameters aktiviert werden.

2.2 Der Dienst "auditd" muss genutzt werden, um sicherheitsrelevante Events zu protokollieren.

2.3 Syscalls "execve" (execute program) müssen protokolliert werden.
2.4 System-Events müssen protokolliert werden.
2.5 Zugriffs- und Authentifizierungs-Events müssen protokolliert werden.
2.6 Konten- und Gruppen-Management-Events müssen protokolliert werden.
2.7 Änderungen der Konfiguration müssen protokolliert werden.
2.8 Die Konfiguration von auditd muss auf "immutable" gesetzt werden.

## Grundlagen

Als Grundlage für die im folgenden beschriebene Audit Standardkonfiguration diente das von der Telekom Security (T-Sec) entwickelte Privacy & Security Assessment Verfahren (PSA), das ebenfalls ein zentraler Baustein zur Gewährleistung der Sicherheit und des Datenschutz der Deutschen Telekom ist.

Das  PSA Verfahren wurde mit dem Gütesiegel des international anerkannten ISO 27001 Zertifikats ausgezeichnet und bildet einen wesentlichen Baustein des zertifizierten Datenschutzmanagementsystems nach Standard PS 980. Zudem bildet das PSA-Verfahren einen elementaren Grundstein für die Einhaltung der Europäischen Datenschutzverordnung (s. Quellen).

Für die Umsetzung der Audit Einstellungen, die vom PSA Verfahren gefordert werden, wurde ein Salt-State entwickelt (audit.sls), das die Datei audit.rules auf die Zielsysteme kopiert.
# Audit Konfigurationen

Das Programm "Auditd" ist ein Zugangsprotokollierungs- und Accounting-System für Linux.
Es kann genutzt werden, um Events zu definieren, die im Linux OS protokolliert werden sollen.
Aus Sicherheitssicht können diese Events genutzt werden, um auffällige Ereignisse zu erkennen oder um potentielle Angriffe zu analysieren.

33: Protokollierung (auditing) muss beim Start durch Festlegen eines Kernelparameters aktiviert werden.
Jeder Prozess auf dem System trägt ein "auditable" Flag, das angibt, ob seine Aktivitäten protokolliert werden können. 

>Obwohl "auditd" dafür sorgt, dass dies für alle Prozesse aktiviert wird, die nach "auditd" gestartet werden, wird durch Hinzufügen des Kernelparameters (audit=1) sichergestellt, dass es während des Startvorgangs für jeden Prozess festgelegt wird.

Beispiel

```
grubby --update-kernel=ALL --args="audit=1"
```

Motivation: 

Die Protokollierung von sicherheitsrelevanten Ereignissen ist eine Grundvoraussetzung, um laufende Angriffe oder auch Angriffe im Nachhinein erkennen zu können. Nur so ist es möglich, sinnvolle Maßnahmen zur Erhaltung oder auch Wiederherstellung der Systemsicherheit durchzuführen. Des Weiteren dienen die Protokollierungsdaten der Beweissicherung, um rechtlich gegen Angreifer vorgehen zu können.

Der Dienst "auditd" muss genutzt werden, um sicherheitsrelevante Events zu protokollieren.
Auf Linux Servern muss das Programm Auditd installiert und konfiguriert werden, um sicherheitsrelevante Events zu
loggen. Jedes sicherheitsrelevante Event muss mit einem möglichst genauen Zeitstempel und einer eindeutigen Systembezeichnung protokolliert werden. 

Motivation: 

Die Protokollierung von sicherheitsrelevanten Ereignissen ist eine Grundvoraussetzung, um laufende Angriffe oder auch Angriffe im Nachhinein erkennen zu können. Nur so ist es möglich, sinnvolle Maßnahmen zur Erhaltung oder auch Wiederherstellung der Systemsicherheit durchzuführen. Des Weiteren dienen die Protokollierungsdaten der Beweissicherung, um rechtlich gegen Angreifer vorgehen zu können.

Für diese Anforderung sind folgende Bedrohungen relevant:

Abstreiten von durchgeführten Aktionen
Unbemerkt durchführbare Angriffe
ID: 3.65-37/4.0

Umsetzung durch die folgenden Salt-State-Sektionen:

```
/etc/audit/audit.rules:
  file.managed:
    - source: salt://psa/files/audit.rules
    - user: root
    - group: root
    - mode: 640
 
/etc/audit/rules.d/audit.rules:
  file.managed:
    - source: salt://psa/files/audit.rules
    - user: root
    - group: root
    - mode: 640
 
auditd_service:
  service.running:
    - name: auditd
    - enable: True
    - restart: True
```

Req 38: Syscalls "execve" (execute program) müssen protokolliert werden.
Auf Linux-Servern müssen die folgenden Events protokolliert werden:

Ausführung von Programmen
Alle Programmausführungen müssen protokolliert werden.
Verbindlich

Motivation: 

Forensik & Mitigation : Im Falle einer Kompromittierung eines Linux-Hosts muss nachvollziehbar sein, welche Aktionen ausgeführt wurden.

Für diese Anforderung sind folgende Bedrohungen relevant:

- Abstreiten von durchgeführten Aktionen
- Unbemerkt durchführbare Angriffe

Umsetzung durch die folgende /etc/audit/audit.rules Sektion:
# Exec events

```
-a exit,always -F arch=b64 -S execve
-a exit,always -F arch=b32 -S execve
```

Req 39: System-Events müssen protokolliert werden.
Auf Linux-Servern müssen die folgenden Events protokolliert werden.

System Start und Shutdown	Jeder Neustart oder Shutdown des Betriebssystems muss protokolliert werden.	Verbindlich
(De-)Installation von Software	Nach der Kommissionierung eines Servers muss jede Deinstallation und Installation von Software protokolliert werden.	Verbindlich
Änderung Systemzeit	Änderung der lokalen Systemzeit und der NTP-Konfiguration muss protokolliert werden.	Verbindlich
Verbindung von externen Geräten
(Speicher)	Die Verbindungen von externen Geräten wie USB-Laufwerken, die auf dem Server gemountet werden können, müssen protokolliert werden.	Verbindlich
Ausführung privilegierter Kommandos	Die Nutzung privilegierter Kommandos mit SUID/SGID muss protokolliert werden.	Verbindlich
Laden und Stoppen von Kernel-Modulen	Das Laden und Stoppen von Kernel-Modulen muss protokolliert werden.	Verbindlich
Änderung von terminierten Jobs	Jobs, die periodisch ausgeführt werden, müssen protokolliert werden, wenn sie verändert oder gelöscht werden.	Optional

Motivation: 

Es ist ungewöhnlich, wenn nach der Inbetriebnahme Veränderungen an den Betriebssystemeinstellungen vorgenommen werden. Ein Angreifer der Zugriff auf einen Server hat, kann die Änderungen vornehmen. Das protokollieren
von Systemevents ist daher notwendig, um solche Angriffe zu erkennen und nachverfolgen zu können. 

Für diese Anforderung sind folgende Bedrohungen relevant:

Abstreiten von durchgeführten Aktionen
Unbemerkt durchführbare Angriffe
ID: 3.65-39/4.0

Umsetzung durch die folgende /etc/audit/audit.rules Sektion:

**System event logging**

```sh
-w /etc/at.deny
-w /etc/at.allow
-w /etc/localtime -p wa -k time-change
-w /var/spool/at/
-a always,exit -F arch=b64 -S clock_settime -k time-change
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time-change
-a always,exit -F arch=b64 -S mount -F auid>=1000 -F auid!=4294967295 -k mounts
-a always,exit -F arch=b64 -S init_module -S delete_module -k modules
-a always,exit -F arch=b64 -S mount -F auid>=500 -F auid!=4294967295 -k export
-w /sbin/modprobe -p x -k modules
-w /etc/anacrontab
-w /etc/crontab
-w /etc/cron.d/
-w /etc/cron.allow
-w /etc/cron.deny
-w /etc/cron.daily
-w /etc/cron.hourly/
-w /etc/cron.monthly/
-w /etc/cron.weekly/
-w /sbin/rmmod -p x -k modules
-w /sbin/insmod -p x -k modules
```

Req 40: Zugriffs- und Authentifizierungs-Events müssen protokolliert werden.
Auf Linux-Servern müssen die folgenden Zugriffs- und Authentifizierungs-Events protokolliert werden:

An-/Abmeldung	Das lokale oder entfernte An- und Abmelden von Benutzern muss protokolliert werden.	Verbindlich
Änderung von Passwörtern	Die Änderung oder das Zurücksetzen von Benutzerpasswörtern muss
protokolliert werden.	Verbindlich
Erweiterung von Privilegien	Es muss protokolliert werden, wenn ein Nutzer sudo ausführt oder korospondierende Dateien verändert (sudoers).	Verbindlich
Änderung der DAC Berechtigungen	Die Änderung der DAC Berechtigungen muss protokolliert werden.	Verbindlich

Motivation: 

Die Protokollierung von Zugriffs- und Authentifizierungsevents können hilfreich sein, um nachzuverfolgen wer Zugriff zu einem bestimmten Zeitpunkt hatte. Mittels dieser Protokollierungsdaten ist es beispielsweise möglich, Konten ausfindig zu machen, die von einem Angreifer missbräuchlich genutzt werden.

Für diese Anforderung sind folgende Bedrohungen relevant:

Abstreiten von durchgeführten Aktionen
Unbemerkt durchführbare Angriffe
ID: 3.65-40/4.0

Umsetzung durch die folgende /etc/audit/audit.rules Sektion:

**Access events**

```
-a always,exit -F arch=b64 -S setxattr -S lsetxattr -S fsetxattr -S removexattr -S lremovexattr -S fremovexattr -F auid>=1000 -F auid!=4294967295 -k perm_mod
-a always,exit -F arch=b64 -S chmod -S fchmod -S fchmodat -F auid>=1000 -F auid!=4294967295 -k perm_mod
-a always,exit -F arch=b64 -S chown -S fchown -S fchownat -S lchown -F auid>=1000 -F auid!=4294967295 -k perm_mod
-w /etc/sudoers -p wa -k scope
-w /etc/sudoers.d -p wa -k scope
-w /var/log/sudo.log -p wa -k actions
-w /var/log/lastlog -p wa -k logins
-w /etc/shadow -p wa -k identity
-w /etc/gshadow -p wa -k identity
-w /etc/security/opasswd -p wa -k identity
```

Req 41: Konten- und Gruppen-Management-Events müssen protokolliert werden.
Auf Linux-Servern müssen die folgenden Zugriffs- und Authentifizierungs-Events protokolliert werden:

Erstellen, modifizieren und entfernen von Konten	Es muss protokolliert werden, wenn Konten erstellt, modifiziert oder gelöscht werden.	Verbindlich
Erstellen, modifizieren und entfernen
von Gruppen	Es muss protokolliert werden, wenn Gruppen erstellt, modifiziert oder gelöscht werden.	Verbindlich
Erweiterung von Privilegien	Es muss protokolliert werden, wenn ein Nutzer sudo ausführt oder korospondierende Dateien verändert (sudoers).	Verbindlich
Änderung der DAC Berechtigungen	Die Änderung der DAC Berechtigungen muss protokolliert werden.	Verbindlich

Motivation: 

Das Protokollieren von Konten- und Gruppen-Management-Events kann hilfreich sein um das Management von Benutzern und Gruppen nachzuverfolgen. Mit diesen Daten ist es möglich maliziöses Anlegen, Verändern oder Löschen von Benutzern und Gruppen zu erkennen.

Für diese Anforderung sind folgende Bedrohungen relevant:

Abstreiten von durchgeführten Aktionen
Unbemerkt durchführbare Angriffe
ID: 3.65-41/4.0

Umsetzung durch die folgende /etc/audit/audit.rules Sektion:
# Group mgmt event

```
-w /etc/group -p wa -k identity
-w /etc/passwd -p wa -k identity
```

Req 42: Änderungen der Konfiguration müssen protokolliert werden.
Auf Linux-Servern müssen die folgenden Änderungen der Konfiguration protokolliert werden:

Deaktivierung der Protokollierung	Es muss protokolliert werden, wenn der Protokollierungsdienst deaktiviert wird.	Verbindlich
Löschen und Verändern von Protokolldaten	Das Löschen von Events muss protokolliert werden. Die nicht autorisierte Änderung von Protokollierungsdaten muss protokolliert werden.	Verbindlich
Änderung der Konfiguration der Protokollierung	Es muss protokolliert werden, wenn die Konfiguration für die Protokollierung verändert wird.	Verbindlich
Änderungen der Konfiguration des
Netzwerks	Es muss protokolliert werden, wenn die Konfiguration von Netzwerk(-
schnittstellen) geändert wird.	Verbindlich
Änderungen des Authentifizierungs-
Subsystems	Änderungen des Authentifizierungs-Subsystems (z.B. LDAP- oder Kerberos-Policy) muss protokolliert werden.	Optional
Kritische Datei-Änderungen	Abhängig von Einsatzzweck müssen kritische Änderungen von Dateien
protokolliert werden.	Optional
Änderung der Konfiguration von SELinux	Wenn SELinux eingesetzt wird, müssen die entsprechenden Ereignisse
und die Änderung der Konfiguration protokolliert werden.	Optional
Änderung der Konfiguration von AppArmor	Wenn AppArmor eingesetzt wird, müssen die entsprechenden Ereignisse und die Änderung der Konfiguration protokolliert werden.	Optional

Motivation: 

Konfigurationsänderungen haben einen großen Einfluss auf das Betriebssystem und sind daher ein Sicherheitsrisiko. Es ist notwendig alle wichtigen Konfigurationen eines Betriebssystems zu identifizieren und Änderungen zu protokollieren.

Für diese Anforderung sind folgende Bedrohungen relevant:

Abstreiten von durchgeführten Aktionen
Unbemerkt durchführbare Angriffe
ID: 3.65-42/4.0

Umsetzung durch die folgende /etc/audit/audit.rules Sektion:

# Konfiguration der Änderung events

```
-w /etc/rsyslog.d/conf
-w /etc/rsyslog.conf
-w /etc/syslog.
-w /etc/issue -p wa -k system-locale
-w /etc/issue.net -p wa -k system-locale
-w /etc/sysctl.conf
-a always,exit -F arch=b64 -S sethostname -S setdomainname -k system-locale
-w /etc/ssh/sshd_config
-w /etc/networks -p wa -k system-locale
-w /etc/network -p wa -k system-locale
-w /etc/profile
-w /etc/profile.d/
-w /var/log/audit/audit[1-4].log
-w /var/log/audit/audit.log
-w /etc/hosts -p wa -k system-locale
-w /etc/modprobe.conf
-w /etc/audit/auditd.conf -p wa
-w /etc/audit/audit.rules -p wa
-w /etc/shells
-w /etc/nsswitch.conf
-w /etc/pam.d/
```

**AppArmor events loggen**

```
-w /etc/apparmor.d/ -p wa -k MAC-policy
-w /etc/apparmor/ -p wa -k MAC-policy
```

**SELinux events loggen**

```
-w /usr/share/selinux/ -p wa -k MAC-policy
-w /etc/selinux/ -p wa -k MAC-policy
```

Req 43: Die Konfiguration von auditd muss auf "immutable" gesetzt werden.
Der "Immutable Mode" muss für den auditd Dienst gesetzt sein, um zu verhindern, dass Audit-Regeln mit dem Kommando "auditctl" verändert werden können.

Motivation: 

Wenn das Programm "auditd" nicht im sogenannten ""Immutable Mode" ist, können unautorisierte Benutzer Veränderungen vornehmen, um böswillige Aktivitäten zu verschleiern.

Für diese Anforderung sind folgende Bedrohungen relevant:

Abstreiten von durchgeführten Aktionen
Unbemerkt durchführbare Angriffe
ID: 3.65-43/4.0

Umsetzung durch die folgende /etc/audit/audit.rules Sektion:

**Ensure the audit configuration is immutable**

-e 2

**Öffnen einer bestimmten Datei überwachen**

```sh
auditctl -a always,exit -F path=/var/log/syslog -F perm=r -k syslog-access
```

**Deaktivieren von Audit-Regeln**

`auditctl -D -w /var/log/syslog -p r -k syslog-access`
# ausearch

**Zum Beispiel kann man alle Zugriffe auf die Datei /var/log/syslog suchen**

```sh
ausearch -f /var/log/syslog
```

https://slack.engineering/syscall-auditing-at-scale/
https://linux-audit.com/linux-audit-framework/linux-audit-framework-101-basic-rules-for-configuration/
https://linux-audit.com/logging-root-actions-by-capturing-execve-system-calls/