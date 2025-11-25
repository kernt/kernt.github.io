---
tags:
  - mongodb
  - nosql
  - datenbanken
---
Die Aggregation in MongoDB ist ein wertvolles Werkzeug zum Analysieren und Filtern von Datenbanken. Das Pipeline-System ermöglicht es, Abfragen zu spezifizieren, die hochgradig angepasste Ausgaben ermöglichen.


# Was ist Aggregation in MongoDB

MongoDB ist eine nicht-relationale und dokumentenorientierte Datenbank, die für den Einsatz mit großen und vielfältigen Datenmengen konzipiert ist. Durch den Verzicht auf starre Tabellen und den Einsatz von Techniken wie Sharding (Speicherung von Daten auf verschiedenen Knoten) kann die NoSQL-Lösung horizontal skalieren und bleibt dabei äußerst flexibel und ausfallsicher. Dokumente im binären JSON-Format BSON werden in Collections gebündelt und können mit der MongoDB Query Language (MQL) abgefragt und bearbeitet werden. Obwohl diese Sprache viele Möglichkeiten bietet, ist sie für die Datenanalyse nicht geeignet (oder vielleicht nicht geeignet genug). Aus diesem Grund bietet MongoDB die Aggregation an. In der Informatik bezieht sich dieser Begriff auf verschiedene Prozesse. In MongoDB bezieht sich Aggregation auf die Analyse und Zusammenfassung von Daten mit Hilfe verschiedener Operationen, um ein einziges und eindeutiges Ergebnis zu erhalten. Dabei werden Daten aus einem oder mehreren Dokumenten analysiert und nach benutzerdefinierten Faktoren gefiltert. In den folgenden Abschnitten gehen wir nicht nur auf die Möglichkeiten ein, die die MongoDB-Aggregation für eine umfassende Datenanalyse bietet, sondern zeigen auch anhand von Beispielen, wie Sie die Aggregatmethode ( ) in einem Datenbankmanagementsystem einsetzen können.

# Was benötige ich für die MongoDB-Aggregation?

Es gibt nur wenige Voraussetzungen für die Verwendung der Aggregation in MongoDB. Die Methode wird in der Shell ausgeführt und funktioniert nach logischen Regeln, die Sie an die Bedürfnisse Ihrer Analyse anpassen können. Um die Aggregation in Mongo DB zu nutzen, muss MongoDB bereits auf Ihrem Computer installiert sein. Falls dies nicht der Fall ist, finden Sie in unserem umfassenden MongoDB-Tutorial Informationen zum Herunterladen, Installieren und Ausführen der Datenbank. Außerdem sollten Sie eine leistungsfähige Firewall verwenden und sicherstellen, dass Ihre Datenbank gemäß allen aktuellen Sicherheitsstandards eingerichtet ist. Um die Aggregation in MongoDB durchzuführen, benötigen Sie Administrationsrechte. Die Datenbank funktioniert plattformübergreifend, so dass die unten beschriebenen Schritte für alle Betriebssysteme gelten.

# Was ist die Pipeline im MongoDB-Aggregationsrahmen

In MongoDB können Sie einfache Suchen oder Abfragen durchführen, wobei die Datenbank die Ergebnisse sofort anzeigt. Diese Methode ist jedoch sehr begrenzt, da sie nur Ergebnisse anzeigen kann, die bereits in den gespeicherten Dokumenten vorhanden sind. Diese Art der Abfrage ist nicht für tiefgreifende Analysen, wiederkehrende Muster oder für die Ableitung weiterer Informationen gedacht. Manchmal müssen verschiedene Quellen innerhalb einer Datenbank berücksichtigt werden, um sinnvolle Schlussfolgerungen zu ziehen. Für solche Situationen wird die MongoDB-Aggregation verwendet. Um solche Ergebnisse zu erzielen, verwendet die Methode aggregate () Pipelines.

## Die Rolle der Pipeline

