---
tags:
  - prometheus
  - monitoring
---
# Installation von  Prometheus

Prometheus ist ein kostenloses Open-Source-Software-Ökosystem,
mit dem Messdaten aus unseren Anwendungen erfasst und in einer Datenbank gespeichert werden können,
insbesondere in einer auf Zeitreihen basierenden Datenbank.

Es ist ein sehr leistungsfähiges Überwachungssystem,
das sich für dynamische Umgebungen eignet.
Prometheus ist in Go geschrieben und verwendet die Abfragesprache für die Datenverarbeitung.
Prometheus bietet Metriken für CPU, Arbeitsspeicher, Festplattennutzung, E/A, Netzwerkstatistik, MySQL-Server und Nginx.

# Prometheus Ansible

[prometheus-community](ansible-galaxy collection install  git+https://github.com/prometheus-community/ansible.git)

`ansible-galaxy collection install  git+https://github.com/prometheus-community/ansible.git`

[[ansible-galaxy]]

# Prometheus Konfiguration testen mit prom

`promtool check config` 

## Prometheus Eigenschaften

Prometheus Hauptmerkmale:

* ein mehrdimensionales Datenmodell mit Zeitreihendaten, die durch Metriknamen und Schlüssel/Wert-Paare identifiziert werden
* PromQL, eine flexible Abfragesprache , um diese Dimensionalität zu nutzen
* keine Abhängigkeit von verteilter Speicherung; einzelne Server-Knoten sind autonom
* Die Zeit Serien Erfassung erfolgt über ein Pull-Modell über HTTP
* Push-Zeitreihen werden über ein zwischengeschaltetes Gateway unterstützt
* Ziele werden über Service Discovery oder statische Konfiguration ermittelt
* Unterstützung für mehrere Grafik- und Dashboard-Modi

`api_http_requests_total{method="POST", handler="/messages"}`

Dies ist die gleiche Schreibweise, die OpenTSDB verwendet.

### Prometheus abfragen mit PromQL

Datentypen für die PromQL Ausdruckssprache:

* **Sofortvektor** - eine Reihe von Zeitreihen, die einen einzelnen Abtastwert für jede Zeitreihe enthalten und alle den gleichen Zeitstempel haben
* **Entfernungsvektor** - eine Reihe von Zeitreihen eine Reihe von Datenpunkten über die Zeit für jede Zeitreihe enthält,
* **Skalar** - ein einfacher numerischer Fließkommawert
* **String** - ein einfacher Stringwert; derzeit nicht verwendet

#### String-Literale

```sh
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t`
```

#### Float Literale

Zeitreihen-Selektoren
Sofortige Vektor-Selektoren

`http_requests_total`

`http_requests_total{job="prometheus",group="canary"}`

* **=:** Wählen Sie Beschriftungen aus, die der angegebenen Zeichenfolge genau entsprechen.
* **!=:** Wählen Sie Beschriftungen aus, die der angegebenen Zeichenfolge nicht entsprechen.
* **=~:** Wählen Sie Beschriftungen aus, die mit der angegebenen Zeichenfolge (oder Teilzeichenfolge) übereinstimmen.
* **!~:** Wählen Sie Beschriftungen aus, die der angegebenen Zeichenfolge (oder Teilzeichenfolge) nicht entsprechen.

**Bereichsvektor-Selektoren**

* **s** - Sekunden
* **m** - Protokoll
* **h** - Std
* **d** - Tage
* **w** - Wochen
* **y** - Jahre

`http_requests_total{job="prometheus"}[5m]`

## Komponenten

* der **Prometheus- Hauptserver**, der Zeitreihendaten speichert und speichert
* Client-Bibliotheken zur Instrumentierung von Anwendungscode
* ein **Push-Gateway** zur Unterstützung kurzlebiger Jobs
* Spezial- Exports für Dienste wie HAProxy, StatsD, Graphit, usw.
* ein **Alertmanager** , der Alerts behandelt
* verschiedene unterstützungstools

### Prometheus Server

Prometheus-Server, der die Datenbank darstellt, in der die eingesammelten Daten abgelegt werden.
Er erfüllt in einem Setup drei Aufgaben: Zunächst empfängt er alle Messdaten, die in einem Setup anfallen.
Die integrierte Storage-Engine des Prometheus-Servers legt diese organisiert auf der Festplatte des Servers ab, auf dem der Prometheus-Server läuft.
Zudem bietet der Prometheus-Server eine API-Schnittstelle, über die sich die Daten auf Basis eines festgelegten Standards auch wieder auslesen lassen.
Dafür hat das SoundCloud-Team sogar eine eigene Abfragesprache entwickelt, die auf den Namen "PromQL" hört. Ein genauerer Blick auf die drei Aufgaben des Servers hilft, um die grundlegenden Design-Ansätze der Lösung zu verstehen.


## Quellen

* [monitor-ubuntu-server-with-prometheus](https://www.howtoforge.com/tutorial/monitor-ubuntu-server-with-prometheus/)
* [PromQL for Humans](https://timber.io/blog/promql-for-humans/)
* [Metrics, tracing, and logging](https://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html)
* [ftsdb format](https://github.com/prometheus/prometheus/blob/master/tsdb/docs/format/README.md)
* [Default Ports](https://github.com/prometheus/prometheus/wiki/Default-port-allocations)
* [prometheus exampels](https://sbcode.net/prometheus/recording-rules/)
* [awesome-prometheus-alerts](https://samber.github.io/awesome-prometheus-alerts/rules.html)