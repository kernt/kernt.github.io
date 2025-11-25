---
tags:
  - postgresql
  - datenbanken
---
Ein Schema wird in PostgreSQL genutzt, um mehrere Tabellen zu gruppieren und so die Datenbank übersichtlicher zu gestalten. Für die Erstellung wird der Befehl `CREATE SCHEMA` genutzt. Mit `DROP SCHEMA` lässt sich ein erstelltes Schema jederzeit wieder löschen.

## Was ist ein PostgreSQL-Schema?

Als Schema wird in [PostgreSQL] eine **Sammlung mehrerer Tabellen** bezeichnet. Ein Schema kann außerdem unter anderem Indizes, Sequenzen, Datentypen und Funktionen enthalten und ist damit vergleichbar mit einem Verzeichnis, wobei im Gegensatz dazu keine Verschachtelung möglich ist. Die Methode wird insbesondere genutzt, um große Datenbanken mit zahlreichen Tabellen einfacher zu verwalten. In der Struktur des [Datenbankmanagementsystems] befindet sich ein PostgreSQL-Schema zwischen der Ebene der Datenbank und der der einzelnen Tabellen. Der gesamte hierarchische Aufbau sieht folgendermaßen aus:

1. Instanz (in PostgreSQL häufig auch als „Server“ bezeichnet, der mehrere Datenbanken enthalten kann)
2. Datenbank
3. Schema
4. Tabelle
5. Zeile

Um ein neues Schema in PostgreSQL zu erstellen, wird der Befehl `CREATE SCHEMA` genutzt. Mit `DROP SCHEMA` wird das Schema wieder entfernt. Beide Anweisungen stellen wir in diesem Artikel näher vor.

## SCHEMA in PostgreSQL: Grundlegende Syntax

Die Syntax von `CREATE SCHEMA` in PostgreSQL ist diese:

```postgresql
CREATE SCHEMA name;
```

Damit weisen Sie die Erstellung eines Schemas in PostgreSQL an und geben diesem Schema einen eigenen, in der Datenbank einzigartigen Namen.

## Neuen Tabellen ein PostgreSQL-Schema zuweisen

Wenn Sie nun mit dem Befehl [PostgreSQL CREATE TABLE](https://www.ionos.at/digitalguide/server/konfiguration/postgresql-create-table/ "PostgreSQL CREATE TABLE") eine neue Tabelle erstellen, können Sie diese zu einem bereits existierenden PostgreSQL-Schema hinzufügen. Die entsprechende Syntax sieht so aus:

```postgresql
CREATE TABLE ihrschema.neuetabelle (
…
);
```

Um diesen Vorgang zu veranschaulichen, zeigen wir Ihnen hier, wie Sie zunächst ein Schema namens „Kunden“ erstellen. Dies ist der Code:

```postgresql
CREATE SCHEMA Kunden;
```

Im Anschluss daran erschaffen Sie eine neue Tabelle namens „Kundenliste_Deutschland“, die Sie in das PostgreSQL-Schema „Kunden“ einfügen. Dafür verwenden Sie diesen Code:

```postgresql
CREATE TABLE Kunden.Kundenliste_Deutschland(
ID INT PRIMARY KEY NOT NULL,
NAME VARCHAR(50) NOT NULL,
LAND VARCHAR(50),
ADRESSE VARCHAR(255)
);
```

Dadurch erhalten Sie eine neue leere Tabelle, die sich in dem angegebenen PostgreSQL-Schema „Kunden“ befindet. Diese können Sie jetzt mit [PostgreSQL INSERT INTO](https://www.ionos.at/digitalguide/server/konfiguration/postgresql-insert-into/ "PostgreSQL INSERT INTO") mit Werten befüllen.

Hinweis

Wird kein Schema spezifiziert, werden neue Tabellen automatisch dem Standard-Schema „public“ zugeordnet.

## Schema entfernen mit `DROP SCHEMA`

Natürlich haben Sie auch die Möglichkeit, ein PostgreSQL-Schema wieder zu löschen. Ist das Schema bereits leer und wird deswegen nicht mehr benötigt, verwenden Sie einfach den Befehl `DROP SCHEMA`. Für unser Beispiel von oben sieht dieser so aus:

```postgresql
DROP SCHEMA Kunden;
```

Möchten Sie in PostgreSQL ein Schema und alle darin enthaltenen Objekte entfernen, ist dies der passende Befehl:

```postgresql
DROP SCHEMA Kunden CASCADE;
```

Diesen Befehl sollten Sie dementsprechend nur mit Vorsicht anwenden, um keine wertvollen Daten zu verlieren.