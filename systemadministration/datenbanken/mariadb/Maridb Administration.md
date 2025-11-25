---
tags:
  - mariadb
  - datenbanken
---
# Mariadb Benutzer Administration 

## Nutzerrechte zuteilen

Die Nutzerrechte (oder „privileges“ im Englischen) legen fest, welche Aktionen mit einem Account in MariaDB durchgeführt werden können. Während der Admin-User über alle Rechte verfügt, sollten die Nutzerrechte für andere Accounts limitiert werden, da es sonst zu Sicherheitsproblemen kommen kann. Die gängigen Nutzerrechte sind diese:

- `ALL`: Stattet den Account mit allen Rechten außer `GRANT OPTION` aus.
- `GRANT OPTION`: Stattet den Account mit den Rechten Ihres Accounts aus.
- `SELECT`: Der Account kann auf Datenbanken oder Tabellen zugreifen.
- `INSERT`: Der Account kann neue Zeilen in eine Tabelle hinzufügen.
- `UPDATE`: Zeilen können aktualisiert werden.
- `DELETE`: Zeilen dürfen gelöscht werden.
- `CREATE`: Neue Tabellen oder Datenbanken können erstellt werden.
- `ALTER`: Kann die Struktur einer Tabelle verändern.
- `DROP`: Kann Tabellen oder Datenbanken löschen.

## `OR REPLACE` und `IF NOT EXISTS`

Wenn Sie mit `CREATE USER` in MariaDB einen neuen Account erstellen möchten und bereits ein User mit demselben Namen existiert, erhalten Sie eine Fehlermeldung. Um dieses Problem zu umgehen, bietet Ihnen das Datenbankmanagementsystem zwei Optionen: `OR REPLACE` und `IF NOT EXISTS`.

Die Syntax von `OR REPLACE` sieht folgendermaßen aus:

```sql
CREATE OR REPLACE USER nutzername@hostname IDENTIFIED BY 'passwort';
```

Es handelt sich dabei um eine Kurzform dieses Codes:

```sql
DROP USER IF EXISTS nutzername@hostname;
CREATE USER nutzername@hostname IDENTIFIED BY 'passwort';
```

Das System überprüft dabei also, ob es bereits einen User mit dem entsprechenden Namen gibt. Ist dies der Fall, wird der **alte Account durch den neuen ersetzt**. Gibt es bisher keinen entsprechenden User, so wird dieser neu erstellt.

Die Syntax von `IF NOT EXISTS` ist diese:

```sql
CREATE USER IF NOT EXISTS nutzername@hostname IDENTIFIED BY 'passwort';
```

Auch hier überprüft das System, ob bereits ein Account mit dem entsprechenden Namen existiert. Ist dies der Fall, erhalten Sie eine Warnmeldung und es wird **kein Account überschrieben**. Gibt es bisher keinen derartigen Nutzer oder eine Nutzerin, wird der User neu angelegt.

> Tipp: In unserem Digital Guide erfahren Sie noch mehr über das Open-Source-Datenbankmanagementsystem. So erklären wir Ihnen unter anderem, wie die Befehle [MariaDB CREATE DATABASE] und [MariaDB CREATE TABLE] funktionieren, welche [Unterschiede und Gemeinsamkeiten MariaDB und MySQL haben] und wie Sie die [Installation von MariaDB] durchführen.

**Benutzer anlegen**

`CREATE USER nutzername@hostname IDENTIFIED BY 'passwort';`

**User an einer DB mit bestimmter Ip Berechtigen**

`MariaDB [(none)]> GRANT ALL ON wpdb.* to 'wpuser'@'208.117.84.50' IDENTIFIED BY 'password' WITH GRANT OPTION;`

**Admin User anlegen zu verwaltung von Datenbanken**

`GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;`

**Bechtigungen expleziet definieren**

`GRANT nutzerrechte ON datenbank.tabelle TO nutzername@hostname;`

> Zur sicherheit sollte kein _%_ verwendetet werden es erlaubt Remote logins !

**Berechtigungen anwenden**

```sql
FLUSH PRIVILEGES;
```

# Mariadb SHOW TABLES

