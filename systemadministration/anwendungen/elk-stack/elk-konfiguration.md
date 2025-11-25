---
tags:
  - elk stack
---
# elk Konfiguration

# [Elasticsearch](../elasticsearch)

Elasticsearch-Konfigurationsdateien befinden sich unter dem Konfigurationsordner im Elasticsearch-Installationsverzeichnis. Der config-Ordner hat zwei Dateien, nämlich elasticsearch.yml und logging.yml. Erstere werden verwendet, um die Konfigurationseigenschaften von verschiedenen Elasticsearch-Modulen festzulegen, wie z. B. Netzwerkadresse, Pfade usw., während letztere loggenbezogene Konfigurationen angeben.

Die Konfigurationsdatei befindet sich im YAML-Format und die folgenden Abschnitte sind einige der Parameter, die konfiguriert werden können.
Netzwerkadresse

Um die Adresse anzugeben, in der alle netzwerkbasierten Module binden und veröffentlichen werden:

```bash
network :
    host : 127.0.0.1
```
### Der Name des Clusters

Um einen Produktionscluster zu benennen, der verwendet wird, um Knoten zu entdecken und automatisch zu verbinden:

```bash
cluster:
  name: "{{ els_elk_nodename }}"
```

### Der Knotenname

So ändern Sie den Standardnamen jedes Knotens:

```
node:
  name: els."{{ansible_nodename}}"
```
### Elastische Sucher-Plugins

Elasticsearch hat eine Vielzahl von Plugins, die die Aufgabe der Verwaltung von Indizes, Cluster und so weiter erleichtern. Einige der meist verwendeten sind das Kopf Plugin, Marvel, Sense, Shield, und so weiter, die in den folgenden Kapiteln abgedeckt werden. Werfen wir einen Blick auf das Kopf Plugin hier.

Kopf ist ein einfaches Web-Administrations-Tool für Elasticsearch, das in JavaScript, AngularJS, jQuery und Twitter Bootstrap geschrieben ist. Es bietet eine einfache Möglichkeit, gemeinsame Aufgaben auf einem Elasticsearch-Cluster durchzuführen. Nicht jede einzelne API wird von diesem Plugin abgedeckt, aber es bietet einen REST-Client an, mit dem Sie das volle Potenzial der Elasticsearch API erkunden können.

Um das `elasticsearch-kopf` Plugin zu installieren, führen Sie den folgenden Befehl aus dem Elasticsearch-Installationsverzeichnis aus:

`bin/plugin -install lmenezes/elasticsearch-kopf`

Gehen Sie nun zu dieser Adresse, um die Schnittstelle zu sehen:

`http://localhost:9200 /_plugin/kopf/`.

Sie können eine ähnliche Seite sehen, die Elasticsearch-Knoten, Scherben, eine Anzahl von Dokumenten, Größe und auch die Abfrage der Dokumente indiziert zeigt.
