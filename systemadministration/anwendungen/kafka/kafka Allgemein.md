---
tags:
  - kafka
  - cloud
---

Der Betrieb von Anwendungen, Webdiensten und Serveranwendungen, um nur einige zu nennen, stellt die Betreiber vor eine Vielzahl von Herausforderungen. Eine der häufigsten Herausforderungen besteht beispielsweise darin, sicherzustellen, dass Datenströme ungehindert übertragen und so schnell und effizient wie möglich verarbeitet werden. Die Messaging- und Streaming-Anwendung Apache Kafka ist eine Software, die diese Herausforderung erheblich vereinfacht. Ursprünglich als Message-Queuing-Dienst für LinkedIn entwickelt, bietet diese Open-Source-Lösung heute eine umfassende Lösung für die Speicherung, Übertragung und Verarbeitung von Daten.

# Was ist Kafka? 

Apache Kafka ist eine plattformunabhängige Open-Source-Anwendung, die zur Apache Software Foundation gehört und sich auf die Verarbeitung von Datenströmen konzentriert. Das Projekt wurde ursprünglich 2011 von LinkedIn, dem Unternehmen hinter dem gleichnamigen sozialen Netzwerk für Berufstätige, ins Leben gerufen. Das Ziel war die Entwicklung einer Nachrichtenwarteschlange. Seit ihrer lizenzfreien Einführung (Apache 2.0) wurden die Möglichkeiten dieser Software stark erweitert, so dass aus einer einfachen Warteschlangenanwendung eine leistungsstarke Streaming-Plattform mit einer Vielzahl von Funktionen wurde. Sie wird von bekannten Unternehmen wie Netflix, Microsoft und Airbnb genutzt.

