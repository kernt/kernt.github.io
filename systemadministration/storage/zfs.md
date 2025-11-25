# ZFS für eilige Sysadmins

ZFS ist das endgültige, letzte Dateisystem. So sehen es zumindest viele Entwickler und Enthusiasten. Ursprünglich wurde es von SUN für Solaris entwickelt. ZFS ist inzwischen aber auch für zahlreiche andere Betriebssysteme verfügbar. ZFS ist nicht nur ein Dateisystem, sondern auch Volume-Manager. Es hat seine Stärken im Bereich von großen Serverinstallationen, aber auch auf einem Raspberry mit USB-Platten kann man ein NAS betreiben.

Was ist an ZFS so interessant?

- Stabilität
- Flexibilität
- Performance

ZFS legt für jeden Datenblock eine Prüfsumme an. Sollte eine Differenz zwischen redundanten Daten im Raid auftauchen (silent bit-rod), wird dies bemerkt und kann _geheilt_ werden. ZFS hat _kein Journal_ und auch _keine Reparaturprogramme_ um Dateitrümmer nach Abstürzen aufzuspüren. Eine Datei ist entweder fehlerfrei geschrieben, oder nicht existent.

ZFS beherrscht alle Raid-Varianten und deren Kombinationen. Selbst in einem einfachen Pool mit _einem_ Datenträger kann man mehrere Kopien der Daten automatisch mitführen.

ZFS kann Snapshots des Dateisystems in Sekundenbruchteilen anlegen und wieder herstellen. Per Auto-Snapshot könnte man folgende Strategie einsetzen:

- frequent = alle 15 min
- hourly = jede Stunde
- daily = jeden Tag
- weekly = jede Woche
- monthly = jeden Monat

Im Schadensfall setzt man das Dataset (Directory) mit einem `rollback` zurück, oder holt sich einzelne Dateien aus dem Snapshot. Die Größe des Datasets spielt dabei keine Rolle. Ein `rollback` ist kein zurück kopieren von Daten, sondern die Metadaten zeigen dann auf die Datenblöcke des Snapshots. Das geht auch mit vielen Gigabyte blitzschnell.

Was kann ZFS noch so? Quotas, SMB und NFS und ISCSI Export, Kompression, Deduplikation (Vorsicht, braucht extrem viel RAM), variable Blockgrößen, ...

ZFS kann - je nach Layout des Pools - Schreib- und Lesegeschwindigkeiten im Bereich GB/s erreichen. Zur Beschleunigung können `LOG` und `Cache` auf SSDs hinzu geschaltet werden.

ZFS gilt als RAM-hungrig. Per default kann bis zur Hälfte des Arbeitsspeichers für den `ARC` Puffer verwendet werden. Für Produktionssysteme wird 1 GB RAM pro Terabyte Plattenplatz empfohlen.

ZFS ist einfach zu managen, es gibt nur zwei Befehle: zpool und zfs. (Aber Unix-üblich gaaanz viele Parameter ;-)
## Installation

Für die ersten Schritte enpfiehlt sich eine Ubuntu 16.* Installation und etwas Plattenplatz, um die unterschiedlichen Konfigurationen für Festplatten zu simulieren.

Zuerst benötigen wir die ZFS Pakete:

```
sudo apt install zfsutils-linux
```

Dann legen wir uns einige _Festplatten_ an, die wir für die Simulation der verschiedenen Raid-Modi brauchen. Im Produktionsbetrieb sollten immer ganze Festplatten gleicher Größe und Geschwindigkeit verwendet werden. Sie sollten dann unter ihrem eindeutigen Pfad eingebunden werden (nicht sdb sdc), sondern z.B.: `/dev/disk/by-id/ata-TOSHIBA_MQ01ABB200_Y3HYTBRKT`

```
truncate -s 100G disk-a-100
truncate -s 100G disk-b-100
truncate -s 100G disk-c-100
truncate -s 100G disk-d-100
```

Damit haben wir vier 'Platten' à 100GB, die aber _noch nicht_ belegt sind.
## Pool anlegen

### Raid-0

Alle Platten als Raid-0 zusammen, maximale Performance, maximaler Platz, 0 Sicherheit.

```
zpool create testpool /PATH/disk-a-100 /PATH/disk-b-100 \
                      /PATH/disk-c-100 /PATH/disk-d-100
```
### Raid-1

Zwei Platten gespiegelt, gute Schreib- Leseperformance, 50% vom Bruttoplatz, Sicherheit +1

```
zpool create testpool mirror /PATH/disk-a-100 /PATH/disk-b-100 
```
### Raid-10

2x2 gespiegelte Platten, Performance +1, 50% Platz, Sicherheit +1

```
zpool create testpool mirror /PATH/disk-a-100 /PATH/disk-b-100 \
                      mirror /PATH/disk-c-100 /PATH/disk-d-100
```
### Raid-5

(mindestens 3 Platten nötig, 'einfache' Schreibperformance, Lesen +1, Sicherheit +1)

