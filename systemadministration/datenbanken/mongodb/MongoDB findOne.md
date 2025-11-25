---
tags:
  - mongodb
  - nosql
  - datenbanken
---
Die MongoDB-Methode findOne eignet sich hervorragend zum Durchsuchen einer Sammlung. Sie gibt jedoch nur ein einziges Ergebnis zurück, was sie für viele Arten von Suchen ungeeignet macht.

## Was ist MongoDB findOne

MongoDB ist ein Datenbankmanagementsystem, das dank seines NoSQL-Ansatzes und seiner bemerkenswerten Skalierbarkeit problemlos große Datenmengen speichern und verwalten kann. Während diese Aspekte den Benutzern erhebliche Vorteile bieten, bedeutet dies auch, dass das System starke Methoden benötigt, um sicherzustellen, dass die Benutzer effektiv in der Datenbank navigieren können. Das System speichert Daten aller Art in Form eines BSON-Dokuments (binäres JSON) und bündelt diese Dokumente in Sammlungen. Wenn Sie eines dieser Dokumente suchen und ausgeben möchten, stehen Ihnen mehrere Möglichkeiten zur Verfügung. Neben der allgemeineren Suchmethode MongoDB find ist MongoDB findOne eine sehr effektive Methode, um große Datenbanken präzise zu filtern. MongoDB findOne durchsucht alle Dokumente und Collections nach bestimmten, vom Benutzer festgelegten Kriterien. Das Besondere an dieser Methode ist, dass sie immer genau ein Dokument zurückgibt. Gibt es nur ein Dokument in der Suchanfrage, wird dieses Dokument zurückgegeben. Wenn mehrere Dokumente mit den vom Benutzer definierten Parametern übereinstimmen, gibt MongoDB findOne das Dokument zurück, das in der natürlichen Reihenfolge der Datenbank zuerst erscheint. Wenn bei der Suche keine Dokumente gefunden werden können, lautet die Ausgabe "null".

## Wie lautet die Syntax für MongoDB findOne?

Unter query können Sie angeben, wie die Methode die Dokumente filtern soll. Unter projection können Sie angeben, welche Felder für das zurückgegebene Dokument angezeigt werden sollen. Verwenden Sie die booleschen Werte 1 (true/include) und 0 (false/exclude), um anzugeben, ob ein Feld einbezogen werden soll. Bleibt dieser Parameter leer, werden alle Felder angezeigt. Mit dem dritten Suchparameter Optionen können Sie die Suche weiter modifizieren und auch die Anzeige ändern. Alle drei Suchparameter sind optional.

> Hinweis: Um Sammlungen mit mehreren Suchparametern effektiv zu durchsuchen, gibt es MongoDB-Abfragen. Die Abfragen basieren auf der MongoDB-Find-Methode. Weitere Informationen zu MongoDB-Abfragen finden Sie in unserem digitalen Leitfaden.

## So erstellen Sie eine Sammlung für Testzwecke

Wenn Sie MongoDB auf Linux, Windows oder Mac installiert haben und MongoDB findOne verwenden möchten, lohnt es sich, eine Testumgebung einzurichten, um mit der Methode zu experimentieren. Wenn Sie noch keine Datenbank eingerichtet haben, lesen Sie unser umfassendes MongoDB-Tutorial, um zu erfahren, wie Sie eine einrichten. In unserem Beispiel unten verwenden wir eine Mitarbeiterdatenbank mit fünf Einträgen. Jeder Eintrag enthält Informationen über den Namen, das Geschlecht und das Alter der Mitarbeiter sowie über die Dauer der Betriebszugehörigkeit. Die Sammlung sieht wie folgt aus:

```bash
db.employee.insertMany ( [
    {
    name : "Smith",
    gender : "Female",
    age : 56,
    year : 2002
    },
    {
    name : "Jones",
    gender : "Female",
    age : 40,
    year : 2017,
    },
    {
    name : "Brown",
    gender : "Male",
    age : 40,
    year : 2019
    },
    {
    name : "Michaels",
    gender : "Female",
    age : 44,
    year : 2015
    },
    name : "Cartwright",
    gender : "Male",
    age : 22,
    year : 2022
    }
]
)
```

## Was sind die verschiedenen Möglichkeiten der Suche mit Mongo DB findOne

Es gibt mehr als eine Möglichkeit, mit MongoDB findOne nach Informationen zu suchen. Sie können ohne Parameter, mit einer ID, mit einem Feld oder mit mehreren Feldern suchen.

### Suche ohne Parameter

Wenn Sie die MongoDB-Methode findOne ohne Parameter verwenden, durchsucht das System Ihre Datenbank und berücksichtigt dabei alle Einträge. In unserem Beispiel bedeutet dies, dass die Methode fünf Einträge identifiziert. Da keines der Dokumente ausgeschlossen wird und die Methode nur ein Ergebnis liefert, wählt MongoDB findOne die erste Person aus, die in die Datenbank eingegeben wurde.

```shell
db.employee.findOne ( )
```

Die Ausgabe ist:

```shell
db.employee.findOne ( )
{
    _id : ObjectID ( "529ete7300of4002bme148om" ),
    name : "Smith",
    gender : "Female",
    age : 56,
    year : 2002
}
```

## Suche nach ID