Wenn Sie eine Übersicht über alle Tabellen innerhalb einer Datenbank benötigen, ist in MariaDB `SHOW TABLES` der passende Befehl. Seit der Version 11.2.0 werden dabei auch provisorische Tabellen (Temporary Tables) gelistet. Für den Einsatz der Anweisung sind die passenden Nutzerrechte notwendig.

## Syntax und Funktionsweise der Anweisung

Die grundsätzliche Syntax von `SHOW TABLES` in MariaDB sieht wie folgt aus:

```sql
SHOW TABLES [FROM name_der_datenbank] [LIKE 'muster'];
```

Nach dem eigentlichen Befehl spezifizieren Sie dabei, aus welcher Datenbank Sie eine Auflistung über alle Tabellen erhalten möchten. Der **Parameter** `LIKE` ist optional. Er hilft Ihnen dabei, die Ergebnisse nach einem eigens definierten Muster zu filtern.

## Beispiel für die Verwendung von `SHOW TABLES` in MariaDB

Die Funktionsweise und der Nutzen von `SHOW TABLES` in MariaDB werden offensichtlich, wenn Sie den Befehl selbst ausprobieren. Dafür können Sie einfach das folgende Beispiel nutzen. Dabei erstellen wir zunächst eine neue Datenbank mit dem [MariaDB-Befehl CREATE DATABASE]:

```sql
CREATE DATABASE Stadt_Land_Fluss;
```

