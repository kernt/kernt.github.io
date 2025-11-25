---
tags:
  - elk-stack
  - kibana5
  - apache
  - logging
---
# elk stack kibana5 apache log

Apache und NGINX sind die meistgenutzten Webserver in der Welt. Es gibt Milliarden von Anfragen, die von diesen Servern da draußen bedient werden, zu internen Netzwerken so viel wie externe Benutzer. Die meiste Zeit sind sie eine der ersten Logik-Layer, die in einer Transaktion berührt werden, so dass man von dort aus eine sehr genaue Sicht auf das, was im Begriff des Service-Gebrauchs vor sich geht, bekommen kann.

In diesem Kapitel konzentrieren wir uns auf den Apache-Server und nutzen die Protokolle, die der Server während der Laufzeit generiert, um die Benutzeraktivität zu visualisieren. Die Protokolle, die wir verwenden werden, wurden von einer Website (www.logstash.net) Apache Webserver generiert. Sie wurden von Peter Kim und Christian Dahlqvist zusammengestellt, zwei meiner Lösungen Architektenkollegen bei Elastic (https://github.com/elastic/elk-index-size-tests).

Wie in der Einleitung erwähnt, können diese Daten aus verschiedenen Blickwinkeln angegangen und analysiert werden, und wir werden versuchen, eine Sicherheits- und eine Bandbreitenanalyse durchzuführen.

Das erste Ziel ist, verdächtiges Verhalten in den Daten zu erkennen, wie z. B. Benutzer, die versuchen, auf Seiten zuzugreifen, die auf der Website nicht mit einem Admin-Wort in der URL existieren; Dieses Verhalten ist sehr häufig. Es ist nicht selten, Webseiten wie WordPress-Blogs zu sehen, die eine Administrations-Seite über den / wp-admin-URI aussetzen. Menschen, die nach Sicherheitslecks suchen, wissen das und werden versuchen, es zu nutzen. Also, mit unseren Daten können wir durch HTTP-Code (hier 404) aggregieren und sehen, was die meistbesuchten unbekannten Seiten sind.

Die zweite zielt darauf ab, zu messen, wie sich der Verkehr auf einer bestimmten Website verhält. Das Verkehrsmuster hängt von der Website ab. Eine regionale Website kann nicht über Nacht besucht werden, und eine weltweite Website könnte ständigen Verkehr haben, aber wenn Sie den Verkehr pro Land brechen, sollten Sie etwas ähnlich dem ersten Fall erhalten. Die Messung der Bandbreite ist dann entscheidend für die Gewährleistung eines optimalen Erlebnisses für Ihre Benutzer, so dass es hilfreich sein könnte zu sehen, dass es regelmäßige Spitzen von Bandbreiten Verbrauch von Benutzern in einem bestimmten Land gibt.

Bevor wir in die Sicherheits- und Bandbreitenanalyse gehen, schauen wir uns an, wie man Daten in Elasticsearch importiert und die verschiedenen Visualisierungen, die Sie als Teil des Dashboards importieren müssen.

## Daten in der Konsole importieren

Die Daten, die wir hier beschäftigen, sind in der Struktur nicht ganz anders als in den vorangegangenen Kapiteln. Sie sind immer noch Ereignisse, die einen Zeitstempel enthalten, also im Wesentlichen zeitbasierte Daten. Der Stichprobenindex, den wir haben, enthält 300.000 Veranstaltungen ab 2015.

Die Art, wie wir die Daten importieren, ist etwas anders als im vorherigen Kapitel, wo wir Logstash verwendet haben, um die Daten aus einer Quelldatei zu indizieren. Hier werden wir die Snapshot-API von Elasticsearch verwenden. Es erlaubt uns, eine zuvor erstellte Indexsicherung wiederherzustellen. Wir werden dann einen Schnappschuss der im Rahmen der Ressourcen des Buches bereitgestellten Daten wiederherstellen.

Hier ist der Link zur Elasticsearch Snapshot-Restore API (https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-snapshots.html), in dem Sie ausführlichere Informationen benötigen.

Im Folgenden sind die Schritte zur Wiederherstellung unserer Daten aufgeführt:

* Legen Sie den Pfad zu einem Snapshot-Repository in der Elasticsearch-Konfigurationsdatei fest
* Registrieren Sie ein Snapshot-Repository in Elasticsearch
* Notieren Sie den Snapshot und prüfen Sie, ob er in der Liste erscheint, um zu sehen, ob unsere Konfiguration korrekt ist
* Wiederherstellen Sie unseren Schnappschuss
* Prüfen Sie, ob der Index ordnungsgemäß wiederhergestellt wurde

Zuerst müssen wir Elasticsearch konfigurieren, um einen neuen Pfad zu einem Repository des Snapshots zu registrieren. Um dies zu tun, musst du die `elasticsearch.yml` bearbeiten, die sich hier befindet:

`ELASTICSEARCH_HOME/conf/elasticsearch.yml`

Hier stellt `ELASTICSEARCH_HOME` den Pfad zu Ihrem Elasticsearch-Installationsordner dar.

Fügen Sie am Ende der Datei folgende Einstellungen hinzu:

`path.repo: ["/PATH_TO_CHAPTER_3_SOURCE/basic_logstash_repository"]`

`Basic_logstash_repository` enthält die Daten in Form eines Snapshots. Sie können nun Elasticsearch neu starten, um die Änderungen zu berücksichtigen.

Jetzt öffnen Sie Kibana und gehen Sie zum Konsolen abschnitt. Konsole ist ein webbasierter Elasticsearch-Abfrage-Editor, mit dem Sie mit der Elasticsearch-API spielen können. Einer der Vorteile von Console ist, dass es Auto-Fertigstellung bringt, was der Benutzer tippt, was viel hilft, wenn man nicht alles über die Elasticsearch API weiß.

In unserem Fall verwenden wir die Snapshot / Restore API, um unseren Index zu erstellen. Hier sind die Schritte zu folgen:

1. Zuerst müssen Sie das neu hinzugefügte Repository mit dem folgenden API-Aufruf registrieren:
```
            PUT /_snapshot/basic_logstash_repository
            {
              "type": "fs",
              "settings": {
              "location":  
                "/Users/bahaaldine/Dropbox/Packt/sources/chapter3/
                  basic_logstash_repository",
              "compress": true
              }
            }
```

2. Einmal registriert, versuchen Sie, die Liste der Schnappschüsse zur Verfügung zu stellen, um die Registrierung richtig zu überprüfen:
`GET _snapshot/basic_logstash_repository/_all`
Sie sollten die Beschreibung unseres Schnappschusses erhalten:

```
{
              "snapshots": [
                {
                  "snapshot": "snapshot_201608031111",
                  "uuid": "_na_",
                  "version_id": 2030499,
                  "version": "2.3.4",
                  "indices": [
                  "basic-logstash-2015"
                  ],
                  "state": "SUCCESS",
                  "start_time": "2016-08-03T09:12:03.718Z",
                  "start_time_in_millis": 1470215523718,
                  "end_time": "2016-08-03T09:12:49.813Z",
                  "end_time_in_millis": 1470215569813,
                  "duration_in_millis": 46095,
                  "failures": [],
                  "shards": {
                  "total": 1,
                  "failed": 0,
                  "successful": 1
                  }
                }
              ]
            }
```

3. Starten Sie nun den Wiederherstellungsvorgang mit folgendem Aufruf:

`POST /_snapshot/basic_logstash_repository/snapshot_201608031111/_restore`

Sie können den Status des Wiederherstellungsvorgangs mit folgendem Aufruf anfordern:

`GET /_snapshot/basic_logstash_repository/snapshot_201608031111/_status`

In unserem Fall ist das Datenvolumen so klein, dass die Wiederherstellung nur eine Sekunde dauern sollte, also könntest du die Erfolgsnachricht direkt bekommen und kein Zwischenzustand:

```
 {
      "snapshots": [
        {
          "snapshot": "snapshot_201608031111",
          "repository": "basic_logstash_repository",
          "uuid": "_na_",
          "state": "SUCCESS",
          "shards_stats": {
            "initializing": 0,
            "started": 0,
            "finalizing": 0,
            "done": 1,
            "failed": 0,
            "total": 1
          },
          "stats": {
            "number_of_files": 70,
            "processed_files": 70,
            "total_size_in_bytes": 188818114,
            "processed_size_in_bytes": 188818114,
            "start_time_in_millis": 1470215525519,
            "time_in_millis": 43625
          },
          "indices": {
            "basic-logstash-2015": {
              "shards_stats": {
                "initializing": 0,
                "started": 0,
                "finalizing": 0,
                "done": 1,
                "failed": 0,
                "total": 1
              },
              "stats": {
                "number_of_files": 70,
                "processed_files": 70,
                "total_size_in_bytes": 188818114,
                "processed_size_in_bytes": 188818114,
                "start_time_in_millis": 1470215525519,
                "time_in_millis": 43625
              },
              "shards": {
                "0": {
                  "stage": "DONE",
                  "stats": {
                    "number_of_files": 70,
                    "processed_files": 70,
                    "total_size_in_bytes": 188818114,
                    "processed_size_in_bytes": 188818114,
                    "start_time_in_millis": 1470215525519,
                    "time_in_millis": 43625
                  }
                }
              }
            }
          }
        }
      ]
    }
```

An dieser Stelle solltest du den neu erstellten Index auflisten können. Noch in der Konsole, geben Sie den folgenden Befehl:

`GET _cat/indices/basic*`

So soll die Konsole in diesem Stadium aussehen:

! [konsole](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_001.jpg)

Wiederherstellen der Daten aus der Konsole

Die Antwort zeigt an, dass unser Index erstellt wurde und enthält 300.000 Dokumente.

Es könnte hier unterschiedliche Ansätze geben, welche Index-Topologie wir verwenden sollten: Da die Daten Zeitstempel haben, könnten wir zB Tagesindizes oder Wochenindizes erstellen. Dies ist in einer Produktionsumgebung sehr verbreitet, da die aktuellsten Daten oft am häufigsten verwendet werden. Wenn also die letzten sieben Tage der Protokolle die wichtigsten sind und Sie täglich Indizes haben, ist es sehr praktisch, Routinen einzurichten, die entweder die alten Indizes archivieren oder entfernen (älter als sieben Tage).

Wenn wir uns den Inhalt unseres Index anschauen, ist hier ein Beispiel für einen Dokumentenauszug:

```
"@timestamp": "2015-03-11T21:24:20.000Z", 
"host": "Astaire.local", 
"clientip": "186.231.123.210", 
"ident": "-", 
"auth": "-", 
"timestamp": "11/Mar/2015:21:24:20 +0000", 
"verb": "GET", 
"request": "/presentations/logstash-scale11x/lib/js/head.min.js", 
"httpversion": "1.1", 
"response": 200, 
"bytes": 3170, 
"referrer": ""http://semicomplete.com/presentations/logstash-scale11x/"", 
"agent": ""Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.102 Safari/537.36"", 
"geoip": { 
  "ip": "186.231.123.210", 
  "country_code2": "BR", 
  "country_code3": "BRA", 
  "country_name": "Brazil", 
  "continent_code": "SA", 
  "latitude": -10, 
  "longitude": -55, 
  "location": [ 
    -55, 
    -10 
  ] 
}, 
"useragent": { 
  "name": "Chrome", 
  "os": "Mac OS X 10.9.0", 
  "os_name": "Mac OS X", 
  "os_major": "10", 
  "os_minor": "9", 
  "device": "Other", 
  "major": "32", 
  "minor": "0", 
  "patch": "1700" 
}
``` 

Das Dokument beschreibt eine gegebene Benutzerverbindung zur Website und besteht aus HTTP-Metadaten wie der Version, dem Rückkehrcode, dem Verb, dem User-Agenten mit der Beschreibung des verwendeten Betriebssystems und dem verwendeten Gerät und sogar der Lokalisierungsinformation.

Die Analyse jedes Dokuments würde es nicht ermöglichen, das Verhalten der Nutzer zu verstehen. Hier kommt Kibana ins Spiel, indem wir uns erlauben, Visualisierungen zu erstellen, die Daten zusammenfassen und Einsichten zeigen.

Wir können nun das Armaturenbrett importieren.
Dashboard importieren

Im Gegensatz zum vorherigen Kapitel werden wir hier nicht die Visualisierung erstellen, sondern die Import-Funktion von Kibana verwenden, mit der Sie Kibana-Objekte wie Suchvorgänge, Visualisierungen und Dashboards, die bereits existieren, importieren können. Alle diese Objekte sind eigentlich JSON-Objekte, die in einem bestimmten Index indiziert sind, der standardmäßig als `.kibana` bezeichnet wird.

Gehen Sie in die Management-Sektion von Kibana und klicken Sie auf **Saved Objects**. Von dort aus können Sie auf die Schaltfläche **Import** klicken und die JSON-Datei auswählen, die in den Ressourcen des Buches zur Verfügung steht. Zuerst importiere ich die `Apache-logs-visualizations.json` Datei und dann die `Apache-logs-dashboard.json` Datei. Die erste enthält alle Visualisierungen, und die zweite enthält das Dashboard, das die Visualisierungen verwendet. Folgende Visualisierungen sollten vorhanden sein:

![edit Sved Objects](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_002.jpg)

## Importierte Visualisierungen

Sie haben alle BasicLogstash Visualisierungen.

Überprüfen Sie das importierte Dashboard. Gehen Sie zum Dashboard-Bereich und versuchen Sie, das Elastische Stack-Apache Logs-Dashboard zu öffnen. Sie sollten folgendes erhalten:

![Apache logs](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_003.jpg)
Apache log Dashboard

In diesem Stadium sind wir nun bereit, die Visualisierungen einzeln zu durchsuchen und zu erklären, was sie meinen.

### Das dashboard verstehen

Wir werden nun jede Visualisierung durchlaufen und die wichtigsten Leistungsindikatoren und Funktionen ansehen, die sie bieten:

### Markdown - Notizen im dashboard

Das folgende ist der Screenshot der grundlegenden elastischen Zusammenfassung:

![basic elastic summary](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_004.jpg)

### Markdown-Visualisierung

Die Markdown-Visualisierung kann verwendet werden, um Kommentare zu Ihrem Dashboard hinzuzufügen, wie in unserem Beispiel, um zu erklären, was das Dashboard betrifft. Sie können auch die Unterstützung von URLs nutzen, um ein Menü zu erstellen, so dass der Benutzer in der Lage ist, von einem Zustand des Dashboards zu einem anderen zu wechseln.

###### Hinweis
Kibana speichert seinen Zustand in der URL, so können Sie einen bestimmten Zustand eines Dashboards teilen, indem Sie einfach den Link teilen.
Metriken - Protokollübersicht

Die folgende Metrik-Visualisierung gibt eine Zusammenfassung der im Dashboard dargestellten Daten:

### Metrics - logs overview

Die folgende Metrik-Visualisierung gibt eine Zusammenfassung der im Dashboard dargestellten Daten:
![BasicLogstash-metrics](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_005.jpg)

Apache protokolliert Metriken

Es ist immer sinnvoll, einige Metriken in deinem Dashboard anzuzeigen. So kannst du beispielsweise die Anzahl der von deiner Filterung betroffenen Dokumente verfolgen, während du mit dem Dashboard spielst.

#### Balkendiagramm - Antwortcode im Laufe der Zeit

Bar-Charts sind ideal für die Visualisierung einer oder mehrerer Dimensionen Ihrer Daten im Laufe der Zeit. In diesem Beispiel zeigen wir eine Aufschlüsselung des Antwortcodes im Laufe der Zeit. So können Sie die Menge der Anfragen auf www.logstash.net sehen und ob es eine Veränderung im Vergleich zu dem, was Sie denken, ist das erwartete Verhalten. Im Folgenden werden etwa 404 Spikes angezeigt, die einen genaueren Blick haben könnten:

![diagram](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_006.jpg)
Antwortcodes im Laufe der Zeit

### Area chart - Bandbreite nach Land

Bereichsdiagramme sind nützlich, um akkumulative Daten über die Zeit anzuzeigen. Hier nutzen wir das Landsitzfeld, um dieses Bereichsdiagramm zu bauen, um uns Einblicke zu geben, auf welchen Ländern wir uns mit unserer Website verbinden:
![chart](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_007.jpg)

Bandwidth by country

Wir werden uns anschauen, wie wir diese Informationen im Rahmen der Bandbreitenanalyse später im Kapitel nutzen können.

**Datentabelle - Anfragen von Agenten**

Datentabellen können verwendet werden, um tabellarische Feldinformationen und Werte anzuzeigen. Hier zeigen wir eine Anzahl von User-Agent-Feld an, mit dem wir die Art der Clients verstehen können:
![requests-by-agent](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_008.jpg)
Requests by agent

Wir werden sehen, dass im Zusammenhang mit der Sicherheitsanalytik dies eine sehr nützliche Information sein kann, um eine Sicherheitsanomalie zu identifizieren.

**Datentabelle - oben angeforderte Ressourcen**

Wenn ein Host eine Verbindung zu Ihrer Website herstellt, können Sie wissen, was zu dem, was die Ressource ist, sucht nach verschiedenen Gründen wie Klick-Stream-Analyse oder Sicherheit weise; Dies ist, was die folgende Datentabelle Ihnen gibt:
![top-requested-resouces](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_009.jpg)

Top angeforderte Ressourcen

Dies ist auch ein Vorteil, den wir in der Sicherheitsanalyse verwenden werden.

**Kreisdiagramm - bedeutende Länder durch Antwort**

Das Antwort-Balkendiagramm stellt bereits zwei Dimensionen dar, wobei hier eine Dimension ein in unserem Dokument enthaltenes Feld ist. Wir konnten uns schließlich nach Land teilen, aber die Analyseerfahrung wäre ein bisschen kompliziert. Stattdessen könnten wir ein Kreisdiagramm verwenden, um den Landausfall basierend auf dem Antwortcode zu zeigen, wie hier gezeigt:
Wichtige Länder nach Antwortcode

### Tile map - Treffer pro Land

Die Daten enthalten Host-Geo-Punkt-Koordinaten. Hier verwenden wir diese Informationen, um die Client-Verbindungen auf eine Fliesenkarte der Welt zu zeichnen. Wieder aus reiner analytischer Sicht wäre die Tatsache, dass Länder über Visualisierungen hinweg erwähnt werden, nicht ausreichen, wenn sie nicht wie folgt auf der Karte gezeichnet würden:
![hits-per-countries](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_011.jpg)

Darüber hinaus ermöglicht Ihnen die Kartenvisualisierung, Polygone zu zeichnen, um Ihre Analyse einzuschränken.

## Stell den daten eine Frage

Sobald das Dashboard erstellt ist, ist die Idee, es Fragen zu stellen, indem es mit einer oder mehreren Visualisierungen interagiert. Dies wird die Analyse verengen und die besonderen Muster, die die Antworten auf unsere Fragen sind, isolieren.

**Bandbreitenanalyse**

Sie haben vielleicht in der Bandbreite nach Land bemerkt, dass das Niveau zwischen August 2015 und Ende März 2015 niedrig ist und dann plötzlich aus einem unbekannten Grund deutlich zunimmt. Wir sehen eine deutliche Erhöhung der heruntergeladenen Daten, dargestellt durch den Pfeil auf dem Diagramm:

![Bandwidth increase](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_012.jpg)
Bandwidth increase

Wir können in diesen Zeitbereich des Diagramms zoomen, der auch alle anderen Diagramme innerhalb des gleichen Zeitbereichs vergrößern wird. Wenn Sie die Anfragen von der Agent-Datentabelle lesen, werden Sie feststellen, dass der erste Benutzer-Agent ein Chef-Agent ist. Chef wird von Operations-Teams verwendet, um Prozesse wie zB die Installation zu automatisieren. Da die Daten, die wir von www.logstash.net bekommen haben, können wir ableiten, dass ein Chef-Agent eine Verbindung zu unserer Website für Installationszwecke herstellt.

Wenn Sie in der Datentabelle auf die Chefzeile klicken, wird hierfür ein Dashboard-Filter angewendet, der auf dem Wert des ausgewählten Feldes basiert. Dies ermöglicht es Ihnen, in Ihre Daten zu bohren, verengen Sie Ihre Analyse, und wir können leicht zu dem Schluss, dass dieser Prozess verbraucht die Mehrheit der Bandbreite:

![Chef bandwidth utilization](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_013.jpg)
Chef bandwidth utilization

Mit wenigen Klicks konnten wir auf eine Bandbreitenauslastung hinweisen und sind nun in der Lage, die richtigen Maßnahmen zu ergreifen, um die Verbindung solcher Agenten auf unserer Website zu drosseln. Von dort aus wechseln wir zu einem anderen Analysewinkel: Sicherheit.

### Sicherheitsanalyse

In diesem Abschnitt werden wir versuchen, eine Sicherheitsanomalie zu suchen, die als ein unerwartetes Verhalten in den Daten definiert ist; Mit anderen Worten, Datenpunkte, die sich von dem normalen Verhalten unterscheiden, das in den Daten beobachtet wird, auf der Grundlage der Tatsachen, die unser Armaturenbrett uns zeigt.

Wenn du das Armaturenbrett zurücksetzst, indem du es öffnest, wirst du im Balkendiagramm bemerken, dass im Hintergrund der Hits eine beträchtliche Menge von 404 Reaktionen sporadisch passieren:
![404 responses](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_014.jpg)

Klicken Sie auf den **404**-Code in der Legende, um das Dashboard zu filtern und die anderen Visualisierungen und Datentabellen zu aktualisieren. Sie werden in der User-Agent-Datentabelle feststellen, dass ein Agent namens - ist die Quelle einer Menge der **404** Antworten. Also, lasst uns die Analyse filtern, indem wir darauf klicken. Jetzt schauen Sie auf die oben angeforderten Ressourcen:

![Basic-logstash](https://www.packtpub.com/graphics/9781786463005/graphics/image_04_015.jpg)

Top angeforderte Ressourcen von User Agent "-"

Es gibt eine sehr ungewöhnliche, möglicherweise verdächtige Tätigkeit, wie ein Versuch, unsere Website anzugreifen. Grundsätzlich ist die WP-Admin-URI die Ressource für den Zugriff auf die WordPress-Blog-Admin-Konsole. Allerdings ist in unserem Fall die Website kein WordPress Blog.

Neue WordPress-Benutzer haben möglicherweise nicht das Wissen, um entweder ändern oder deaktivieren Sie die Admin-Konsole auf ihrer Website, und könnte auch die Standard-Benutzer / Passwort. Also, meine Vermutung hier ist, dass der Benutzer-Agent versucht, eine Verbindung zu der wp-admin-Ressource und geben Sie die Standard-Anmeldeinformationen, um die vollständige Kontrolle über unsere Website zu nehmen.