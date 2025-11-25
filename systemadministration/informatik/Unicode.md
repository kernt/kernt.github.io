Die Entstehung von Sprachen erlaubte das Weitergeben von Wissen und dessen Erhalt über Generationen hinweg. Die Entwicklung von Schrift erleichterte den Wissenstransfer dann nochmal.

Viele der heute gebräuchlichen **Schriftsysteme sind Tausende von Jahren alt**. Demgegenüber ist digitaler Text relativ neu. Und die digitale Abbildung von Texten war in den Anfangsjahren primär auf die englische Sprache ausgerichtet. Doch mittlerweile spielt sich ein beachtlicher Anteil menschlicher Interaktionen in den weltumspannenden digitalen Netzen ab. **Menschen tauschen Informationen über Sprach- und Ländergrenzen hinweg aus**. Diese Veränderung erforderte ein Umdenken und den Aufbau einer einheitlichen Struktur für den Austausch von Texten unterschiedlichster Schrift- und Zeichensysteme. Gleichzeitig eröffneten sich durch den technologischen Fortschritt vollkommen neue Möglichkeiten, Zeichen darzustellen.

Ein anschauliches Beispiel: **Denken Sie an die beliebten Emojis auf Ihrem Handy**. Die bunten Icons lassen sich mit einer speziellen Tastatur wie Buchstaben verwenden, fast als wären Sie ein natürlicher Teil des Alphabets. Aber wie funktioniert das? Der Unicode-Standard liefert die Erklärung.

## Was ist Unicode?

Unicode, das ist die „Universelle Zeichencodierung”, abgeleitet vom englischen Begriff _„Universal Character Encoding”_. Es handelt sich dabei um einen **Standard zum Kodieren von Schriftzeichen in Binärdarstellung**. Dies ermöglicht das Speichern und Verarbeiten von Texten in digitalen Systemen.

Unicode ist insofern eine Neuerung, dass es nicht an die Formate und Kodierungen eines einzelnen Alphabets bestimmter menschlicher Sprache gebunden ist. Unicode wurde vielmehr mit dem Ziel geschaffen, als einheitlicher Standard für die Abbildung **sämtlicher** **von Menschen entwickelter Schriftsysteme und Zeichen** zu dienen.

Seit der Veröffentlichung von Unicode 1.0 zum Ende des Jahres 1991 ist der Standard seiner Bestimmung gerecht geworden. Unicode wird **intern von Browsern und Betriebssystemen als einheitliches Format verwendet**. Mit der vom Unicode Consortium im Jahr 2020 veröffentlichten Version 13.0 umfasste der Unicode-Standard inzwischen ein Repertoire von insgesamt 143.859 Zeichen.

Das Unicode Consortium ist eine in Kalifornien beheimatete nicht-gewinnorientierte Organisation, die die Weiterentwicklung des Standards vorantreibt. Mitglieder des Konsortiums sind führende Technologie-Unternehmen wie Adobe, Apple, Facebook, Google, IBM, Microsoft, Netflix und SAP. Der vom Unicode-Standard abgedeckte Zeichensatz ist vollständig **deckungsgleich mit dem** „Universal Coded Character Set“ **(UCS)**, der als ISO/IEC 10646 international genormt ist.

### Technische Basis für Schriftzeichenkodierung

Schrift und Texte sind im Leben eines modernen Menschen allgegenwärtig. Lesen und Schreiben gehören zu den ersten kulturellen Fähigkeiten, die ein Mensch in der Schule erlernt. Daher verwundert es nicht, dass viele Menschen das Vorhandensein digitaler Schrift schlicht als gegeben hinnehmen. Doch wie genau funktioniert die technische Darstellung von Schrift? Machen Sie mit uns einen **Ausflug in die Welt der digitalen Schriftkodierung**.

Zunächst ist es wichtig, zu verstehen, dass alle in einem digitalen System vorhandenen Informationen auf einer tieferen Ebene aus endlosen Ketten von Nullen und Einsen bestehen. Man spricht dabei auch von der „Binärdarstellung”. Der Binärcode ist an sich so etwas ein Alphabet. Allerdings gibt es im Binärcode nur zwei „Buchstaben”: Nullen und Einsen. Jede Stelle innerhalb einer Sequenz von Nullen und Einsen wird als „Bit“ bezeichnet.

