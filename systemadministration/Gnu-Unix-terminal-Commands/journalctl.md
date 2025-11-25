---
tags:
  - journalctl
  - timedatectl
  - konsole
  - system-administration
  - gnu-tools
---
# journalctl

**Log in UTC**

`journalctl --utc`

**Boot vorgang**

`journalctl -b`

**Boot Vorgänge**

`journalctl --list-boots`

**Dach Datum**

`journalctl --since "2015-01-10 17:15:00"`

**Von bis**

`journalctl --since "2015-01-10" --until "2015-01-11 03:00"`

**Seit**

`journalctl --since yesterday`

**Via Unit seit**

`journalctl -u nginx.service --since today`

**Über Process**

`journalctl _PID=8088`

**Log Daten älter als 1 Monat Löschen**

`journalctl --vacuum-time=1month`

**Speicher verbrauch des journels anzeigen**

`journalctl --disk-usage`

**Ohne Pager ausgeben**

`journalctl --no-pager`

**Bootvorgänge auflisten**

`journalctl --list-boots`

**Gefiltert nach Datum**

`journalctl --since "2025-07-01" --until "2 minutes ago"`

**Rechte Prüfen**

`usermod -aG systemd-journal BENUTZERNAME`

**Filter nach Datum und Fehler Level**

`journalctl -p err -b --since "2019-01-11"`

# Journal space

**Clear (reduce) the folder size instantly**

```shell
sudo journalctl --vacuum-size=100M
```

Note that **vacuuming only removes archived journal files**, not active ones. To remove everything, you must first rotate the files so that recent entries are moved to inactive files.

```shell
sudo journalctl --rotate
sudo journalctl --vacuum-time=1s
```

[anchor to Control the amount of disc space that /var/log/journal can utilise](https://www.sitelint.com/blog/how-do-i-clear-a-big-var-log-journal-folder#control_the_amount_of_disc_space_that_varlogjournal_can_utilise)

Control the amount of disc space that /var/log/journal can utilise

_/etc/systemd/journald.conf_

```shell
SystemMaxUse=25M
```

```shell
sudo systemctl restart systemd-journald.service
```

Speicher verbrauch

```sh
journalctl --disk-usage
```

Integrrität Prüfen

```sh
journalctl --verify
```

### Query only by priority ERROR

Show only the entries flagged with priority ERROR.

`journalctl -p err`

Or since last boot:

`journalctl -b -p err`