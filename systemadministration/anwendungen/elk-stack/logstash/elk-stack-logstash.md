---
tags:
  - elk
  - elk-stack
  - logstash
---
# Logstash ausführen

Führen Sie Logstash mit `-e` Flag aus, gefolgt von der Konfiguration der Standard-Eingabe und Ausgabe:

```bash
cd logstash-1.5.0
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

Nun, wenn wir etwas in die Eingabeaufforderung eingeben, sehen wir seine Ausgabe in Logstash wie folgt:

```bash
hello logstash
2015-05-15T03:34:30.111Z 0.0.0.0 hello logstash
```

Hier betreiben wir Logstash mit dem `stdin`-Eingang und dem `stdout`-Ausgang, da diese Konfiguration alles, was Sie in ein strukturiertes Format als Ausgabe eingeben, druckt. Mit dem Flag `-e` können Sie die Konfiguration schnell über die Befehlszeile testen.

Nun, versuchen wir die `codec`-Einstellung für die Ausgabe für eine ziemlich formatierte Ausgabe. Beenden Sie den laufenden Logstash, indem Sie einen **Strg + C**-Befehl ausgeben, und dann müssen wir Logstash mit dem folgenden Befehl neu starten:
`bin/logstash -e 'input { stdin { } } output { stdout { codec => rubydebug } }'`

Geben Sie nun noch mehr Testeingaben ein:

```bash
Hello PacktPub

{
  "message" => " Hello PacktPub",
  "@timestamp" => "2015-05-20T23:48:05.335Z",
  "@version" => "1",
  "host" => "packtpub"
}
```

Die Ausgabe, die Sie sehen, ist die häufigste Ausgabe, die wir normalerweise von Logstash sehen:

* "message" enthält die komplette Eingangsnachricht oder die Ereigniszeile

* "@timestamp" enthält den Zeitstempel der Zeit, in der das Ereignis indiziert wurde; Oder wenn Datumsfilter verwendet wird, kann dieser Wert auch eines der Felder in der Nachricht verwenden, um einen für das Ereignis spezifischen Zeitstempel zu erhalten

* "host" wird in der Regel die Maschine repräsentieren, in der dieses Ereignis erzeugt wurde


### Logstash mit Datei-Eingabe

Logstash kann einfach konfiguriert werden, um aus einer Protokolldatei als Eingabe zu lesen.

Zum Beispiel, um Apache-Protokolle aus einer Datei zu lesen und an eine `stout` auszugeben, ist die folgende Konfiguration hilfreich:

```c
input {
  file {
    type => "apache"
    path => "/user/packtpub/intro-to-elk/elk.log"
  }
}
output {
  stdout {
    codec => rubydebug
  }
}
```

### Logstash mit Elastisches Sektor

Logstash kann so konfiguriert werden, dass alle Eingaben an eine Elasticsearch-Instanz ausgegeben werden. Dies ist das häufigste Szenario in einer ELK-Plattform:

`bin/logstash -e 'input { stdin { } } output { elasticsearch { host = localhost } }'`

Dann geben Sie , für Protokolle 'you know ein

Sie werden in der Lage sein, Indizes in Elasticsearch zu sehen durch

`http://localhost:9200/_search`.

### Konfigurieren von Logstash

Logstash-Konfigurationsdateien befinden sich im JSON-Format. Eine Logstash-Konfigurationsdatei hat einen separaten Abschnitt für jede Art von Plugin, die Sie der event processing pipeline hinzufügen möchten. Beispielsweise:

```c
# This is a comment. You should use comments to describe
# parts of your configuration.
input {
  ...
}

filter {
  ...
}

output {
  ...
}
```

Jeder Abschnitt enthält die Konfigurationsoptionen für ein oder mehrere Plugins. Wenn Sie mehrere Filter angeben, werden sie in der Reihenfolge ihres Erscheinens in der Konfigurationsdatei angewendet.

Wenn Sie `logstash` ausführen, verwenden Sie das `-flag`, um Konfigurationen aus einer Konfigurationsdatei oder sogar aus einem Ordner mit mehreren Konfigurationsdateien für jeden Typ von Plugin-Input, Filter und Output zu lesen:

`bin/logstash –f ../conf/logstash.conf`

### Notiz
Wenn Sie Ihre Konfigurationen für Syntaxfehler vor dem Ausführen testen möchten, können Sie einfach mit folgendem Befehl überprüfen:

`bin/logstash –configtest ../conf/logstash.conf`

Dieser Befehl überprüft nur die Konfiguration ohne `logstash` auszuführen.

Logstash läuft auf JVM und verbraucht eine große Menge an Ressourcen. Logstash hat manchmal einen erheblichen Speicherverbrauch. Offensichtlich könnte dies eine große Herausforderung sein, wenn Sie Protokolle von einer kleinen Maschine senden möchten, ohne die Anwendungsleistung zu beeinträchtigen.

