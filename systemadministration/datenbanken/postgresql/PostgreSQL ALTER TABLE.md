---
tags:
  - postgresql
  - datenbanken
---
Mit dem Befehl `ALTER TABLE` können Sie in PostgreSQL Tabellen anpassen. Das Kommando wird mit einer Aktion genutzt, um zum Beispiel Spalten hinzuzufügen oder zu modifizieren.

## Was ist PostgreSQL `ALTER TABLE`?

Der Befehl `ALTER TABLE` wird in [PostgreSQL] dafür benötigt, bereits bestehende **Tabellen zu modifizieren**. Die Anweisung kann dabei verwendet werden, um eine Spalte hinzuzufügen, zu löschen oder sie nach eigenen Vorstellungen anzupassen. Zudem wird PostgreSQL `ALTER TABLE` eingesetzt, um für eine Tabelle in dem [Datenbankmanagementsystem] Einschränkungen festzulegen oder wieder aufzuheben. Um das gewünschte Ergebnis zu erzielen, müssen Sie den Befehl **mit einer Aktion spezifizieren**.

## `ALTER TABLE`: Syntax

Für ein grundlegendes Verständnis lohnt sich ein Blick auf die grundsätzliche Syntax von `ALTER TABLE`. Diese sieht wie folgt aus:

```postgresql
ALTER TABLE name_der_tabelle aktion;
```

Auf den eigentlichen Befehl folgt also zunächst der Name der Tabelle, an der Sie Änderungen vornehmen möchten. Danach spezifizieren Sie den PostgreSQL-Befehl `ALTER TABLE` mit der gewünschten Aktion.

Tipp

Um eine neue Tabelle zu erstellen, verwenden Sie in PostgreSQL den Befehl [CREATE TABLE](https://www.ionos.at/digitalguide/server/konfiguration/postgresql-create-table/ "PostgreSQL CREATE TABLE").

## PostgreSQL `ALTER TABLE`: Beispiele

In den nachfolgenden Abschnitten wollen wir die Funktionsweise von `ALTER TABLE` anhand von praktischen Beispielen verdeutlichen. Wir legen hierfür eine exemplarische Tabelle mit dem Namen „Kunden“ an, die zunächst drei Spalten und drei Zeilen enthält. Sie sieht so aus:

|ID|Name|Stadt|
|---|---|---|
|1|Schulz|Berlin|
|2|Meyer|Hamburg|
|3|Schmidt|Dortmund|

Diese lässt sich mit `ALTER TABLE` in PostgreSQL nun auf verschiedene Art anpassen.

### Spalte hinzufügen mit PostgreSQL `ADD COLUMN`

Um eine zusätzliche Spalte hinzuzufügen, verwenden Sie `ALTER TABLE` in Kombination mit PostgreSQL `ADD COLUMN`. Diese Aktion verfügt über zwei Parameter: den Namen der Spalte und den Datentyp der Spalte. Die Syntax gestaltet sich folgendermaßen:

```postgresql
ALTER TABLE name_der_tabelle ADD COLUMN name_der_spalte datentyp;
```

Möchten wir unsere Tabelle „Kunden“ um eine Spalte für die Adresse erweitern, nutzen wir PostgreSQL `ADD COLUMN` wie folgt:

```postgresql
ALTER TABLE Kunden ADD COLUMN Adresse VARCHAR(255);
```

Die neue, erweiterte Tabelle sieht damit wie folgt aus:

|ID|Name|Stadt|Adresse|
|---|---|---|---|
|1|Schulz|Berlin|NULL|
|2|Meyer|Hamburg|NULL|
|3|Schmidt|Dortmund|NULL|

### Spalten löschen mit `DROP COLUMN`

Ganz ähnlich gehen wir vor, wenn wir eine Spalte aus der existierenden Tabelle entfernen möchten. Dafür nutzen wir PostgreSQL `ALTER TABLE` in Kombination mit der Aktion `DROP COLUMN`. Diese hat als einzigen Parameter den Namen der Spalte. Dies ist die Syntax:

```postgresql
ALTER TABLE name_der_tabelle DROP COLUMN name_der_spalte;
```

Um die Spalte „Stadt“ zu entfernen, verwenden wir diesen Code:

```postgresql
ALTER TABLE Kunden DROP COLUMN Stadt;
```

Die Tabelle besteht damit wieder nur noch aus drei Spalten:

|ID|Name|Adresse|
|---|---|---|
|1|Schulz|NULL|
|2|Meyer|NULL|
|3|Schmidt|NULL|

### Spalten umbenennen mit `RENAME COLUMN`

Wir können auch eine existierende Spalte umbenennen, was eine gute Alternative dazu sein kann, Spalten zu löschen oder neu hinzuzufügen. Die Syntax von `RENAME COLUMN` innerhalb von `ALTER TABLE` sieht folgendermaßen aus:

```postgresql
ALTER TABLE name_der_tabelle RENAME COLUMN name_der_spalte TO neuer_name;
```

Hier machen wir aus der Spalte „Name“ die Spalte „Kundenname“:

```postgresql
ALTER TABLE Kunden RENAME COLUMN Name TO Kundenname;
```

In unserer Tabelle verändert sich wie gewünscht lediglich der Name der Spalte:

|ID|Kundenname|Adresse|
|---|---|---|
|1|Schulz|NULL|
|2|Meyer|NULL|
|3|Schmidt|NULL|

### Weitere PostgreSQL-Aktionen für `ALTER TABLE`

Es gibt zahlreiche weitere Aktionen, die Sie zusammen mit `ALTER TABLE` verwenden können. Dies sind die wichtigsten:

So verändern Sie den **Datentyp** einer Spalte:

```postgresql
ALTER TABLE name_der_tabelle ALTER COLUMN name_der_spalte TYPE datentyp;
```

So legen Sie fest, dass eine Spalte einen **Wert erhalten** muss:

```postgresql
ALTER TABLE name_der_tabelle ALTER COLUMN name_der_spalte SET NOT NULL;
```

Um für eine Spalte Einschränkungen wie `UNIQUE` oder `PRIMARY KEY` zu etablieren, nutzen Sie PostgreSQL `ALTER TABLE` in Kombination mit der Aktion `ADD CONSTRAINT`:

```postgresql
ALTER TABLE name_der_tabelle ADD CONSTRAINT name_der_einschränkung definition_der_einsch
```

