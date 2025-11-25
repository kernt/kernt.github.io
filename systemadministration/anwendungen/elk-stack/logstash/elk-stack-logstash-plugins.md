---
tags:
  - elk
  - elk-stack
  - logstash
---
# elk stack logstash plugins

Logstash hat eine Vielzahl von Plugins um zu helfen, wir integrieren sie mit einer Vielzahl von Input-und Output-Quellen. Lassen Sie uns die verschiedenen Plugins erkunden.

### Auflisten aller Plugins in Logstash

Sie können den folgenden Befehl ausführen, um alle verfügbaren Plugins in Ihrer installierten Logstash-Version aufzulisten:

`bin/plugin list`

Außerdem können Sie alle Plugins mit einem `namensfragment` auflisten, indem Sie diesen Befehl ausführen:

`bin/plugin list <namefragment>`

Um alle Plugins für Gruppennamen, Input, Output oder Filter aufzulisten, können wir diesen Befehl ausführen:

```sh
bin/plugin list --group <group name>
bin/plugin list --group output
```

Bevor Sie verschiedene Plugin-Konfigurationen erkunden, werfen wir einen Blick auf die Datentypen und bedingten Ausdrücke, die in verschiedenen Logstash-Konfigurationen verwendet werden müssen.

### Datentypen für Plugin-Eigenschaften

Ein Logstash-Plugin erfordert bestimmte Einstellungen oder Eigenschaften. Diese Eigenschaften haben bestimmte Werte, die zu einem der folgenden wichtigen Datentypen gehören.

#### Array

Ein Array ist die Erfassung von Werten für eine Eigenschaft.
Ein Beispiel lässt sich wie folgt ansehen:

`path => ["value1","value2"]`

###### Hinweis

Das Zeichen `=>` ist der Zuweisungsoperator, der für alle Eigenschaften von Konfigurationswerten in der Logstash-Konfiguration verwendet wird.

### Boolesch werte

Ein boolescher Wert ist entweder `true` oder `false` (ohne Anführungszeichen).
Ein Beispiel lässt sich wie folgt sehen:

`periodic_flush => false`

#### Codec
Codec ist eigentlich kein Datentyp, sondern eine Möglichkeit, Daten am Eingang(input) oder Ausgang(output) zu codieren oder zu dekodieren.

Ein Beispiel lässt sich wie folgt veranschaulichen:

`codec => "json"`

Diese Instanz legt fest, dass dieser Codec am Ausgang alle Ausgaben im JSON-Format kodiert.

#### Hash

Hash ist im Grunde eine Key-Value-Paar-Sammlung. Es wird als `"key" => "value"` angegeben und mehrere Werte in einer Sammlung werden durch ein Leerzeichen getrennt.

Ein Beispiel lässt sich wie folgt veranschaulichen:

```c
match => {
"key1" => "value1" "key2" => "value2"}
```

#### String

String stellt eine Folge von Zeichen dar, die von Anführungszeichen eingeschlossen sind.

Ein Beispiel lässt sich wie folgt veranschaulichen:

`value => "Welcome to ELK"` 

#### Kommentare 
Kommentare beginnen mit dem `#` Zeichen.

Ein Beispiel lässt sich wie folgt veranschaulichen:

`#this represents a comment`

#### Feldreferenzen

Felder können mit [`field_name`] oder verschachtelten Feldern mit [`level1`] [`level2`] bezeichnet werden.

#### Logstash-Bedingungen

Logstash-Bedingungen werden verwendet, um Ereignisse oder Protokollzeilen unter bestimmten Bedingungen zu filtern. Bedingungen in Logstash werden wie andere Programmiersprachen behandelt und arbeiten mit `if`, `if else` und `else` Aussagen. Mehrere, `if else` andere Blöcke können auch verschachtelt werden.

Die Syntax für die Bedingungen ist wie folgt:

```c
if  <conditional expression1>{
[[some]] statements here.
}
else if  <conditional expression2>{
[[some]] statements here.
}
else{
[[some]] statements here.
}
```

Conditionals arbeiten mit Vergleichsoperatoren, booleschen Operatoren und unären Operatoren:

Vergleichsoperatoren:

* Gleichstellungs Operator: `==`, `!=`, `<`, `>`, `<=`, `>=`
* Reguläre Ausdrücke:` =~`,` !~`
* Inclusion(Aufnahme): `in`, `not in`
* Boolesche Operatoren : `and`, `or`, `nand`, `xor`
* Unärerer Operator Include: `!`

Schauen wir uns das mit einem Beispiel an:

```c
filter {
  if [action] == "login" {
    mutate { remove => "password" }
  }
}
```

Mehrere Ausdrücke können in einer einzigen Anweisung mit booleschen Operatoren angegeben werden.

Ein Beispiel lässt sich wie folgt veranschaulichen:

```c
output {
  # Send Email on Production Errors
  if [loglevel] == "ERROR" and [deployment] == "production" {
   email{

     }

  }
}
```

### Arten von Logstash-Plugins

Im Folgenden sind Typen von Logstash-Plugins aufgeführt:

*  Input 
*  Filter 
*  Output 
*  Codec 

Nun wollen wir einen Blick auf einige der wichtigsten Input-, Output-, Filter-und Codec-Plugins, die nützlich sein wirden für den Bau der meisten der Log-Analysen Pipeline Use Cases.

#### Eingabe-Plugins

Ein Eingabe-Plugin wird verwendet, um einen Satz von Ereignissen zu konfigurieren, die Logstash zugeführt werden sollen. Einige der wichtigsten Eingabe-Plugins sind:

#### Datei(file)

Das `file` Plugin wird verwendet, um Ereignisse und Protokollzeilen-Dateien in Logstash zu streamen. Es erkennt automatisch Dateivrehungen und liest aus dem zuletzt gelesenen Punkt.

