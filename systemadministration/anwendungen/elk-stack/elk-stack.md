---
tags:
  - elk
  - elk-stack
---
# ELK Stack

Die ELK-Plattform ist eine komplette Log Analytics-Lösung, die auf einer Kombination von drei Open-Source-Tools - Elasticsearch, Logstash und Kibana - aufgebaut ist.
ELK nutzt den Open Source Stack von Elasticsearch für tiefe Suche und Datenanalytik; Logstash für das zentrale Protokollierungsmanagement, das das Verschicken und Weiterleiten der Protokolle von mehreren Servern, Protokollanreicherung und Parsing beinhaltet;
Und schließlich Kibana für kraftvolle und schöne Datenvisualisierungen. ELK-Stack wird derzeit gepflegt und aktiv unterstützt von der Firma Elastic (ehemals Elasticsearch).

* [elk-stack-logstash](../elk-stack-logstash)
* [elk-stack-data-pipeline](../elk-stack-data-pipeline)
* [elk-stack-einrichten](../elk-stack-einrichten)
* [elk-stack-kibana-visualisierung](../elk-stack-kibana-visualisierung)
* [elk-stack-kibana5-apache-log](../elk-stack-kibana5-apache-log)
* [elk-stack-logstash-konfiguration](../elk-stack-logstash-konfiguration)
# [elk-stack-logstash-plugins](elk-stack-logstash-plugins.md)

# [elk-stack-logstsh-input-dataset](../elk-stack-logstsh-input-dataset)

# [elk-stack-logstash](../elk-stack-logstash)

Schauen wir uns einen kurzen Überblick über jedes dieser Systeme:

### [Elasticsearch](../elasticsearch)

Elasticsearch ist eine verteilte Open-Source-Suchmaschine, die auf Apache Lucene basiert und unter einer Apache 2.0-Lizenz veröffentlicht wurde (was bedeutet, dass sie kostenlos heruntergeladen, verwendet und modifiziert werden kann). Es bietet horizontale Skalierbarkeit, Zuverlässigkeit und Multitenant-Fähigkeit für die Echtzeit-Suche. Elasticsearch-Funktionen sind über JSON über eine RESTful API verfügbar. Die Suchfunktionen werden durch eine schema-less Apache Lucene Engine unterstützt, die es erlaubt, Daten dynamisch zu indexieren, ohne vorher die Struktur zu kennen. Elasticsearch ist in der Lage, schnelle Suchantworten zu erreichen, weil es Indizierung verwendet, um über die Texte zu suchen.

Elasticsearch wird von vielen großen Firmen wie GitHub, SoundCloud, FourSquare, Netflix und vielen anderen verwendet. Einige der Anwendungsfälle sind wie folgt:

* **Wikipedia**: Das nutzt Elasticsearch, um eine Volltextsuche zu liefern und Funktionalitäten wie Such-as-you-type und did-you-mean Vorschläge zu bieten.

* **The Guardian**: Das nutzt Elasticsearch, um 40 Millionen Dokumente pro Tag zu verarbeiten, die Echtzeit-Analytik des Standortverkehrs über die gesamte Organisation hinweg zu vermitteln und das Publikum Engagement besser zu verstehen.

* **StumbleUpon**: Das nutzt Elasticsearch, um intelligente Suchanfragen über seine Plattform zu machen und bietet großen Empfehlungen für Millionen von Kunden.

* **SoundCloud**: Dies nutzt Elasticsearch, um Echtzeit-Suchfunktionen für Millionen von Benutzern über Regionen bereitzustellen.

* **GitHub**: Dies nutzt Elasticsearch, um über 8 Millionen Code-Repositories zu indizieren und mehrere Ereignisse über die Plattform zu indexieren und somit Echtzeit-Suchfunktionen zu bieten.

Einige der Hauptmerkmale von Elasticsearch sind:

* Es ist ein Open-Source-verteilter, skalierbarer und hoch verfügbarer Echtzeit-Dokumenten-speicher

* Es bietet Echtzeit-Such- und Analysefunktionen

