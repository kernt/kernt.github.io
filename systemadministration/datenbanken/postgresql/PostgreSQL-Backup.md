---
tags:
  - postgresql
  - datenbanken
  - backup
---
Die Kommandozeilen-Tools pg_dump und pg_restore dienen zum Exportieren bzw. Importieren einer [PostgreSQL](https://www.ionos.at/digitalguide/server/knowhow/postgresql/ "PostgreSQL")-Datenbank. Zum Einsatz kommen sie beim Erstellen eines PostgreSQL-Backups sowie beim Migrieren von PostgreSQL-Datenbanken zwischen Servern.

## Worum handelt es sich bei einem PostgreSQL-Dump?

Unter einem PostgreSQL-Dump versteht man die **Ausgabe-Datei, die beim Export einer PostgreSQL-Datenbank** erzeugt wird. PostgreSQL ist ein ausgefeiltes Datenbank-Management-System, das Daten in optimierten Datenstrukturen speichert. Das Extrahieren strukturierter Daten aus einer PostgreSQL-Datenbank erfordert daher ein spezielles Verfahren.

> Hinweis: [Was ist ein Backup?](https://www.ionos.at/digitalguide/server/sicherheit/was-ist-ein-backup/ "Was ist ein Backup?") Diese grundsätzliche Frage beantworten wir in einem anderen Ratgeber.

Ähnlich wie beim [MySQL-Backup via MySQL-Dump] erzeugt das pg_dump-Tool eine **Textdatei mit SQL-Anweisungen**. Das Ausführen der Anweisungen stellt die Datenbank zum Zeitpunkt des Dumps wieder her. In den Worten der offiziellen PostgreSQL-Dokumentation:

> Zitat: „The idea behind this dump method is to generate a text file with SQL commands that, when fed back to the server, will recreate the database in the same state as it was at the time of the dump.“

„Die Idee hinter dieser Dump-Methode besteht darin, eine Textdatei mit SQL-Anweisungen zu generieren, die, wenn sie am Server wieder eingespielt wird, die Datenbank zu dem Zustand wiederherstellt, an dem sie sich zum Zeitpunkt des Dumps befand.“

Es ist wichtig, zu verstehen, was mit „PostgreSQL-Datenbank“ gemeint ist. Der Begriff wird oft verwechselt mit dem PostgreSQL-Server. In der Tat ist es nicht ungewöhnlich, dass **ein einzelner PostgreSQL-Server mehrere Datenbanken enthält**. Hier die Hierarchie der Objekte einer PostgreSQL-Installation im Überblick:

|PostgreSQL-Objekt|Enthält|
|---|---|
|Server|Datenbanken|
|Datenbank|Tabellen|
|Tabelle|Datensätze|
|Datensatz|Felder|

## Wie funktionieren pg_dump und pg_restore?

Die Kommandozeilen-Tools pg_dump und pg_restore werden für gewöhnlich als Teil der PostgreSQL-Client-Anwendungen mitsamt der PostgreSQL-Kommandozeilen-Schnittstelle psql installiert. Die Tools **folgen der Unix-Philosophie und nutzen Textströme für Ein- und Ausgabe**. Dies ermöglicht, sie durch sogenannte Pipes mit anderen Programmen zu verknüpfen. Ferner lassen sich Ein- und Ausgabe mittels Umleitungen aus Dateien lesen bzw. an Dateien ausgeben.

Wir zeigen das **generelle Muster bei der Nutzung von pg_dump** zum Erzeugen eines PostgreSQL-Dumps:

```sh
pg_dump dbname > db.dump
```

Wir nutzen eine Umleitung der Ausgabe ('>') an eine Datei. Der dabei **erzeugte PostgreSQL-Dump enthält SQL-Anweisungen**. Diese lassen sich mit dem psql-Tool ausführen. Wir zeigen das generelle Muster bei der Nutzung von psql zum Einlesen eines PostgreSQL-Dumps:

```sh
psql dbname < db.dump
```

Wie Sie sehen, erfolgt der **Aufruf analog zum Aufruf von pg_dump**. Jedoch nutzen wir eine Umleitung der Eingabe ('<') aus der PostgreSQL-Dump-Datei.

Das pg_dump-Tool kann noch einiges mehr. So ist es möglich, **Datenbanken in speziellen Dump-Formaten auszugeben**. Je nach Format des erzeugten PostgreSQL-Dumps wird das pg_restore- oder psql-Tool zum Wiederherstellen benötigt.

Hier ein Überblick der beim **Erstellen und Wiederherstellen von PostgreSQL-Backups** zum Einsatz kommenden Kommandozeilen-Tools:

|Tool|Erklärung|
|---|---|
|pg_dump|Kommandozeilen-Tool zum Erstellen eines PostgreSQL-Dumps|
|pg_restore|Kommandozeilen-Tool zum Wiederherstellen einer PostgreSQL-Datenbank aus einem PostgreSQL-Dump; erlaubt ggf. spezielle Operationen wie partielle Imports, Umsortieren der Import-Daten, parallelen Import mehrerer Tabellen etc.|
|psql|PostgreSQL-Kommandozeilen-Schnittstelle; nimmt SQL-Anweisungen von der Kommandozeile oder aus einer PostgreSQL-Dump-Datei entgegen und führt diese aus|

Betrachten wir das **generelle Muster bei der Nutzung von** pg_restore zum Wiederherstellen eines PostgreSQL-Dumps:

```sh
pg_restore --dbname=dbname db.dump
```

Die Nutzung von pg_restore **erfordert das Erzeugen des PostgreSQL-Dumps in einem speziellen Format**. Schauen wir uns die möglichem Ausgabeformate von pg_dump an:

|pg_dump-Ausgabeformat|Erklärung|Import via|
|---|---|---|
|Plain|Plain-Text-Datei mit SQL-Anweisungen; Komprimieren erfordert weiteres Tool wie Gzip|psql|
|Custom|Komprimiertes Dump-Format; Import lässt sich im Detail steuern|pg_restore|
|Directory|Erstellt Verzeichnis mit je einer Datei pro Tabelle/Blob; zusätzlich Inhaltsverzeichnis; Llsst sich mit Standard-Unix-Tools bearbeiten; erlaubt parallelen Export mehrerer Tabellen|pg_restore|
|Tar|Archivierungsformat „Tape Archive“; lässt sich in Directory-Format umwandeln; Komprimieren erfordert weiteres Tool wie Gzip; Abfolge der Imports nicht steuerbar|pg_restore|

Abschließend das generelle Muster beim Aufruf von pg_dump zur **Erstellung eines PostgreSQL-Dumps im „Custom“-Dump-Format**:

```sh
pg_dump --format=custom dbname > db.dump
```

> Tipp: Wenn Ihre PostgreSQL-Installation in einem Docker-Container läuft, können Sie pg_dump innerhalb des Containers verwenden, um ein PostgreSQL-Backup anzulegen. Es gibt auch die Möglichkeit, den gesamten Container als [Docker-Backup](https://www.ionos.at/digitalguide/server/sicherheit/docker-backup/ "Docker-Backup") zu speichern. Wie das geht, erklären wir in unserem Detailartikel.

## Schritt-für-Schritt-Anleitung zum Erstellen und Wiederherstellen eines PostgreSQL-Backups

Es gibt unterschiedliche Wege, PostgreSQL-Backups anzulegen und wiederherzustellen. **Je nach Einsatzszenario und Anforderungen** bieten die Methoden verschiedene Vor- und Nachteile. Das Vorgehen beim Erstellen des PostgreSQL-Dumps bestimmt darüber, welche Methode beim Import zum Einsatz kommt.

Bevor wir uns mit der spezifischen Methode zum Anlegen und Wiederherstellen von PostgreSQL-Dumps beschäftigen, ein kleiner Hinweis: Die nachfolgenden Beispiele referenzieren lediglich den Namen der zu nutzenden Datenbank. Sie enthalten jedoch keine Datenbank-Nutzernamen oder -Passwörter. Diese sind bei PostgreSQL der Konvention folgend in der Passwort-Datei _.pgpass_ abgelegt. Die Datei befindet sich im Heimverzeichnis des Nutzers und enthält **PostgreSQL-Verbindungsdaten**. Dabei kommt das folgende Format zum Einsatz:

```sh
hostname:port:database:username:password
```

Die in der Passwort-Datei enthaltenen Informationen werden **automatisch bei Aufruf der Kommandozeilen-Tools genutzt**. So entfällt das riskante Angeben sensibler Daten auf der Kommandozeile.

### Überprüfen, ob Tools vorhanden sind, und diese ggf. installieren

Vorab überprüfen wir, ob pg_dump und pg_restore installiert sind. Wir **versuchen, die Tools aufzurufen und deren Version auszugeben**. Schlägt dies fehl, dann fehlt das Tool und muss installiert werden.

- Überprüfen, ob pg_dump installiert ist:

```sh
pg_dump --version
```

- Überprüfen, ob pg_restore installiert ist:

```sh
pg_restore --version
```

- Ferner überprüfen wir, ob die PostgreSQL-Kommandozeilenschnittstelle psql installiert ist:

```sh
psql --version
```

Sollten die **Tools nicht gefunden** werden, lassen sie sich leicht installieren.

- Auf dem **Mac** installieren wir die PostgreSQL-Client-Anwendungen mit Homebrew:

```sh
brew install libpq
brew link --force libpq
```

- Unter **Ubuntu-Linux** nutzen wir die eingebaute Paketverwaltung:

```sh
sudo apt-get install postgresql-client
```

- Folgen Sie unter **Windows** unserer Anleitung, um [PostgreSQL auf Windows Server 2016 zu installieren](https://www.ionos.at/digitalguide/server/konfiguration/postgresql-auf-windows-server-2016-installieren/ "PostgreSQL auf Windows Server 2016 installieren").

### PostgreSQL-Backup erstellen und wiederherstellen

Schauen wir uns zunächst das einfachste Szenario zum Erstellen eines PostgreSQL-Backups an. Dabei **extrahieren wir eine einzelne Datenbank** von einem PostgreSQL-Server. Struktur und Inhalte der Datenbank werden dabei in Form von SQL-Anweisungen in eine Datei geschrieben. Der folgende Aufruf des pg_dump-Tools leistet genau das:

```sh
pg_dump dbname > db.dump
```

Wie sieht es nun aus, wenn man den erzeugten **PostgreSQL-Dump auf einem anderen Server wiederherstellen** möchte? Zu diesem Zweck kommt die PostgreSQL-Kommandozeilen-Schnittstelle psql zum Einsatz. Der Befehl ist denkbar einfach:

```sh
psql dbname < db.dump
```

Neben der Ausgabe eines PostgreSQL-Dumps als Textdatei mit SQL-Anweisungen erlaubt das pg_dump-Tool das **Erzeugen spezialisierter PostgreSQL-Dump-Formate**. Diese werden beim Aufruf über Optionen gesteuert. Hier ein Überblick der beiden nützlichsten Dump-Formate:

|PostgreSQL-Dump-Format|Ausführliche Options-Syntax|Kurze Options-Syntax|
|---|---|---|
|Custom|pg_dump --format=custom|pg_dump -Fc|
|Directory|pg_dump --format=directory|pg_dump -Fd|

Wir erstellen einen **PostgreSQL-Dump im Custom-Format**:

```sh
pg_dump --format=custom dbname > db.dump
```

Zum **Wiederherstellen an einem anderen Server** nutzt man das pg_restore-Tool:

```sh
pg_restore --dbname=dbname db.dump
```

Möchte man den **PostgreSQL-Dump am selben Server wiederherstellen**, erfordert dies einen zusätzlichen Schritt. Denn Datenbank und Tabellen sind auf dem Server bereits vorhanden und müssen vor dem Import entfernt werden. Konzeptuell ist das ähnlich wie '--add-drop-tables' bei MySQL-Dump. Praktischerweise lässt sich bei PostgreSQL die Funktionalität beim Importieren hinzuschalten:

```sh
pg_restore --clean --create --dbname=dbname db.dump
```

Die Option '--clean' **entfernt eine existierende Datenbank vor dem Import**; mit der Option '--create' erstellen wir die Datenbank unter dem angegebenen Namen. So läuft der Import ohne Kollisionen ab.

### PostgreSQL-Datenbank zwischen entfernten Servern migrieren

Export und Import eines PostgreSQL-Dumps lassen sich über eine Pipe ('|') verbinden. So werden **die exportierten Daten direkt in den Import eingespeist**. Da pg_dump, pg_restore und psql bei Bedarf auf einem entfernten Host operieren, ist es möglich, den Export von einem Server direkt auf einem anderen Server zu importieren. Schauen wir uns den dafür verwendeten Befehlsaufruf an. Wir nutzen die Option '--host', um die Host-Namen anzugeben:

```sh
pg_dump --host=export_host dbname | psql --host=import_host dbname
```

### PostgreSQL-Backup großer Datenbanken erstellen

Bei PostgreSQL handelt es sich um ein professionelles Datenbank-Management-System. Um Backups umfangreicher Datenbanken anzulegen, wird ein spezielles Vorgehen benötigt. Denn erzeugte **PostgreSQL-Dumps können sehr groß werden**. Zunächst bietet sich an, Komprimierung zu nutzen. Als Textdatei exportierte PostgreSQL-Dumps enthalten für gewöhnlich große Mengen redundanter SQL-Anweisungen. Diese lassen sich gut komprimieren.

Wir leiten die Ausgabe von pg_dump per Pipe an das Gzip Kompressions-Tool weiter und schreiben anschließend in eine komprimierte _.gz_-Datei:

```sh
pg_dump dbname | gzip > db.dump.gz
```

Zum **Wiederherstellen aus einem komprimierten PostgreSQL-Dump** drehen wir den Prozess um. Das Gunzip-Tool entpackt die komprimierten Daten und gibt diese an die Standardausgabe aus. Wir leiten die Ausgabe per Pipe an das psql-Tool weiter. Beim Entpacken nutzen wir die Option '-c', um die Eingabedatei unberührt zu lassen:

```sh
gunzip -c db.dump.gz | psql dbname
```

Die 3-2-1-Backup-Regel erfordert mindestens ein Backup in der Cloud. Der Upload sehr großer Dateien kann unter gewissen Umständen problematisch sein. Ggf. ist es sinnvoll, das **PostgreSQL-Dump mit dem split-Tool in mehrere Dateien zu zerlegen**. Wiederum nutzen wir eine Pipe, um die Ausgabe von pg_dump an split weiterzuleiten. In unserem Beispiel spalten wir den PostgreSQL-Dump in einzelne Dateien mit einer maximalen Größe von 1 GB:

```sh
pg_dump dbname | split -b 1G - db.dump
```

Wie lassen sich die **beim Import durch split erzeugten partiellen Dateien wieder zusammenführen**? Hierfür wird keine spezielle Software benötigt; stattdessen nutzen wir das Standard-cat-Tool. Wir übergeben eine Liste der partiellen PostgreSQL-Dumps, die vom cat-Tool zu einem zusammenhängenden Datenstrom kombiniert werden. Diesen leiten wir wie gehabt per Pipe an psql weiter:

```sh
cat db.dump* | psql dbname
```

Bei Verwendung des Directory-Dump-Formats ist es möglich, **mehrere Tabellen parallel zu dumpen**. Das geht schneller, führt jedoch auch zu einer höheren Last am Datenbank-Server. Über die Option '--jobs' steuern wir die Anzahl parallel exportierter Tabellen. In unserem Beispiel exportieren wir drei Tabellen parallel. Da wir ein Verzeichnis schreiben, ist es nicht möglich, die Ausgabe an eine Datei umzuleiten. Stattdessen nutzen wir die Option '--file' mit Angabe des Verzeichnisnamens:

```sh
pg_dump --jobs=3 --format=directory --file=dump.dir dbname
```