Um Ressourcen zu sparen, kannst du den Logstash Forwarder (früher als Lumberjack bekannt) verwenden. Der Forwarder nutzt das Protokoll von Lumberjack, so dass Sie sicher komprimierte Protokolle versenden können, wodurch der Ressourcenverbrauch und die Bandbreite reduziert werden. Die einzige Eingabe sind Datei/en, während die Ausgabe auf mehrere Ziele gerichtet werden kann.

Es gibt auch andere Optionen, um Log daten zu senden. Sie können rsyslog auf Linux-Rechnern verwenden, und es gibt andere Agenten für Windows-Rechner wie z.B. `nxlog` und `syslog-ng`. Es gibt ein weiteres einfgaches Werkzeug, um Logs zu versenden namens `Log-Courier` (https://github.com/driskell/log-courier), was eine verbesserte fork des Logstash-Forwarders mit einigen Verbesserungen ist.

### Installieren von Logstash Forwarder

Laden Sie die neueste Logstash Forwarder-Version von der Download-Seite herunter.

### Tip
Überprüfen Sie die neueste Logstash-Forwarder-Version unter https://www.elastic.co/downloads/logstash.

Bereiten Sie eine Konfigurationsdatei vor, die Eingabe-Plugin-Details und ssl-Zertifikatdetails enthält, um eine sichere Kommunikation zwischen Ihrem Forwarder und Indexer-Servern herzustellen und es mit dem folgenden Befehl auszuführen:

```c
input {
  lumberjack {
    # The port to listen on
    port => 12345

    # The paths to your ssl cert and key
    ssl_certificate => "path/to/ssl.crt"
    ssl_key => "path/to/ssl.key"

    # Set the type of log.
    type => "log type"
  }
```

### Logstash-Plugins

Einige der beliebtesten Logstash-Plugins sind:

* Input plugin
* Filters plugin
* Output plugin

### Input plugin
Einige der beliebtesten Logstash-Eingabe-Plugins sind:

* **file**: Diese Streams protokollieren Ereignisse aus einer Datei
* **redis**: streamt Ereignisse aus einer redis-Instanz
* **stdin**: streamt Ereignisse von der Standard-Eingabe
* **syslog**: streamt zu Syslog-Nachrichten über das Netzwerk
* **ganglia**: streamt ganglia Pakete über das Netzwerk über udp
* **lumberjack**: Dies empfängt Ereignisse mit dem lumberjack-Protokoll
* **eventlog**: Dies empfängt Ereignisse aus dem Windows-Ereignisprotokoll
* **S3**: Dies streamt Ereignisse aus einer Datei aus einem s3-bucket
* **elasticsearch**: Das liest aus dem Elasticsearch-Cluster auf der Grundlage der Ergebnisse einer Suchanfrage

### Filter-Plugin

Einige der beliebtesten Logstash-Filter-Plugins sind wie folgt:

* **date**: Dies wird verwendet, um Datumsfelder aus eingehenden Ereignissen zu analysieren und diese als Logstash-Zeitstempelfelder zu verwenden, die später für die Analyse verwendet werden können.

* **drop**: Das verwirft alles von eingehenden Ereignissen, die mit der Filterbedingung übereinstimmen.

* **grok**: Dies ist der leistungsstärkste Filter, um unstrukturierte Daten von Protokollen oder Ereignissen zu einem strukturierten Format zu analysieren.

* **multiline**: Dieser hilft, mehrere Zeilen aus einer einzigen Quelle als ein Logstash-Ereignis zu analysieren

* **dns**: Dieser Filter löst eine IP-Adresse aus allen angegebenen Feldern auf

* **mutate**: Dies hilft Umbenennen, Entfernen, Ändern und Ersetzen von Feldern in Ereignissen

* **geoip**: Dies fügt geografische Informationen auf der Grundlage von IP-Adressen hinzu, die aus der `Maxmind`-Datenbank abgerufen werden

### Ausgangs-Plugin

Eine Liste der beliebtesten Logstash-Ausgangs-Plugins sind wie folgt:

* **file** : Dies schreibt Ereignisse in eine Datei auf Datenträger

* **e-mail** : Dies sendet eine E-Mail, die auf einigen Bedingungen basiert, wenn sie eine Ausgabe erhält

* **elasticsearch**: Dies sendet die Ausgabe an den Elasticsearch-Cluster, die häufigste und empfohlene Ausgabe für Logstash

* **stdout**: Es schreibt Ereignisse zur Standardausgabe.

* **redis**: Dies schreibt Ereignisse in die Redis-Warteschlange und wird als Broker für viele ELK-Implementierungen verwendet

* **mongodb**: Das schreibt die Ausgabe an die Mongodb

* **kafka**: Dies schreibt Ereignisse nach Kafka Topic.
