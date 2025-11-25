# ASCII – Erklärung und Beispiele

Der ASCII-Code ist eine Zeichenkodierung, die die Darstellung von Zeichen durch elektronische Geräte wie beispielsweise PCs festlegt. Hierzu werden die einzelnen Zeichen in Binär-, Dezimal- und Hexadezimalwerte konvertiert, die der Computer verarbeiten kann.

## Was ist ASCII?

ASCII ist ein Standard zur Darstellung von Zeichen durch elektronische Geräte. Um zu verstehen, was das heißt, muss man sich darüber im Klaren sein, wie ein Rechner überhaupt funktioniert: Bei einem Computer basieren die Rechenprozesse immer auf dem **binären System**. Das heißt: Einsen und Nullen bestimmen die Vorgänge eines Computers. Deshalb ist auch ASCII auf diesem System aufgebaut. Der ursprüngliche ASCII-Standard definiert innerhalb von sieben [Bits](https://www.ionos.at/digitalguide/websites/web-entwicklung/was-ist-ein-bit/ "Was ist ein Bit") – also sieben Stellen, die entweder eine 0 oder 1 zeigen – unterschiedliche Zeichen.

Das achte Bit, das zu einem vollen Byte gehört, wird traditionell für Prüfzwecke verwendet. Die auf ASCII basierenden, erweiterten Versionen nutzen genau dieses Bit, um die verfügbaren Zeichen auf 256 (28) zu erweitern.

> Fakt: Ursprünglich sollte das achte Bit die Überprüfung der Daten auf Fehler ermöglichen. Das sogenannte Paritätsbit erlaubt dem Empfänger der Bitfolge, Ungereimtheiten zu erkennen. Dabei ist allerdings nur ersichtlich, dass ein Fehler aufgetreten ist, jedoch nicht, wo genau die Ursache liegt. Deshalb eignet sich die Paritätsprüfung kaum zur Korrektur von Fehlern.

So entspricht jedem Zeichen eine siebenstellige Folge von Nullen und Einsen, die man als Dezimalzahl oder als [hexadezimale Zahl](https://www.ionos.at/digitalguide/server/knowhow/hexadezimalsystem/ "Hexadezimalsystem") darstellen kann. Die ASCII-Zeichen lassen sich in mehrere Gruppen aufteilen:

- **Steuerzeichen (_0–31 & 127_)**: Die Steuerzeichen sind nicht druckbare Zeichen. Sie dienen dazu, Kommandos an den PC oder den Drucker zu übermitteln und beruhen auf Techniken von Fernschreibern. Mit diesen Zeichen werden beispielsweise Zeilenumbrüche oder Tabulatoren gesetzt. Viele dieser Zeichen sind heute kaum noch in Gebrauch.
- **Sonderzeichen (_32–47 / 58–64 / 91–96 / 123–126_)**: Sonderzeichen umfassen alle druckbaren Zeichen, die weder Buchstaben noch Ziffern sind, wie z. B. Satzzeichen oder technisch-mathematische Zeichen. ASCII zählt dazu auch das Leerzeichen, das dort als **nicht sichtbares, aber druckbares Zeichen** gilt – und das somit nicht zu den Steuerzeichen gehört, wie man vermuten könnte.
- **Ziffern _(30–39)_**: Die Ziffern umfassen die zehn arabischen Ziffern von Null bis Neun.
- **Buchstaben _(65–90 / 97–122)_**: Die Buchstaben sind in zwei Blöcke unterteilt, wobei die erste Gruppe die Groß- und die zweite die Kleinbuchstaben enthält.

> Tipp: Um verschiedene Zeichen mühelos in ASCII-Code zu konvertieren, lohnt sich ein Blick auf die [ASCII-Tabelle](https://www.ionos.at/digitalguide/server/knowhow/ascii-tabelle/ "ASCII Tabelle"), die die binären, dezimalen und hexadezimalen Werte für die einzelnen Zeichen enthält.

## Beispiel: ASCII-Zeichen

Bei ASCII konvertiert das System [Binärcode](https://www.ionos.at/digitalguide/websites/web-entwicklung/binaercode/ "Binärcode") gemäß einem festgelegten Standard in druckbare und nicht druckbare Zeichen. Wirft man einen Blick auf die ASCII-Tabelle, lassen sich für verschiedene numerische Werte die durch sie dargestellten Zeichen finden.

Beispiel:

Die Binärzahl 01000001 kann dezimal als 65, hexadezimal als 41 geschrieben werden. Das Zeichen, das mit dieser Zahl kodiert wird, ist ein „A“. Zählen Sie nun weiter nach unten, finden Sie die Großbuchstaben in alphabetischer Reihenfolge aufgelistet. Das Beispielwort „ASCII“ entspräche also folgenden Zahlenwerten:

|   |   |   |   |   |   |
|---|---|---|---|---|---|
||A|S|C|I|I|
|binär|01000001|01010011|01000011|01001001|01001001|
|dezimal|65|83|67|73|73|
|hexadezimal|41|53|43|49|49|
Tipp

Unter Windows können Sie Unicode-Zeichen – und somit auch ASCII-Zeichen – per Tastenkombination eingeben. Halten Sie dafür die Alt-Taste gedrückt und geben Sie über den Nummernblock der Tastatur den Dezimalwert des Zeichens ein.
## ASCII-Code: Nutzen und Anwendungsgebiete

Auch heute noch findet ASCII vielfältige Verwendung, wenn auch [UTF-8](https://www.ionos.at/digitalguide/websites/webseiten-erstellen/utf-8-codierung-globaler-digitaler-kommunikation/ "UTF-8: Codierung globaler digitaler Kommunikation") inzwischen bei der Darstellung von Text wichtiger geworden sein dürfte. Doch erst ab ca. 2008 hat der [Unicode](https://www.ionos.at/digitalguide/websites/webseiten-erstellen/unicode/ "Unicode") die ältere Zeichenkodierung beim Einsatz im World Wide Web von Platz eins verdrängt. Der Vorteil von UTF-8 ist, dass der Code quasi abwärtskompatibel ist: ASCII ist eine Teilmenge von UTF-8 und so sind die ersten 128 Zeichen identisch. Da ASCII als kleinster gemeinsamer Nenner der meisten neueren Kodierungsformen gesehen werden kann, findet die alte Kodierung immer noch Verwendung in E-Mails und URLs.

> Fakt: Anwender können Unicode inzwischen auch bei der Erstellung von E-Mails verwenden, und sogar Domains lassen sich heutzutage dank [Internationalized Domain Names](https://www.ionos.at/digitalguide/domains/domainverwaltung/was-ist-ein-internationalisierter-domain-name-idn/ "Was ist ein internationalisierter Domain-Name (IDN)?") mit Umlauten versehen. Vor der Übertragung muss der Text allerdings in beiden Fällen in das ASCII-Format konvertiert werden. Dies geschieht automatisiert, sodass Nutzer davon nichts mitbekommen.

Darüber hinaus wird ASCII seit Langem für einen weniger technischen als vielmehr künstlerischen Zweck gebraucht: Als **ASCII-Art** bezeichnet man Kunst, die ausschließlich druckbare Zeichen der ASCII-Tabelle verwendet, um Bilder zu erzeugen. Die Bandbreite reicht dabei von Schriftzügen über einfache Strichfiguren bis hin zu richtigen Gemälden. Die ASCII-Künstler nutzen dafür die unterschiedlichen Helligkeiten der einzelnen Zeichen aus. So lassen sich sogar Schattierungen darstellen.
## Kurze Geschichte des ASCII-Codes

Die American Standards Association (ASA, inzwischen als ANSI für American National Standards Institute“ bekannt) hat den American Standard Code for Information Interchange (ASCII) bereits im Jahre 1963 genehmigt. Damit hat sie für verbindliche Vorgaben gesorgt, wie elektronische Geräte Zeichen abbilden sollen. Da es sich um einen **rein US-amerikanischen Standard** handelt, wird auch häufig von US-ASCII gesprochen.

Als **Vorgänger** kann man u. a. den **Morse-Code** ansehen – oder auch Kodierungen, die man bei Fernschreiben einsetzt: Ein standardisierter Code (z. B. eine festgelegte Abfolge akustischer Signale) wird dabei in Text übersetzt. Da auch Computer nicht mit unserem Alphabet umgehen können – schließlich basieren ihre internen Vorgänge auf dem binären System – hat man ASCII eingeführt.

Bis heute hat man den Zeichenstandard nur wenige Male verändert, um ihn an neue Anforderungen anzupassen. So existieren **erweiterte Versionen**, die ein achtes Bit verwenden, damit auch nationale Eigenheiten – wie etwa die deutschen Umlaute (ä, ö und ü) – dargestellt werden können. Das in Deutschland immer noch beliebte Latin-1 (ISO 88591-1) beruht auf dem ASCII-Code.

Ein Wechsel zwischen dem lateinischen Alphabet und beispielsweise arabischen Schriftzeichen ist allerdings ausgeschlossen. Deshalb haben sich inzwischen weitestgehend auf **Unicode basierende Zeichensätze wie UTF-8** durchgesetzt: Unicode bietet Platz für mehr als eine Million verschiedener Zeichen. UTF-8 ist darüber hinaus kompatibel mit ASCII, kodiert also die ersten 128 Zeichen ebenso wie dieser.
