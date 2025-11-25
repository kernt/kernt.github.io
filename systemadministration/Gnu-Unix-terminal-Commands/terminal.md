# Nützle befehle


**Wer ist online (Unter Spalte TTY wird das Terminal angezeigt) und  etwas schreiben?:**

```sh
# who
bgxxxx  pts/2        Jul  9 13:14 (139.1.164.55)
maxxxx  pts/3        Jul  9 13:15 (139.1.164.75)
sjxxxx  pts/4        Jul  9 04:58 (10.216.11.8)
ubxxxx  pts/1        Jun 29 07:16 (139.1.164.55)
 
# write maxxxx pts/3
Hallo ! Ich melde mich jetzt ab.
CTRL-C

while [ 1 = 1 ]; do write maxxxx pts/3 < /tmp/test.txt ; done

```

**Protokoll git:// lokal auf https:// umleiten**

`git config --global url."https://".insteadOf git://`

**Installationsdatum abfragen**

`ls --sort=t /dev -l | tail -n1 | awk '{print $8 " " $7 " " $9}'`

`top -b -n 1 | awk '{if (NR <=7) print; else if ($8 == "D") {print; count++} } END {print "Total status D (I/O wait probably): "count}'`

Gelöschte/verlorene Dateien wiederherstellen:
Problem: Man editiert mit VI eine Datei und die SSH-Session fliegt weg. SCREEN wurde nicht genutzt.
Lösung:
Man lokalisiert den VI Prozess (32563):

```
ps ax | grep -i vi
2061 ?        Ss     0:08 /usr/sbin/hald --daemon=yes --retain-privileges
32563 pts/3    S+     0:00 vi test.txt
```

Nun welchselt man in /proc/[PROZESSID]/fd/:

cd /proc/32563/fd/

Wenn man sich das Verzeichnis anzeigen lässt, findet man einen Link, der auf die offene Datei zeigt:

```
total 4
2134048778 0 dr-x------ 2 root root  0 Oct 15 13:16 .
2134048770 0 dr-xr-xr-x 5 root root  0 Oct 15 13:15 ..
2134081536 1 lrwx------ 1 root root 64 Oct 15 13:16 0 -> /dev/pts/3
2134081537 1 lrwx------ 1 root root 64 Oct 15 13:16 1 -> /dev/pts/3
2134081538 1 lrwx------ 1 root root 64 Oct 15 13:16 2 -> /dev/pts/3
2134081539 1 lrwx------ 1 root root 64 Oct 15 13:16 3 -> /root/.test.txt.swp
```

Nun kann man sich den Inhalt der Datei mit 'cat' anzeigen lassen oder in eine neue Datei umleiten. 
Da der Inhalt meist in umgekehrter Reihenfolge angezeigt wird, sollte man das Kommando 'tac' benutzen.
Das Ganze funktioniert nur solange die Datei noch von einem Prozess offen gehalten wird !!!


Systemleistung

**Summe CPU und Kerne:**

`grep physical\ id /proc/cpuinfo |sort -u| echo "Anzahl CPUs: $(wc -l)" && grep core\ id /proc/cpuinfo |sort -u| echo "Kerne: $(wc -l)"`

**Summe des RAM-Verbrauchs aller laufenden Prozesse anzeigen:**

`ARRAY=$(ps -e -orss=,args= | sort -b -k1,1n | pr -TW$COLUMNS | awk '{print $1}'); for entry in ${ARRAY[*]}; do sum=$(( $sum + $entry )); done; echo $sum`

**RAM-Nutzung laufender Prozesse anzeigen:**

`for file in /proc/*/status ; do awk '/VmSize|Name|^Pid/{printf $2 " " $3}END{ print ""}' $file; done | sort -k 3 -n -r | less`
 
#oder
 
`ps -e -o pid,vsz,comm= | sort -n -k 2`

Maximale (PEAK) RAM-Nutzung laufender Prozesse anzeigen:

`for file in /proc/*/status ; do awk '/VmPeak|Name|^Pid/{printf $2 " " $3}END{ print ""}' $file; done | sort -k 3 -n -r | less`

SWAP-Nutzung laufender Prozesse anzeigen:

```sh
LINES=10; for file in /proc/*/status; do awk '/VmSwap|Name/ { printf $2 " " $3 }END{ print ""}' $file; done | sort -k 2 -n -r | head -n $LINES | grep -iv "no such" | awk '{ print $1 "\t\t" $2}'
LINES=10; for file in /proc/*/status; do awk '/VmSwap|Name/ { printf $2 " " $3 }END{ print ""}' $file 2>&1; done | sort -k 2 -n -r | head -n $LINES | column -ts" "
```
 
oder
 
`for file in /proc/*/status ; do awk '/VmSwap|Name|^Pid/{printf $2 " " $3}END{ print ""}' $file; done | sort -k 3 -n -r | less`


Die Variable LINES definiert die Anzahl der auszugebenden Resulate. Je grösser, desto mehr Output.

Wesentlich aussagekräftiger ist folgendes Script (POSIX konform, also auf allen unixoiden OS lauffähig):

```sh
#!/bin/sh
 
SUM=0
OVERALL=0
TMPf=$(mktemp)
 
for DIR in `find /proc/ -maxdepth 1 -type d | egrep "^/proc/[0-9]" | sort -V` ;   do
   PID=`echo $DIR | cut -d / -f 3`
 
   SUM=0
    for SWAP in `grep Swap $DIR/smaps 2>/dev/null| awk '{ print $2 }'`;   do
        SUM=$((SUM+SWAP))
    done
 
    [ "x$SUM" = "x0" -o "x$SUM" = "x" ] && continue
   PROGNAME=`readlink $DIR/exe`
   #PROGNAME=`awk '$1=="Name:" { print $2 }' $DIR/status 2>/dev/null`
    echo "PID=$PID   \t- Swap benutzt: $SUM   \t- ($PROGNAME )"
    OVERALL=$((OVERALL+SUM))
done > $TMPf
 
cat $TMPf  | sort -n -k5
rm $TMPf
echo "SWAP Nutzung: $OVERALL"
IO-Messung:
vmstat -a > /tmp/vmstat.log; vmstat -d >> /tmp/vmstat.log; vmstat -D >> /tmp/vmstat.log; vmstat -p /dev/cciss/c0d0p2 >> /tmp/vmstat.log; vmstat -s >> /tmp/vmstat.log; vmstat -m >> /tmp/vmstat.log; vmstat -n 1 10 >> /tmp/vmstat.log; mv /tmp/vmstat.log /tmp/vmstat-$(date "+%Y%m%d-%H%M%S").log
```


