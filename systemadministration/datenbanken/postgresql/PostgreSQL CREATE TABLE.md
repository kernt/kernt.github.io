---
tags:
  - postgresql
  - datenbanken
---
Mit dem Befehl CREATE TABLE werden in PostgreSQL neue Tabellen innerhalb einer Datenbank erstellt. Bei der Nutzung des Kommandos legen Sie außerdem direkt unterschiedliche Spezifikationen für die Tabelle und ihre einzelnen Spalten fest.
## Was ist PostgreSQL `CREATE TABLE`?

Der Befehl `CREATE TABLE` wird in [PostgreSQL] genutzt, um eine **neue Tabelle in einer bestehenden Datenbank zu erstellen**. Dabei legen Sie immer auch bereits einen eindeutigen und in der Datenbank einzigartigen Namen für die Tabelle und ihre einzelnen Spalten fest. Auch die Spalten erhalten jeweils einen Namen und einen Datentyp, den sie am Ende enthalten müssen. Außerdem können bereits während der Erstellung Einschränkungen für einzelne oder alle Spalten definiert werden.

> Tipp: Möchten Sie die Einstellungen Ihrer Tabelle zu einem späteren Zeitpunkt verändern, können Sie dafür innerhalb des [Datenbankmanagementsystems] den Befehl [ALTER TABLE] nutzen und so einzelne Spalten bedarfsgenau anpassen.
## Syntax und Funktionsweise von `CREATE TABLE`

Die grundlegende Syntax von PostgreSQL `CREATE TABLE` ist wie folgt:

```sql
CREATE TABLE name_der_tabelle(
spalte1 datentyp PRIMARY KEY,
spalte2 datentyp,
spalte3 datentyp,
…
);
```

Sie nutzen also zunächst den Hauptbefehl `CREATE TABLE`, um PostgreSQL anzuweisen, eine **neue Tabelle anzulegen**. Diese bezeichnen Sie dann mit einem eindeutigen Namen. In Klammern folgen die Bezeichnungen der einzelnen Spalten und eine Festsetzung der erlaubten Datentypen.

Möchten Sie bereits Einschränkungen (Constraints) einbauen, so verändert sich die Syntax wie folgt:

```sql
CREATE TABLE name_der_tabelle(
spalte1 datentyp PRIMARY KEY einschränkung,
spalte2 datentyp einschränkung,
spalte3 datentyp einschränkung,
…
);
```

PostgreSQL unterstützt, abseits von `PRIMARY KEY`, folgende Arten von Einschränkungen:

- `NOT NULL`: Auf diese Weise stellen Sie sicher, dass die jeweilige Spalte keine NULL-Werte enthalten darf.
- `UNIQUE`: Definieren Sie diese Einschränkung, um sicherzugehen, dass alle Werte in einer Spalte oder Kombination von Spalten einzigartig sind.
- `CHECK`: Mit `CHECK` legen Sie Bedingungen fest, die beim Einfügen oder Aktualisieren von Daten erfüllt sein müssen.
- `FOREIGN KEY`: Diese Einschränkung wird benötigt, um Beziehungen zu einer Spalte in einer anderen Tabelle zu setzen.
- `DEFAULT`: Definiert einen Standardwert für eine Spalte, falls kein expliziter Wert beim Einfügen angegeben wird.

Dedizierte Server mit modernsten Prozessoren

- 100 % Enterprise-Hardware
- Minutengenaue Abrechnung
- Nur bei uns: Cloud-Funktionen

## Praxisbeispiel für PostgreSQL `CREATE TABLE`

Die Funktionsweise von `CREATE TABLE` in PostgreSQL wird deutlicher, wenn Sie sie an einem praktischen Beispiel nachvollziehen können. Dafür erstellen wir nun eine neue Tabelle namens „Kundenliste“. Diese soll zunächst vier Spalten enthalten: „ID“, „Name“, „Land“ und „Adresse“. „ID“ definieren wir als `PRIMARY KEY` und die Spalten für „ID“ und „Name“ dürfen nicht leer bleiben. So sieht der entsprechende Code aus:

```sql
CREATE TABLE Kundenliste(
ID INT PRIMARY KEY NOT NULL,
Name VARCHAR(50) NOT NULL,
Land VARCHAR(50),
Adresse VARCHAR(255)
);
```

Nun wird die Datenbank eine leere Tabelle mit diesem Namen und den von Ihnen definierten Spalten anlegen, die Sie im Anschluss mit Werten füllen können. Die Ausgabe der fertig ausgefüllten Tabelle sieht dann in etwa folgendermaßen aus:

|ID|Name|Land|Adresse|
|---|---|---|---|
|1|Max Mustermann|Deutschland|Musterstraße 1, 12345 Musterstadt|
|2|…|…|…|
|3|…|…|…|
## Erstellte Tabellen ansehen mit `\d`

Um sicherzustellen, dass die PostgreSQL-Aktion mit `CREATE TABLE` erfolgreich war, können Sie den Befehl `\d` verwenden. Dieser listet Ihnen alle Tabellen innerhalb einer Datenbank auf. So wird er angewendet:

Den Befehl können Sie alternativ auch nutzen, um eine nähere Beschreibung einer bestimmten Tabelle zu erhalten. Für eine Auflistung aller Spezifikationen in unserem Beispiel von oben nutzen Sie ihn wie folgt:

```postgresql
testdb-# \d Kundenliste
```
