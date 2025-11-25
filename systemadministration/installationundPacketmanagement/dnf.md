---
tags:
  - packet-management
  - dnf
---

# dnf Packet Manager

**Find The Package That Provides A Specific File**

`dnf provides '*filename'`

**List Installed Packages From A Certain Repository In Linux**

`yum list installed | grep @epel`
# dnf history

In der Spalte "Aktion(en)" wird jede Art von Aktion aufgeführt, die in der Transaktion durchgeführt wurde. Die möglichen Werte sind:

 - Install (I): ein neues Paket wurde auf dem System installiert
 - Downgrade (D): eine ältere Version eines Pakets hat die zuvor installierte Version ersetzt
 - Obsolete (O): ein veraltetes Paket wurde durch ein neues Paket ersetzt
 - Upgrade (U): eine neuere Version des Pakets ersetzt die zuvor installierte Version
 - Entfernen (E): ein Paket wurde aus dem System entfernt
 - Reinstall (R): ein Paket wurde mit der gleichen Version neu installiert
 - Grundänderung (C): ein Paket wurde im System belassen, aber der Grund für seine Installation wurde geändert

In der Spalte "Geändert" ist die Anzahl der in jeder Transaktion durchgeführten Aktionen aufgeführt, möglicherweise gefolgt von einem oder zwei der folgenden Symbole:

- >: Die RPM-Datenbank wurde nach der Transaktion geändert, außerhalb von DNF
- <: Die RPM-Datenbank wurde vor der Transaktion außerhalb von DNF geändert
- *: Die Transaktion wurde vor dem Abschluss abgebrochen
- #: Die Transaktion wurde abgeschlossen, aber mit einem Status ungleich Null
- E: Die Transaktion wurde erfolgreich abgeschlossen, aber mit einer Warnung/Fehlerausgabe

# dnf List

`dnf [options] list [--all] [<package-file-spec>...]`

**Lists all packages, present in the RPMDB, in a repository or both.**

`dnf [options] list --installed [<package-file-spec>...]`

**Lists installed packages.**

`dnf [options] list --available [<package-file-spec>...]`

**Lists available packages.**

`dnf [options] list --extras [<package-file-spec>...]`

**Lists extras, that is packages installed on the system that are not available in any known repository.**

`dnf [options] list --obsoletes [<package-file-spec>...]`

**List packages installed on the system that are obsoleted by packages in any known repository.**

`dnf [options] list --recent [<package-file-spec>...]`

**List packages recently added into the repositories.**

`dnf [options] list --upgrades [<package-file-spec>...]`

**List upgrades available for the installed packages.**

`dnf [options] list --autoremove`

# dnf fehler

Modular dependency problems

Lösung
Das Problem kann umgangen werden, wenn man die entsprechenden Pakete exkludiert, z.B.:

`yum update --exclude=nagios-plugins* -y`

Ein weitere Möglichkeit besteht darin, das entsprechende Modul zu deaktivieren, wie in dem folgenden Beispiel: 

`yum module disable freeradius -y`
