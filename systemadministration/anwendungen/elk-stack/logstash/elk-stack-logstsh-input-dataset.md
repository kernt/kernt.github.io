---
tags:
  - elk
  - elk-stack
  - logstash
---
# elk stack logstsh input dataset

Für unser Beispiel ist der Datensatz, den wir hier verwenden werden, der tägliche Google (GOOG) Quotes Preisdatensatz über einen Zeitraum von 6 Monaten vom 1. Juli 2014 bis zum 31. Dezember 2014. Dies ist ein guter Datensatz, um zu verstehen, wie wir schnell können Analysieren einfache Datensätze, wie diese, mit ELK.

### Hinweiß

Dieser Datensatz kann einfach aus folgender Quelle heruntergeladen werden:

Http://finance.yahoo.com/q/hp?s=GOOG

### Datenformat für Eingabedatensatz

Die wichtigsten Felder dieses Datensatzes sind `Date`, `Open Price`, `Close Price`, `High Price`, `Volume`, und `Adjusted Price`.

Die folgende Tabelle zeigt einige der Beispieldaten aus dem Datensatz. Der aktuelle Datensatz befindet sich im CSV-Format.

| Date | Open | High | Low | Close | Volume | Adj Close |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
|Dec 31, 2014 | 531.25 | 532.60 | 525.80 | 526.40 | 1,368,200 | 526.40 |
|...|...|...|...|...|...|...|

Wir müssen diese Daten an einen Ort stellen, von wo aus ELK Stack auf sie zur weiteren Analyse zugreifen kann.

Wir betrachten einige der Top-Einträge der CSV-Datei mit dem Unix `head` -Befehl wie folgt:

```sh
$ head GOOG.csv
2014-12-31,531.25244,532.60236,525.80237,526.4024,1368200,526.4024
2014-12-30,528.09241,531.1524,527.13239,530.42242,876300,530.42242
2014-12-29,532.19244,535.48242,530.01337,530.3324,2278500,530.3324
2014-12-26,528.7724,534.25244,527.31238,534.03247,1036000,534.03247
2014-12-24,530.51245,531.76141,527.0224,528.7724,705900,528.7724
2014-12-23,527.00238,534.56244,526.29236,530.59241,2197600,530.59241
2014-12-22,516.08234,526.4624,516.08234,524.87238,2723800,524.87238
2014-12-19,511.51233,517.72235,506.9133,516.35229,3690200,516.35229
2014-12-18,512.95233,513.87231,504.7023,511.10233,2926700,511.10233
```

Jede Zeile repräsentiert die Quote Preisdaten für ein bestimmtes Datum durch ein Komma getrennt.

Jetzt, wenn wir mit den Daten vertraut sind, richten wir den ELK Stack ein, wo wir die Daten mit Logstash analysieren und verarbeiten können, in Elasticsearch indexieren und dann in Kibana schöne Visualisierungen bauen.