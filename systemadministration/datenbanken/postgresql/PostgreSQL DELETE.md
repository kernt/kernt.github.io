---
tags:
  - postgresql
  - datenbanken
---
Mit PostgreSQL `DELETE` können Inhalte aus einer Tabelle gelöscht werden. Der Befehl lässt sich durch Bedingungen so spezifizieren, dass nur bestimmte Zeilen für die Löschung in Frage kommen. Da sich die Entfernung nicht rückgängig machen lässt, sollte der Befehl nur sehr vorsichtig eingesetzt werden.

## Was ist PostgreSQL `DELETE`?

Der Befehl `DELETE` wird in [PostgreSQL] verwendet, um **Einträge aus einer Tabelle zu löschen**. Mit der Bedingung `WHERE` können Sie dabei bestimmte Zeilen auswählen, aus denen Inhalte entfernt werden sollen. Verzichten Sie bei der Nutzung des Kommandos auf die Klausel `WHERE`, werden alle Inhalte aus der betreffenden Tabelle **unwiederbringlich gelöscht**. Aus diesem Grund sollten Sie den Befehl nur mit größter Vorsicht anwenden.

## ostgreSQL `DELETE`: Syntax und Funktionsweise

Die Syntax von PostgreSQL `DELETE` hat folgende Grundstruktur:

```postgresql
DELETE FROM name_der_tabelle
WHERE [Bedingung];
```

Das Keyword `DELETE FROM` initiiert die Löschung in der angegebenen Tabelle. Mit der Bedingung `WHERE` können Sie spezifizieren, in welchen Zeilen Inhalte entfernt werden sollen. Möchten Sie mehrere Bedingungen berücksichtigen, können Sie diese mit `AND` oder `OR` auflisten.

> Hinweis: Bevor Sie Daten löschen, sollten Sie sicherstellen, dass Sie eine aktuelle Sicherung der Datenbank besitzen oder den Löschprozess in einer Transaktion ausführen. Auf diese Weise können Sie verhindern, dass wichtige Daten verlorengehen, wenn das Kommando versehentlich falsch ausgeführt wird.

## Alle Inhalte einer Tabelle entfernen

Die Funktionsweise von `DELETE` in PostgreSQL lässt sich am einfachsten mit einem praktischen Beispiel erläutern. Dafür nutzen wir den Befehl [CREATE TABLE](https://www.ionos.at/digitalguide/server/konfiguration/postgresql-create-table/ "PostgreSQL CREATE TABLE"), um eine neue PostgreSQL-Tabelle namens „Kundenliste“ zu erstellen. Mit [INSERT INTO](https://www.ionos.at/digitalguide/server/konfiguration/postgresql-insert-into/ "PostgreSQL INSERT INTO") füllen wir diese dann mit verschiedenen Inhalten. Die Tabelle verfügt über drei Spalten namens „ID“, „Name“ und „Ort“ und erhält zunächst vier Einträge. So sieht sie aus:

```postgresql
|ID|Name|Ort|
|-|-|-|
|1|Schulz|Berlin|
|2|Meyer|Hamburg|
|3|Schmidt|Dortmund|
|4|Schulz|Stuttgart|
```

Möchten wir nun die Tabelle beibehalten, dabei aber **sämtliche Inhalte löschen**, verwenden wir PostgreSQL `DELETE` ohne zusätzliche Bedingung. Der Befehl sieht für unser Beispiel dann so aus:

```postgresql
DELETE FROM Kundenliste;
```

## Eine Zeile löschen mit PostgreSQL `DELETE`

Häufiger werden Sie allerdings in der Situation sein, dass Sie lediglich **eine bestimmte Zeile** entfernen möchten. Auch dies wird mit PostgreSQL `DELETE` erledigt. Dafür nutzen wir den Befehl mit einer Klausel `WHERE`. Für unser Beispiel möchten wir den Kunden „Meyer“ mit der ID „2“ löschen. Der passende Code sieht dann so aus:

```postgresql
DELETE FROM Kundenliste
WHERE ID = 2;
```

## Zeilen mit mehreren Bedingungen spezifizieren

Insbesondere bei langen Tabellen kann es doppelte und damit nicht eindeutige Einträge geben. Möchten Sie sichergehen, dass ausschließlich die gewünschte Zeile gelöscht wird, können Sie PostgreSQL `DELETE` mit **mehreren Bedingungen** verwenden. In unserem Beispiel haben wir zwei Kunden namens „Schulz“ und möchten lediglich den zweiten Eintrag entfernen. Daher kombinieren wir zwei Bedingungen. Der Code dafür ist dieser:

```postgresql
DELETE FROM Kundenliste
WHERE Name = 'Schulz'
AND ID >= 3;
```

Auf diese Weise werden alle Zeilen gelöscht, in denen der Name „Schulz“ und die ID größer oder gleich „3“ ist. Da der erste Eintrag mit dem Namen eine ID-Nummer hat, die kleiner als „3“ ist, bleibt dieser Eintrag auch nach dem Löschbefehl in der Datenbank.