---
tags:
  - kafka
  - cloud
---

Die Source Software ist eine der besten Lösungen für die Speicherung und Verarbeitung von Datenströmen. Diese **Messaging- und Streaming-Plattform**, die unter Apache 2.0 lizenziert ist, bietet Verwerfungstoleranz, hervorragende Skalabilität und eine hohe Lese- und Schreibgeschwindigkeit. Diese Faktoren, die für Big-Data-Anwendungen sehr attraktiv sind, basieren auf einem Cluster, der **verteilte Datenspeicherung und -Replikation** ermöglicht. Es gibt vier verschiedene Schnittstellen zur Kommunikation mit dem Cluster mit einem einfachen TCP-Protokoll, das als Kommunikationsgrundlage dient.

Dieses Kafka-Tutorial erklärt, wie man mit der Scala-basierten Anwendung beginnt, beginnend mit **der Installation** **von _Kafka_ und der _Apache_ ZooKeeper-Software**, **die benötigt wird**, **um sie zu verwenden**.

## Anforderungen an Apache Kafka

Um einen leistungsstarken Kafka-Cluster zu betreiben, benötigen Sie die richtige Hardware. Das Entwicklungsteam empfiehlt die Verwendung von Quad-Core **Intel Xeon-Maschinen** mit 24 Gigabyte Speicher. Es ist wichtig, dass Sie **genug Speicher** haben, um immer die Lese- und Schreibzugriffe für alle Anwendungen zu speichern, die aktiv auf den Cluster zugreifen. Da _der_ hohe Datendurchsatz von _Apache Kafka eine_ seiner Ziehungen ist, ist es entscheidend, eine geeignete Festplatte zu wählen. Die Apache Software Foundation empfiehlt eine **SATA-Festplatte** (8 x 7200 U/min). Wenn es darum geht, Leistungsengpässe zu vermeiden, gilt das folgende Prinzip: Je mehr Festplatten, desto besser.