Aggregationspipelines in MongoDB sind Prozesse, bei denen vorhandene Daten mit Hilfe verschiedener Schritte analysiert und gefiltert werden, um das von den Benutzern gesuchte Ergebnis anzuzeigen. Diese Schritte werden als Stages bezeichnet. Je nach Bedarf können ein oder mehrere Schritte eingeleitet werden. Diese werden nacheinander ausgeführt und verändern Ihre ursprüngliche Eingabe so, dass am Ende die Ausgabe (die gesuchten Informationen) angezeigt werden kann. Während die Eingabe aus zahlreichen Daten besteht, ist die Ausgabe (d. h. das Endergebnis) singulär. Wir werden die verschiedenen Stufen der MongoDB-Aggregation später in diesem Abschnitt erläutern.


## Syntax der MongoDB-Aggregationspipeline

Zunächst lohnt es sich, einen kurzen Blick auf die Syntax der Aggregation in MongoDB zu werfen. Die Methode ist immer nach dem gleichen Schema aufgebaut und kann an Ihre spezifischen Anforderungen angepasst werden. Die Grundstruktur sieht wie folgt aus:

```shell
db.collection_name.aggregate ( pipeline, options )
```

Hier ist collection_name der Name der betreffenden Sammlung. Die Stufen der MongoDB-Aggregation sind unter pipeline aufgeführt. options kann für weitere optionale Parameter verwendet werden, die die Ausgabe definieren.

## Stufen der Pipeline

Es gibt zahlreiche Stufen für die Aggregationspipeline in MongoDB. Die meisten von ihnen können innerhalb einer Pipeline mehrfach verwendet werden. Es würde den Rahmen dieses Artikels sprengen, hier alle Optionen aufzulisten, zumal einige nur für ganz bestimmte Vorgänge erforderlich sind. Um Ihnen jedoch eine Vorstellung von den einzelnen Schritten zu geben, werden hier einige der am häufigsten verwendeten aufgeführt:

- $Zahl: Diese Stufe gibt an, wie viele BSON-Dokumente für die Stufe oder Stufen in der Pipeline berücksichtigt wurden. 
- $group: Diese Stufe sortiert und bündelt Dokumente nach bestimmten Parametern. $limit: Begrenzt die Anzahl der Dokumente, die an die nächste Stufe in der Pipeline übergeben werden. 
- $match: Mit der Stufe $match schränken Sie die Dokumente ein, die für die folgende Stufe verwendet werden. 
- $out: Diese Stufe wird verwendet, um die Ergebnisse der MongoDB-Aggregation in die Sammlung aufzunehmen. Diese Stufe kann nur am Ende einer Pipeline verwendet werden. 
- $project: Verwenden Sie $project, um bestimmte Felder aus einer Sammlung auszuwählen. 
- $skip: In dieser Phase wird eine bestimmte Anzahl von Dokumenten ignoriert. Sie können dies mit einer Option angeben. 
- $sort: Diese Operation sortiert die Dokumente in der Sammlung des Benutzers. Darüber hinaus werden die Dokumente jedoch nicht verändert. 
- $unset: $unset schließt bestimmte Felder aus. Es bewirkt das Gegenteil von dem, was $project tut.

## Ein Beispiel für die Aggregation in MongoDB

Damit Sie besser verstehen, wie die Aggregation in MongoDB funktioniert, zeigen wir Ihnen einige Beispiele für die verschiedenen Stufen und wie Sie sie verwenden. Um die MongoDB-Aggregation zu verwenden, öffnen Sie die Shell als Administrator. Normalerweise wird zunächst eine Testdatenbank angezeigt. Wenn Sie eine andere Datenbank verwenden wollen, benutzen Sie den Befehl use. Für dieses Beispiel stellen wir uns eine Datenbank vor, die die Daten von Kunden enthält, die ein bestimmtes Produkt gekauft haben. Der Einfachheit halber hat diese Datenbank nur zehn Dokumente, die alle gleich strukturiert sind:

```shell
{
	"name" : "Smith",
	"city" : "Los Angeles",
	"country" : "United States",
	"quantity" : 14
}
```