Prüfen der Systemauslastung aufgrund I/O WAITS und Anzahl der CPU-Kerne/Threads:
Das Script meldet, wenn die Festplatten den geforderten I/O nicht mehr schaffen. Die Werte können/müssen noch feingetuned werden, liefern aber schon einen guten Anhaltspunkt.
Angepasst werden kann im Bedarfsfall der Parameter

`IOWAITMAX=4`

am Anfang der Befehlszeile. Um etwas kritischere Prüfungen zu erreichen, sollte der Wert 4 verringert werden.

```sh
IOWAITMAX=4; clear; CPUCOUNT=$(echo "$(cat /proc/cpuinfo | grep -i "processor" | wc -l) / 1"|bc); MAXWAT=$(echo "$(echo "scale=1; 1 / $CPUCOUNT * 100" | bc -l | cut -d "." -f1) / 1"|bc); WAITINGACCESSTIME=$(echo "$(top -b -n 1 | head -n 5 | grep -i "cpu(s)" | cut -d "," -f5 | sed 's/ //g' | sed 's/\..*//g') / 1" | bc); IOWAIT=$(echo "$(iostat -c -d -m | head -n 4 | tail -n 1 | tr -s " " | awk -F " " '{ print $4 }' | cut -d "." -f1) / 1"|bc); if [ $WAITINGACCESSTIME -gt $MAXWAT ] || [ ${IOWAIT} -ge ${IOWAITMAX} ]; then echo "INFO: Mom. WaitingAccessTime: ${WAITINGACCESSTIME} darf nicht groesser sein als Max. WAT: ${MAXWAT}"; echo "I/O Wait: ${IOWAIT}"; if [ ${IOWAIT} -gt 4 ]; then echo "IO Wait ist groesser als ${IOWAITMAX} --> Festplatten zu langsam"; fi; else echo "Keine Performanceprobleme aufgrund mangelnder Ressourcen feststellbar"; fi
```


System langsam (X1P), „top“ zeigt nur 23% CPU-Last, Swap ist nur zur Hälfte belegt:
Ermitteln der I/O-Systemlast

`iostat -xd`

zeigt, dass der meiste I/O auf

```
Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await  svctm  %util
cciss/c0d0       57.48   136.99   21.20   49.51   773.20  1492.21    32.04    14.29  202.12   3.66  25.91
liegt.
```

`cat /proc/devices | grep ccis`

zeigt, dass die lokale Platte Devicenummer „104 cciss0“ besitzt.
Nun lassen wir uns anzeigen, welcher Prozess/User die meiste Last auf Device 104 erzeugt:

```sh
lsof | grep "104," | awk ' { print $7"=SIZE  "$1"=COMMAND  "$2"=PID  "$3"=USER  "$6"=DEVICE  "$9"=NODENAME" }' | sort -n
...
1613048=SIZE  rscd=COMMAND  8056=PID  root=USER  104,2=DEVICE  /usr/nsh/NSH/bin/rscd_full=NODENAME
4396928=SIZE  rscd=COMMAND  8052=PID  root=USER  104,2=DEVICE  /usr/nsh/NSH/lib/libBLCfgParser.so.1.0=NODENAME
4396928=SIZE  rscd=COMMAND  8055=PID  root=USER  104,2=DEVICE  /usr/nsh/NSH/lib/libBLCfgParser.so.1.0=NODENAME
4396928=SIZE  rscd=COMMAND  8056=PID  root=USER  104,2=DEVICE  /usr/nsh/NSH/lib/libBLCfgParser.so.1.0=NODENAME
```

Wie man sieht, ist Device „104,2“ am meisten betroffen. BladeLogic Prozesse erzeugen eine hohe I/O-Last.

Um SWAP als Ursache letztendlich auszuschließen, prüft man mit

```sh
ls -l /dev/cciss/
 
brw-r----- 1 root disk 104, 0 Nov 20 00:12 c0d0
brw-r----- 1 root disk 104, 1 Nov 20 00:12 c0d0p1
brw-r----- 1 root disk 104, 2 Nov 20 00:12 c0d0p2   ---> Höchster IO
brw-r----- 1 root disk 104, 3 Nov 20 00:12 c0d0p3
```

welche Partition device „104,2“ ist und erfährt anschließend mit

```sh
fdisk -l | grep -i c0d0p2
 
/dev/cciss/c0d0p2   *         263        1307     8393962+  83  Linux
```

… dass es KEIN SWAP-Problem ist, IO wird auf dem normalen /-FS erzeugt.
(SWAP wäre “/dev/cciss/c0d0p1 1 262 2104483+ 82 Linux swap / Solaris“)

Die Frage, die sich hier stellt… Ist BladeLogic wirklich für den hohen I/O verantwortlich ???
Eine Gegenprüfung mit…

```sh
iostat -xdN | awk ' { print $7"=WSEC/S  "$3"=WRQM/S  "$8"=AVGRQ-SZ  "$1"=DEVICE" }' | sort -n
```

ergibt nämlich, dass die meiste Last im „rootvg-varlv“ erzeugt wird,…

```
...
163.97=WSEC/S  0.00=WRQM/S  8.01=AVGRQ-SZ  rootvg-varlv=DEVICE
198.08=WSEC/S  19.61=WRQM/S  38.53=AVGRQ-SZ  cciss/c0d0=DEVICE
```

… während eine Suche nach der grössten durchschnittlichen „Service Time“…

```sh
iostat -xdN | awk ' { print $11"=SVCTM  "$3"=WRQM/S  "$8"=AVGRQ-SZ  "$7"=WSEC/S  "$1"=DEVICE" }' | sort -n
...
1.20=SVCTM  0.00=WRQM/S  21.43=AVGRQ-SZ  0.01=WSEC/S  sdu=DEVICE
1.30=SVCTM  0.00=WRQM/S  8.00=AVGRQ-SZ  0.00=WSEC/S  rootvg-swaplv=DEVICE
1.60=SVCTM  0.00=WRQM/S  21.20=AVGRQ-SZ  0.01=WSEC/S  sdv=DEVICE
1.99=SVCTM  0.00=WRQM/S  18.57=AVGRQ-SZ  0.00=WSEC/S  sdl=DEVICE
2.06=SVCTM  0.00=WRQM/S  8.00=AVGRQ-SZ  0.00=WSEC/S  rootvg-sapswaplv=DEVICE
3.25=SVCTM  0.00=WRQM/S  14.50=AVGRQ-SZ  0.00=WSEC/S  sdb=DEVICE
```

… verrät, dass “/dev/sdb“, also ein Pfad im SAN und auch SWAP die grössten Antwortzeiten fordern.

Zusätzlich kann mit

```sh
top -b -n 1 | awk '{if (NR <=7) print; else if ($8 == "D") {print; count++} } END {print "Total status D (I/O wait probably): "count}'
```

