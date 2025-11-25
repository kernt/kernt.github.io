---
tags:
  - postgresql
  - datenbanken
---
PostgreSQL `INSERT INTO` wird genutzt, um eine oder mehrere neue Zeilen in eine Tabelle hinzuzufügen. Dabei werden auch bereits die jeweiligen Werte hinterlegt.

## Was PostgreSQL `INSERT INTO`?

Mit dem Befehl INSERT INTO können Sie in [PostgreSQL] **neue Zeilen in eine Tabelle einfügen**. Dabei ist es sowohl möglich, eine einzelne neue Zeile einzufügen, als auch mehrere Zeilen hinzuzufügen. Im Rahmen der Nutzung von PostgreSQL `INSERT` werden die Spalten, die auch bei der Erstellung der Tabelle definiert wurden, mit angegeben. Auch die Werte, die in der neuen Zeile abgebildet werden sollen, sind bereits in dem Befehl enthalten.

## PostgreSQL `INSERT`: Syntax und Funktionsweise

Die grundsätzliche Syntax von PostgreSQL `INSERT INTO` sieht wie folgt aus:

```postgresql
INSERT INTO name_der_tabelle (spalte1, spalte2, spalte3, …, spalteN)
VALUES (wert1, wert2, wert3, …, wertN);
```

Bei der Verwendung von PostgreSQL `INSERT INTO` geben Sie also zunächst die Tabelle an, in der Sie Ihre Anpassungen vornehmen möchten. Es folgen die einzelnen Spalten, wobei Sie diesen Parameter **theoretisch weglassen** können, insofern Sie Werte für alle hinterlegten Spalten einsetzen. In diesem Fall sieht die Syntax so aus:

```postgresql
INSERT INTO name_der_tabelle
VALUES (wert1, wert2, wert3, …, wertN);
```

In jedem Fall müssen Sie die einzelnen Werte in der richtigen Reihenfolge hinterlegen. Diese werden in die einzelnen Spalten von links nach rechts eingesetzt.

## Beispiel für den PostgreSQL-Befehl `INSERT INTO`

Wie PostgreSQL `INSERT INTO` in der Praxis funktioniert, kann man am besten mit einem praktischen Beispiel veranschaulichen. Dafür erstellen wir mit [PostgreSQL CREATE TABLE](https://www.ionos.at/digitalguide/server/konfiguration/postgresql-create-table/ "PostgreSQL CREATE TABLE") eine Tabelle namens „Kundenliste“. Diese enthält vier Spalten mit den Bezeichnungen „ID“, „Name“, „Stadt“ und „Adresse“. So sieht der entsprechende Code aus:

```postgresql
CREATE TABLE Kundenliste(
ID INT PRIMARY KEY NOT NULL,
Name VARCHAR(50) NOT NULL,
Stadt VARCHAR(50),
Adresse VARCHAR(255)
);
```

Um nun eine Zeile einzufügen, nutzen wir PostgreSQL `INSERT`:

```postgresql
INSERT INTO Kundenliste (ID, NAME, STADT, ADRESSE)
VALUES (1, 'Schulz', 'Berlin', 'Hauptstrasse 1');
```

Im nächsten Beispiel kennen wir die Adresse eines Kunden nicht und lassen dieses Feld bei der Eingabe frei. Es erhält dadurch den Standardwert, der in der Tabelle definiert wurde. Sollte kein Wert festgelegt worden sein, ist der Wert `NULL`. Dies ist der Code:

```postgresql
INSERT INTO Kundenliste (ID, NAME, STADT)
VALUES (2, 'Meyer', 'Hamburg');
```

## Mehrere Zeilen gleichzeitig mit PostgreSQL `INSERT` einfügen

Es ist auch möglich, in PostgreSQL mit `INSERT INTO` gleich **mehrere Zeilen hinzuzufügen**. Im folgenden Code setzen wir zwei weitere Kunden ein:

```postgresql
INSERT INTO Kundenliste (ID, NAME, STADT, ADRESSE)
VALUES (3, 'Schmidt', 'Dortmund', 'Kleistweg 17'), (4, 'Müller', 'Stuttgart', 'Waldgasse 73');
```

Die einzelnen Zeilen werden dabei **in Klammern gefasst und durch Kommata voneinander abgetrennt**.

> Tipp: Möchten Sie den Inhalt einer Zeile entfernen, können Sie dies mit dem Befehl [PostgreSQL DELETE](https://www.ionos.at/digitalguide/server/konfiguration/postgresql-delete/ "PostgreSQL DELETE") erledigen.