###### Notiz:
Das Logstash `file` Plugin unterhält `sincedb` Dateien, um die aktuellen Positionen in überwachten Dateien zu verfolgen. Standardmäßig schreibt es `sincedb` Dateien auf `$HOME/.sincedb*` Pfad. Der Standort und die Frequenz können mit `sincedb_path` und `sincedb_write_interval` Eigenschaften des Plugins geändert werden.

Eine möglichst einfache Dateikonfiguration sieht so aus:

```c
input{
file{
path => "/path/to/logfiles"
}
```

Die einzige erforderliche Konfigurationseigenschaft ist der Pfad zu den Dateien. Schauen wir uns an, wie wir einige der Konfigurationseigenschaften des Datei-Plugins nutzen können, um verschiedene Arten von Dateien zu lesen.
Konfigurationsoptionen

Für das Dateieingabe-Plugin stehen folgende Konfigurationsoptionen zur Verfügung:

#### add_field

Es wird verwendet, um ein Feld zu eingehenden Ereignissen hinzuzufügen, sein Werttyp ist `Hash` und der Standardwert ist `{}`.

Nehmen wir die folgende Instanz als Beispiel:

`add_field => { "input_time" => "%{@timestamp}" }`

#### codec
Es wird verwendet, um einen Codec anzugeben, der eine bestimmte Art von Eingabe decodieren kann.

Zum Beispiel: `codec => "json"` wird verwendet, um den `Json`-Typ der Eingabe zu decodieren.

Der Standardwert des Codecs ist `"plain"`.

#### delimiter (Trennzeichen)

Es wird verwendet, um ein Trennzeichen anzugeben, das separate Zeilen identifiziert. Standardmäßig ist es `"\n"`.

#### exclude (ausschließen)

Um bestimmte Dateitypen aus dem Eingabepfad auszuschließen, ist der Datentyp Array.

Nehmen wir die folgende Instanz als Beispiel:

```c
path =>["/app/packtpub/logs/*"]
exclude => "*.gz"
```

Dies schließt alle gzip-Dateien aus der Eingabe aus.

#### path (Pfad)

Dies ist die einzige erforderliche Konfiguration für das Datei-Plugin. Es gibt ein Array von Pfadpositionen an, von wo aus Protokolle und Ereignisse gelesen werden sollen.

#### Sincedb_path

Es gibt den Ort an, an dem die `sincedb` Dateien geschrieben werden sollen, die die aktuelle Position der zu überwachenden Dateien verfolgen. Die Voreinstellung ist `$HOME/.sincedb*`

#### Sincedb_write_interval

Es gibt an, wie oft (Anzahl in Sekunden) die `sincedb` Dateien, die die aktuelle Position der überwachten Dateien verfolgen, geschrieben werden sollen. Die Voreinstellung ist 15 Sekunden.

#### start_position

Es hat zwei Werte: `"beginning"` und `"end"`. Es gibt an, wo man anfängt, eingehende Dateien zu lesen. Der Standardwert ist `"end"`, da in den meisten Situationen dies für Live-Streaming-Daten verwendet wird. Obwohl, wenn Sie an alten Daten arbeiten, kann es auf "beginning" gesetzt werden.

###### Notiz

Diese Option hat nur Auswirkungen, wenn eine Datei zum `"first contact"` gelesen wird, genannt "erster Kontakt", da sie den Standort im Ort `"sincedb"` pflegt. Also für die nächste Einstellung, diese Option hat keine Auswirkungen, es sei denn, Sie entscheiden, die `sincedb` Dateien zu entfernen.

#### tags

Es gibt das Array von Tags an, die zu eingehenden Ereignissen hinzugefügt werden können. Hinzufügen von Tags zu Ihren eingehenden Ereignissen hilft bei der Verarbeitung später bei der Verwendung von Bedingungen. Es ist oft hilfreich, bestimmte Daten als `"processed"` zu markieren und diese Tags zu verwenden, um eine zukünftige Vorgehensweise zu entscheiden.

Zum Beispiel, wenn wir `"processed"` in Tags:

`tags =>["processed"]`

Im Filter können wir die Voraussetzungen beachten:

```c
filter{
if  "processed" in tags[]{

}
}
```

#### type

Die `type` Option ist wirklich hilfreich, um die verschiedenen Arten von eingehenden Streams mit Logstash zu verarbeiten. Sie können mehrere Eingabewege für verschiedene Arten von Ereignissen konfigurieren, geben Sie einfach einen `type` namen, und dann können Sie sie separat filtern und verarbeiten.

Nehmen wir die folgende Instanz als Beispiel:

```c
input {
file{
path => ["var/log/syslog/*"]
type => "syslog"
}
file{
path => ["var/log/apache/*"]
type => "apache"
}
}
```

Im `filter` können wir je nach Typ filtern:

```c
filter {
if [type] == "syslog" {
grok {

}

}
if [type] == "apache" {
grok {

}
}
}
```

Wie im vorigen Beispiel haben wir einen separaten Typ für eingehende Dateien konfiguriert. `"Syslog"` und `"apache"`. Später in der Filterung des Streams, können wir die Bedingungen für die Filterung auf der Grundlage dieser Art.

### stdin

Das `stdin` Plugin dient zum Streamen von Ereignissen und Protokolllinien von der Standard-Eingabe.

Eine Grundkonfiguration für `stdin` sieht so aus:

```c
stdin {

}
```

Wenn wir stdin so konfigurieren, wird alles, was wir in die Konsole eingeben, als Eingabe in die Logstash-Event-Pipeline gehen. Dies wird vor allem als erste Stufe des Testens der Konfiguration verwendet, bevor die eigentliche Datei oder die Ereigniseingabe eingefügt wird.

### Konfigurationsoptionen

Für das `stdin` Eingabe-Plugin stehen folgende Konfigurationsoptionen zur Verfügung:

#### add_field

Die `add_field` Konfiguration für `stdin` ist die gleiche wie `add_field` im `file` eingabe-Plugin und wird für ähnliche Zwecke verwendet.

