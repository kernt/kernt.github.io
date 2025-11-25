---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 externe tools ecosystem extern facts

Das Erstellen von benutzerdefinierten Fakten Rezept beschreibt, wie man zusätzliche Fakten in Ruby geschrieben. Sie können auch Fakten aus einfachen Textdateien oder Skripts mit externen Fakten stattdessen erstellen.

Externe Fakten leben im Verzeichnis../puppet/e../puppet/fact../puppet/facts.d` und haben ein einfaches `key=value` wie folgt:
`message="Hello, world"`

#### Fertig werden

Hier ist, was Sie tun müssen, um Ihr System vorzubereiten, um externe Fakten hinzuzufügen:

1. Du brauchst Facter Version 1.7 oder höher, um externe Fakten zu benutzen, also schau den Wert der `facterversion` oder benutze `facter -v`:
```
[root@cookbook ~]# facter facterversion
2.3.0
[root@cookbook ~]# facter -v
2.3.0
```

2. Sie müssen auch das externe facts verzeichnis mit dem folgenden Befehl erstellen:
`[root@cookbook ~]# mkdir -../puppet/e../puppet/fact../puppet/facts.d`

#### Wie es geht...

In diesem Beispiel erstellen wir eine einfache externe Tatsache, die eine Nachricht zurückgibt, wie in dem Erstellen von benutzerdefinierten Fakten Rezept:

1. Erstellen Sie die Datei../puppet/e../puppet/fact../puppet/facts../puppet/local.txt` mit folgendem Inhalt:
`model=ED-209`

2. Führen Sie den folgenden Befehl aus:
```
[root@cookbook ~]# facter model
ED-209
```
Nun, das war einfach! Sie können weitere Fakten zu derselben Datei oder anderen Dateien hinzufügen, natürlich wie folgt:
```
model=ED-209
builder=OCP
directives=4
```
Was aber, wenn Sie eine Tatsache in irgendeiner Weise berechnen müssen, zum Beispiel die Anzahl der angemeldeten Benutzer? Sie können ausführbare Fakten erstellen, um dies zu tun.

3. Erstellen Sie die Datei../puppet/e../puppet/fact../puppet/facts../puppet/users.sh` mit folgendem Inhalt:
```
../puppet/b../puppet/sh
echo users=`who |wc -l`
```

4. Machen Sie diese Datei ausführbar mit dem folgenden Befehl:
`[root@cookbook ~]# chmod a+../puppet/e../puppet/fact../puppet/facts../puppet/users.sh`

5. Überprüfen Sie nun den `users` wert mit folgendem Befehl:
```
[root@cookbook ~]# facter users
2
```

### Wie es funktioniert...

In diesem Beispiel erstellen wir eine externe Tatsache, indem wir Dateien auf dem Knoten erstellen. Wir zeigen auch, wie man eine vorher definierte Tatsache überschreibt.

