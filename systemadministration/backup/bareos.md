# Restore mit Bareos

**Status des Backup Servers**

`* status dir`

**Status eines Clients (auch check ob erreichbar)**

`* status client=srv3.local`

Liste der Jobs und Backups eines Clients (ggf. mit Beschränkung der letzten Tage)

```sh
* list jobs client=srv3.local
* list backups client=srv3.local* list backups client=floh.rz.uni-jena.de days=4
```

**Testen, wie viel ein aktuelles Backup benötigen würde:**

`* estimate job=floh.rz`

Anschauen der Reconfiguration in Bareos:

* show client=floh.rz.uni-jena.de
* show job=floh.rz * show jobs * show jobdefs=ServerDefaultJob3W
* show fileset=``default``-win-fs1 show schedule=3WS`

Restore

`* restore`
`* restore client=floh.rz.uni-jena.de`

Für die nächste Frage sind vermutlich meist nur die Punkte **2**, **5** und **6** relevant.

```sh
`To select the JobIds, you have the following choices:`<br><br>     
`1`: List last` `20` `Jobs run`<br><br>     
`2`: List Jobs where a given File is saved`<br><br>     
`3`: Enter list of comma separated JobIds to select`<br><br>     
`4`: Enter SQL list command`<br><br>     
`5`: Select the most recent backup` `for` `a client`<br><br>     
`6`: Select backup` `for` `a client before a specified time`<br><br>     
`7`: Enter a list of files to restore`<br><br>     
`8`: Enter a list of files to restore before a specified time`<br><br>     
`9`: Find the JobIds of the most recent backup` `for` `a client`<br><br>    
`10`: Find the JobIds` `for` `a backup` `for` `a client before a specified time
`11: Enter a list of directories to restore` `for` `found JobIds
`12: Select full restore to a specified Job date`<br><br>    
`13: Cancel`
```

- **2**: um eine Datei (ohne Pfad) zu finden in den unterschiedlichen Backup Versionen
- **3**: wenn man die JobIDs kennt
- **5**: Wiederherstellen der letzten Version
- **6**: Wiederherstellen zu einem bestimmten Zeitpunkt
- Danach kommt man in einen virtuellen Verzeichnisbaum um die Dateien/Verzeichnisse zu markieren (Befehl **mark**). Die Auswahl wird dann mit **done** abgeschlossen.

Nun erfolgt nochmal eine Kontrolle. Normalerweise wird nach /tmp/bareos-restore zurückgesichert. Wenn dies bestätigt wird, startet der Restore. Mit **status dir**

Nun erfolgt nochmal eine Kontrolle. Normalerweise wird nach /tmp/bareos-restore zurückgesichert. Wenn dies bestätigt wird, startet der Restore. Mit **status dir**
##### Beispiel 1

|`* restore client=floh.rz.uni-jena.de`
`First you select one or more JobIds that contain files`
`to be restored. You will be presented several methods`
`of specifying the JobIds. Then you will be allowed to`
`select which files from those JobIds are to be restored.`
`To select the JobIds, you have the following choices:`
`1`: List last` `20` `Jobs run`
`2: List Jobs where a given File is saved`
`3`: Enter list of comma separated JobIds to select`
`4: Enter SQL list command`
`5`: Select the most recent backup` `for` `a client`
`6`: Select backup` `for` `a client before a specified time`
`7`: Enter a list of files to restore`
`8: Enter a list of files to restore before a specified time`
`9: Find the JobIds of the most recent backup` `for` `a client`  
`10: Find the JobIds` `for` `a backup` `for` `a client before a specified time`
`11`: Enter a list of directories to restore` `for` `found JobIds`
`12`: Select full restore to a specified Job date`
`13`: Cancel`<br><br>`Select item:  (``1``-``13``):` `2`
`Enter Filename (no path):blabla.txt`
`+-------+-----------------------+---------------------+---------+-----------+----------+----------------+`| jobid \| name                  \| starttime           \| jobtype \| jobstatus \| jobfiles \| jobbytes       \|`<br><br>`+-------+-----------------------+---------------------+---------+-----------+----------+----------------+`<br><br>`\|` `9419`  `\| /home/floh/blabla.txt \|` `2016``-``01``-``31` `12``:``00``:``06` `\| B       \| W         \|` `723222`   `\|` `25847959779`    `\|`
`|` `8095`  `| /home/floh/blabla.txt \|` `2016``-``01``-``14` `09``:``02``:``21` `\| B       | W         |` `720177`   `\|` `26887045352`    `\|`
`|` `7187`  `| /home/floh/blabla.txt \|` `2016``-``01``-``07` `16``:``02``:``12` `\| B       \| T         |` `1952`     `\|` `206679936`      `\|`
`+-------+-----------------------+---------------------+---------+-----------+----------+----------------+`
`To select the JobIds, you have the following choices:`
`1`: List last` `20` `Jobs run`
`2`: List Jobs where a given File is saved
`3`: Enter list of comma separated JobIds to select`
`4`: Enter SQL list command` 
`5`: Select the most recent backup` `for` `a client`
`6: Select backup` `for` `a client before a specified time`
`7`: Enter a list of files to restore`
`8: Enter a list of files to restore before a specified time`
`9`: Find the JobIds of the most recent backup` `for` `a client`
`10: Find the JobIds` `for` `a backup` `for` `a client before a specified time`
`11``: Enter a list of directories to restore` `for` `found JobIds`
`12``: Select full restore to a specified Job date`
`13``: Cancel`

`Select item:  (``1``-``13``):` `3`
`Enter JobId(s), comma separated, to restore:` `8095`
`You have selected the following JobId:` `8095`
`Building directory tree` `for` `JobId(s)` `8095` `+++++++++++++++++++++++++++++++++++++++++++++`
`658``,``342` `files inserted into the tree.`
`You are now entering file selection mode where you add (mark) and`
`remove (unmark) files to be restored. No files are initially added, unless`
`you used the` `"all"` `keyword on the command line.`
`Enter` `"done"` `to leave` `this` `mode.`
`cwd is: /`
`$ ls`
`etc/`
`home/`
`root/`
`$ mark /home/floh/blabla.txt`
`1` `file marked.`
`$ done`
`Bootstrap records written to /var/lib/bareos/bareos1-dir.restore.``13``.bsr`
`The job will require the following`
`Volume(s)                 Storage(s)                SD Device(s)`
`===========================================================================`
`VFBP-``21149`                `VirtualStorageVFB         VirtualVFB`
`Volumes marked with` `"*"` `are online.`
`1` `file selected to be restored.``Automatically selected Job: RestoreDummy`<br><br>
`Run Restore job`
`JobName:         RestoreDummy`
`Bootstrap:       /var/lib/bareos/bareos1-dir.restore.``13``.bsr``Where:           /tmp/bareos-restores`
`Replace:         Always`
`FileSet:`         `default``-linux-fs0`
`Backup Client:   floh.rz.uni-jena.de`
`Restore Client:  floh.rz.uni-jena.de`
`Format:          Native`
`Storage:         VirtualStorageVFB`
`When:`            `2016``-``02``-``11` `15``:``28``:``11`
`Catalog:         MyCatalog`
`Priority:`        `10`
`Plugin Options:  *None*`
`OK to run? (yes/mod/no): yes`
`Job queued. JobId=``10670`

Wenn der Restorepfad /tmp/bareos-restores nicht passt, insbesondere für WIndows muss dieser angepasst weden z.B.: '**D:/bareos-restores/**'