Die folgenden Informationen über die Kunden wurden aufgenommen: Name, Wohnort, Land und die Anzahl der gekauften Produkte. Wenn Sie die Aggregation in MongoDB ausprobieren möchten, können Sie die Methode insertMany ( ) verwenden, um alle Dokumente mit Kundendaten zu der Sammlung "customers" hinzuzufügen:

```shell
db.customers.insertMany ( [
	{ "name" : "Smith", "city" : "Los Angeles", "country" : "United States", "quantity" : 14 },
	{ "name" : "Meyer", "city" : "Hamburg", "country" : "Germany", "quantity" : 26 },
	{ "name" : "Lee", "city" : "Birmingham", "country" : "England", "quantity" : 5 },
	{ "name" : "Rodriguez", "city" : "Madrid", "country" : "Spain", "quantity" : 19 },
	{ "name" : "Nowak", "city" : "Krakow", "country" : "Poland", "quantity" : 13 },
{ "name" : "Rossi", "city" : "Milano", "country" : "Italy", "quantity" : 10 },
{ "name" : "Arslan", "city" : "Ankara", "country" : "Turkey", "quantity" : 18 },
{ "name" : "Martin", "city" : "Lyon", "country" : "France", "quantity" : 9 },
{ "name" : "Mancini", "city" : "Rome", "country" : "Italy", "quantity" : 21 },
{ "name" : "Schulz", "city" : "Munich", "country" : "Germany", "quantity" : 2 }
] )
```

Es wird eine Liste der Objekt-IDs für jedes einzelne Dokument angezeigt.

## Wie man $match verwendet

Um die Möglichkeiten der Aggregation in MongoDB zu veranschaulichen, wenden wir zunächst die Stufe $match auf unsere Sammlung "customers" an. Ohne zusätzliche Parameter würde dies einfach die komplette Liste der oben aufgeführten Kundendaten ausgeben. Im folgenden Beispiel haben wir jedoch die Anweisung gegeben, uns nur Kunden aus Italien anzuzeigen. Hier ist der Befehl:

```shell
db.customers.aggregate ( [
	{ $match : { "country" : "Italy" } }
] )
```

Jetzt werden Ihnen nur noch die Objekt-IDs und Informationen der beiden Kunden aus Italien angezeigt.

## Begrenzen Sie die Ausgabe mit $project

Bei den bisher verwendeten Stufen werden Sie sehen, dass die Ausgabe relativ umfangreich ist. So wird beispielsweise neben den eigentlichen Informationen in den Dokumenten immer auch die Objekt-ID ausgegeben. Sie können $project in der MongoDB-Aggregationspipeline verwenden, um zu bestimmen, welche Informationen ausgegeben werden sollen. Dazu setzen wir den Wert 1 für erforderliche Felder und 0 für Felder, die nicht in die Ausgabe aufgenommen werden müssen. In unserem Beispiel wollen wir nur den Kundennamen und die Anzahl der gekauften Produkte sehen. Zu diesem Zweck geben wir Folgendes ein:

```shell
db.customers.aggregate ( [
	{ $project : { _id : 0, name : 1, city : 0, country : 0, quantity : 1 } }
] )
```

## Kombinieren mehrerer Stufen mit Aggregation in MongoDB

Die MongoDB-Aggregation bietet Ihnen auch die Möglichkeit, mehrere Stufen nacheinander anzuwenden. Diese werden dann nacheinander durchlaufen, und am Ende gibt es eine Ausgabe, die alle gewünschten Parameter berücksichtigt. Wenn Sie z. B. nur die Namen und Käufe von US-Kunden in absteigender Reihenfolge anzeigen möchten, können Sie die oben beschriebenen Stufen wie folgt verwenden:

```shell
db.customers.aggregate ( [
	{ $match : { "country" : "United States" } }
	{ $project : { _id : 0, name : 1, city : 0, country : 0, quantity : 1 } }
	{ $sort : { "quantity" : -1 } }
] )
```