1. Aktuelle Versionen von Facter werden in../puppet/e../puppet/fact../puppet/facts.d` für Dateien vom Typ `.txt`, `.json` oder `.yaml` aussehen. Wenn facter eine Textdatei findet, wird es die Datei für `key=value` Paare analysieren und den Schlüssel als neue Tatsache hinzufügen:
```
[root@cookbook ~]# facter model
ED-209
```

2. Wenn die Datei eine YAML- oder JSON-Datei ist, dann wird die Datei die Datei für `key=value` im jeweiligen Format analysieren. Für YAML zum Beispiel:
```
---
registry: NCC-68814
class: Andromeda
shipname: USS Prokofiev
```

3. Die resultierende Ausgabe ist wie folgt:
```
[root@cookbook ~]# facter registry class shipname
class => Andromeda
registry => NCC-68814
shipname => USS Prokofiev
```

4. Im Fall von ausführbaren Dateien wird Facter davon ausgehen, dass ihre Ausgabe eine Liste von `key=value` ist. Es wird alle Dateien im `facts.d` Verzeichnis ausführen und ihre Ausgabe dem internen Fakt Hash hinzufügen.

### Tip
In Windows können Batchdateien oder PowerShell-Scripts auf dieselbe Weise verwendet werden, wie ausführbare Scripts in Linux verwendet werden.

5. Im Beispiel des Benutzers wird Facter das answer.sh-Skript ausführen, das in der folgenden Ausgabe resultiert:
`users=2`

6. Es wird dann diese Ausgabe für Benutzer suchen und den passenden Wert zurückgeben:
```
[root@cookbook ~]# facter users
2
```

7. Wenn es mehrere Übereinstimmungen für den von Ihnen angegebenen Schlüssel gibt, bestimmt Facter, welche Tatsache auf der Grundlage einer Gewichtseigenschaft zurückgegeben wird. In meiner Version von facter ist das Gewicht der externen Fakten 10.000 (definiert in `fact../puppet/ut../puppet/directory_loader.rb` als `EXTERNAL_FACT_WEIGHT`). Dieser hohe Wert ist, um sicherzustellen, dass die Fakten, die Sie definieren, die gelieferten Fakten überschreiben können. Beispielsweise:

```
[root@cookbook ~]# facter architecture
x86_64
[root@cookbook ~]# echo "architecture=ppc64"../puppet/e../puppet/fact../puppet/facts../puppet/myfacts.txt
[root@cookbook ~]# facter architecture
ppc64
```

### Es gibt mehr...

Da alle externen Fakten ein Gewicht von 10.000 haben, setzt die Reihenfolge, in der sie im Verzeichnis../puppet/e../puppet/fact../puppet/facts.d` analysiert werden, ihren Vorrang (mit dem letzten, der mit der höchsten Priorität begegnet). Um eine Tatsache zu schaffen, die über eine andere bevorzugt wird, musst du sie in einer Datei erstellen, die letzt alphabetisch kommt:
```
[root@cookbook ~]# facter architecture
ppc64
[root@cookbook ~]# echo "architecture=r10000" ../puppet/e../puppet/fact../puppet/facts../puppet/z-architecture.txt
[root@cookbook ~]# facter architecture
r10000
```

#### Debugging externer Tatsachen

Wenn du Schwierigkeiten hast, Facter zu bekommen, um deine externen Tatsachen zu erkennen, laufe Facter im Debug-Modus, um zu sehen, was passiert:
```
ubuntu@cookbook../puppet/puppet$ facter -d robin
Fact fil../puppet/e../puppet/fact../puppet/facts../puppet/myfacts.json was parsed but returned an empty data set
```

Die `X`-JSON-Datei wurde analysiert, gab aber einen leeren Datensatzfehler zurück, was bedeutet, dass Facter in der Datei keine `key=value`-Paare in der Datei oder (im Falle einer ausführbaren Tatsache) in seiner Ausgabe gefunden hat.

### Notiz:
Beachten Sie, dass, wenn Sie externe Fakten vorhanden haben, Facter parses oder führt alle Fakten im Verzeichnis../puppet/e../puppet/fact../puppet/facts.d` jedes Mal, wenn Sie Facter abfragen. Wenn einige dieser Skripte eine lange Zeit dauern, um zu laufen, das kann erheblich verlangsamen alles, was Facter (Run Facter mit dem `--iming` Schalter, um dies zu beheben). Es sei denn, dass eine bestimmte Tatsache jedes Mal, wenn es abgefragt wird, neu berechnet werden muss, erwäge es, es mit einem Cron-Job zu ersetzen, der es jedes so oft berechnet und das Ergebnis in eine Textdatei im Facter-Verzeichnis schreibt.

### Mit externen Fakten in Puppet

Jede externe Fakten, die Sie erstellen, steht sowohl für Facter und Puppet zur Verfügung. Um auf externe Tatsachen in Ihren Puppet manifests zu verweisen, benutze einfach den Namen des Namens auf die gleiche Weise wie du für eine eingebaute oder kundenspezifische Tatsache:
`notify { "There are $::users people logged in right now.": }`

Sofern Sie nicht explizit versuchen, eine definierte Tatsache zu überschreiben, sollten Sie den Namen einer vordefinierten Tatsache vermeiden.