Anschließend fügen wir mit [MariaDB CREATE TABLE](https://www.ionos.at/digitalguide/hosting/hosting-technik/mariadb-create-table/ "MariaDB Create Table") neue Tabellen in dieser Datenbank hinzu. Dafür nutzen wir diese Codes:

```sql
CREATE TABLE stadt
(
Postleitzahl INT,
Name VARCHAR(50)
);
```

```sql
CREATE TABLE land
(
Vorwahl INT,
Name VARCHAR(50)
);
```

```sql
CREATE TABLE fluss
(
Name VARCHAR(50),
Laenge INT
);
```

Im Anschluss nutzen wir `SHOW TABLES` für MariaDB, um eine Übersicht über alle Tabellen innerhalb der Datenbank „Stadt_Land_Fluss“ zu erhalten. Dieser sieht so aus:

```sql
SHOW TABLES;
```

Wenn Sie mehrere Datenbanken erstellt haben und die Auflistung Ihrer Tabellen ausdrücklich auf eine bestimmte Datenbank zuschneiden möchten, können Sie den Befehl spezifizieren. Der Code sieht dann für unser Beispiel so aus:

```sql
SHOW TABLES FROM Stadt_Land_Fluss;
```

## Suchparameter eingrenzen über `LIKE`

Im Abschnitt über die Syntax haben wir bereits kurz den optionalen Parameter `LIKE` angesprochen. Diesen können Sie einsetzen, wenn Sie die Ausgabe von SHOW TABLES in MariaDB **nach eigenen Vorstellungen einschränken** möchten. Gerade bei umfangreichen Datenbanken mit zahlreichen Tabellen kann eine solche Klausel einen großen Mehrwert bieten. Unser Beispiel ist zwar nicht so umfangreich, aber die Funktionsweise von `LIKE` lässt sich auch daran veranschaulichen. Im folgenden Code weisen wir das System daher an, zwar die gesamte Datenbank zu durchsuchen, dabei aber nur Tabellen auszugeben, die dem Suchparameter „fluss“ entsprechen. Dies sieht so aus:

```sql
SHOW TABLES LIKE 'fluss%';
```

Unsere Auflistung wird dann ausschließlich die Tabelle „fluss“ enthalten.

## Tabellentyp anzeigen mit `FULL`

Wenn Sie nicht nur den Namen der vorhandenen Tabellen in einer Datenbank listen möchten, sondern gleichzeitig auch Informationen über die Art der Tabelle benötigen, können Sie `SHOW TABLES` in MariaDB mit der **Option** `FULL` verwenden. Diese fügt neben den Namen eine zweite Spalte in die Ausgabe ein, die den Namen „table_type“ trägt. Hier wird Ihnen angezeigt, um welche Art Tabelle es sich jeweils handelt. Die verschiedenen Typen sind `BASE TABLE`, `VIEW` und `SEQUENCE`. Dies ist der passende Code:

```sql
SHOW FULL TABLES FROM Stadt_Land_Fluss;
```

## Wofür wird `SHOW TABLES` in MariaDB benötigt?

Nachdem Sie sich einen Überblick über alle Tabellen verschafft haben, können Sie entweder eine Tabelle Ihrer Wahl aufrufen oder sie mit dem [MariaDB-Befehl DROP TABLES](https://www.ionos.at/digitalguide/hosting/hosting-technik/mariadb-drop-tables/ "MariaDB Drop Tables") aus der entsprechenden Datenbank löschen. `SHOW TABLES` ist für MariaDB daher ein sehr **elementares Werkzeug**, um die Übersicht über sämtliche Datensammlungen zu behalten und weitere Arbeitsschritte zu planen.

# Mariadb Tabelen anlegen

Die Anweisung `CREATE TABLE` wird in MariaDB verwendet, um eine **neue Tabelle zu erstellen**, die dann im Anschluss mit Daten gefüllt werden kann. Da MariaDB ein relationales Datenbankmanagementsystem (DBMS) ist, dienen diese Tabellen als Basis für sämtliche Speichervorgänge. Bereits während der Erstellung werden die einzelnen Spalten definiert und dabei vor allem festgelegt, für welche Datentypen diese vorgesehen sein sollen. Tabellen sind innerhalb einer neuerstellten Datenbank – einzigartig, sodass eine Fehlermeldung erscheint, wenn eine Tabelle mit demselben Namen bereits existiert. In den folgenden Abschnitten erklären wir, wie Sie `CREATE TABLE` in MariaDB anwenden und welche Spezifikationen Sie nutzen können.

## Syntax und Funktionsweise

Die generelle Syntax von `CREATE TABLE` in MariaDB folgt immer diesem Prinzip:

```sql
CREATE TABLE Name_der_Tabelle(
	Name_der_ersten_Spalte Datentyp_der_ersten_Spalte,
	Name_der_zweiten_Spalte Datentyp_der_zweiten_Spalte,
	…
);
```

Dabei erstellen Sie zunächst eine neue Tabelle, der Sie einen eigenständigen Namen anstelle des Platzhalters „Name_der_Tabelle“ geben. Erlaubt sind dabei alle Zeichen des . Anschließend spezifizieren Sie die einzelnen Spalten. Jede dieser Spalten erhält einen **eigenen Namen und den Datentyp**, der innerhalb dieser Spalte hinterlegt werden darf. Alle Spalten werden durch Kommata voneinander getrennt.

## `OR REPLACE` und `IF NOT EXISTS`

Da Tabellen einzigartig sein müssen, erhalten Sie eine Fehlermeldung, **falls bereits eine gleichnamige Tabelle existiert**. Um dieses Problem zu umgehen, haben Sie zwei Möglichkeiten: Die Option `OR REPLACE` überprüft, ob eine Tabelle mit demselben Namen bereits in der Datenbank existiert. Ist dies der Fall, wird die **alte Tabelle durch die neue ersetzt**. Andernfalls wird die neue Tabelle einfach erstellt. Die Syntax dieser Anweisung sieht so aus:

```sql
CREATE OR REPLACE TABLE Name_der_Tabelle(
	Name_der_ersten_Spalte Datentyp_der_ersten_Spalte,
	Name_der_zweiten_Spalte Datentyp_der_zweiten_Spalte,
	…
);
```

Beachten Sie dabei allerdings, dass die alte Tabelle überschrieben wird und dadurch mögliche Inhalte verloren gehen. Die Option funktioniert als Kurzform dieses Codes:

```sql
DROP TABLE IF EXISTS Name_der_Tabelle;
CREATE TABLE Name_der_Tabelle(
	Name_der_ersten_Spalte Datentyp_der_ersten_Spalte,
	Name_der_zweiten_Spalte Datentyp_der_zweiten_Spalte,
	…
);
```

Eine weitere Möglichkeit, um Dopplungen beziehungsweise die darauffolgenden Fehlermeldungen zu umgehen, ist die Option `IF NOT EXISTS`. Diese überprüft, ob eine namensgleiche Tabelle bereits in der Datenbank geführt wird. Ist dies der Fall, erhalten Sie lediglich eine Benachrichtigung und es wird **keine Tabelle überschrieben**. Gibt es keine Tabelle unter diesem Namen, wird eine neue Tabelle erstellt. Die entsprechende Syntax sieht so aus:

```sql
CREATE TABLE IF NOT EXISTS Name_der_Tabelle(
	Name_der_ersten_Spalte Datentyp_der_ersten_Spalte,
	Name_der_zweiten_Spalte Datentyp_der_zweiten_Spalte,
	…
);
```

## Beispiel für `CREATE TABLE` in MariaDB

Die Funktionsweise von `CREATE TABLE` in MariaDB lässt sich am einfachsten mit einem kleinen Beispiel veranschaulichen. Dafür erstellen wir eine Tabelle für eine fiktive Projektliste mit insgesamt acht Spalten. Diese sieht so aus:

```sql
CREATE TABLE Projekte(
	projektnummer INT AUTO_INCREMENT,
	nachname VARCHAR(50) NOT NULL,
	vorname VARCHAR(50),
	anfang DATE,
	ende DATE,
	kosten DOUBLE,
	aufgaben VARCHAR(255) NOT NULL,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	PRIMARY KEY (projektnummer)
);
```

Im ersten Schritt erstellen wir so eine neue Tabelle und nennen sie „Projekte“. In den folgenden Zeilen spezifizieren wir die einzelnen Spalten:

- **projektnummer**: In dieser Spalte wird dem Projekt eine individuelle Nummer zugeteilt. Sie wird als Primärschlüssel behandelt und dient damit zur klaren Zuteilung jeder einzelnen Zeile. Durch `AUTO_INCREMENT` weisen wir das Programm an, die Einträge in „projektnummer“ automatisch fortzuführen, um so eine einheitliche Reihenfolge zu gewährleisten.
- **nachname**: Hier wird der Nachname des Kunden oder der Kundin eingetragen. Der Eintrag darf eine Abfolge von bis zu 50 Zeichen umfassen. Durch `NOT NULL` legen wir fest, dass diese Spalte nicht leerbleiben darf.
- **vorname**: Die Spalte „vorname“ funktioniert nach einem ähnlichen Prinzip wie die vorangegangene Spalte. Da der Vorname allerdings nicht obligatorisch für die Abrechnung ist, kann diese Spalte auch leerbleiben.
- **anfang**: Hier wird der Beginn eines fortlaufenden Projekts hinterlegt. Die zulässigen Werte sind ein Datum im eingestellten Datumsformat oder der Nullwert.
- **ende**: „ende“ beschreibt die Deadline oder den tatsächlichen Abschluss eines Projektes. Auch diese Werte dürfen im Format `DATE` oder `NULL` sein.
- **kosten**: In dieser Spalte wird der Rechnungsbetrag aufgelistet. Er wird im Format `DOUBLE` hinterlegt.
- **aufgaben**: Unter „aufgaben“ ist Platz für eine kurze Beschreibung der Serviceleistungen, die für das Projekt durchgeführt wurden. Die Spalte bietet Platz für bis zu 255 Zeichen und darf nicht leerbleiben.
- **created_at**: In der letzten Spalte wird das Datum der jeweiligen Projekterstellung hinterlegt. Basis dafür ist die aktuelle Uhrzeit und Datumsangabe des Systems.

# Wofür wird `CREATE DATABASE` in MariaDB verwendet?

Der Befehl `CREATE DATABASE` in MariaDB wird genutzt, um eine **neue Datenbank innerhalb des freien und relationalen Datenbankmanagementsystems zu erstellen**. Dabei wird nicht nur der Name dieser Datenbank festgelegt, sondern auch optional verschiedene Parameter. Für die Erstellung sind Root- oder Admin-Rechte notwendig.

Der Name der neuen Sammlung muss innerhalb der Serverstruktur einzigartig sein. Versuchen Sie, einen bereits vergebenen Namen zu nutzen, erhalten Sie ohne weitere Vorkehrungen eine Fehlermeldung. Wie Sie CREATE DB in MariaDB anwenden und welche Vorkehrungen Sie treffen können, erfahren Sie in den nächsten Abschnitten.

## Syntax und Beispiel

Die grundsätzliche Syntax von `CREATE DATABASE` in MariaDB sieht immer wie folgt aus:

```sql
CREATE DATABASE Name_der_Datenbank;
```

Mit dem Befehl weisen Sie die **Erstellung einer neuen Datenbank** an, deren Namen Sie anstelle des Platzhalters „Name_der_Datenbank“ hinterlegen. Für diesen Namen sind sämtliche Zeichen des [ASCII-Codes] (American Standard Code for Information Interchange) zulässig. Dazu gehören unter anderem alle Buchstaben des lateinischen Alphabets in Groß- und Kleinschreibung, die Zahlen von 0 bis 9 sowie zahlreiche Sonderzeichen.

Ein mögliches Beispiel für eine neue Datenbank könnte wie folgt aussehen:

```sql
CREATE DATABASE kundenliste_2024;
```

Ist die neue Datenbank einmal angelegt, können Sie im Anschluss mit  neue Nutzer  erstellen und verwenden, um neue Tabellen anzulegen.

Ist die neue Datenbank einmal angelegt, können Sie im Anschluss mit [MariaDB CREATE USER] neue Nutzer und Nutzerinnen erstellen und [MariaDB CREATE TABLE] verwenden, um neue Tabellen anzulegen.

## `CREATE OR REPLACE DATABASE`

Die Syntax von `CREATE DATABASE` für MariaDB lässt sich um zwei Parameter erweitern. Beide dienen dazu, die Fehlermeldung zu verhindern, die ausgeliefert wird, sobald eine Datenbank mit demselben Namen bereits existiert. Die erste optionale Erweiterung nennt sich `OR REPLACE` und wird genutzt, um eine **Datenbank zu ersetzen**, sofern diese denselben Namen hat. Dies ist ihre Syntax:

```sql
CREATE OR REPLACE DATABASE Name_der_Datenbank;
```

Diese Schreibweise ist im Endeffekt eine Verkürzung dieses Codes:

```sql
DROP DATABASE IF EXISTS Name_der_Datenbank;
CREATE DATABASE Name_der_Datenbank;
```

`OR REPLACE` wird seit der Version 10.1.3 unterstützt.

## `CREATE DATABASE` mit `IF NOT EXISTS`

Der zweite optionale Parameter für CREATE DB unter MariaDB lautet `IF NOT EXISTS`. Durch diesen überprüft das Programm ebenfalls, ob eine Datenbank mit demselben Namen bereits existiert. Ist dies nicht der Fall, wird die Datenbank neu erstellt. Gibt es allerdings bereits eine entsprechende Datenbank, erhalten Sie eine **Warnung statt einer Fehlermeldung**. Die Datenbank wird nicht erstellt. Die entsprechende Syntax mit dem Parameter sieht so aus:

```sql
CREATE DATABASE IF NOT EXISTS Name_der_Datenbank;
```

# DROP TABLE in MariaDB

wenn Sie mit dem freien Datenbankmanagementsystem eine oder mehrere Tabellen löschen möchten, ist MariaDB `DROP TABLE` die richtige Anweisung. Da die Löschung unwiderruflich ist, sollte der Befehl nur mit äußerster Vorsicht genutzt werden, auch weil neben der eigentlichen Tabelle auch sämtliche Inhalte entfernt werden.

## Voraussetzung und Syntax

Um eine Tabelle zu löschen, benötigen Sie die entsprechenden **Nutzerrechte**. Diese erhalten Sie entweder als Admin oder über die Neuerstellung mit [MariaDB CREATE USER].

Die Syntax von `DROP TABLE` in MariaDB sieht wie folgt aus:

```sql
DROP TABLE name_der_tabelle;
```

Den Platzhalter „name_der_tabelle“ ersetzen Sie dabei durch den tatsächlichen Tabellennamen.

Sollten Sie versuchen, eine Tabelle zu entfernen, die entweder bereits entfernt wurde oder sich nie in der Datenbank befand, erhalten Sie eine Fehlermeldung. Um dies zu verhindern, bietet MariaDB für `DROP TABLE` die Option `IF EXISTS`. Mit dieser überprüft das System zunächst, ob eine entsprechende Tabelle hinterlegt ist. Ist dies der Fall, wird sie ohne weitere Zwischenschritte entfernt. Existiert die Tabelle nicht, erhalten Sie lediglich eine Warnung und es werden keine weiteren Schritte unternommen. Der Befehl mit der Option sieht so aus:

```sql
DROP TABLE IF EXISTS name_der_tabelle;
```

## Beispiel für `DROP TABLE` in MariaDB

Die Funktionsweise von `DROP TABLE` in MariaDB lässt sich am einfachsten mit einem kleinen Beispiel veranschaulichen. Dafür nehmen wir an, dass Sie mit [MariaDB CREATE DATABASE] eine Datenbank namens „Aufgaben“ erstellt haben. In dieser haben Sie mit der Anweisung [MariaDB CREATE TABLE] verschiedene Tabellen eingefügt. Die Tabelle „Aufgaben_2023“ benötigen Sie allerdings nicht länger und möchten sie endgültig entfernen. Sie rufen daher die entsprechende Datenbank auf und geben dann den folgenden Befehl ein:

```sql
DROP TABLE IF EXISTS Aufgaben_2023;
```


Die Tabelle und alle in ihr gespeicherten Daten werden nun entfernt.

## Mehrere Tabellen entfernen

Es ist auch möglich, **mehrere Tabellen gleichzeitig zu löschen**. Diese werden durch Kommata voneinander abgegrenzt. So sähe ein praktisches Beispiel aus:

```sql
DROP TABLE IF EXISTS Aufgaben_2023, Aufgaben_2022, Aufgaben_2021;
```

## Provisorische Tabellen löschen

Wenn Sie mit `DROP TABLE` in MariaDB eine **provisorische Tabelle** (engl. temporary table) löschen möchten, ist auch dies möglich. Für unser Beispiel von oben würde der Befehl dann folgendermaßen aussehen:

```sql
DROP TEMPORARY TABLE IF EXISTS Aufgaben_2023;
```

In diesem Fall überprüft das System, ob es eine temporäre Tabelle namens „Aufgaben_2023“ gibt. Ist dies der Fall, wird sie gelöscht. Ist dies nicht der Fall oder ist die Tabelle nicht temporär, entfällt die Löschung.

# DROP DATABASE in MariaDB

Mit `DROP DATABASE` werden in MariaDB ganze Datenbanken unwiderruflich entfernt. Der Befehl kann daher nur mit Root- oder Admin-Rechten ausgeführt werden und sollte nur mit großer Vorsicht zum Einsatz kommen.

## `DROP DATABASE` in MariaDB

`DROP DATABASE` ist für MariaDB eine sehr wirkungsvolle Anweisung, die nur **äußerst vorsichtig eingesetzt** werden sollte. Sie wird verwendet, um eine Datenbank aus einer Serverstruktur zu löschen. Wurde der Befehl durchgeführt, ist die **gesamte Datenbank inklusive aller Tabellen und Daten unwiederbringlich verloren** und kann nicht mehr aufgerufen werden. Lediglich Nutzerrechte, die während des Einsatzes von [MariaDB CREATE USER] etabliert wurden, sind nicht automatisch aufgehoben. `DROP DATABASE` kann in MariaDB nur mit Admin- oder Root-Privilegien durchgeführt werden. Andere Befehle wie `DELETE DATABASE` für MariaDB oder `REMOVE DATABASE` für MariaDB existieren nicht.

## yntax mit und ohne `IF EXISTS`

Die Syntax von `DROP DATABASE` in MariaDB sieht wie folgt aus:

```sql
DROP DATABASE Name_der_Datenbank;
```

Dabei ersetzen Sie den Platzhalter „Name_der_Datenbank“ lediglich durch die entsprechende Datenbank, die Sie entfernen möchten.

Optional können Sie `IF EXISTS` einsetzen, um zu verhindern, dass eine Fehlermeldung erscheint, wenn die gesuchte Datenbank sich nicht auf Ihrem Server befindet.

```sql
DROP DATABASE IF EXISTS Name_der_Datenbank;
```

## Die Funktionsweise mit einem Beispiel erklärt

Um die Funktionsweise von `DROP DATABASE` in MariaDB zu veranschaulichen, nutzen wir ein einfaches Beispiel. Dafür stellen wir uns vor, dass eine Datenbank namens „Aufgaben_2023“ nicht länger benötigt wird. Daher überprüfen wir mit `SHOW DATABASES`, ob sich die Datenbank noch auf dem Server befindet, und entfernen sie dann. Dies ist der Code:

```sql
mysql> SHOW DATABASES;
mysql> DROP DATABASE Aufgaben_2023;
```

# JOIN für MariaDB

In einem relationalen Datenbankmanagementsystem können Datensätze in unterschiedlichen Tabellen miteinander verglichen werden. So ist es möglich, Verbindungen herzustellen und übereinstimmende Werte aus zwei Tabellen zu extrahieren. Diese Aufgabe wird in MariaDB durch `JOIN` übernommen. Die Anweisung wird in Kombination mit `SELECT` angewendet und kann in verschiedene Kategorien unterteilt werden, von denen wir Ihnen hier `INNER JOIN`, `LEFT OUTER JOIN` und `RIGHT OUTER JOIN` näher vorstellen.

## Syntax und Funktionsweise

Damit Sie einen Überblick über die unterschiedlichen `JOIN`-Anweisungen in MariaDB gewinnen können, zeigen wir Ihnen zunächst die grundsätzliche Syntax der Anweisung. Diese sieht für `INNER JOIN` so aus:

```sql
SELECT spalte(n)
FROM tabelle_1
INNER JOIN tabelle_2
ON tabelle_1.spalte = tabelle_2.spalte;
```

Mit `SELECT` wählen Sie dabei die Spalte (oder die Spalten) aus, die berücksichtigt werden sollen. Statt des Platzhalters „tabelle_1“ hinterlegen Sie Ihre erste Tabelle und statt „tabelle_2“ die zweite Tabelle, die mit der ersten verbunden werden soll. Durch `INNER JOIN` werden **alle Zeilen der ersten Tabelle mit allen Zeilen der zweiten Tabelle verglichen**. Ergebnisse, die übereinstimmen (also in beiden Tabellen vorhanden sind), werden dann gemeinsam in einer Ergebnistabelle ausgegeben. Einträge, die nicht übereinstimmen, werden hingegen auch nicht berücksichtigt.

## Beispiel für `INNER JOIN` in MariaDB

Um zu veranschaulichen, wie `INNER JOIN` in MariaDB funktioniert, wählen wir ein einfaches Beispiel. Dafür gehen wir von einer Datenbank aus, in der sich zwei Tabellen befinden. Die erste Tabelle nennt sich „Kundenliste“ und die zweite „Bestellungen“. „Kundenliste“ erstellen wir mit [MariaDB CREATE TABLE](https://www.ionos.at/digitalguide/hosting/hosting-technik/mariadb-create-table/ "MariaDB Create Table"). Sie enthält die Spalten „Kundennummer“, Nachname“, „Vorname“, „Stadt“ und „Erstellungsdatum“. Der Code sieht aus wie folgt:

```sql
CREATE TABLE Kundenliste (
	Kundennummer INT PRIMARY KEY,
	Nachname VARCHAR(50),
	Vorname VARCHAR(50),
	Stadt VARCHAR(50),
	Erstellungsdatum DATE
);
```

Diese Tabelle füllen wir nun mit einigen Werten. Dafür nutzen wir `INSERT INTO`:

```sql
INSERT INTO Kundenliste VALUES
(1, 'Schmidt', 'Martina', 'Berlin', '2022-07-19'),
(2, 'Schulz', 'Bernd', 'Hamburg', '2023-03-03'),
(3, 'Meyer', 'Peter', 'Hamburg', '2023-07-09'),
(4, 'Schneider', 'Sarah', 'Dortmund', '2023-12-10'),
(5, 'Bauer', 'Lisa', 'Berlin', '2024-01-17');
```

Im Anschluss erstellen wir die Tabelle „Bestellungen“. Diese enthält die Spalten „Bestellnummer“, „Artikelnummer“, „Kundenname“ und „Bestelldatum“. Der Code sieht so aus:

```sql
CREATE TABLE Bestellungen (
	Bestellnummer INT AUTO_INCREMENT PRIMARY KEY,
	Artikelnummer INT,
	Kundenname VARCHAR(50),
	Bestelldatum DATE
);
```

Diese Tabelle füllen wir nun mit einigen Werten. Dafür nutzen wir `INSERT INTO`:

```sql
INSERT INTO Kundenliste VALUES
(1, 'Schmidt', 'Martina', 'Berlin', '2022-07-19'),
(2, 'Schulz', 'Bernd', 'Hamburg', '2023-03-03'),
(3, 'Meyer', 'Peter', 'Hamburg', '2023-07-09'),
(4, 'Schneider', 'Sarah', 'Dortmund', '2023-12-10'),
(5, 'Bauer', 'Lisa', 'Berlin', '2024-01-17');
```

Im Anschluss erstellen wir die Tabelle „Bestellungen“. Diese enthält die Spalten „Bestellnummer“, „Artikelnummer“, „Kundenname“ und „Bestelldatum“. Der Code sieht so aus:

```sql
CREATE TABLE Bestellungen (
	Bestellnummer INT AUTO_INCREMENT PRIMARY KEY,
	Artikelnummer INT,
	Kundenname VARCHAR(50),
	Bestelldatum DATE
);
```

Auch diese Tabelle füllen wir mit Beispielwerten:

```sql
INSERT INTO Bestellungen VALUES
(101, 247, 'Müller', '2024-02-20'),
(102, 332, 'Meyer', '2024-03-03'),
(103, 247, 'Hallmann', '2024-03-09'),
(104, 191, 'Schulz', '2024-03-17'),
(105, 499, 'Brandt', '2024-03-17');
```

Nun nutzen wir `INNER JOIN` für MariaDB, um jene Kunden und Kundinnen auszufiltern, die **in der Kundenliste auftauchen und eine Bestellung aufgegeben haben**, die in der Tabelle „Bestellungen“ aufgelistet wird. Der entsprechende Code sieht so aus:

```sql
SELECT Kundenliste.Kundennummer, Kundenliste.Nachname, Bestellungen.Bestellnummer, Bestellungen.Artikelnummer
FROM Kundenliste
INNER JOIN Bestellungen
ON Kundenliste.Nachname = Bestellungen.Kundenname;
```

Hier legen wir den Fokus auf den Nachnamen in der Kundenliste und den Kundennamen in den Bestellungen. Stimmen diese Werte überein, werden sie übernommen. Da die Kunden Meyer und Schulz in beiden Tabellen auftauchen, würde die entsprechende Ausgabe so aussehen:

|Kundennummer|Kundenname|Bestellnummer|Artikelnummer|
|---|---|---|---|
|3|Meyer|102|332|
|2|Schulz|104|191|

## `LEFT OUTER JOIN`

`LEFT OUTER JOIN` in MariaDB funktioniert nach einem ähnlichen Prinzip und verwendet auch eine fast identische Syntax. Im Gegensatz zu `INNER JOIN` werden in diesem Fall allerdings **alle Datensätze aus der ersten oder linken Tabelle** (in unserem Beispiel „Kundenliste“) ausgegeben und nur die passenden Datensätze aus der zweiten oder rechten Tabelle („Bestellungen“). Gibt es keine Entsprechung in der zweiten Tabelle, wird der entsprechende Wert **mit NULL angegeben**. Für unser Beispiel von oben sieht der Befehl so aus:

```sql
SELECT Kundenliste.Nachname, Bestellungen.Artikelnummer
FROM Kundenliste
LEFT OUTER JOIN Bestellungen
ON Kundenliste.Nachname = Bestellungen.Kundenname;
```

Dadurch erhalten wir das folgende Ergebnis:

|Kundenname|Artikelnummer|
|---|---|
|Schmidt|NULL|
|Schulz|191|
|Meyer|332|
|Schneider|NULL|
|Bauer|NULL|

## `RIGHT OUTER JOIN`

Genau andersherum funktioniert `RIGHT OUTER JOIN` in MariaDB. Dieses Mal werden die **Daten der zweiten oder rechten Tabelle mit übereinstimmenden Werten der ersten oder linken Tabelle verknüpft**. Gibt es keine Übereinstimmung, bleibt der Wert NULL. Dies ist der Code:

```sql
SELECT Kundenliste.Nachname, Bestellungen.Artikelnummer
FROM Kundenliste
RIGHT OUTER JOIN Bestellungen
ON Kundenliste.Nachname = Bestellungen.Kundenname;
```

So sieht die Ausgabe aus:

|Kundenname|Artikelnummer|
|---|---|
|NULL|247|
|Meyer|332|
|NULL|247|
|Schulz|191|
|NULL|499|