* Es bietet eine anspruchsvolle RESTful API zu spielen mit Lookup, und verschiedene Funktionen, wie mehrsprachige Suche, Geolocation, Autovervollständigung, kontextuelle did-you-mean Vorschläge und Ergebnis Snippets

* Es kann horizontal leicht skaliert werden und bietet einfache Integration mit Cloud-basierten Infrastrukturen wie AWS und anderen
### [Logstash](../logstash)

Logstash ist eine Datenpipeline, die eine Vielzahl von strukturierten und unstrukturierten Daten und Ereignissen, die über verschiedene Systeme generiert werden, sammelt, analysiert und analysiert. Es stellt Plugins zur Verfügung, um eine Verbindung zu verschiedenen Arten von Eingangsquellen und Plattformen herzustellen und ist so konzipiert, dass es Protokolle, Ereignisse und unstrukturierte Datenquellen für die Verteilung in eine Vielzahl von Ausgängen mit der Verwendung seiner Ausgangs-Plugins, nämlich Datei, Stdout (als Ausgabe) effizient verarbeiten kann Auf Konsole mit Logstash) oder Elasticsearch.

Es hat folgende Hauptmerkmale:

* Zentrale Datenverarbeitung: Logstash hilft beim Aufbau einer Datenpipeline, die die Datenverarbeitung zentralisieren kann. Mit der Verwendung einer Vielzahl von Plugins für Input und Output, kann es eine Vielzahl von verschiedenen Input-Quellen zu einem einzigen gemeinsamen Format zu konvertieren.

* Unterstützung für benutzerdefinierte Log-Formate: Protokolle, die von verschiedenen Anwendungen geschrieben werden, haben oft spezielle Formate, die für die Anwendung spezifisch sind. Logstash hilft bei der Bearbeitung und Bearbeitung von benutzerdefinierten Formaten in großem Maßstab. Es bietet Unterstützung, um eigene Filter für die Tokenisierung zu schreiben und bietet auch gebrauchsfertige Filter.

* Plugin-Entwicklung: Benutzerdefinierte Plugins können entwickelt und veröffentlicht werden, und es gibt eine Vielzahl von benutzerdefinierten Plugins bereits verfügbar.

### [filebeat](../filebeat)

### [Kibana](../kibana)

Kibana ist eine Open Source Apache 2.0 lizenzierte Datenvisualisierungsplattform, die bei der Visualisierung jeglicher Art von strukturierten und unstrukturierten Daten hilft, die in Elasticsearch Indizes gespeichert sind. Kibana ist komplett in HTML und JavaScript geschrieben. Es nutzt die leistungsstarken Such- und Indizierungsfunktionen von Elasticsearch, die durch seine RESTful API ausgesetzt sind, um leistungsstarke Grafiken für die Endbenutzer anzuzeigen. Von der grundlegenden Business Intelligence bis hin zum Echtzeit-Debugging spielt Kibana seine Rolle durch die Darstellung von Daten durch schöne Histogramme, Geomaps, Kreisdiagramme, Graphen, Tabellen und so weiter.

Kibana macht es leicht, große Datenmengen zu verstehen. Seine einfache browserbasierte Schnittstelle ermöglicht es Ihnen, schnell erstellen und teilen dynamische Dashboards, die Änderungen an Elasticsearch Abfragen in Echtzeit anzeigen.

Einige der wichtigsten Merkmale von Kibana sind wie folgt:

* Es bietet flexible Analysen und eine Visualisierungsplattform für Business Intelligence.

* Es bietet Echtzeit-Analyse, Verdichtung, Charting und Debugging-Fähigkeiten.

* Es bietet eine intuitive und benutzerfreundliche Schnittstelle, die sehr anpassbar ist durch einige Drag & Drop-Funktionen und Ausrichtungen wie und wann benötigt.

* Es ermöglicht das Speichern des Armaturenbretts und die Verwaltung von mehr als einem Armaturenbrett. Dashboards können einfach in verschiedenen Systemen geteilt und eingebettet werden.

* Es ermöglicht das Teilen von Snapshots von Protokollen, die Sie bereits durchsucht haben, und isoliert mehrere Problemtransaktionen

* [Kibana Settings](https://www.elastic.co/guide/en/kibana/current/settings.html)