```
zpool create testpool raidz /PATH/disk-a-100 /PATH/disk-b-100 \
                            /PATH/disk-c-100 /PATH/disk-d-100
```
### Raid-6 mit doppelter Parity

(mindestens 4 Platten, davon können 2 ausfallen, nur 50% vom Brutto nutzbar)

```
zpool create testpool raidz2 /PATH/disk-a-100 /PATH/disk-b-100 \
                             /PATH/disk-c-100 /PATH/disk-d-100
```

Damit wird ein Pool angelegt und sofort unter /testpool gemountet. Bei 'echten' Festplatten sollte die Sektorgröße beim Erstellen mit `-o ashift=12` deklariert werden (Aktuelle Platten haben 4096 das sind 2^12). Bevor er genutzt wird, sollten einige Optionen noch gesetzt werden. Diese Werte vererben sich im Pool und gelten innerhalb eines sog. Dataset. Diese Werte können jederzeit geändert werden, betreffen dann aber nur neue Daten.

- compression = `zfs set compression=on testpool`
- Zugriffszeit = `zfs set atime=off testpool`
- Ersatz von Disks = `zfs set autoreplace=on testpool`
- größere Disks = `zfs set autoexpand=on testpool`
- erweiterte Dateiattribute = `zfs set xattr=sa testpool`

Innerhalb unseres Pools sollten wir nicht direkt Daten in 'normalen' Directories speichern, sondern immer Datasets anlegen, die individuell behandelt werden können. Das ist besonders wichtig für spätere Snapshots. Der Poolname hat keinen führenden /! Hier ein Beispiel:

- `zfs create testpool/home`
- `zfs create testpool/home/bill`
- `zfs set quota=10G testpool/home`
- `zfs set quota=100G testpool/home/bill`
## Nützliche Befehle

- zpool status # gehts gut?
- zpool iostat -v 1 # Durschsatz/s
- zpool list # Brutto Belegung des Pools
- zpool import | export POOLNAME # 'mounten' eines ZFS-Pools
- zfs list -o space # Belegung der Datasets mit Snapshots
- zfs list -t snapshot # alle Snapshots des Pools
- zfs list -H -t snapshot -o name -S creation | tail -n1 # ältester Snapshot im Pool
- zfs get all testpool/home/bill | more # alle Parameter des Datasets
# ZFS Snaphots

Einen rekursiven Snapshot aller unter testpool/home liegenden Datasets legt man so an:

```
now=`date +%Y-%m-%d_%H:%M`
/sbin/zfs snapshot -r testpool/home@$now
```

Snapshots listen: `zfs list -t snapshot | grep testpool/home@` Sollen alle Daten eines Dataset wieder auf einen Zeitpunkt zurückgesetzt werden: `zfs rollback SNAPSHOTNAME` Snapshots löscht man mit `zfs destroy SNAPSHOTNAME`

Snapshots liegen im Directory `.zfs/snapshot` im Dataset und sind nicht mit `ls -a` sichtbar!

**List snapshots**

`zfs list -t snapshot`

**Snapshot anlegen**

Mit ZFS-Bordmitteln ist das Übertragen eines Snapshots auf einen Backup-Server schnell erledigt:

```
zfs snapshot pool01/daten@snap01
zfs send pool01/daten@snap01 | \
ssh <server02> zfs recv -F pool02/backup
```

Das sichert einen ersten Snapshot von _pool01_ mit dem Dataset _daten_ auf „Server01“ und überträgt ihn in _pool02_ ins Dataset _backup_ auf „Server02“_._ Die Option _–__F_ stellt auf dem Zielserver den Zustand des vorigen Snapshots sicher, etwaige Änderungen verfallen. Alle weiteren Snapshots überträgt man per

```
zfs snapshot pool01/daten@snap02
zfs send -i pool01/daten@snap01 pool01/daten@snap02 | ssh <server02> 
zfs recv pool02/backup
```
### Vereinfachtes Handling

Ist _zrep_ wie oben beschrieben eingerichtet, muss man es zunächst mit Informationen über die zu sichernden Datasets und das Backup-System initialisieren:

`zrep init pool01/daten <server02> pool02/backup`

Falls nicht vorhanden, legt _zrep_ das Dataset _backup_ automatisch an. Soll das Werkzeug weitere Datasets sichern, müssen diese ebenfalls initialisiert werden, damit spätere Backups sie berücksichtigen. Die eigentliche Replikation startet der Befehl

`zrep sync pool01/daten`

```sh
while true;
  do zrep sync all;
  # sleep 60
done
```
## Auto-Snapshots

[https://github.com/zfsonlinux/zfs-auto-snapshot](https://github.com/zfsonlinux/zfs-auto-snapshot)
# Zfs und Ansible

* [zfs_facts_module](https://docs.ansible.com/ansible/latest/collections/community/general/zfs_facts_module.html)
* [zpool-facts](https://docs.ansible.com/ansible/latest/collections/community/general/zpool_facts_module.html#ansible-collections-community-general-zpool-facts-module)