Der grundlegende Trick der digitalen Informationstechnologie ist, die **Zeichen verschiedener Alphabete als Sequenzen von Nullen und Einsen** abzubilden. So lassen sich Zahlen und Buchstaben kodieren, jedoch auch alle sonstigen unterscheidbaren Zustände. Generell spricht man dabei von „Symbolen”. Je länger die Sequenz von Nullen und Einsen für die Darstellung eines einzelnen Symbols ist, desto mehr Symbole lassen sich darstellen. Dabei verdoppelt sich die Anzahl der möglichen Symbole mit jedem hinzugefügten Bit.

Ein konkretes Beispiel: Stellen Sie sich vor, wir hätten binäre „Wörter”, welche **zwei Bits** lang sind. Dann ließen sich damit **vier Zahlen** kodieren:

|   |   |
|---|---|
|**2-Bit-Wort**|**Zahl**|
|00|0|
|01|1|
|10|2|
|11|3|

Fügen wir **ein weiteres Bit** am Anfang der Sequenz dazu, **verdoppelt sich die Anzahl möglicher Bit-Wörter**. Diese bestehen aus den bereits bekannten Bit-Sequenzen, jeweils mit einer vorangestellten Null oder Eins. Somit können wir acht Zahlen kodieren:

|   |   |
|---|---|
|**3-Bit-Wort**|**Zahl**|
|000|0|
|001|1|
|010|2|
|011|3|
|100|4|
|101|5|
|110|6|
|111|7|

Fakt

Ein 8-Bit-Wort wird als **Octet oder Byte** bezeichnet.

Der Einfachheit halber haben wir hier exemplarisch die Kodierung von Zahlen gezeigt. Dasselbe Prinzip kommt jedoch in digitalen Systemen auch für die **Kodierung von Buchstaben oder jeglichen anderen Zeichen und Zuständen** zum Tragen. Hier das stark vereinfachte Beispiel einer Binärkodierung von Buchstaben:

|   |   |
|---|---|
|**3-Bit-Wort**|**Buchstabe**|
|000|A|
|001|B|
|010|C|

Beachten Sie, dass unsere bisherigen Erklärungen nichts mit Schriftarten zu tun haben. Wir besprechen hier lediglich das interne Modell, auf dessen Grundlage Zeichen digital abgebildet werden. Die **grafische Repräsentation eines Zeichens nennt man Glyphe**. Je nach eingesetztem Font gibt es unterschiedliche Glyphen für dasselbe Zeichen, und sogar innerhalb eines Fonts kann es mehrere Varianten für eine Glyphe geben. Denken Sie z. B. an verschiedene Gewichte, Ligaturen, Kursivschrift etc. Hier eine erweiterte Darstellung, die die Zuordnung vom Zeichen bis zur Glyphe umfasst:

|Binärdarstellung|Dezimalzahl|Kodiertes Zeichen|Glyphe|
|---|---|---|---|
|1000001|65|großes „A“ des lateinischen Alphabets|A|
|1100001|97|kleines „a“ des lateinischen Alphabets|a|
|0110000|48|Arabische Ziffer „0”|0|
|0111001|57|Arabische Ziffer „9”|9|
|11000100|196|großes „Ä“|Ä|
|11000001|193|großes „Á“|Á|

#### Terminologie der Schriftzeichenkodierung

Die digitale Schriftzeichenkodierung umfasst eine Reihe spezifischer Konzepte. Im Deutschen findet sich zum Teil eine **synonyme Nutzung der verschiedenen Begriffe**. Um eine präzise Unicode-Definition geben zu können, bilden wir hier auch die englischen Terme ab:

|Begriff|Bedeutung|Englischer Term|
|---|---|---|
|Zeichensatz|Menge möglicher Zeichen, z.B. Ziffern „0–9“, Buchstaben „a–z“, etc.|Character set|
|Codepunkt|Einem bestimmten Zeichen zugeordnete Nummer innerhalb der Code-Domäne|Code point|
|Zeichencode|Zuordnung eines jeden Zeichens zu genau einem Codepunkt|Coded character set|
|Zeichencodierung|Prozess zur Umwandlung eines Zeichens in eine technische Struktur, z. B. Binärdarstellung|Character encoding|