Normalerweise möchten Sie eine Art von Kriterien für Ihre Suche angeben, damit Sie am Ende das Dokument finden, das Sie tatsächlich benötigen. Eine Möglichkeit, mit MongoDB findOne nach Dokumenten zu suchen, ist die Suche nach einer ID. Das Feld _id_ ist in jedem Dokument eindeutig. In unserem Beispiel steht jede ID für genau einen Mitarbeiter. Wenn Sie MongoDB findOne mit der Objekt-ID ausführen, erhalten Sie genau das Ergebnis, das Sie suchen.

```shell
db.employee.findOne ( { _id : ObjectID ( "582pfh773813tw982qj411l0" ) } )
```

Die Ausgabe sieht wie folgt aus:

```shell
db.employee.findOne ( { _id : ObjectID ( "582pfh773813tw982qj411l0" ) } )
{
    _id : ObjectID ( "582pfh773813tw982qj411l0"
    name : "Cartwright",
    gender : "Male",
    age : 22,
    year : 2022
}
```

## Suche über bestimmte Felder

Wenn Sie die ID nicht kennen oder Ihre Sammlung nach anderen Parametern durchsuchen möchten, können Sie auch MongoDB findOne verwenden, um nach bestimmten Feldern zu suchen. Wenn es nur ein Dokument gibt, das dem Parameter entspricht, wird dieses Dokument angezeigt. Wenn jedoch mehrere Dokumente Ihren Suchkriterien entsprechen, wird nur der erste Eintrag angezeigt. Im folgenden Beispiel suchen wir nach allen Einträgen mit dem Geschlecht "Male". Hier ist der Befehl:

```shell
db.employee.findOne ( { gender : "Male" } )
```

Es gibt zwei Einträge, die diesen Kriterien entsprechen, aber nur einer wird angezeigt. Die Ausgabe zeigt den Mitarbeiter Mr. Brown an:

```shell
db.employee.findOne ( { gender : "Male" } )
{
    _id : ObjectID ( "498p0t173mv489fh63th00kh"
    name : "Brown",
    gender : "Male",
    age : 40,
    year : 2019
}
```

## Suche über mehrere Felder

Sie haben auch die Möglichkeit, Ihre Suche weiter einzuschränken, um Überschneidungen zu vermeiden. Für unsere kleine Beispielsammlung ist dies vielleicht nicht notwendig, aber wenn Sie mit mehreren hundert oder sogar tausenden von Einträgen arbeiten, ist diese Option sehr nützlich. MongoDB findOne erlaubt es Ihnen, mehrere Felder für die Suche zu verwenden. Wenn Sie einen Mitarbeiter nach Geschlecht (männlich) und Alter identifizieren möchten, können Sie Folgendes schreiben:

```shell
db.employee.findOne ( { gender : "Male", age: 40 } )
```

Die Ausgabe zeigt wiederum Mr. Brown, der als einzige Person in der Sammlung männlich und 40 Jahre alt ist. Frau Jones hat das gleiche Alter, aber ihr Geschlecht entspricht nicht den Kriterien, so dass das Ergebnis wie folgt aussieht:

```shell
db.employee.findOne ( { gender : "Male", age: 40 } )
{
    _id : ObjectID ( "498p0t173mv489fh63th00kh"
    name : "Brown",
    gender : "Male",
    age : 40,
    year : 2019
}
```


## Festlegen von Bedingungen für ein Feld mit MongoDB findOne

Es ist auch möglich, Bedingungen für ein bestimmtes Feld zu definieren und diese als Suchkriterien zu verwenden. Im folgenden Beispiel suchen wir nur nach Mitarbeitern, die über 30 Jahre alt sind. 

Dieser Eintrag sieht wie folgt aus:

```shell
db.employee.findOne ( { age : { $gt : 30 } } )
```

Dies schließt Herrn Cartwright aus. Da Frau Smith über 30 ist und die erste Person in der Liste ist, erscheint sie als Ergebnis:

```shell
db.employee.findOne ( { age : { $gt : 30 } } )
{
    _id : ObjectID ( "529ete7300of4002bme148om" ),
    name : "Smith",
    gender : "Female",
    age : 56,
    year : 2002
}
```

## Wie man mit MongoDB findOne Felder ausschließt

Wenn Sie umfangreiche Sammlungen haben, die viele Informationen enthalten, kann die Ausgabe zu viele Informationen enthalten. Glücklicherweise bietet MongoDB findOne auch die Möglichkeit, Felder von der Ausgabe auszuschließen. Im folgenden Beispiel möchten wir ID, Geschlecht und Alter in der Ausgabe nicht anzeigen.

```shell
db.employee.findOne ( { name : "Smith" }, { _id : 0, gender : 0, age : 0 } )
```

Sie werden die folgende Ausgabe erhalten:

```shell
db.employee.findOne ( { name : "Smith" }, { _id : 0, gender : 0, age : 0 } )
{
    name : "Smith",
    year : 2002
}
```

## Was passiert, wenn MongoDB findOne keine Ergebnisse für meine Suche finden kann?

Wenn es keine Ergebnisse für Ihre Suche mit MongoDB findOne gibt, wird trotzdem eine Ausgabe angezeigt. Um zu veranschaulichen, was passiert, suchen wir nach Frau Larkham, die sich nicht in der Sammlung befindet.

```shell
db.employee.findOne ( { name : "Larkham" }  )
```

Die Ausgabe hierfür ist:

```shell
db.employee.findOne ( { name : "Larkham" }  )
null
```