[Netflix Data Pipline mit kafka](https://youtu.be/6ocfbpxBobQ)

Confluent wurde 2014 von den ursprünglichen Entwicklern von Apache Kafka gegründet und bietet mit Confluent Platform die vollständigste Version von Apache Kafka. Es erweitert das Programm um zusätzliche Funktionen, von denen einige ebenfalls Open Source sind, während andere kommerziell sind.


# Was sind die Kernfunktionen von Apache Kafka

Apache Kafka wurde in erster Linie entwickelt, um die Übertragung und Verarbeitung von Datenströmen zu optimieren, die über eine direkte Verbindung zwischen dem Datenempfänger und der Datenquelle übertragen werden. Kafka fungiert als Messaging-Instanz zwischen dem Sender und dem Empfänger und bietet Lösungen für die üblichen Herausforderungen, die bei dieser Art von Verbindung auftreten. Die Apache-Plattform bietet beispielsweise eine Lösung für die Unmöglichkeit, Daten oder Nachrichten zwischenzuspeichern, wenn der Empfänger nicht verfügbar ist (z. B. aufgrund von Netzwerkproblemen). Darüber hinaus verhindert eine richtig eingerichtete Kafka-Warteschlange, dass der Sender den Empfänger überlastet. Diese Art von Situation tritt immer dann auf, wenn Informationen schneller gesendet werden, als sie bei einer direkten Verbindung empfangen und verarbeitet werden können. Schließlich ist die Kafka-Software auch ideal für Situationen, in denen das Zielsystem die Nachricht empfängt, aber während der Verarbeitung abstürzt. Während der Absender normalerweise davon ausgehen würde, dass die Verarbeitung trotz des Absturzes stattgefunden hat, meldet Apache Kafka den Fehler an den Absender. Im Gegensatz zu reinen Message-Queuing-Diensten wie Datenbanken ist Apache Kafka fehlertolerant. Das bedeutet, dass die Software den Anforderungen an die weitere Verarbeitung von Nachrichten und Daten gerecht wird. In Verbindung mit der hohen Skalierbarkeit und der Fähigkeit, übermittelte Informationen auf eine beliebige Anzahl von Systemen zu verteilen (verteiltes Transaktionsprotokoll), macht dies Apache Kafka zu einer hervorragenden Lösung für alle Dienste, die eine schnelle Speicherung und Verarbeitung von Daten bei gleichzeitig hoher Verfügbarkeit gewährleisten müssen.

# Ein Überblick über die Architektur von Apache Kafka

Apache Kafka läuft als Cluster auf einem oder mehreren Servern, die sich über mehrere Rechenzentren erstrecken können. Die einzelnen Server im Cluster, die so genannten Broker, speichern und kategorisieren eingehende Datenströme in Themen. Die Daten werden in Partitionen aufgeteilt, innerhalb des Clusters repliziert und verteilt und mit einem Zeitstempel versehen. Dadurch gewährleistet die Streaming-Plattform eine hohe Verfügbarkeit und eine schnelle Lesezugriffszeit. Apache Kafka unterscheidet zwischen normalen Topics und komprimierten Topics. In normalen Topics kann Kafka Nachrichten löschen, sobald die Speicherdauer oder das Speicherlimit überschritten ist, während sie in verdichteten Topics keinen zeitlichen oder räumlichen Beschränkungen unterliegen.

Anwendungen, die Daten in einen Kafka-Cluster schreiben, werden als Produzenten bezeichnet, während Anwendungen, die Daten aus einem Kafka-Cluster lesen, als Konsumenten bezeichnet werden. Die zentrale Komponente, auf die Produzenten und Konsumenten bei der Verarbeitung von Datenströmen zugreifen, ist eine Java-Bibliothek namens Kafka Streams. Durch die Unterstützung des transaktionalen Schreibens wird sichergestellt, dass Nachrichten nur einmal zugestellt werden (ohne Duplikate). Dies wird als Exact-once-Delivery bezeichnet.

> Hinweis: Die Kafka Streams Java-Bibliothek ist die empfohlene Standardlösung für die Verarbeitung von Daten in Kafka-Clustern. Sie können Apache Kafka jedoch auch mit anderen Datenstromverarbeitungssystemen verwenden.

# Die technischen Grundlagen: Die Schnittstellen von Kafka

Diese Software bietet fünf verschiedene grundlegende Schnittstellen, um Anwendungen den Zugang zu Apache Kafka zu ermöglichen: 
- Kafka Producer: Die Kafka Producer API ermöglicht es Anwendungen, Datenströme an den/die Broker in einem Apache-Cluster zu senden, um sie zu kategorisieren und zu speichern (in den zuvor genannten Topics).
- Kafka Consumer: Die Kafka Consumer API ermöglicht Apache Kafka-Konsumenten den Lesezugriff auf Daten, die in den Topics des Clusters gespeichert sind.
- Kafka Streams: Mit der Kafka-Streams-API kann eine Anwendung als Stream-Prozessor fungieren, um eingehende Datenströme in ausgehende Datenströme umzuwandeln. 
- Kafka Connect: Die Kafka Connect API ermöglicht es, wiederverwendbare Producer und Consumer zu erstellen, die Kafka-Themen mit bestehenden Anwendungen oder Datenbanksystemen verbinden. 
- Kafka AdminClient: Die Kafka AdminClient API ermöglicht die einfache Verwaltung und Kontrolle von Kafka-Clustern.

Die Kommunikation zwischen Client-Anwendungen und einzelnen Servern in Apache-Clustern erfolgt über ein einfaches, leistungsfähiges und sprachunabhängiges Protokoll auf Basis von TCP. Die Entwickler stellen standardmäßig einen Java-Client für Apache Kafka zur Verfügung, aber es gibt auch Clients in einer Vielzahl anderer Sprachen wie PHP, Python, C/C++, Ruby, Perl und Go.

# Anwendungsszenarien für Apache Kafka

Apache Kafka wurde von Anfang an für einen hohen Lese- und Schreibdurchsatz konzipiert. In Kombination mit den bereits erwähnten APIs und seiner hohen Flexibilität, Skalierbarkeit und Fehlertoleranz macht dies die Open-Source-Software für eine Vielzahl von Anwendungsfällen attraktiv. Apache Kafka eignet sich besonders gut für die folgenden Anwendungen:

- Veröffentlichen und Abonnieren von Datenströmen: Dieses Open-Source-Projekt begann mit der Verwendung von Apache Kafka als Nachrichtensystem. Obwohl die Funktionen der Software erweitert wurden, eignet sie sich nach wie vor am besten für die direkte Nachrichtenübertragung über das Warteschlangensystem sowie für die Übertragung von Broadcast-Nachrichten.
- Verarbeitung von Datenströmen: Apache Kafka ist ein leistungsfähiges Werkzeug für Anwendungen, die in Echtzeit auf bestimmte Ereignisse reagieren müssen und zu diesem Zweck Datenströme möglichst schnell und effektiv verarbeiten müssen. 
- Speichern von Datenströmen: Apache Kafka kann auch als fehlertolerantes, verteiltes Speichersystem eingesetzt werden, egal ob Sie 50 Kilobyte oder 50 Terabyte konsistente Daten auf dem/den Server(n) speichern müssen.

Natürlich lassen sich all diese Elemente beliebig kombinieren, so dass Apache Kafka als komplexe Streaming-Plattform Daten nicht nur speichern und jederzeit verfügbar machen, sondern auch in Echtzeit verarbeiten und mit allen gewünschten Anwendungen und Systemen verbinden kann

Ein Überblick über gängige Anwendungsfälle für Apache Kafka: 

- Nachrichtensystem 
- Webanalyse 
- Speichersystem 
- Datenstromprozessor 
- Ereignisbeschaffung 
- Protokolldateianalyse und -verwaltung
- Überwachungslösungen 
- Transaktionsprotokoll

https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330