#### Übersicht gebräuchlicher Zeichenkodierungen

Vor dem Aufkommen von Unicode gab es eine große Vielzahl spezifischer Kodierungen. Die Norm war, eine **eigene Kodierung für jede Sprache oder Sprachfamilie** einzusetzen. Dies führte häufig zu Darstellungsfehlern und Daten-Inkonsistenzen. Um dem entgegenzuwirken, wurden Zeichenkodierungen oft als abwärtskompatible Übermenge eines bereits existierenden Standards modelliert. So baut der moderne Unicode-Standard auf der früheren Zeichencodierung ISO Latin-1 auf, welche wiederum auf dem [ASCII-Zeichencode](https://www.ionos.at/digitalguide/server/knowhow/ascii-american-standard-code-for-information-interchange/ "ASCII-American-Standard-Code-for-Information-Interchange") basiert.

|Zeichencodierung|Bits pro Zeichen|Mögliche Zeichen|Zeichensatz|
|---|---|---|---|
|ASCII|7 Bit|128|Buchstaben, Zahlen und Sonderzeichen der US-Amerikanischen Tastatur, sowie Steuerzeichen für Fernschreiber|
|ISO Latin-1 (ISO 8859-1)|8 Bit|256|Erste 128 Zeichen wie ASCII, weitere 128 Zeichen für Sonderzeichen europäischer Sprachen|
|Universal Coded Character Set 2 (UCS-2)|16 Bit|65.536|Zeichen des „Basic Multilingual Plane”(BMP); erste 256 Zeichen wie bei ISO Latin-1|
|Universal Coded Character Set 4 (UCS-4)|32 Bit|1.114.111|Zeichen des BMP und weitere darüber hinausgehende; insgesamt 143.859 Zeichen in Unicode Version 13.0; erste 256 Zeichen wie ISO Latin-1|
|UCS Transformation Format 8 Bit (UTF-8)|8/16/24/32 Bit|1.114.111|Beliebige Zeichen aus UCS-2 und UCS-4; erste 256 Zeichen wie ISO Latin-1|

### Aufbau des Unicode-Standards

Der Unicode-Standard definiert **Zeichen und korrespondierende Code-Punkte für Buchstaben, Silbenzeichen, Ideogramme, Satzzeichen, Sonderzeichen und Ziffern**. Dabei werden neben dem lateinischen das griechische, kyrillische, arabische, hebräische, thailändische Alphabet unterstützt. Ferner die japanischen (Katakana, Hiragana), chinesischen und koreanischen Schriften (Hangul). Hinzu kommen mathematische, kaufmännische und technische Sonderzeichen, sowie historische Steuerzeichen für Fernschreiber.

Die Zeichen sind in einer **Reihe von Zeichentabellen** zusammengefasst. Wir geben hier einen Überblick der gebräuchlichsten Zeichentabellen.

#### Schriftsysteme des Unicode-Standards

|Zeichentabelle|Enthält u. A. die Alphabete|
|---|---|
|Europäische Schriftsysteme|Armenisch, Georgisch, Griechisch, Lateinisch|
|Afrikanische Schriftsysteme|Äthiopisch, Ägyptische Hieroglyphen, Koptisch|
|Schriftsysteme des mittleren Ostens|Arabisch, Hebräisch, Syrisch|
|Zentralasiatische Schriftsysteme|Mongolisch, Tibetisch, Alt-Türkisch|
|Südasiatische Schriftsysteme|Brahmi, Tamilisch, Vedisch|
|Südostasiatische Schriftsysteme|Khmer, Rohingya, Thai|
|Schriftsysteme Indonesiens und Ozeaniens|Balinesisch, Buginesisch, Javanesisch|
|Ostasiatische Schriftsysteme|CJK (Chinesisch, Japanisch, Koreanisch), Hangul (Koreanisch), Hiragana (Japanisch)|
|Amerikanische Schriftsysteme|Cherokee, Kanadische Silbenschrift, Osage|

#### Symbole und Satzzeichen des Unicode-Standards

|Zeichentabelle|Enthält u. A. die Zeichen|
|---|---|
|Notationssysteme|Braille-Muster, musikalische Notenschrift, Duployan Stenographie|
|Satzzeichen|Satzzeichen der englischen Sprache, Satzzeichen europäischer Sprachen, CJK-Satzzeichen|
|Alphanumerische Symbole|Mathematische Buchstaben, eingekreiste Buchstaben|
|Technische Symbole|Symbole der Programmiersprache APL, Symbole für optische Texterkennung|
|Zahlen & Ziffern|Maya-Ziffern, Osmanische Siyaq-Zahlzeichen, Ziffern der sumerischen Keilschrift|
|Mathematische Symbole|Pfeile, Mathematische Operatoren, Geometrische Formen|
|Emoji & Piktogramme|Emoticons, Dingbats, weitere Piktogramme|
|Andere Symbole|Alchemistische Symbole, Währungszeichen, Schach-, Domino-, und Mahjong-Zeichen|

## Wofür wird Unicode verwendet?

Der Unicode-Standard dient in erster Linie als universelle Grundlage für **Verarbeitung, Speicherung und Austausch von Text in jeder Sprache**. Die meisten modernen Software-Komponenten, wie Bibliotheken, Protokolle, Datenbanken, etc., welche auf Text operieren, bauen auf Unicode auf. Wir verdeutlichen die Bandbreite an Einsatzmöglichkeiten anhand der folgenden Beispiele.

### Betriebssysteme

Unicode ist in den meisten modernen Betriebssystemen der **interne Standard für die Abbildung von Text**. Manche Betriebssysteme, wie Apples macOS, erlauben die Nutzung von Unicode-Zeichen in Dateinamen.

### Websites

Die Unicode-Variante [UTF-8](https://www.ionos.at/digitalguide/websites/webseiten-erstellen/utf-8-codierung-globaler-digitaler-kommunikation/ "UTF-8: Codierung globaler digitaler Kommunikation") hat sich als **Standard für die Kodierung von HTML-Dokumenten** durchgesetzt. Bereits 2016 benutzten mehr als 80 Prozent der weltweit meistbesuchten Websites UTF-8 für die Speicherung und Darstellung ihrer HTML-Dokumente. Für die Nutzung von nicht-ASCII Buchstaben in Domänen-Namen hat sich der [Punycode-Standard](https://www.ionos.at/digitalguide/domains/domainverwaltung/punycode/ "Punycode") durchgesetzt.

### Programmiersprachen

Viele moderne Programmiersprachen nutzen Unicode als Basis für die Verarbeitung von Text. Eine neuere Entwicklung ist die Möglichkeit, **Unicode-Zeichen zur Benennung von Variablen und Funktionen** zu nutzen. Dies ist unter anderem in ECMAScript/JavaScript möglich und wird im folgenden Code illustriert:

```none
let ︎ = true;
let  = false;
if (bool_var === ︎) {
 // …
}
```

### Datenbanken

Die beliebte und auf breiter Basis eingesetzte Datenbank **MySQL unterstützt den kompletten Unicode-Zeichensatz** mit der Zeichenkodierung „utf8mb4“. Bei Nutzung der Zeichenkodierung „utf8“ kommt es hingegen zum Verlust von Zeichen, deren Codepunkt mehr als 3 Bytes umfasst.

### Fonts

Fonts enthalten die für die grafische Darstellung von Text zum Einsatz kommenden Glyphen. Aufgrund der großen Menge der im Unicode-Standard enthaltenen Zeichen gibt es **kein Font, der sämtliche Zeichen enthält**. Selbst die Untermenge des Basic Multilingual Plane wird nur von wenigen Fonts komplett abgedeckt. Hier ein paar Beispiele:

|Unicode-Font|Glyphen|Lizenz|
|---|---|---|
|Noto|ca. 65.000|Open Font License|
|Sun-ExtA/B|ca. 50.000|Freeware|
|Unifont|ca. 63.000|GNU GPL|
|Code2000|ca. 63.000|Shareware|

## Wie setzt man Unicode ein?

In vielen Fällen setzen Nutzer Unicode ein, ohne sich dessen je bewusst zu werden. **Digitaler Text liegt in den meisten Dokumenten und Anwendungen als Unicode vor** und lässt sich vom Nutzer beliebig kopieren, einfügen und bearbeiten. Manchmal ergibt sich die Notwendigkeit für den Endnutzer, ein bestimmtes Unicode-Zeichen in Text einzufügen. Dafür gibt es verschiedene Möglichkeiten, die wir im Folgenden vorstellen.

### Spezielle Software-Tastaturen

Die Nutzung spezieller Software-Tastaturen ist die wohl gebräuchlichste Methode, Unicode-Zeichen in Text einzufügen. Auf Mobilgeräten allgegenwärtig erlauben Software-Tastaturen das **Umschalten zwischen Sprachen und den dazugehörigen Alphabeten**. Dabei ändert sich die Tastenbelegung, wobei alle Zeichen dem Unicode-Repertoire entstammen. Die Zeichen lassen sich beliebig mischen und miteinander in Texten kombinieren.

Ein gutes Beispiel hierfür sind die Emojis: **Emojis sind in Unicode ganz normale Zeichen** wie Buchstaben, Zahlen und Sonderzeichen. Wie von digitalen Zeichen gewohnt ist die Darstellung der Emojis unabhängig von deren interner Modellierung. Jedes Betriebssystem stellt ein und dasselbe Emoji leicht unterschiedlich dar.

Die nützlichen Software-Tastaturen finden sich nicht nur auf Mobilgeräten. Auch auf dem Desktop sind sie vorhanden. Sie lassen sich in Windows, macOS und vielen Linux-Distributionen leicht öffnen und stellen **je nach ausgewählter Sprache eine unterschiedliche Menge an Zeichen** dar. Da die Anzahl an Tasten limitiert ist, werden nicht sämtliche Unicode-Zeichen dargestellt. Vielmehr handelt es sich um eine sprachspezifische Auswahl der gebräuchlichsten Zeichen.

### Unicode-Zeichentabellen

Neben den Software-Tastaturen sind Unicode-Zeichentabellen der wohl nützlichste Weg, auf Unicode-Zeichen zuzugreifen. Zur Erinnerung: ein Zeichencode (_„Coded character set”_) ist die **Menge aller Zeichen mitsamt ihrer korrespondierenden eindeutigen Codepunkte**. Für eine solche Struktur bietet sich die Anordnung als Tabelle an, und in der Tat umfasst der Unicode-Standard genau solche, als [Code Charts](https://www.unicode.org/charts/ "Code Charts") bezeichnete Tabellen. Aus diesen Tabellen lassen sich einerseits spezifische Zeichen kopieren, um diese anderweitig zu verwenden. Andererseits kann man als Endnutzer den entsprechenden Codepunkt ablesen, z.B. um diesen als numerische Zeichen-Referenz zu nutzen — mehr dazu im nächsten Abschnitt.

Auch viele Desktop-Betriebssysteme enthalten eine Unicode-Zeichentabelle. Diese gibt einen Überblick aller verfügbaren **Unicode-Zeichen samt Codepunkt, Beschreibung und Glyphe**. Ein Zeichen lässt sich per Klick einfügen, bzw. kopieren. Eine Zeichentabelle lässt sich mit wenigen Zeilen Code auch selbst erzeugen. Wir zeigen im weiteren Verlauf des Artikels ein Beispiel in der Programmiersprache Python.

### Numerische Zeichen-Referenz

Im Mittelpunkt des Unicode-Standards steht die Zuordnung von Zeichen zu Codepunkten. Kennt man den Codepunkt eines Zeichens, lässt sich dieser verwenden, um das entsprechende **Zeichen in verschiedenen Kontexten einzubinden**. Unter Windows erfolgt die [Eingabe zum Einfügen von Unicode-Symbolen](https://support.microsoft.com/de-de/office/einf%C3%BCgen-von-ascii-und-unicode-symbolen-oder-zeichen-westliche-sprachen-d13f58d3-7bcb-44a7-a4d5-972ee12e50e0 "Einfügen von ASCII- und Unicode-Symbolen oder -Zeichen (westliche Sprachen) - Office-Support") über die normale Hardware-Tastatur unter Nutzung einer speziellen Tastenkombination. Beachten Sie, dass die Nummer des Codepunkts normalerweise in Hexadezimal-Darstellung eingegeben werden muss.

Am häufigsten benötigen Programmierer die numerischen Zeichen-Referenzen. Die hexadezimale Darstellung der Codepunkte erlaubt die **Abbildung eines Unicode-Zeichens in Zeichen des ASCII-Zeichensatzes**. Wir zeigen das Vorgehen hier in HTML; prinzipiell funktioniert dieses ebenso in Python, C++ etc.

Das generelle Schema, um ein Zeichen per numerischer Referenz einzubinden, **umfasst die Referenz selbst, sowie einen öffnenden und schließenden Terminus**: In HTML-Dokumenten wird die numerische Referenz mit „&#x“ eröffnet und mit „;“ abgeschlossen. Dazwischen wird ohne Leerzeichen der zwei- bis vierstellige Hexadezimal-Codepunkt eintragen. Es ergibt sich das Muster „&#xNNNN;“.

Um beispielshalber das **Copyright-Zeichen** „©“ **in ein HTML-Dokument einzufügen**, gehen wir nach folgendem Schema vor:

1. In einer **Unicode-Tabelle** nach dem Zeichen suchen.
2. Den zum Zeichen gehörigen **Codepunkt ablesen**.

Im Falle unseres Beispiels ist der Codepunkt als „U+00A9“ angegeben, wobei es sich um die hexadezimale Darstellung handelt.

3. Die **Zeichen-Referenz komponieren** und in HTML-Quelltext oder ein Markdown-Dokument eintragen.

In unserem Fall geben wir „©“ ein; dies ergibt das gerenderte Zeichen „©“.

Weniger gebräuchlich ist ein verwandter Ansatz, der die Nutzung von **Codepunkten in Dezimal- statt Hexadezimaldarstellung** erlaubt. In diesem Fall beginnt die numerische Referenz mit „&#“ (ohne das „x“) und schließt wie gehabt mit „;“ ab. Dazwischen wird der Codepunkt in Dezimaldarstellung geschrieben. Im Fall unseres Beispiels ergibt sich die numerische Referenz „&#169;“ für das Copyright-Zeichen.

Tipp

Nutzen Sie den [Unicode Character Inspector](https://apps.timwhitlock.info/unicode/inspect?s=%C2%A9%0D%0A "Unicode character inspector"), um schnell die verschiedenen Codes zu einem Zeichen zu erfahren.

### Benannte Zeichen-Entitäten

Da die Schreibweise von Unicode-Zeichen als numerische Referenzen für den Menschen nicht intuitiv ist, gibt es noch eine weitere Methode. Es handelt sich dabei um die benannten Zeichen-Entitäten. Diese sind für häufig gebrauchte Zeichen definiert und **ordnen dem Zeichen einen kurzen, einprägsamen Namen zu**. Eine benannte Zeichen-Entität beginnt mit dem Und-Zeichen „&“ und endet mit einem Semikolon „;“. Dazwischen wird ohne Leerzeichen der definierte Name platziert. Um das Copyright-Zeichen „©“ in HTML einzufügen schreibt man einfach „©“.

Tipp

Die komplette Liste der definierten Zeichen-Entitäten ist im [HTML-Standard](https://html.spec.whatwg.org/multipage/named-characters.html#named-character-references "HTML Standard") hinterlegt.

### Programmiersprachen

Die meisten Programmiersprachen enthalten grundlegende Funktionen, **mit denen sich Zeichen und Codepunkte umwandeln lassen**. Die entsprechenden Funktionen heißen oft „ord(Zeichen)“ und „chr(Codepunkt)“. Dabei gilt:

'chr(ord(Zeichen)) == Zeichen'

Beachten Sie, dass es immer möglich ist, den **zu einem Zeichen korrespondierenden Codepunkt zu ermitteln**. Andersherum funktioniert die Zuordnung nur für Zahlen, welche tatsächlich als Codepunkte des Zeichencodes definiert sind. Wir zeigen hier das grundlegende Schema anhand eines kurzen Python-Beispiels:

```none
# Dezimal-Codepunkt eines Zeichens ermitteln
ord('A') # `65`
# Hexadezimal-Codepunkt eines Zeichens ermitteln
hex(ord('A')) # `0x41`
# zu Codepunkt gehöriges Zeichen ermitteln
chr(65) # `'A'`
chr(0x41) # `'A'`
chr(0x110001) # Fehler, da Codepunkt > `0x110000`
```

Mithilfe dieser Funktionen ist es problemlos möglich, eine **Zeichentabelle für Codepunkte des Unicode-Zeichensatzes zu erstellen**. Dazu iteriert man die Codepunkte und gibt die korrespondierenden Zeichen aus. Mit Python ist dies innerhalb weniger Codezeilen erledigt:

```none
# `range` bei `32` beginnen, da bei kleinerem Wert Steuerzeichen ausgegeben werden
# ASCII-Zeichensatz ausgeben
# for code_point in range(32, 128):
# ISO Latin-1 ausgeben
for code_point in range(32, 256):
# Code-Punkt in Dezimal- und Hexadezimaldarstellung samt zugehörigem Zeichen ausgeben
    print(code_point, hex(code_point), chr(code_point))
```

### Programm-Bibliothek ICU

Die internationalen Komponenten für Unicode (_„International Components for Unicode”_, ICU) sind in einer vom Unicode Consortium bereitgestellten Programm-Bibliothek zusammengefasst. Die Bibliothek wird unter einer Open-Source-Lizenz veröffentlicht und lässt sich auf vielen Betriebssystemen einsetzen. Die Software **dient der programmatischen Internationalisierung** (_„Internationalization“_, oft abgekürzt als _„i18n”_). Zu ihren Einsatzbereichen zählen unter anderem:

- Verarbeitung von Unicode-Texten
- Unterstützung regulärer Ausdrücke in Unicode
- Parsen und Formatierung von Kalenderdaten, Zeitangaben, Zahlen, Währungen und Nachrichten

Die ICU-Bibliothek liegt in **zwei Versionen** vor:

- „icu4c“ ist in C/C++ geschrieben und stellt ein API für diese Sprachen bereit.
- „icu4j“ ist in Java geschrieben und stellt für diese Sprache ein API bereit.

Der Einsatz der Komponenten liefert **konsistente Resultate** unabhängig von der zugrundeliegenden Plattform.

### Charset Meta-Angabe im Head von HTML-Dokumenten

Die meisten HTML-Dokumente liegen heutzutage in der Zeichencodierung UTF-8 vor. Um sicherzustellen, dass einem Seitenbesucher das **Dokument ohne fehlerhafte Zeichen angezeigt** wird, sollte eine _„Charset”-_Meta-Angabe im Head des HTML-Dokuments platziert werden. Diese weist den Browser an, das abgerufene Dokument als UTF-8 zu interpretieren und ist nachfolgend exemplarisch dargestellt:

```none
<head>
<meta charset="utf-8">
<!-- weitere Head-Elemente -->
</head>
```

### Twitter-Fonts

### Twitter-Fonts

Der beliebte Kurznachrichtendienst Twitter erlaubt keine Text-Formatierung für Tweets, Profile, oder Nutzernamen. Die kreativen Möglichkeiten der Nutzer sind damit beschränkt. Findige Entwickler haben jedoch mit einem Trick Abhilfe geschaffen: Twitter setzt durchgehend auf Unicode, und so ist es möglich, **aus speziellen Zeichen einen formatiert wirkenden Text zu komponieren**. Dabei kommen insbesondere Zeichen zum Einsatz, welche lateinischen Buchstaben ähneln. Am einfachsten lässt sich ein derartiger Text mit einem [Twitter Fonts Generator](https://lingojam.com/TwitterFonts "Twitter Fonts Generator – LingoJam") erzeugen.