ermittelt werden, um welche Art von Problem es sich handelt.

Die folgenden Status sind möglich:

R  running or runnable (on run queue)
D  uninterruptible sleep (usually IO)
S  interruptible sleep (waiting for an event to complete)
Z  defunct/zombie, terminated but not reaped by its parent
T  stopped, either by a job control signal or because
   it is being traced

Platzbelegung des aktuellen Verzeichnisses SCHNELL anzeigen lassen:
(schneller als ein 'du -h . –max-depth=1)

`du * -sch`

oder auch
```
ergebnis=0; for zahl in $(df -hP | awk '{ print $3 }' | grep "G" | cut -d "G" -f1); do ergebnis="$(echo "$ergebnis + $zahl" | bc)"; done; echo $ergebnis
Auswertung NUTZDATEN / FREIER PLATZ:
uzahl=""; USEDSIZES=$(df -P | egrep "^/dev/" | awk -F " " {'print $3'}| egrep "[1-9]"); USEDSUM=0; for uzahl in ${USEDSIZES[@]}; do let USEDSUM="$USEDSUM+$uzahl"; d
```

Alle Prozesse anzeigen, die auf ein/e Datei/Verzeichnis zugreifen:
In diesem Fall auf die Datei „std_server0.out“…

```sh
PROCESS=`fuser -m std_server0.out | egrep "[0-9]" | tr -s " "`; for entry in $PROCESS; do ps -A | grep $entry; done
```

Besser:

```sh
PROCESS=$(fuser -m uptime.txt | egrep "[[:digit:]]"); for process in ${PROCESS[@]}; do ps -Ao pid,tt,user,fname,tmout,f,wchan -q $process -o comm= | tail -n 1; do
```

Space Utilisation –> Alle Filesysteme anzeigen, die mehr als 70% voll sind:

`df -hP| tr -d "%"|awk '$5>70 { print ; }'`

Zeit eines fsck auf allen LVs, ohne Änderungen vorzunehmen:

`time for line in $(lvscan | cut -d "'" -f2); do fsck -fn $line;done`

Prozentsatz der belegten Inodentabelleneinträge bei VXFS überwachen:

```
while true; do ALLOC=`vxfsstat -a /usr/sap/PPO/JC37 | grep -i "alloc" | tr -s " " | cut -d " " -f2`; FREE=`vxfsstat -a /usr/sap/PPO/JC37 | grep -i "alloc" | tr -s " " | cut -d " " -f5`; echo $FREE*100/$ALLOC | bc; sleep 3; done
```

Alle RO-Mounts auflisten und deren RO-Zeitpunkt listen:

```sh
ARRAY=`grep -i "ro," /proc/mounts | cut -d " " -f2 | cut -d "/" -f2`; for entry in $ARRAY; do ls -arlt / | grep -i $entry; done
```

Stale-Mounts finden:

```sh
mount | sed -n "s/^.* on \(.*\) type nfs .*$/\1/p" | while read mount_point ; do ls $mount_point >& /dev/null || echo "stale $mount_point" ; done
```

oder

```sh
lsof -N
lsof: WARNING: can't stat() nfs file system /mnt/backup2
```

Output information may be incomplete.

```sh
ls /mnt/backup2
/bin/ls: /mnt/backup2: Stale NFS file handle
```

oder

```sh
for MP in $(mount -t nfs|cut -d " " -f3); do mountpoint $MP > /dev/null;done mountpoint: /usr/sap/export: Stale NFS file handle
```


oder in Kombination mit WATCH –> Alle Stales und RO´s zeigen:

```sh
watch 'echo "STALES:"; MPOINTS=$(mount | grep -i " nfs " | cut -d " " -f3); for i in $MPOINTS; do stat $i 2>&1 | grep -i "stale"; done; echo "READONLIES:"; mount | grep -i " nfs " | grep -i "ro,";'
```


RO-Mounts finden, umounten und neu mounten:

```sh
ROMOUNTS=$(grep -i "$(cat /proc/mounts | grep -i "ro," | awk '{print $2}')" /etc/fstab | egrep -iv ",ro|ro," | awk '{print $2}'); for entry in $ROMOUNTS; do echo "umounte $entry..."; umount -l $entry; done; mount -a
```

Manchmal kann es sein, dass Mounts auf ReadOnly gehen. Mit dieser Befehlsfolge wird auf RO-Mounts geprüft, in der /etc/fstab gegengeprüft, ob das so sein soll, danach ein „lazy umount“ gemacht und laut /etc/fstab neu gemounted. Im Idealfall spart man sich so einen Server-Reboot.
RO-Mounts per Schreibtest finden:

```sh
for fs in $( df -l | awk '$1 != "Filesystem" && $NF != "/dev" && $NF != "/dev/shm" && $NF != "/sys/fs/cgroup" { print $NF }' ); do echo "stale mount test" > $fs/stalemounttest.txt 2>/dev/null || echo "ERROR: $fs is readonly" && rm $fs/stalemounttest.txt 2>/dev/null; done
```

Eine andere Möglichkeit ist, in jedes Filesystem eine Testdatei zu schreiben. Es erfolgt eine Ausgabe, wenn dies fehl schlägt.

Partitionstabelle im laufenden Betrieb neu einlesen:

partprobe
Netzwerk

Den Internen IP-Traffic auflisten:

```sh
INTERNALIPS=$(ifconfig | grep -i "inet addr" | cut -d ":" -f2 | awk '{ print $1 }');for entry in $INTERNALIPS; do for eintrag in $INTERNALIPS; do netstat -tapen | awk '{ print $4 "  " $5 "  " $6}' | grep -i $entry | grep -i $eintrag; done; done
```


Testen, ob Port offen:
(„netcat“ muss installiert sein)
Beispiel 1: DNS über UPD

`nc -v -w 1 8.8.8.8 -u -z 53`

Beispiel 1: SSH über TCP

`nc -v -w 1 192.168.1.20 -z 22`

Standard Linux FW Ports auf Status testen:
(„netcat“ muss installiert sein) –> Wenn das Script nichts ausgibt, ist alles OK.

```sh
#!/bin/bash
#
# Checking firewall ports
if [ -f /tmp/fwlog.txt ] ; then rm -f /tmp/fwlog.txt; fi
if [ ! "$(nc -v -n -z -w1 139.1.165.95 80 | grep succeeded)" ]; then echo "ERROR: firewall port not configured - ffminstall 80/tcp" >> /tmp/fwlog.txt; fi
if [ ! "$(nc -v -n -z -w1 139.1.165.117 80 | grep succeeded)" ]; then echo "ERROR: firewall port not configured - spacewalk 80/tcp" >> /tmp/fwlog.txt; fi
if [ ! "$(nc -v -n -z -w1 139.1.165.95 111 | grep succeeded)" ]; then echo "ERROR: firewall port not configured - ffminstall 111/tcp" >> /tmp/fwlog.txt; fi
if [ ! "$(nc -v -n -z -w1 139.1.165.95 111 -u | grep succeeded)" ]; then echo "ERROR: firewall port not configured - ffminstall 111/udp" >> /tmp/fwlog.txt; fi
if [ ! "$(nc -v -n -z -w1 139.1.165.95 443 | grep succeeded)" ]; then echo "ERROR: firewall port not configured - ffminstall 443/tcp" >> /tmp/fwlog.txt; fi
if [ ! "$(nc -v -n -z -w1 139.1.165.117 443 | grep succeeded)" ]; then echo "ERROR: firewall port not configured - spacewalk 443/tcp" >> /tmp/fwlog.txt; fi
if [ ! "$(nc -v -n -z -w1 139.1.165.95 2049 | grep succeeded)" ]; then echo "ERROR: firewall port not configured - ffminstall 2049/tcp" >> /tmp/fwlog.txt; fi
if [ $(nc -v -n -z -w1 139.1.166.111 4505-4506 | grep succeeded | wc -l) -lt 2 ]; then echo "ERROR: firewall ports not configured - saltmaster 4505-4506/tcp" >> /tmp/fwlog.txt; fi
 
if [ -f /tmp/fwlog.txt ]; then
    echo "NETWORK / FIREWALL PROBLEM"
    echo "=========================="
    cat /tmp/fwlog.txt
    read -p "Press key to enter console..."
    exit 1
fi
```


Switchport vom Betriebssystem aus rausfinden:

```sh
tcpdump -nvi <interface> -s 1500 ether dst 01:00:0c:cc:cc:cc
 
        Device-ID (0x01), length: 28 bytes: 'franxdeffm08-18(SSI173708JM)'
        Address (0x02), length: 13 bytes: IPv4 (1) 172.22.0.49
        Port-ID (0x03), length: 15 bytes: 'Ethernet100/1/3'
        Capability (0x04), length: 4 bytes: (0x00000228): L2 Switch, IGMP snooping
        Version String (0x05), length: 66 bytes:
          Cisco Nexus Operating System (NX-OS) Software, Version 7.3(3)N1(1)
        Platform (0x06), length: 11 bytes: 'N5K-C5548UP'
        Native VLAN ID (0x0a), length: 2 bytes: 172
        AVVID trust bitmap (0x12), length: 1 byte: 0x00
        AVVID untrusted ports CoS (0x13), length: 1 byte: 0x00
        Duplex (0x0b), length: 1 byte: full
        System Name (0x14), length: 15 bytes: 'franxdeffm08-18'
        System Object ID (not decoded) (0x15), length: 14 bytes:
          0x0000:  060c 2b06 0104 0109 0c03 0103 883c
        Management Addresses (0x16), length: 13 bytes: IPv4 (1) 172.22.0.49
        Physical Location (0x17), length: 13 bytes: 0x00/snmplocation
```

## Dateien finden


In vielen Dateien/Unterverzeichnissen nach bestimmtem Text suchen:
Manchmal möchte man über mehrere Unterverzeichnisse in allen Dateien nach einem String suchen. Da hier oft auch grosse Binaries liegen, die 'grep' nur mit Mühe durchforsten kann, sollte man zuerst nach Textdateien filtern und nur diese dann durchsuchen. Genau das macht die folgende Befehlszeile, in der man nur die Variable FINDTEXT anpassen muss. Die Befehlszeile setzt man dann in dem Verzeichnis ab, in dem man abwärts suchen möchte.
ACHTUNG: Es erfolgt sehr lange keine Ausgabe. Eine Suche kann u.U. Stunden oder Tage dauern. Diese Variante ist aber immer noch schneller, als ein FIND über alle Dateitypen.

```sh
FINDTEXT="139.1.144."
FILETYPE="ASCII text"
declare -a
FILEARRAY=$(find . -type f -exec bash -c 'file {} | grep -i "ASCII text" | cut -d ":" -f1' \;)

for entry in $FILEARRAY
  do grep -Hin "$FINDTEXT" $entry
done

```

Paketmanagement
RPM

Datei libnetsnmp.so in RPMs suchen:

`ls /media/ISO/Packages/*.rpm | xargs rpm --query --filesbypkg --package | grep -i "libnetsnmp.so"`

Datei /bin/extract in RPMs suchen… (dauert sehr lange!):

`rpm -qf /bin/extract`

Umständlich (ich weiss nicht mehr genau warum…):

```sh
FILE2SEARCH="/bin/extract"
rpm -qali | while read -r line ; do { echo $line | grep -qi "^Name" && Name=$line ; }; { echo $line | grep -q "$FILE2SEARCH" && echo $Name ; } ; done
```

## APT/DEB


Pakete nach Ursprung/Origin suchen:

ll /var/lib/apt/lists/
...

```sh
BACKPORTS=$(grep ^Package /var/lib/apt/lists/*trusty-backports* | awk '{print $2}' | sort | uniq); for package in ${BACKPORTS[@]}; do dpkg -l | grep -i $package; done
```

Sucht alle Pakete, die aus dem Repository „BACKPORTS“ kommen.

**Aufräumen**

Dateien in /tmp löschen, die NICHT im Zugriff sind (non-destruktiv):
(Muss evtl. für Unterverzeichnisse wiederholt werden !)

```sh
cd /tmp; DELFILES=$(find . -type f); for entry in $DELFILES; do lsof | grep -i "/tmp" | grep -i "$entry" || echo "Datei $entry kann geloescht werden..." && rm -f ${entry} ; done
```

Alle Dateien im aktuellen Verzeichnis, die älter als 943 Tage sind, in ein anderes Verzeichnis (hier: /usr/sap/trans/log_old/) verschieben:

```sh
ARRAY=$(find . -mtime +943); for entry in $ARRAY; do mv $entry /usr/sap/trans/log_old/; done
```


Alle REAR Backups, die älter als 365 Tage sind und NICHT die aktuellen Backups beinhalten, löschen: (Fraport Hana Backup Server)

```
for server in $(ls -1 | awk -F "." {' print $1 }' | awk -F "_" '{ print $1 }' | uniq | egrep "^fra")
do echo "========="
echo "${server}"; echo "---------"
find /daten/rear_sdc/ -name "${server}.*" -mtime +365 -exec rm -rf {} \;
done
```


Im aktuellen Verzeichnis alle Dateien „älter 100 Tage“ als Einzelarchiv packen und die Dateien dann löschen:

```
logfile=/tmp/aelter100tage.txt; echo > $logfile
LIST=$(echo p*)
for entry in $LIST
  do find . -mtime +100 -name $entry >> $logfile
done

ARCHIVLIST=$(cat $logfile | cut -d "/" -f2)
for entry in $ARCHIVLIST
  do tar --remove-files -czf $entry.tgz $entry
done
```

Dieser Befehl kann nützlich sein, wenn auf dem Dateisystem nicht mehr genug Platz ist, um alle Dateien „älter 100 Tage“ in EIN Archiv zu packen. Durch die Option “–remove-files“ wird jede gepackte Datei gleich gelöscht.
Hat man noch genug Platz und möchte alle Dateien „älter 100 Tage“ in EIN Archiv packen, dann reicht z.B ein

```sh
find . -mtime +100 -type f > neuefiles.txt; tar -c -T neuefiles.txt -v -z -p -f sicherung.tgz
```

Dies ist besser als ein

```sh
tar cfvzp sicherung.tgz $( find / -mtime +100 -type f)
```

da im letzten Befehl Dateilisten GRÖSSER als 32768 Zeichen ignoriert werden würden.

Suche im aktuellen Verzeichnis nach Unterverzeichnis mit dem grössten Datenvolumen und wechsle dorthin:

```sh
LIST=($(du -x . --max-depth=1 | sort -nr))
if [[ $(echo ${LIST[2]}*2 | bc) -lt ${LIST[0]} ]]
then while [ ${#LIST[@]} -gt 2 ]
do if [[ "${LIST[3]}" =~ "^\.$" ]]
then DIR2CHANGE=$(echo ${LIST[1]} | cut -d "/" -f2)
else DIR2CHANGE=$(echo ${LIST[3]} | cut -d "/" -f2)
fi
cd "$DIR2CHANGE"; LIST=($(du -x $1 --max-depth=1 | sort -nr))
echo "Wechsle nach $DIR2CHANGE"
done
fi
```

**Tipp**: Natürlich landet man zuletzt nicht im Verzeichnis mit den grössten Dateien. Dieses findet man durch schrittweises „cd ..“ und „du . -h“ nach oben.

**ANMERKUNG**: Manchmal macht dieser Befehl noch Probleme, wenn man ihn in / ausführt. Sobald man in ein Unterverzeichnis wechselt, funktioniert er dann. Andernfalls den untenstehenden Befehl verwenden, dieser funktioniert auch unter /.

```sh
CURRENTDIR=($(pwd));if [ "$CURRENTDIR" == "/" ]; then LIST=($(du . --max-depth=1 --exclude=proc --exclude=dev --exclude=run | sort -nr)); else LIST=($(du . --max-depth=1 | sort -nr)); fi; if [[ $(echo ${LIST[2]}*2 | bc) -lt ${LIST[0]} ]]; then while [ ${#LIST[@]} -gt 2 ]; do if [[ "${LIST[3]}" =~ "^\.$" ]]; then DIR2CHANGE=$(echo ${LIST[1]} | cut -d "/" -f2); else DIR2CHANGE=$(echo ${LIST[3]} | cut -d "/" -f2); fi; cd "$DIR2CHANGE"; LIST=($(du $1 --max-depth=1 | sort -nr)); echo "Wechsle nach $DIR2CHANGE"; done; fi
```

Wenn man im aktuellen Filesystem bleiben möchte, benutzt man:

```sh
CURRENTDIR=($(pwd));if [ "$CURRENTDIR" == "/" ]; then LIST=($(du . -x --max-depth=1 --exclude=proc --exclude=dev --exclude=run | sort -nr)); else LIST=($(du . -x --max-depth=1 | sort -nr)); fi; if [[ $(echo ${LIST[2]}*2 | bc) -lt ${LIST[0]} ]]; then while [ ${#LIST[@]} -gt 2 ]; do if [[ "${LIST[3]}" =~ "^\.$" ]]; then DIR2CHANGE=$(echo ${LIST[1]} | cut -d "/" -f2); else DIR2CHANGE=$(echo ${LIST[3]} | cut -d "/" -f2); fi; cd "$DIR2CHANGE"; LIST=($(du $1 -x --max-depth=1 | sort -nr)); echo "Wechsle nach $DIR2CHANGE"; done; fi
```

Liste im aktuellen Verzeichnis alle Dateien der Grösse nach sortiert auf:

`ls -l --block-size=M | sort -n -k 5`

Alle LOG Files 3 x durchrotieren: (schafft Platz auf /var/log, Fehlermeldungen können ignoriert werden.)

```
for i in {1..3}; do logrotate -f /etc/logrotate.conf; done; for rotate in $(ls -1 /etc/logrotate.d/); do for i in {1..3}; do logrotate -d /etc/logrotate.d/${rotate}; done; done
```

# Storage

Seriennummern aller Platten auslesen (auf dem Salt-Master):

```sh
for drive in $(lsblk -dn | awk '{print $1}'); do echo -e "-----------------\n"; udevadm info --query=all --name=/dev/${drive} | grep -Ei "ID_SERIAL|DEVNAME"; done
```

Lun-Rescan über alle SCSI-Hosts:

```sh
rescan-scsi-bus.sh -a --forcerescan
lsscsi -s
```

Das Device sollte nun die neue Grösse zeigen. Siehe: http://manpages.ubuntu.com/manpages/wily/man8/rescan-scsi-bus.8.html

Im Kernel-Ringbuffer nach Fehlern im SAN schauen und die betroffenen Devices suchen:

```sh
devicetest=$(dmesg | grep -i "error" | grep -i "device-mapper" | grep -i "multipath" | cut -d " " -f3 | sed 's/\.*[:]/, /g' | sed 's/[ ,]*$//'); ls -arlt /dev/mapper/ | grep "$TEST" || echo "Device $devicetest nicht gefunden"
```
## Multipathing

Prüfen, ob alle in der /etc/multipath.conf gesetzen Pfade auch vorhanden und erreichbar sind:

```
WWIDS=$(grep -i "wwid" /etc/multipath.conf | awk '{print $2}' | sort | uniq | grep -v "wwid"); for entry in $WWIDS; do multipath -ll | grep -i $entry || echo "$entry nicht gefunden"
done
```

Herausfinden, welche LUN-IDs zu einer Diskgruppe gehören:

(Danke an Ulf, siehe auch http://oswiki-linux.de.t-internal.com/doku.php?id=conti:lun_online_vergroessern_-_mit_veritas_vxvm)

```sh
for i in $(vxprint -g dgx2pdata|grep ^dm|awk '{print $3}');do vxdmpadm list dmpnode dmpnodename=$i|grep -iE "dmpdev|lun-sno";done
```

Anzeigen von allen an einen Server sichtbaren LUNs mit ihren wichtigsten Informationen.
Beispiel Format der Ausgabe:

```
mpdev-10000 360060e80164f390000014f390000000f size 50G
mpdev-10001 360060e80164f390000014f3900000010 size 50G
mpdev-10002 360060e80164f390000014f3900000011 size 50G
mpdev-10003 360060e80164f390000014f3900000012 size 50G
mpdev-10004 360060e80164f390000014f3900000013 size 50G
mpdev-10005 360060e80164f390000014f3900000014 size 50G
mpdev-10006 360060e80164f390000014f3900000015 size 250G
mpdev-10007 360060e80164f390000014f3900000016 size 250G
mpdev-10008 360060e80164f390000014f3900000017 size 250G
mpdev-10009 360060e80164f390000014f3900000018 size 250G
mpdev-10010 360060e80164f390000014f3900000019 size 250G
```

Befehlszeile:

```sh
for i in $(multipath -ll|grep -i ^m|awk '{print $2}'|tr -d "("|tr -d ")")
do export S=$(multipath -ll $i|grep ^size|awk '{print $1}'|cut -d "=" -f2)
export N=$(multipath -ll ${i}|head -1|awk '{print $1}')
echo "$N $i size ${S}"
done|sort
```
## VMs

WWNs in einer VM herausfinden:
Im Folgenden wollen wir die WWN einer virtuellen Platte mit 2 TB herausfinden.
# pvs

```
  PV         VG           Fmt  Attr PSize   PFree
  /dev/sda3  rootvg       lvm2 a-    35.83G   4.83G
  /dev/sdb   sapEFIvg     lvm2 a-   358.00G   4.09G
  /dev/sdc   rootvg       lvm2 a-    20.00G      0
  /dev/sdd   oralogEFIvg  lvm2 a-    16.00G 396.00M
  /dev/sde   backupvg     lvm2 a-     1.95T      0
  /dev/sdf   oradataEFIvg lvm2 a-     1.95T  88.00M
  /dev/sdg   oraarchtmpvg lvm2 a-    50.00G  96.00M
  /dev/sdh   backup2vg    lvm2 a-     1.95T      0
  /dev/sdi1  backupvg     lvm2 a-   199.99G 392.00M
  /dev/sdj1  backup2vg    lvm2 a-   199.99G  52.38G
```


## Monitoring


Kleine Serverüberwachung per PING:
Man legt eine Textdatei im CSV-Format mit den Spalten KUNDE,IP-ADRESSE,SERVERNAME an –> EmiTech,192.168.23.12,FRASV000111
Diese speichert man als „serverliste.txt“.
Mit folgendem Befehl überwacht man dann alle Server in der Liste periodisch alle 5 Sekunden:

```
until [ "1" == "2" ]; do for i in $(cat serverliste.txt); do CUSTOMER=$(echo $i|cut -d "," -f1); ADDRESS=$(echo $i|cut -d "," -f2); SERVER=$(echo $i|cut -d "," -f3); echo "Pinge $CUSTOMER - $SERVER"; ping -c 1 $ADDRESS | grep -i "ttl" || echo "ACHTUNG" ; done; sleep 5; clear; done
```

## CUPS / Drucker


VESI-IT / Procxys:
Im Auftrag steht meistens:

Bitte folgende Drucker neu einrichten, bzw. IP-Adressen ändern.

```
19BAU102        10.71.46.51
19BOC101        10.71.17.51
19BOD101        10.71.21.51
19DRA101        10.71.18.51
19GEN102        10.71.13.51
19GOT101        10.71.14.51
19HAL101        10.71.12.52
19HAL103        10.71.12.51
19HAL211        10.71.4.51
19HAM101        10.71.16.51
19HER101        10.71.11.52
19HER201        10.71.11.51
19KAK101        10.71.15.51
19MAG137        10.71.0.51
19NOR101        10.71.22.51
19NOR102        10.71.22.52
```

Zuerst speichert man sich diese Liste in eine Datei. Z.B. 'INC000002463421_printerlist.txt'

Danach prüft man, ob die einzurichtenden Drucker schon zu erreichen sind:

```sh
for ip in $(cat INC000002463421_printerlist.txt | awk -F " " '{print $2}'); do ping -c 1 -W 5 $ip > /dev/null ; if [ $? != 0 ]; then echo "$ip nicht erreichbar"; else echo "$ip OK"; fi; done
```

```
10.71.46.51 OK
10.71.17.51 OK
10.71.21.51 OK
10.71.18.51 OK
10.71.13.51 OK
10.71.14.51 OK
10.71.12.52 OK
10.71.12.51 OK
10.71.4.51 OK
10.71.16.51 OK
10.71.11.52 nicht erreichbar
10.71.11.51 OK
10.71.15.51 OK
10.71.0.51 OK
10.71.22.51 OK
10.71.22.52 OK
```


Negative Ergebnisse meldet man dann an Herrn Volland (VESI-IT)

Als Nächstes prüft man, welche Drucker bereits existieren und wie sie konfiguriert sind:

```sh
for printer in $(cat INC000002463421_printerlist.txt | awk -F " " '{print $1}'); do awk -vRS="</Printer>" -vFS="<Printer" '{print $2}' /etc/cups/printers.conf | grep -i -A12 "$printer" > /dev/null; if [ $? == 0 ]; then echo "$printer: $(grep -i -A12 "$printer" /etc/cups/printers.conf | grep -i "DeviceURI")"; else echo "$printer noch nicht eingerichtet"; fi;  done
```

```
19BAU102: DeviceURI lpd://10.71.46.31/PASSTHRU
19BOC101: DeviceURI lpd://10.71.17.31/PASSTHRU
19BOD101: DeviceURI lpd://10.71.21.31/PASSTHRU
19DRA101: DeviceURI lpd://10.71.18.31/PASSTHRU
19GEN102: DeviceURI lpd://10.71.13.35/PASSTHRU
19GOT101: DeviceURI lpd://10.71.14.31/passthru
19HAL101: DeviceURI lpd://10.71.12.35/PASSTHRU
19HAL103: DeviceURI lpd://10.71.12.36/PASSTHRU
19HAL211: DeviceURI lpd://10.71.4.31/PASSTHRU
19HAM101: DeviceURI lpd://10.71.16.33/PASSTHRU
19HER101: DeviceURI lpd://10.71.11.36/PASSTHRU
19HER201 noch nicht eingerichtet
19KAK101: DeviceURI lpd://10.71.15.32/PASSTHRU
19MAG137: DeviceURI lpd://10.71.0.31/PASSTHRU
19NOR101: DeviceURI lpd://10.71.22.32/PASSTHRU
19NOR102 noch nicht eingerichtet
```

**Problemlösungen**

Bash Befehle „tracen“…

```sh
set -x
bash -x '/etc/init.d/network' start bond0 2>&1 | more
```

–> liefert die Befehlskette zum Init-Script „network start“ für bond0.
Danach wieder

`set +x`

**Hohe CPU-Auslastung durch 'kswapd':**

`echo 3 > /proc/sys/vm/drop_caches`

Prüfen, ob GRUB installiert ist:

```sh
xxd -l 512 /dev/sda | grep -i grub
xxd -l 512 /dev/cciss/c0d0p2 | grep -i grub    (auf HP Hardware)
```

System-Reboot funktioniert nicht:

```sh
echo 10 > /proc/sys/kernel/panic
echo 1 > /proc/sys/kernel/sysrq
echo s > /proc/sysrq-trigger
sleep 5
echo s > /proc/sysrq-trigger
sleep 1
echo b > /proc/sysrq-trigger
```

„umount“ klappt nicht, device busy…

```sh
$ umount /dev/cdrom
umount: /cdrom: device is busy
$ kill -9 $(lsof -t /dev/cdrom)
$ umount /dev/cdrom
$ eject
```

NFS-Mount per SSH Tunnel

Beispiel: NFS-Server exportiert: 139.1.165.95/srv/ISO-FILES
SSH ist erlaubt, NFS aber durch die Firewall gesperrt.

Zuerst muss man in der /etc/exports auf dem NFS-Server die Zeile für den zu erreichenden Export duplizieren und die hinzugefügte Zeile wie folgt anpassen:

/srv/ISO-FILES  *(ro,root_squash,sync,no_subtree_check)
/srv/ISO-FILES  127.0.0.1(insecure,ro,sync,no_subtree_check)
Danach reloaded man den NFS-Server neu (NICHT NEU STARTEN ! –> stale mounts !!!) Nun noch ein

# exportfs -ra

Auf dem CLIENT führt man nun Folgendes aus:

```sh
ssh username@139.1.165.95 'grep -i "MOUNTD_PORT=" /etc/sysconfig/nfs'
MOUNTD_PORT="35508"

ssh -Nv username@139.1.165.95 -L 2222:localhost:2049 -f sleep 600m
ssh -Nv username@139.1.165.95 -L 3333:localhost:35508 -f sleep 600m
netstat -tulpen | egrep "2222|3333"
sudo mount -v -t nfs -o port=2222,mountport=3333,tcp localhost:/srv/ISO-FILES /mnt
```

Nun ist der Export auf dem Client unter /mnt erreichbar.

## Lustiges

TREE-Kommando für AIX / HPUX nachbilden:

```sh
ls -R | grep ":$" | sed -e 's/:$//' -e 's/[^-][^\/]*\//--/g' -e 's/^/   /' -e 's/-/|/'
```

## Userverwaltung


Speziellen User anlegen:

```sh
groupadd -g 1113 sstojak && useradd -u 11013 -g 1113 -G 1100 -c "Silvia Stojak" -p '$2y$10$18tnDNCUhAkgSs30Hu.r0OaNkRoLxL6.JmKsABiXu3f93qgN13FRu' -s /bin/bash -d /home/sstojak -m sstojak
```

Die Werte müssen entsprechend angepasst werden (analog zur userlist.cfg von Franks Userskript):

```sh
-g ⇒ ID der Gruppe
-u ⇒ UID des Users
-G ⇒ GIDs der zusätzlichen Gruppen
-c ⇒ Anzeigename des Benutzers
-p ⇒ Passwort-Hash des Users (optional)
-s ⇒ Standard-Shell
-d ⇒ Home-Directory
-m ⇒ Home-Directory anlegen
```

Alle User ausser ROOT und dem Eigenen zwangsabmelden:

`USERS=$(who | cut -d " " -f1 | egrep -i -v "bglaessn|root"); for user in $USERS; do skill -KILL -v $user; done`


Usern in einer definierten UID-Range ein neues Passwort zuweisen und zwingen, Passwort beim ersten Login zu ändern (FUNKTIONIERT NUR ALS ROOT):
(im Beispiel UID 31001 - 31013 mit einem Passwort 'A12345678!)

```sh
USERLIST=$(for i in {31001..31013}; do grep -i "$i" /etc/passwd | awk -F ":" '{ print $1 }' ; done); for user in ${USERLIST}; do echo -e 'A12345678!\nA12345678!' | passwd ${user} && chage -d 0 ${user}; done
```

Als root zu einem System-User wechseln, der */bin/nologin* als Shell zugewiesen hat:

`su -s /bin/bash [username]`

Saltstack
http://oswiki-linux.de.t-internal.com/doku.php?id=linuxteam-intern:projektdoku:osintern:saltstack_server#nuetzliche_befehle

Alle 'acceppted keys' anzeigen:

`salt-key -L | awk '/Accepted Keys:/{f=1;next} /Denied Keys:/{f=0} f'`

Nur für akzeptierte Minions einen SALT-Befehl ausführen lassen:
Grundlage ist eine serverliste.txt mit allen zu bearbeitenden Servern untereinander.

```sh
#!/bin/bash
SERVERS=( "$(cat ./serverliste.txt | tr [:upper:] [:lower:])" )
MINIONS=( "$(salt-key -L | awk '/Accepted Keys:/{f=1;next} /Denied Keys:/{f=0} f' | tr [:upper:] [:lower:])" )

for server in ${SERVERS[@]}; do
        found="false"
        for minion in ${MINIONS[@]}; do
                if [ "${server}" == "${minion}" ]; then
                        found="true"
                        echo "Server ${server} gefunden."
                        # Hier nun einen SALT-Befehl ausführen:
                        salt "${server}" cmd.run "awk -F\: '{ print \$1\",\"\$3\",\"\$4\",\"\$5\",\"\$6 }' /etc/passwd | sort -k 1 | tr -s \",\""
                        break
                fi
        done
        if [ "${found}" == "false" ]; then
                echo "*** ERROR: Server ${server} nicht gefunden. ***"
        fi
done
```

Alle Scripte mit dem String 'check' in 'file_root' anzeigen lassen:

`root@fravm009025:# salt-run fileserver.file_list | grep "check"`

Sein Passwort per Salt auf einem Minion ändern:

`echo "linuxpassword" | passwd --stdin linuxuser`

## OpenStack


Brach liegenden Storage zusammen rechnen:

```sh
LOSTSTORAGE=$(cinder list --all-tenants | grep "available" | awk -F "|" '{print $7}'); amount=0; for count in ${LOSTSTORAGE[@]}; do amount=$(echo "$amount + $count"|bc); done; echo"
```

Allokierter, nicht benutzter Storage: $amount GB"
Allokierter, nicht benutzter Storage: 3380 GB

Volume eines Tenants einem anderen Tenant zuweisen/umziehen:

```sh
source openrc Tenant_A Tenant_A
cinder transfer-create <volume_id>

source openrc Tenant_B Tenant_B
cinder transfer-accept <transfer_id> <auth_key>
```

Kurzstatus eines JUJU-Charms anzeigen lassen: (hier beispielsweise an #RabbitMQ)

`juju status --color rabbitmq |& sed -n '/^Unit/,/^Machine/p'`

```
Unit            Workload  Agent  Machine    Public address  Ports               Message
rabbitmq/6      active    idle   6/lxd/142  172.30.14.30    5672/tcp,15672/tcp  Unit is ready and clustered
  filebeat/636  active    idle              172.30.14.30                        Filebeat ready.
  nrpe/404      active    idle              172.30.14.30    5666/tcp            ready
rabbitmq/7      active    idle   8/lxd/135  172.30.14.42    5672/tcp,15672/tcp  Unit is ready and clustered
  filebeat/638  active    idle              172.30.14.42                        Filebeat ready.
  nrpe/406      active    idle              172.30.14.42    5666/tcp            ready
rabbitmq/8*     active    idle   9/lxd/112  172.30.14.108   5672/tcp,15672/tcp  Unit is ready and clustered
  filebeat/637  active    idle              172.30.14.108                       Filebeat ready.
  nrpe/405      active    idle              172.30.14.108   5666/tcp            ready
```

Alle openrc auf allen Maschinen anzeigen lassen: [adminpw] muss hier mit dem aktuellen Passwort ersetzt werden.

```
for machine in $(juju status |& sed -n '/^Machine/,//p' | tail -n +2 | sed 's/^$//g' | awk '{ print $1 }')
do echo "=== Machine: ${machine} ==="
uju run --machine ${machine} 'sudo find / -name openstackrc* 2> /dev/null
sudo grep -rI "Exp3l1mus!" /home/* /root/* 2> /dev/null'
done
```

Alle Admin Passworte ändern: [adminpw] muss hier mit dem aktuellen Passwort ersetzt werden.
Admin-Accounts anzeigen lassen:

```
for dom in $(openstack domain list | grep "True" | awk -F "|" '{ print $3 }' | tr -d " ")
do echo "DOMAIN: ${dom}"
openstack user list --domain ${dom}| grep -i "admin"
done
```

Danach zu jedem Account jede passenden openstackrc sourcen und als dieser aktive Admin folgenden Befehl absetzen:

`openstack user password set --password 'TollesNeuesPW' --original-password '[adminpw]'`

**Service User Passwort ändern:**

`openstack user set --password-prompt glance`

**Alle Juju Leader anzeigen lassen:**

```sh
for app in $(juju status --color |& sed -n '/^App/,/^Unit/p' | head -n -2 | tail -n +2 | awk '{ print $1 }'); do juju status ${app} | egrep "${app}/[[:digit:]]\*" | awk -F "*" '{ print $1 }' | tr -d " "; done
```

**Alle Clusterstatus anzeigen lassen:**

```sh
MACHINES=$(juju status --color |& sed -n '/^Machine/,/^$/p' | head -n -1 | tail -n +2 | awk '{ print $1 }'); for machine in ${MACHINES[@]}; do if [ "$(juju ssh ${machine} 'which crm_mon' 2> /dev/null)" ]; then echo "============"; echo "${machine}"; echo "-----------"; juju ssh ${machine} 'sudo crm_mon -1 --failcounts 2> /dev/null'; fi; done
```


Status der darunter liegenden Physiken für z.B. 'ubuntu-api' anzeigen lassen:

```sh
for server in $(juju status --color ubuntu-api |& sed -n '/^Machine/,/^$/p' | head -n -1 | tail -n +2 | awk '{ print $1 }'); do juju ssh ${server} 'hostname; free -h'; done
```

Alle bereits vergebenen Hitnet Routing Targets anzeigen lassen:

```sh
TOKEN="$(openstack token issue | grep '| id' | awk '{print $4}')"; TARGETS=$(curl -X GET "http://172.30.14.2:8082/route-targets" -H "Accept: application/json" -H "X-Auth-Token: ${TOKEN}" | python -m json.tool | jq -r '.[]|.[]|.href'); for target in ${TARGETS[@]}; do curl -s -X GET "${target}" -H "Accept: application/json" -H "X-Auth-Token: ${TOKEN}" | python -m json.tool | grep '"fq_name":' -A 2 | grep "target" | grep ":2"; done
```

Alle auf einem Nova Compute Knoten laufende VMs anzeigen lassen:

```sh
NOVAHOST="cldsv00006"; for instance in $(nova hypervisor-servers ${NOVAHOST} | awk -F "|" '{ print $2}' | tr -d " " | head -n -1 | tail -n +4); do openstack server show ${instance} | egrep "instance_name|power_state|vm_state| id | name |project_id"; echo "-------------------------------" ; done
```
## Systemd

Systemd-Service wurde gelöscht, ist aber noch zu sehen:

```sh
# systemctl status hp-health.service
● hp-health.service
   Loaded: not-found (Reason: No such file or directory)
   Active: failed (Result: timeout) since Wed 2019-04-03 17:11:46 CEST; 5min ago

Apr 03 17:10:46 he113150 hpasmlited[3036]: check_ilo2: BMC Returned Error:  ccode  0x0,  Req. Len:  15, Resp. Len:  21
Apr 03 17:10:46 he113150 hpasmlited[3036]: The Integrated Lights-Out Management Processor is not responding!
Apr 03 17:10:46 he113150 hpasmlited[3036]: Sleeping 30 seconds and will retry . ..
Apr 03 17:11:16 he113150 hpasmlited[3036]: check_ilo2: BMC Returned Error:  ccode  0x0,  Req. Len:  15, Resp. Len:  21
Apr 03 17:11:16 he113150 hpasmlited[3036]: The Integrated Lights-Out Management Processor is not responding!
Apr 03 17:11:16 he113150 hpasmlited[3036]: Sleeping 30 seconds and will retry . ..
Apr 03 17:11:46 he113150 systemd[1]: hp-health.service start operation timed out. Terminating.
Apr 03 17:11:46 he113150 systemd[1]: Failed to start HP System Health Monitor.
Apr 03 17:11:46 he113150 systemd[1]: Unit hp-health.service entered failed state.
Apr 03 17:11:46 he113150 systemd[1]: hp-health.service failed.
# systemctl reset-failed
# systemctl status hp-health.service
Unit hp-health.service could not be found.
```
