---
tags:
  - mariadb
  - datenbanken
---
Nachdem Sie sich mit dem freien Datenbankmanagementsystem verbunden haben, müssen Sie in MariaDB eine `Select Database`-Aktion durchführen, um auszuwählen, in welcher Datenbank Sie arbeiten möchten. Für dieses Vorhaben haben Sie zwei verschiedene Optionen: Entweder nutzen Sie den Befehl `USE` in der Kommandozeile von MySQL oder die Funktion `mysql_select_db` über PHP. Wir stellen Ihnen beide Wege vor.

## Der Befehl `USE` in der Kommandozeile

Die Syntax von `USE` sieht aus wie folgt:

```sql
USE name_der_datenbank;
```

Den Befehl müssen Sie immer in Kombination mit einer speziellen Datenbank verwenden und diese an Stelle des Platzhalters „name_der_datenbank“ einsetzen. Verzichten Sie auf diesen Parameter, **erhalten Sie eine Fehlermeldung (ERROR 1046)**.

Um Ihnen die Funktionsweise einfacher veranschaulichen zu können, nutzen wir ein einfaches Beispiel. Dafür stellen wir uns vor, dass wir die Datenbank „Kunden“ aufrufen möchten. Folgende Schritte sind dafür nötig:

1. Loggen Sie sich auf Ihrem Server über die Kommandozeile ein:
```sql
mysql -u root -p
Enter password: ************
```

2. Nutzen Sie den Befehl `SHOW DATABASES`, um sich einen Überblick über alle verfügbaren Datenbanken auf Ihrem Server zu verschaffen:

```sql
mysql> SHOW DATABASES;
```

3. Um nun die gewünschte Datenbank auszuwählen, verwenden Sie den Befehl `USE`:

```sql
mysql> USE Kunden;
```

Jetzt können Sie in der Datenbank arbeiten und mit [MariaDB CREATE TABLE]eine neue Tabelle erstellen. Ist die gewünschte Datenbank noch nicht gelistet, erstellen Sie sie mit dem [MariaDB-Befehl CREATE DATABASE]. Wird eine Datenbank nicht mehr benötigt, entfernen Sie sie mit der [MariaDB-Anweisung DROP DATABASE].

## `SELECT DATABASE` für MariaDB in PHP

Die Funktion `SELECT DATABASE` für MariaDB findet sich auch in PHP (hier: `mysqli_select_db`). Die Syntax für den Verbindungsaufbau gestaltet sich hier folgendermaßen:

```php
$connection = mysqli_connect("server", "username", "password");
```

Zum Auswählen der Datenbank sieht der anschließende Befehl wie folgt aus:

```php
mysqli_select_db($connection, "Kunden");
```