#### codec

Es wird verwendet, um eingehende Daten zu decodieren, bevor sie an die Datenpipeline weitergegeben werden. Der Standardwert ist `"line"`.

##### tags

Die `tags` Konfiguration für `stdin` ist die gleiche wie `tags` im `file` Eingabe-Plugin und wird für ähnliche Zwecke verwendet.

#### type

Die `typ` konfiguration für `stdin` ist die gleiche wie der `type` im `file` eingabe-Plugin und wird für ähnliche Zwecke verwendet.

### Twitter

Möglicherweise müssen Sie einen Twitter-Stream basierend auf einem Thema von Interesse für verschiedene Zwecke, wie Sentiment-Analyse, Trending Themen-Analyse, und so weiter zu analysieren. Das `Twitter` Plugin ist hilfreich, um Ereignisse von der Twitter-Streaming-API zu lesen. Dies erfordert einen Verbraucher Schlüssel, Verbraucher Geheimnis, Keyword, oauth Token und oauth Token Geheimnis zu arbeiten.

Diese Details können durch die Registrierung einer Anwendung auf der Twitter-Entwickler-API-Seite (https://dev.twitter.com/apps/new) erhalten werden:

```c
twitter {
    consumer_key => "your consumer key here"
    keywords => "keywords which you want to filter on streams" 
    consumer_secret => "your consumer secret here"
    oauth_token => "your oauth token here"
    oauth_token_secret => "your oauth token secret here"
}
```

#### Konfigurationsoptionen

Für das `twitter` input plugin stehen folgende Konfigurationsoptionen zur Verfügung:

#### add_field

Die `Add_field` Konfiguration für das `Twitter` Plugin ist das gleiche wie `add_field` im `file` eingabe-Plugin und wird für ähnliche Zwecke verwendet.

#### codec

Die `codec`Konfiguration für `twitter` ist die gleiche wie das Codec-Plugin im `file` ingabe-Plugin und wird für ähnliche Zwecke verwendet.

##### Consumer_key

Dies ist eine erforderliche Konfiguration ohne Standardwert. Sein Wert kann von der Twitter-App-Registrierungsseite bezogen werden. Sein Wert ist der `string` Typ.

##### Consumer_secret

Das gleiche wie `consumer_key`, sein Wert kann von der Twitter dev App Registrierung erhalten werden.

##### full_tweet

Dies ist eine boolesche Konfiguration mit dem Standardwert `false` . Es gibt an, ob ein vollständiges tweet-Objekt aufgeführt wird, das von der Twitter-Streaming-API erhalten wird.

#### keywords (Schlüsselwörter)

Dies ist eine `array` Typ-erforderliche Konfiguration ohne Standardwert. Es gibt eine Reihe von `keywords` , die aus dem Twitter-Stream zu verfolgen sind.

Ein Beispiel lässt sich wie folgt sehen:

`keywords  => ["elk","packtpub"]`

#### oauth_token

Die Option `oauth_token` wird auch von der Twitter dev API Seite erhalten.

###### Notiz

Nachdem du dein Benutzerschlüssel- und Konsumentengeheimnis bekommen hast, klicke auf **Create My Access Token**, um dein oauth-Token und oauth-Token-Geheimnis zu erstellen.

#### Oauth_token_secret

Die Option `oauth_token_secret` wird von der `twitter` dev API Seite erhalten.

### tags

Die `tags` Konfiguration für das `twitter` Eingangs-Plugin ist das gleiche wie die Tags im `file` eingabe-Plugin und wird für ähnliche Zwecke verwendet.

#### typ

`typ`Konfiguration für `twitter`Eingabe-Plugins ist die gleiche wie `type` in der `file` Eingabe-Plugin und wird für ähnliche Zwecke verwendet.

### lumberjack

Das `lumberjack` Plugin ist nützlich, um Ereignisse über das Holzfäller-Protokoll zu erhalten, das im Logstash-Forwarder verwendet wird.

Die grundlegende erforderliche Konfigurationsoption für das `lumberjack` Plugin sieht so aus:

```c
lumberjack {
    port => 
    ssl_certificate => 
    ssl_key => 
}
```

###### Tip
Holzfäller oder Logstash Forwarder ist ein leichter Log-Absender, der verwendet wird, um Log-Events aus Quellsystemen zu versenden. Logstash ist ein sehr speicherbedingter Prozess, so dass die Installation auf jedem Knoten, von wo aus Sie Daten versenden möchten, nicht empfohlen wird. Logstash Forwarder ist eine leichte Version von Logstash, die eine geringe Latenz, sichere und zuverlässige Übertragung bietet und eine geringe Ressourcennutzung bietet.

Mehr Details über Holzfäller oder Logstash Forwarder finden Sie hier:

Https://github.com/elastic/logstash-forwarder

#### Konfigurationsoptionen

Folgende Konfigurationsoptionen stehen für das `lumberjack` Eingangs-Plugin zur Verfügung:

#### add_field

Die `Add_field` Konfiguration für das `lumberjack` Plugin ist das gleiche wie `add_field` im `file` eingabe-Plugin und wird für ähnliche Zwecke verwendet.

#### codec

Die `codec` Konfiguration für das `lumberjack` Plugin ist das gleiche wie das `codec` Plugin im Dateieingabe-Plugin und wird für ähnliche Zwecke verwendet.

#### host

Es spezifiziert den Host, auf dem zu hören ist. Der Standardwert: ``"0.0.0.0".

#### port

Dies ist eine Nummer Typ erforderlich Konfiguration und es gibt den Port zu lauschen an. Es gibt keinen Standardwert.

#### ssl_certificate

Es gibt den Pfad zum SSL-Zertifikat an, das für die Verbindung verwendet werden soll. Es ist eine erforderliche Einstellung.

Ein Beispiel ist wie folgt:

`ssl_certificate => "/etc/ssl/logstash.pub"`

##### ssl_key
Es gibt den Pfad zum SSL-Schlüssel an, der für die Verbindung verwendet werden muss. Es ist auch eine erforderliche Einstellung.

Ein Beispiel ist wie folgt:

`ssl_key => "/etc/ssl/logstash.key"`

#### Ssl_key_passphrase

Es gibt die SSL-Schlüsselpassphrase an, die für die Verbindung verwendet werden muss.

#### tags

Die `tags` Konfiguration für das `lumberjack` Eingangs-Plugin ist das gleiche wie die `tags` im `file` eingabe-Plugin und wird für ähnliche Zwecke verwendet.

#### type

Die `type` Konfiguration für die `lumberjack` Eingangs-Plugins ist die gleiche wie der Typ in der Datei-Eingabe-Plugin und wird für ähnliche Zwecke verwendet.

### Redis
Das `redis` Plugin dient zum Lesen von Ereignissen und Protokollen aus der `redis` Instanz.

###### Notiz
Redis wird häufig in ELK Stack als Broker für eingehende Protokolldaten aus dem Logstash Forwarder verwendet, was hilft, Daten zu warten, bis der Indexer bereit ist, Protokolle zu erfassen. Dies hilft, das System unter starker Last zu kontrollieren.

Die Grundkonfiguration des redis-Eingangs-Plugins sieht wie folgt aus:

```
redis {
}
```

#### Konfigurationsoptionen

Für das `redis` input plugin stehen folgende Konfigurationsoptionen zur Verfügung:

#### add_field

Die `add_field` Konfiguration für `redis` ist die gleiche wie `add_field` im `file` eingabe-Plugin und wird für ähnliche Zwecke verwendet.

#### codec

Die `codec` Konfiguration für `redis` ist die gleiche wie `codec` in der `file` Eingabe-Plugin und wird für ähnliche Zwecke verwendet.

##### data_type

Die Option `data_type` kann einen Wert als `"list"`, `"channel"` oder `"pattern_channel"` haben.

Aus der Logstash-Dokumentation zum `redis` Plugin (https://www.elastic.co/guide/de/logstash/current/plugins-inputs-redis.html):

Wenn `redis_type` eine `list` ist, dann werden wir den Schlüssel BLPOP.Wenn `redis_type` `channel` ist, dann werden wir den Schlüssel abmelden.Wenn `redis_type` ist `pattern_channel`, dann werden wir PSUBSCRIBE zum Schlüssel.

###### Tip

Bei der Verwendung von `redis` auf der Consumer- und Publisher-Seite sollten `key` und `data_type` auf beiden Seiten gleich sein.

#### host

Es gibt den Hostnamen des `redis` Servers an. Der Standardwert ist `"127.0.0.1"`.

#### key

Es gibt den Schlüssel für redis an; `"list"` oder `"channel"`.

#### password

Es handelt sich um eine `password` typkonfiguration, die das Kennwort für die Verbindung festlegt.

##### Port

Es gibt den Port an, auf dem die `redis` Instanz läuft. Die Voreinstellung ist `6379`.

Eine ausführliche Liste und neueste Dokumentation zu allen verfügbaren Logstash-Eingabe-Plugins finden Sie unter https://www.elastic.co/guide/de/logstash/current/input-plugins.html.

Nun, da wir einige der wichtigsten Input-Plugins für Logstash gesehen haben, lasst uns einen Blick auf einige Output-Plugins werfen.

### Output-Plugins

Logstash bietet eine Vielzahl von Output-Plugins, die dazu beitragen, eingehende Ereignisse mit fast jeder Art von Ziel zu integrieren. Schauen wir uns einige der am meisten verwendeten Output-Plugins im Detail an.

#### Csv

Das `csv` Plugin wird verwendet, um eine `csv` Datei als Ausgabe zu schreiben, wobei die Felder in `csv` und der Pfad der Datei angegeben werden.

Die Grundkonfiguration des `csv` Ausgangs-Plugins sieht so aus:

```
csv {
    fields => ["date","open_price","close_price"]
    path => "/path/to/file.csv"
}
```

##### Konfigurationsoptionen

Im Folgenden stehen die Konfigurationsoptionen für das `csv` Plugin zur Verfügung:

#### codec

Es wird verwendet, um die Daten zu codieren, bevor es aus Logstash geht. Der Standardwert ist `"plain"`, der die Daten so ausgibt, wie er ist.

### csv_options

Die Option `csv_options` wird verwendet, um erweiterte Optionen für die `csv` Ausgabe festzulegen. Es beinhaltet das Ändern der Standardspalte und der Zeilen-Trennzeichen.

Ein Beispiel ist wie folgt:

`csv_options => {"col_sep" => "\t" "row_sep" => "\r\n"}` 

#### fieldes

Die `fields` einstellung ist eine erforderliche Einstellung, mit der die Felder für die CSV-Ausgabe ausgegeben werden. Es wird als Array von Feldnamen angegeben und in der gleichen Reihenfolge wie im Array geschrieben. Für diese Einstellung ist kein Standardwert.

#### gzip

Die `gzip` Einstellung ist eine boolesche Art von Einstellung, die angibt, ob sie als gzip komprimiertes Format ausgegeben werden soll oder nicht. Die Voreinstellung ist `false`.

##### path

Die `path` einstellung ist eine erforderliche Einstellung und wird verwendet, um den Pfad zur CSV-Datei anzugeben.

#### file

Das `file` ausgabe-Plugin, genau wie das `file` Eingabe-Plugin, wird verwendet, um Ereignisse in eine Datei im Dateisystem zu schreiben.

Die Grundkonfiguration des Dateianausgangs-Plugins sieht wie folgt aus:

```c
file
{
path = > "path/to/file"
}
```

#### Konfigurationsoptionen

Die verfügbaren Konfigurationsoptionen sind:

* codec
* gzip
* max_size
* path

Die meisten dieser Konfigurationsoptionen wurden früher behandelt und sind mit ihrem Namen gut verstanden.

#### Email

Das `email` Plugin ist ein sehr wichtiges Ausgangs-Plugin, da es sehr nützlich ist, E-Mails für bestimmte Ereignisse und Fehlerszenarien zu senden.

Die grundlegende erforderliche Konfiguration sieht so aus:

```c
email {
    to => "abc@example.com"
}
```

#### Konfigurationsoptionen

Für das `email` Plugin stehen folgende Konfigurationsoptionen zur Verfügung:

#### attachments

Die Option `attachments` ist ein Array von Dateipfaden, die mit der E-Mail angehängt werden sollen. Der Standardwert ist `[]`

#### body

Die `body` Option legt den Text der E-Mail im Klartextformat fest. Der Standardwert ist "".

#### cc

Die Option `cc` gibt die Liste der E-Mails an, die als cc-Adressen in der E-Mail aufgenommen werden sollen. Es akzeptiert mehrere E-Mail-IDs in einem kommagetrennten Format.

#### from

Die Option `from` gibt die E-Mail-Adresse an, die als Absenderadresse in der E-Mail verwendet werden soll. Der Standardwert lautet `"logstash.alert@nowhere.com"` und muss nach der Art der Warnungen oder des Systems überschrieben werden.

#### to

Die Option `to` ist eine erforderliche Einstellung, die die Empfängeradresse für die E-Mail spezifiziert. Es kann auch als eine Folge von durch Kommas getrennten E-Mail-Adressen ausgedrückt werden.

#### htmlbody

Die Option `htmlbody` gibt den Text der E-Mail im HTML-Format an. Es enthält HTML-Mark-up-Tags in der E-Mail-Stelle.
Antwort an

#### replyto

Die Option `replyto` gibt die E-Mail-Adresse an, die für das `Reply-To` Feld für die E-Mail verwendet werden soll.

#### subject

Die `subject` gibt den Betreff für die E-Mail an. Der Standardwert ist `""`.

### elasticsearch

Das `elastiksearch` Plugin ist das wichtigste Plugin, das in ELK Stack verwendet wird, denn es ist, wo Sie Ihre Ausgabe schreiben möchten, um gespeichert zu werden, um später in Kibana zu analysieren.

Die Grundkonfiguration für das Elastiksearch Plugin sieht so aus:

```c
elasticsearch {
}
```

Konfigurationsoptionen

Einige der wichtigsten Konfigurationsoptionen werden wie folgt erwähnt:

|option|daten typ|Essential|default wert|
| :----: | :---: | :---: | :---: |
|`action`|`string`|`N`|` "index" `|
|`bind_host`|`string`|`N`|``|
|`bind_port`|`number`|`N`|``|
|`cacert`|ein passender system Pfad|`N`|``|
|`cluster`|`string`|`N`|``|
|`document_id`|`string`|`N`|``|
|`document_type`|`string`|`N`|``|
|`host`|`array`|`N`|``|
|`index`|`string`|`N`|` "logstash-%{+YYYY.MM.dd}" `|
|`max_retries`|`number`|`N`|`3`|
|`node_name`|`string`|`N`|``|
|` password `|`password`|`N`|``|
|`port`|`string`|`N`|``|
|`user`|`string`|`N`|``|


### ganglia

ganglia ist ein Monitoring-Tool, das verwendet wird, um die Leistung eines Clusters von Maschinen in einer verteilten Computerumgebung zu überwachen. ganglia nutzt einen Dämon namens Gmond, der ein kleiner Dienst ist, der auf jeder Maschine installiert ist, die überwacht werden muss.

Das `ganglia` Ausgangs-Plugin in Logstash wird verwendet, um Metriken an den `gmond` Service zu senden, basierend auf Ereignissen in Protokollen.

Die grundlegende `ganglia` Output-Plugin-Konfiguration sieht so aus:

```c
ganglia {
    metric => 
    value => 
}
```

### Konfigurationsoptionen

Für das Ganglien-Plugin stehen folgende Konfigurationsoptionen zur Verfügung

#### metric

Die `metric` option gibt die Metrik an, die für die Leistungsüberwachung verwendet werden soll. Es kann sogar Werte aus den Veranstaltungsfeldern nehmen.

#### unit

Die `unit` Option gibt die Einheit wie kb/s, ms für die verwendete Metrik an.

##### value

Die `value` option gibt den Wert der verwendeten Metrik an.

### Jira

Das `jira` Plugin kommt bei der Logstash-Installation nicht standardmäßig zur Verfügung, kann aber problemlos von einem Plugin `install` befehl wie folgt installiert werden:

`bin/plugin install logstash-output-jira`

Das `jira` Plugin wird verwendet, um Ereignisse an eine JIRA-Instanz zu senden, die JIRA-Tickets basierend auf bestimmten Ereignissen in Ihren Protokollen erstellen kann. Um dies zu verwenden, muss die JIRA-Instanz REST-API-Aufrufe akzeptieren, da sie intern die JIRA REST-API verwendet, um die Ausgabeereignisse von Logstash zu JIRA zu übergeben.

Die Grundkonfiguration des `jira` Ausgangs-Plugins sieht so aus:

```c
jira {
    issuetypeid => 
    password => 
    priority => 
    projectid =>
    summary => 
    username => 
}
```

### Konfigurationsoptionen

Im Folgenden finden Sie die Konfigurationsoptionen und die entsprechenden Datentypen für das jira plugin:

|Option|Daten Typ|Benötigt|
| :---: | :---: | :---: |
|`assignee`|`string`|`N`|
|`issuetypeid`|`string`|`Y`|
|`password`|`string`|`Y`|
|`priority`|`string`|`Y`|
|`projectid`|`string`|`Y`|
|`reporter`|`string`|`N`|
|`summary`|`string`|`Y`|
|`username`|`string`|`Y`|


### Kafka

Wie auf der Hortonworks Kafka Seite (http://hortonworks.com/hadoop/kafka/) erklärt:

"Apache ™ Kafka ist ein schnelles, skalierbares, langlebiges und fehlertolerantes Publish-Abonnement-Messaging-System."

Das `kafka` Ausgangs-Plugin wird verwendet, um bestimmte Ereignisse zu einem Thema auf `kafka` zu schreiben. Es nutzt die Kafka Producer API, um Nachrichten zu einem Thema auf dem Broker zu schreiben.

Die grundlegende `kafka` Konfiguration sieht so aus:

```c
kafka {
    topic_id => 
}
```

#### Konfigurationsoptionen

Es gibt viele `kafka` spezifische Konfigurationsoptionen, die aus der offiziellen Dokumentation erhalten werden können, aber die einzige erforderliche Konfiguration ist `topic_id`.

#### topic_id

Die Option `topic_id` definiert das Thema zum Senden von Nachrichten an.

### lumberjack

Das `lumberjack` Plugin wird verwendet, um Ausgabe an einen Logstash-Forwarder oder `lumberjack` zu schreiben.

Die Grundkonfiguration für das `lumberjack` Plugin sieht so aus:

```c
lumberjack {
    hosts => 
    port => 
    ssl_certificate => 
}
```

#### Konfigurationsoptionen

Für das `lumberjack` Plugin stehen folgende Konfigurationsoptionen zur Verfügung:

#### hosts

Die `hostd` Option gibt die Liste der Adressen an, in denen `lumberjack` Nachrichten senden können.

#### port

Die Port-Option gibt den Port an, um eine Verbindung zur `lumberjack` -Kommunikation herzustellen.

#### ssl_certificate

Es gibt den Pfad zu `ssl_certificate` an, der für die Kommunikation verwendet werden soll.

### Redis

Das `redis`Plugin wird verwendet, um Ereignisse an eine `redis`Instanz zu senden.
Konfigurationsoptionen

Die Konfigurationsoptionen entsprechen denen, die für das `redis` input plugin definiert wurden.

### Rabbitmq

###### Hinweis

RabbitMQ ist eine Open-Source-Message-Broker-Software (manchmal auch als Message-orientierte Middleware bezeichnet), die das Advanced Message Queuing Protocol (AMQP) implementiert. Weitere Informationen finden Sie in der offiziellen Dokumentation unter http://www.rabbitmq.com.

In RabbitMQ sendet der Produzent immer Nachrichten an einen Austausch, und der Austausch entscheidet, was mit den Nachrichten zu tun ist. Es gibt verschiedene Austauscharten, die einen weiteren Vorgang für die Nachrichten definieren, nämlich `direct`, `topic`, `headers `and `fanout`.

Das `rabbitmq` Plugin drückt die Ereignisse von Protokollen an die RabbitMQ-Börse.

Die Grundkonfiguration des `rabbitmq` Plugins sieht so aus:

```c
rabbitmq {
    exchange => 
    exchange_type => 
    host => 
} 
```

### stdout

Das `stdout` Plugin schreibt die Ausgabeereignisse an die Konsole. Es wird verwendet, um die Konfiguration zu debuggen, um die Ereignisausgabe von Logstash zu testen, bevor sie mit anderen Systemen integriert wird.

Die Grundkonfiguration sieht so aus:

```c
output {
  stdout {}
}
```

### Mongodb

MongoDB ist eine dokumentorientierte NoSQL-Datenbank, die Daten als JSON-Dokumente speichert.

Wie das `Jira` Plugin ist das auch ein Community-Plugin und wird nicht mit Logstash ausgeliefert. Es kann einfach mit dem folgenden Plugin-Installationsbefehl `install` werden:
`bin/plugin install logstash-output-mongodb`

Die Grundkonfiguration für das `mongodb` Ausgangs-Plugin ist:

```c
mongodb {
    collection => 
    database => 
    uri => 
}
```

#### Konfigurationsoptionen

Für das `mongodb` Plugin stehen folgende Konfigurationsoptionen zur Verfügung:

#### collection

Die `collection` option gibt an, welche `mongodb` Sammlung zum Schreiben von Daten verwendet werden muss.

#### database

Die `database` option gibt die `mongodb` Datenbank an, die zum Speichern der Daten verwendet werden soll.

#### uri

Die Option `uri` gibt die Verbindungszeichenfolge an, die für die Verbindung zu Mongodb verwendet werden soll.

Eine ausführliche Liste und neueste Dokumentation zu allen verfügbaren Logstash-Ausgabedateien finden Sie unter https://www.elastic.co/guide/de/logstash/current/output-plugins.html.

### Filter-Plugins filtern

Filter-Plugins werden verwendet, um die Zwischenverarbeitung auf Ereignissen zu tun, die von einem Eingangs-Plugin gelesen werden und bevor sie als Ausgabe über ein Ausgangs-Plugin übergeben werden. Sie werden häufig verwendet, um die Felder in Eingabeereignissen zu identifizieren und bestimmte Teile von Eingabeereignissen bedingt zu verarbeiten.

Werfen wir einen Blick auf einige der wichtigsten Filter-Plugins.

#### csv

Der `csv` Filter wird verwendet, um die Daten aus einer eingehenden CSV-Datei zu analysieren und den Feldern Werte zuzuordnen.

#### Konfigurationsoptionen

Konfigurationsoptionen für das `csv` Filter-Plugin wurden in einem Beispiel in, [Aufbau Ihrer ersten Daten-Pipeline mit ELK](../elk-stack-data-pipeline) abgedeckt.

#### date

Bei ELK ist es sehr wichtig, dem Ereignis den richtigen Zeitstempel zuzuordnen, damit er auf dem `time` filter in Kibana analysiert werden kann. Das Datumsfilter soll den entsprechenden Zeitstempel basierend auf Feldern in Protokollen oder Ereignissen zuweisen und dem Zeitstempel ein entsprechendes Format zuweisen.

Wenn das `date` filter nicht gesetzt ist, wird Logstash einen Zeitstempel zuweisen, wenn das erste Mal das Ereignis sieht oder wenn die Datei gelesen wird.

Die Grundkonfiguration des `date` filters sieht wie folgt aus:

```c
date {
} 
```

#### Konfigurationsoptionen
Die Konfigurationsoptionen für das Datumsfilter sind bereits in einem Beispiel enthalten
[Aufbau Ihrer ersten Daten-Pipeline mit ELK](../elk-stack-data-pipeline)

#### drop

Der `drop` filter wird verwendet, um alles zu fallen, was den Bedingungen für diesen Filter entspricht.

Nehmen wir die folgende Instanz als Beispiel:

```c
filter {
if [fieldname == "test"] {
drop {
}
}
}
```

Der vorhergehende Filter führt dazu, dass alle Ereignisse mit dem Testfeldname gelöscht werden. Dies ist sehr hilfreich, um nicht nützliche Informationen aus den eingehenden Ereignissen herauszufiltern.

#### Konfigurationsoptionen

Folgende Konfigurationsoptionen sind für diesen Filter vorhanden:

* `add_field` 
* `add_tag` 
* `remove_field`
* `remove_tag`

### geoip

Der `geoip` Filter wird verwendet, um die geografische Lage der im eingehenden Ereignis vorhandenen IP-Adresse hinzuzufügen. Es holt diese Informationen aus der Maxmind-Datenbank.

###### Hinweis

Maxmind ist ein Unternehmen, das sich auf Produkte spezialisiert hat, die gebaut wurden, um nützliche Informationen aus IP-Adressen zu erhalten. GeoIP ist ihr IP-Intelligenzprodukt, mit dem der Standort einer IP-Adresse verfolgt wird. Alle Logstash-Versionen haben eine MaxMinds GeoLite-Stadtdatenbank mit ihnen ausgeliefert. Es ist auch unter http://dev.maxmind.com/geoip/legacy/geolite/ verfügbar.

Die Grundkonfiguration des `geoip` Filters sieht so aus:

```c
geoip {
    source => 
}
```

#### Konfigurationsoptionen

Für das Geoip Plugin steht folgende Konfigurationsoption zur Verfügung.

#### source

Die `source` option ist eine erforderliche Einstellung, die vom `string` Typ ist. Es wird verwendet, um eine IP-Adresse oder einen Hostnamen anzugeben, der über den `geoip` Dienst abgebildet werden muss. Jedes Feld aus Ereignissen, die die IP-Adresse oder den Hostnamen enthalten, kann bereitgestellt werden, und wenn das Feld vom Array-Typ ist, wird nur der erste Wert genommen.

#### grok

Die `grok` Option ist bei weitem das beliebteste und leistungsstärkste Plugin, das Logstash hat. Es kann jedes unstrukturierte Log-Ereignis analysieren und es in einen strukturierten Satz von Feldern umwandeln, die weiter verarbeitet und in der Analyse verwendet werden können.

Es wird verwendet, um jede Art von Protokollen zu analysieren, sei es Apache-Logs, MySQL-Protokolle, benutzerdefinierte Anwendungsprotokolle oder einfach nur unstrukturierter Text in Ereignissen.

Logstash, standardmäßig, kommt mit einem Satz von Grok-Mustern, die direkt verwendet werden können, um bestimmte Arten von Feldern zu markieren, und benutzerdefinierte reguläre Ausdrücke werden auch unterstützt.

Alle verfügbaren `grok` Muster sind erhältlich unter:

Https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns

Einige Beispiele für die `grok` Muster sind wie folgt:

```sh
HOSTNAME \b(?:[0-9A-Za-z][0-9A-Za-z-]{0,62})(?:\.(?:[0-9A-Za-z][0-9A-Za-z-]{0,62}))*(\.?|\b)
DAY (?:Mon(?:day)?|Tue(?:sday)?|Wed(?:nesday)?|Thu(?:rsday)?|Fri(?:day)?|Sat(?:urday)?|Sun(?:day)?)
YEAR (?>\d\d){1,2}
HOUR (?:2[0123]|[01]?[0-9])
MINUTE (?:[0-5][0-9])
```


Die vorangehenden `grok` Muster können direkt verwendet werden, um Felder diesem Typens mit einem Operator wie folgt zu markieren:

`%{HOSTNAME:host_name}`

Hier ist `host_name` der Feldname, den wir dem Teil des Protokollereignisses zuordnen wollen, der den Hostnamen wie string darstellt.

Lassen Sie uns versuchen, `grok` im Detail zu betrachten:

Die `grok` Muster in Protokollen werden durch dieses allgemeine Format dargestellt: `%{SYNTAX: SEMANTIC}`

Hier ist `SYNTAX` der Name des Musters, das dem Text im Protokoll entspricht, und `SEMANTIC` ist der Feldname, den wir diesem Muster zuordnen wollen.

Nehmen wir die folgende Instanz als Beispiel:

Angenommen, Sie möchten die Anzahl der in einem Ereignis übertragenen Bytes darstellen:

`%{NUMBER:bytes_transferred}`

Hier bezieht sich `bytes_transferred` auf den tatsächlichen Wert der im Log-Ereignis übertragenen Bytes.

Lassen Sie uns einen Blick darauf werfen, wie wir eine Zeile aus HTTP-Protokollen darstellen können:

`54.3.245.1 GET /index.html 14562  0.056`

Das `grok` Muster würde wie folgt dargestellt:

```c
%{IP:client_ip} %{WORD: request_method } %{URIPATHPARAM:uri_path} %{NUMBER:bytes_transferred} %{NUMBER:duration}
```

Die grundlegende `grok` Konfiguration für das vorhergehende Ereignis sieht so aus:

```c
filter{
grok{
match =>  { "message" =>"%{IP:client_ip} %{WORD:request_method} %{URIPATHPARAM:uri_path} %{NUMBER:bytes_transferred} %{NUMBER:duration}"}
}
}
```

Nach der Bearbeitung mit diesem `grok`Filter können wir die folgenden Felder mit den Werten hinzufügen:

* `client_ip : 54.3.245.1`
* `request_method : GET`
* `uri_path :/index.html`
* `bytes_transferred :14562`
* `duration :0.056`

#### spezifische grok muster anpassen

Benutzerdefinierte `grok` Muster können basierend auf einem regulären Ausdruck erstellt werden, wenn nicht in der Liste der Grok-Muster zur Verfügung gestellt.

Diese URLs sind nützlich, um `grok` Muster für den passenden Text nach Bedarf zu entwerfen und zu testen:

Http://grokdebug.herokuapp.com und http://grokconstructor.appspot.com/

### mutieren

Der `mutate` Filter ist ein wichtiges Filter-Plugin, das die Umbenennung, das Entfernen, Ersetzen und Ändern von Feldern bei einem eingehenden Ereignis unterstützt. Es wird auch speziell verwendet, um den Datentyp von Feldern zu konvertieren, zwei Felder zu verschmelzen und Text von Kleinbuchstaben in Großbuchstaben umzuwandeln und umgekehrt.

Die Grundkonfiguration des `mutierte` Filters sieht so aus:

```c
filter {
mutate {
}
}
```

#### Konfigurationsoptionen

Es gibt verschiedene Konfigurationsmöglichkeiten für `mutate` und die meisten von ihnen werden unter dem Namen verstanden:

|Option|Daten Typ|Benötigt|Default wert|
| :---: | :----: | :---: | :----: |
|`add_field`|`add_field`|`N`|`{}`|
|`add_tag `|`array`|`N`|`[]`|
|`convert`|`hash`|`N`|``|
|`join`|`hash`|`N`|``|
|`lowercase`|`array`|`N`|``|
|`merge`|`hash`|`N`|``|
|`remove_field`|`array`|`N`|`[]`|
|`remove_tag`|`array`|`N`|`[]`|
|`rename`|`hash`|`N`|``|
|`replace`|`hash`|``|``|
|`split`|`hash`|`N`|``|
|`strip`|`array`|`N`|``|
|`update`|`hash`|`N`|``|
|`uppercase`|`array`|`N`|``|

#### sleep

Die `sleep`Option wird verwendet, um Logstash im `sleep`-Modus für die angegebene Zeit einzustellen. Wir können auch die Häufigkeit der Schlafintervalle auf der Grundlage der Anzahl der Ereignisse angeben.

Nehmen wir die folgende Instanz als Beispiel:

Wenn wir Logstash für 1 Sekunde für jedes fünfte Event verarbeiten lassen wollen, können wir es so konfigurieren:

```c
filter {
  sleep {
    time => "1"   # Sleep 1 second
    every => 5   # Sleep on every 5th event.
  }
}
```

Eine umfangreiche Liste und die aktuellste Dokumentation zu allen verfügbaren Logstash-Filter-Plugins finden Sie unter https://www.elastic.co/guide/de/logstash/current/filter-plugins.html.

### Codec-Plugins


Codec-Plugins werden verwendet, um eingehende oder ausgehende Ereignisse von Logstash zu codieren oder zu decodieren. Sie fungieren als Stromfilter in Input- und Output-Plugins.

Einige der wichtigsten Codec-Plugins sind:

* avro
* json
* line
* multiline
* plain
* rubydebug

Werfen wir einen Blick auf einige Details über einige der am häufigsten verwendeten.

### json

Wenn Ihr Eingabeereignis oder Ausgabeereignis aus vollständigen `json`Dokumenten besteht, ist das Json-Codec-Plugin hilfreich. Es kann definiert werden als:

```c
input{
stdin{
codec => json{
}
}
}
```

Oder es kann einfach definiert werden als:

```c
input{
stdin{
codec => "json"
}
}
```

### line

Der `line` codec wird verwendet, um jede Zeile in einer Eingabe als Ereignis zu lesen oder jedes ausgehende Ereignis als eine Zeile zu decodieren. Es kann definiert werden als:

```c
input{
stdin{
codec => line{
}
}
}
```

Oder es kann einfach definiert werden als:

```c
input{
stdin{
codec => "line"
}
}
```

### multiline

Der `multiline` Codec ist sehr hilfreich für bestimmte Arten von Veranstaltungen, wo Sie gerne mehr als eine Zeile als ein Ereignis zu nehmen. Das ist sehr hilfreich in Fällen wie Java Exceptions oder Stack Traces.

Beispielsweise kann die folgende Konfiguration eine vollständige Stapelüberwachung als ein Ereignis ausführen:

```c
input {
  file {
    path => "/var/log/someapp.log"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601} "
      negate => true
      what => previous
    }
  }
}
```

Dies wird alle Zeilen, die nicht mit einem Zeitstempel als Teil der vorherigen Zeile beginnen und betrachten alles als ein einziges Ereignis.

### plain

Das `plain` Plugin wird verwendet, um festzulegen, dass es keine Codierung oder Decodierung gibt, die für Ereignisse erforderlich ist, da sie von entsprechenden Input- oder Output-Plugin-Typen selbst übernommen werden. Für viele Plugins, wie `redis`, `mongodb` und so weiter, ist dies der Standard-Codec-Typ.

#### rubydebug

Das `rubydebug` Plugin wird nur mit Ausgabe-Ereignisdaten verwendet und druckt Ausgabe-Ereignisdaten mit der Ruby Awesome Print-Bibliothek.

Eine ausführliche Liste und neueste Dokumentation zu allen verfügbaren Logstash-Codec-Plugins finden Sie unter https://www.elastic.co/guide/de/logstash/current/codec-plugins.html.