**In Bezug auf die Software gibt es auch einige Anforderungen, die erfüllt werden müssen,** um _Apache Kafka_ für die Verwaltung eingehender und ausgehender Datenströme zu verwenden. Bei der Wahl Ihres Betriebssystems sollten Sie sich für ein **Unix-Betriebssystem** wie _Solaris_ oder eine Linux-Distribution entscheiden. Dies liegt daran, dass Windows-Plattformen nur begrenzte Unterstützung erhalten. _Apache Kafka_ ist in Scala geschrieben, die nach Java bytecode kompiliert, so dass Sie die neueste Version der [Java SE Development Kits (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html "Oracle.com: JDK Download-Center") auf Ihrem System installiert benötigen. Dazu gehört auch die **Java Runtime Environment,** die für den Einsatz von Java-Anwendungen benötigt wird. Sie benötigen auch den [_Apache_](https://zookeeper.apache.org/ "Offizieller Projektstandort Apache ZooKeeper") ZooKeeper-Dienst, der verteilte Prozesse synchronisiert.

ZooKeeper explained[](https://youtu.be/gZj16chk0Ss)

## Apache Kafka Tutorial: Wie man Kafka, ZooKeeper und Java installiert

Eine Erklärung, welche Software benötigt wird, siehe den vorherigen Teil dieses Kafka-Tutorials. Wenn es nicht bereits auf Ihrem System installiert ist, empfehlen wir, die **Java Runtime Environment** zuerst zu installieren. Viele neuere Versionen von Linux-Distributionen, wie _Ubuntu_, das als Beispiel-Betriebssystem in diesem Apache Kafka Tutorial verwendet wird (Version 17.10), hat bereits OpenJDK, eine kostenlose Implementierung von JDK, in ihrem **offiziellen Paket-Repository**. Das bedeutet, dass Sie das Java Development Kit einfach über dieses Repository installieren können, indem Sie den folgenden Befehl in das Terminal eingeben:

```bash
sudo apt-get install openjdk-8-jdk
```

Sofort nach der Installation von Java installieren Sie den **_Apache ZooKeeper_** Prozess-Synchronisationsservice. Das Paket-Repository Ubuntu bietet auch ein gebrauchsfertiges Paket für diesen Dienst, das mit der folgenden Befehlszeile ausgeführt werden kann:

```bash
sudo apt-get install zookeeperd
```

Sie können dann mit einem zusätzlichen Befehl überprüfen, ob der **ZooKeeper-Dienst aktiv ist** :

```bash
sudo systemctl status zookeeper
```

Wenn der Synchronisationsdienst nicht läuft, können Sie ihn jederzeit mit diesem Befehl starten:

```bash
sudo systemctl start zookeeper
```

Damit _ZooKeeper_ immer automatisch beim Start startet, fügen Sie am Ende einen Autostart-Eintrag hinzu:

```bash
sudo systemctl enable zookeeper
```

Erstellen Sie schließlich ein **Benutzerprofil für Kafka**, das Sie später verwenden müssen. Um dies zu tun, öffnen Sie das Terminal erneut und geben Sie folgenden Befehl ein:

```bash
sudo useradd kafka -m
```

Mit dem _passwd_ Passwort-Manager können Sie dem Benutzer ein Passwort hinzufügen, indem Sie den folgenden Befehl eingeben, gefolgt vom gewünschten Passwort:

```bash
sudo passwd kafka
```

Als nächstes gewähren Sie dem Benutzer „kafka“ sudo Rechte:

```bash
sudo adduser kafka sudo
```

Sie können sich nun jederzeit mit dem neu erstellten Benutzerprofil anmelden:

```bash
su – kafka
```

Wir sind an der Stelle in diesem Tutorial angekommen, an der es Zeit ist, Kafka herunterzuladen und zu installieren. Es gibt eine Reihe von vertrauenswürdigen Quellen, in denen Sie sowohl ältere als auch aktuelle Versionen der Datenstromverarbeitungssoftware herunterladen können. So können Sie z.B. die **Installationsdateien** direkt aus dem [Download-Verzeichnis](https://www.apache.org/dist/kafka/ "Kafka Index auf apache.org") der [Apache Software Foundation](https://www.apache.org/dist/kafka/ "Kafka index on apache.org") erhalten. Es wird dringend empfohlen, dass Sie mit einer **aktuellen Version von Kafka** arbeiten, so dass Sie möglicherweise den folgenden Download-Befehl anpassen müssen, bevor Sie ihn in das Terminal eingeben:

```bash
wget http://www.apache.org/dist/kafka/2.1.0/kafka_2.12-2.1.0.tgz
```

Da die heruntergeladene Datei komprimiert wird, müssen Sie sie auspacken:

```bash
sudo tar xvzf kafka_2.12-2.1.0.tgz --strip 1
```

Verwenden Sie das ---strip 1-Flag, um sicherzustellen, dass die extrahierten Dateien direkt in das Verzeichnis **" / kafka"** gespeichert werden. Andernfalls, basierend auf der in diesem Kafka-Tutorial verwendeten Version, würde Ubuntu alle Dateien im Verzeichnis **"/kafka/kafka" (2122-2,1.0)** speichern. Um dies zu tun, müssen Sie zuvor ein Verzeichnis namens "kafka" mit mkdir erstellt und darauf umgestellt haben (über "cd kafka").

## Kafka: Wie man das Streaming- und Messaging-System einlegt

Jetzt, da Sie Apache Kafka, die Java Runtime Environment und ZooKeeper installiert haben, können Sie den Kafka-Dienst jederzeit ausführen. Bevor Sie dies tun, sollten Sie jedoch ein paar kleine **Anpassungen an seinen Konfigurationen** vornehmen, damit die Software für ihre kommenden Aufgaben optimal konfiguriert ist.

### Thema löschen

Kafka **erlaubt es Ihnen nicht, Themen (d.h. die Speicher- und Kategorisierungskomponenten in einem Kafka-Cluster) in seinem Standard-Setup zu löschen**. Sie können dies jedoch leicht ändern, indem Sie die Konfigurationsdatei _server.properties_ Kafka verwenden. Um diese Datei zu öffnen, die sich im Konfigurationsverzeichnis befindet, verwenden Sie den folgenden Terminal-Befehl im Standardtext-Editor _nano_ :

```bash
sudo nano ~/kafka/config/server.properties
```

Am Ende dieser Konfigurationsdatei fügen Sie einen neuen Eintrag hinzu, mit dem Sie Themen löschen können:

```bash
delete.topic.enable=true
```

### Erstellen von .service-Dateien für ZooKeeper und Kafka

Der nächste Schritt im Kafka-Tutorial besteht darin, Unit-Dateien für ZooKeeper und Kafka zu erstellen, mit denen Sie **gemeinsame Aktionen** ausführen können, wie z. B. das Starten, Stoppen und Neustarten der beiden Dienste in einer Weise, die mit anderen Linux-Diensten übereinstimmt. Um dies zu tun, müssen Sie _._service-Dateien für den _systemd_ session Manager für beide Anwendungen erstellen und einrichten.

#### Wie man die passende ZooKeeper-Datei für den Ubuntu-System-Sitzungsmanager erstellt

Erstellen Sie zuerst die Datei für den ZooKeeper-Synchronisationsdienst, indem Sie den folgenden Befehl im Terminal eingeben:

```bash
sudo nano /etc/systemd/system/zookeeper.service
```

Das wird nicht nur die Datei erstellt, sondern auch im Nano-Texteditor öffnen. Jetzt geben Sie die folgenden Zeilen ein und speichern Sie dann die Datei.

```bash
[Unit]
Requires=network.target remote-fs.target
After=network.target remote-fs.target
[Service]
Type=simple
User=kafka
ExecStart=/home/kafka/kafka/bin/zookeeper-server-start.sh /home/kafka/kafka/config/zookeeper.properties
ExecStop=/home/kafka/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal
[Install]
WantedBy=multi-user.target
```

Infolgedessen wird _systemd_ verstehen, dass _ZooKeeper_ verlangt, dass das **Netzwerk** und das **Dateisystem bereit sind**, bevor es starten kann. Dies ist im Abschnitt [Einheit] definiert. Der Abschnitt [Service] gibt an, dass der Session-Manager die Dateien _zookeeper-server-start.sh_ und _zookeeper-server-stop.sh verwenden_ sollte, um **_ZooKeeper_** zu **starten** und **zu stoppen**. Es wird auch angegeben, dass ZooKeeper **automatisch neu gestartet** werden sollte, wenn es unerwartet stoppt. Die Eingabesteuerung [Installieren] wird beim Start der Datei mit ".multi-user.target" als Standardwert für ein Multi-User-System (z.B. ein Server) aktiviert.

#### Wie das Erstellen einer Kafka-Datei für den Ubuntu-Systemd-Sitzungsmanager funktioniert

Um die _._service-Datei für _Apache Kafka_ zu erstellen, verwenden Sie den folgenden Terminalbefehl:

```bash
sudo nano /etc/systemd/system/kafka.service
```

Kopieren Sie dann den folgenden Inhalt in die neue Datei, die bereits im Nano-Texteditor geöffnet wurde:

```bash
[Unit]
Requires=zookeeper.service
After=zookeeper.service
[Service]
Type=simple
User=kafka
ExecStart=/bin/sh -c '/home/kafka/kafka/bin/kafka-server-start.sh /home/kafka/kafka/config/server.properties > /home/kafka/kafka/kafka.log 2>&1'
ExecStop=/home/kafka/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal
[Install]
WantedBy=multi-user.target
```

Der Abschnitt [Einheit] in dieser Datei gibt an, dass **der _Kafka-Dienst_ auf _ZooKeeper_ angewiesen ist**. Dies stellt sicher, dass der Synchronisationsdienst automatisch gestartet wird, wenn die _Datei kafka.service_ ausgeführt wird. Der Abschnitt [Service] gibt an, dass die Shell-Dateien _kafka-server-start.sh_ und _kafka-server-stop.sh zum_ Starten und Stoppen des Kafka-Servers verwendet werden sollten. Die Spezifikation für einen automatischen Neustart finden Sie auch nach einer unerwarteten Trennung sowie den Multi-User-Eintrag in dieser Datei.

### Kafka: Erststarten und Erstellen eines Autostart-Eintrags

Sobald Sie die Session-Manager-Einträge für _Kafka_ und _ZooKeeper_ erfolgreich erstellt haben, können Sie Kafka mit dem folgenden Befehl starten:

```bash
sudo systemctl start kafka
```

Standardmäßig _systemd_verwendet das Systemd-Programm ein zentrales Protokoll oder ein Journal, in dem alle **Log-Nachrichten** automatisch geschrieben werden. So können Sie leicht prüfen, ob der Kafka-Server wie gewünscht gestartet wurde:

```bash
sudo journalctl -u kafka
```

Wenn Sie _Apache Kafka_ erfolgreich gestartet haben manuell gestartet, aktivieren Sie das Ende, indem Sie **den automatischen Start** während des Systemstarts aktivieren:

```none
sudo systemctl enable kafka
```

## Apache Kafka Tutorial: mit Apache Kafka beginnen

Dieser Teil des Kafka-Tutorials beinhaltet das Testen von _Apache Kafka_ durch die Verarbeitung einer ersten Nachricht über die Messaging-Plattform. Dazu benötigen Sie einen **Hersteller** und einen **Verbraucher** (d. h. eine Instanz, mit der Sie Daten zu Themen und eine Instanz lesen können, die Daten aus einem Thema lesen kann). Zunächst einmal müssen Sie ein **Thema** erstellen, das in diesem Fall **TutorialTopic** heißen soll**.** Da es sich um ein einfaches Testthema handelt, sollte es nur eine einzelne Partition und eine einzige Replik enthalten:

```none
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic TutorialTopic
```

Als nächstes müssen Sie dem neu erstellten Thema einen Produzenten erstellen, der die erste Beispielnachricht " **Hello, Welt!** " hinzufing. Verwenden Sie dazu das **Shell-Skript** _kafka-console-producer.sh_, das den Hostnamen und Port des Kafka-Servers (in diesem Beispiel Kafkas Standardpfad) sowie den Themennamen als Argumente benötigt:

```none
echo "Hello, World!" | ~/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic TutorialTopic > /dev/null
```

Als nächstes erstellen Sie mit dem _kafka-console-consumer._sh-Skript einen Kafka-Konsumenten, der **Nachrichten von** **TutorialTopic** **verarbeitet und anzeigt**. Sie benötigen den Hostnamen und Port des Kafka-Servers sowie den Themennamen als Argumente. Darüber hinaus wird das Argument „---from-anfang“ beigefügt, damit der Verbraucher die vor der Erstellung des Verbrauchers veröffentlichte „Hallo, Welt!“-Botschaft verarbeiten kann:

```none
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TutorialTopic --from-beginning
```

Als Ergebnis präsentiert das Terminal die **Botschaft „Hallo, Welt!"** mit dem Skript, das kontinuierlich läuft und darauf wartet, dass weitere Nachrichten zum Testthema veröffentlicht werden. Wenn der Hersteller also in einem anderen Terminalfenster für **zusätzliche Dateneingabe** verwendet wird, sollten Sie dies auch in dem Fenster sehen, in dem das Verbraucherskript läuft.

https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330