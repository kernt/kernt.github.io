---
tags:
  - elk
  - elk-stack
  - kibana5
---
# elk-stack-kibana-visualisierung

Jetzt, wenn Sie sich vergewissern, dass Ihre Daten in Elasticsearch erfolgreich indiziert werden, können wir voran gehen und die Kibana-Schnittstelle anschauen, um einige nützliche Analysen aus den Daten zu erhalten.
Laufen Kibana

Wie im vorigen Kapitel beschrieben, starten wir den Kibana-Service aus dem Kibana-Installationsverzeichnis.
`$ bin/kibana`

![kibana-logstash-log](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_02.jpg)

# Kibana Discover page

Da wir Kibana bereits eingerichtet haben, um standardmäßig logstash- * indexes zu nehmen, werden die indizierten Daten als Histogramm der Zählungen und die zugehörigen Daten als Felder im JSON-Format angezeigt.

Zuerst müssen wir das Datumsfilter auf Basis unseres Datumsbereichs so einstellen, dass wir unsere Analyse auf demselben bauen können. Seit wir vom 1. Juli 2014 bis zum 31. Dezember 2014 Daten gemacht haben, werden wir unseren Datumsfilter für das gleiche konfigurieren.

Wenn Sie auf das Time-Filter-Symbol an der äußersten rechten Ecke klicken, können wir einen absoluten Zeitfilter auf der Basis unseres Sortiments wie folgt einstellen:

![kibana4-discover-abulute](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_03.jpg)

# Kibana Zeitfilter

Jetzt sind wir alle eingestellt, um schöne Visualisierungen auf dem gesammelten Datensatz mit dem reichen Satz von Visualisierungsmerkmalen zu erstellen, die Kibana bietet.

Bevor wir die Visualisierung aufbauen, lassen Sie uns bestätigen, ob alle Felder mit den zugehörigen Datentypen korrekt indiziert sind, damit wir die entsprechenden Operationen durchführen können.

Hierzu klicken Sie auf die Seite Einstellungen oben auf dem Bildschirm und wählen Sie das `Logstash- *` Index-Muster auf der linken Seite des Bildschirms. Die Seite sieht so aus:

![Kibana Settings page](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_04.jpg)
Kibana Settings page

Es zeigt alle unsere Felder, die indiziert wurden, ihre Datentypen, Indexstatus und Beliebtheitswert.

## Kibana Visualisierungen

Lassen Sie uns einige grundlegende Visualisierungen von der Kibana Visualisierungsseite aufbauen, und wir werden sie später im Armaturenbrett verwenden.

Klicken Sie auf den Link zur Visualisierungsseite oben auf der Kibana-Homepage und klicken Sie auf das neue Visualisierungssymbol.

Diese Seite zeigt verschiedene Arten von Visualisierungen, die mit der Kibana-Schnittstelle möglich sind:

![Kibana visualization menu](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_05.jpg)
Kibana visualization menu

### Erstellen eines Liniendiagramms

Die erste Visualisierung, die wir bauen werden, ist ein Liniendiagramm, das eine wöchentlich enge Preisindexbewegung für das GOOG-Skript über einen Zeitraum von sechs Monaten zeigt.

Wählen Sie Line-Diagramm aus dem Visualisierungsmenü, und dann wählen wir Y-Achsen-Metriken als Max und Feld als nah. Wählen Sie im Bereich Eimer die Option Aggregation als Datumshistogramm auf der Grundlage des @timestamp-Feldes und Intervall als wöchentlich aus, und klicken Sie auf Übernehmen.

![Kibana Line chart](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_06.jpg)
Kibana Line chart

Jetzt speichern Sie die Visualisierung mit einem Namen für das Liniendiagramm, das wir später in das Armaturenbrett ziehen werden.

### Erstellen eines Balkendiagramms

Wir werden ein vertikales Balkendiagramm erstellen, das die Bewegung der wöchentlich gehandelten Bände über einen Zeitraum von sechs Monaten darstellt.

Wählen Sie im **Vertical Bar Chart** die Option Vertikale Balkendiagramme aus und wählen Sie die **Y-Axis Aggregation** als **Sum** und **Field** als `volume` aus. Wählen Sie im Bereich **buckets ** die **X-Axis Aggregation** als **Date Histogram** und **Field** als **@timestamp** und **Interval** als **Weekly** aus. Klicken Sie auf **Apply**, um ein vertikales Balkendiagramm zu sehen, das das wöchentliche Gesamtvolumen darstellt, das über einen Zeitraum von sechs Monaten gehandelt wird.

![Kibana Vertical Bar Chart](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_07.jpg)
Kibana Vertical Bar Chart

Jetzt speichern Sie die Visualisierung mit einem Namen für das Balkendiagramm, das wir später in das Armaturenbrett ziehen werden.

### Eine Metrik bauen

Metrik stellt eine große Zahl dar, die wir als etwas Besonderes über Daten zeigen wollen.

Wir werden das Höchste Volumen, das an einem einzigen Tag in einem Zeitraum von sechs Monaten mit Metrik gehandelt wird, zeigen.

Klicken Sie im Visualisierungsmenü auf Metric und wählen Sie Metric Aggregation als Max, Field als Volume. Klicken Sie auf Anwenden, um das Ergebnis der Visualisierung auf der rechten Seite wie folgt zu sehen:

![Kibana Metric](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_08.jpg)
Kibana Metric

Jetzt speichern Sie die Visualisierung mit einem Namen für die Metrik, die wir später in das Armaturenbrett ziehen werden.

### Erstellen einer Datentabelle

Datentabellen sollen detaillierte Ausfälle in einem tabellarischen Format für die Ergebnisse einiger zusammengesetzter Aggregationen darstellen.

Wir erstellen eine Datentabelle der **Monthly Average** Volumen, die über sechs Monate gehandelt wird.

Wählen Sie im **Data table** die Datentabelle aus, klicken Sie auf **split rows** und wählen Sie **Aggregation** als **Average** und `Fields` als `volume` aus. Wählen Sie im Bereich `buckets` die Option `Aggregation` als `Date Histogram`, `Fields` als `@timestamp` und `Interval` als `Monthly` aus. Klicken Sie auf `Apply`, um das Bild wie im folgenden Screenshot zu sehen:

![Kibana Data table](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_09.jpg)
Kibana Data table

Jetzt speichern Sie die Visualisierung mit einem Namen für die Datentabelle, die wir später in das Dashboard ziehen werden.

Nachdem wir einige Visualisierungen erstellt haben, bauen wir ein Dashboard, das diese Visualisierungen enthält.

Wählen Sie oben auf der Seite den Dashboard-Seitenlink aus und klicken Sie auf den Link "Visualisierung hinzufügen", um Visualisierungen aus Ihren gespeicherten Visualisierungen auszuwählen und zu arrangieren.

Das Dashboard, nach dem Einfügen eines Liniendiagramms, Balkendiagramm, Datentabelle und Metric, sieht so aus:

![Kibana Dashboard](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_10.jpg)
Kibana Dashboard

Jetzt können wir dieses Dashboard mit der Save-Taste speichern, und es kann später gezogen und leicht geteilt werden.

Dashboards können als IFrame in andere Systeme eingebettet werden oder können direkt als Links geteilt werden.

Klicken Sie auf die Schaltfläche Freigeben, um die Optionen zu sehen:
![Kibana Share options](https://www.packtpub.com/graphics/9781787288546/graphics/B04876_02_11.jpg)

Wenn Sie alles bis zu diesem Punkt abgeschlossen haben, dann haben Sie erfolgreich Ihre erste ELK-Datenpipeline eingerichtet.