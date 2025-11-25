Während das Dezimalsystem mit seinen zehn Ziffern ein fester Bestandteil unseres normalen Alltags ist, verlassen sich die Informatik und Datenverarbeitung auf das **Binärsystem** oder den **[Binärcode](https://www.ionos.at/digitalguide/websites/web-entwicklung/binaercode/ "Binärcode")**. Dies ermöglicht die Darstellung komplexer Sachverhalte mit nur zwei Zuständen: 0 und 1. Große Binärzahlen haben jedoch den Nachteil, dass sie schnell unübersichtlich werden. Und hier schafft das **Hexadezimalsystem** Abhilfe: Denn Informationen, die im Binärsystem 8 Stellen bräuchten, können mit nur 2 Hexadezimalzahlen ausgedrückt werden.
## Was ist das Hexadezimalsystem?

Das Wort **Hexadezimal** setzt sich aus den Begriffen _hexa_ und _decem_ zusammen. _hexa_ kommt aus dem Griechischen und bedeutet „sechs“_,_ während _decem_ das lateinische Wort für „zehn“ ist. Das Hexadezimalsystem ist also ein Stellenwertsystem, das Zahlen zur Basis 16 darstellt. Das heißt, dass das hexadezimale Zahlensystem 16 verschiedene Ziffern verwendet. Mit anderen Worten: Es gibt 16 mögliche Ziffernsymbole – im Gegensatz zu zwei im Binärsystem (1 und 0) oder zehn im Dezimalsystem (0 bis 9). Aber welchen Zweck hat dieses System in der Praxis?
## Wozu dient das Hexadezimalsystem?

Das Hexadezimalsystem kommt in der Computertechnik zum Einsatz und erleichtert die Lesbarkeit von großen Zahlen bzw. **langen Bitfolgen**. Diese werden zu je vier Bit gruppiert und in Hexadezimalzahlen umgerechnet. Das Ergebnis: Aus einer langen Folge von Einsen und Nullen werden kürzere hexadezimale Zahlen, die sich wiederum in Zweier- oder Vierergruppen unterteilen lassen. So sind Hexadezimalzahlen eine **kompaktere Form** der Darstellung von Bitfolgen. Gebraucht wird das System u. a. in der Quell- und Zieladresse von [Internet Protocols (IPs)](https://www.ionos.at/digitalguide/server/knowhow/was-ist-das-internet-protocol-definition-von-ip-co/ "Was ist das Internet Protocol? – Definition von IP & Co."), in [ASCII-Codes](https://www.ionos.at/digitalguide/server/knowhow/ascii-american-standard-code-for-information-interchange/ "ASCII-American-Standard-Code-for-Information-Interchange") oder der Beschreibung von Farbcodes beim Webdesign mit der [Stylesheet-Sprache CSS](https://www.ionos.at/digitalguide/websites/webdesign/css-lernen-leicht-gemacht/ "CSS-Lernen leicht gemacht").

Tipp

## Hexadezimalsystem: Die Schreibweise

Wie bereits erwähnt, stehen Ihnen im Hexadezimalsystem 16 mögliche Ziffernsymbole zur Verfügung. Allerdings ergibt sich dadurch ein potenzielles Problem: Mit der herkömmlichen Ziffernschreibweise werden die Dezimalzahlen 10, 11, 12, 13, 14 und 15 genutzt, die jeweils aus zwei zusammenhängenden Symbolen bestehen. Wenn Sie also die Zahl 10 in hexadezimaler Schreibweise ausdrücken, dann ist unklar, ob Sie die Dezimalzahl 10 meinen oder etwa die Binärzahl 2 (1 + 0).

Um dieses Problem zu umgehen, werden hexadezimale Zahlen, die die Werte von 10 bis 15 bezeichnen, durch die Großbuchstaben A, B, C, D, E und F ersetzt. Im Hexadezimalsystem werden also die Zahlen von **0 bis 9** und die Großbuchstaben **A bis F** verwendet, um das binäre oder dezimale Zahlenäquivalent darzustellen. Damit Sie Hexadezimalzahlen von dezimalen Zahlen unterscheiden können, stehen Ihnen mehrere Schreibweisen zur Verfügung (in den Beispielen unten wird die Hexadezimalzahl „73“ beschrieben):

- 7316
- 73hex
- 73h
- 73H
- 73H
- 0x73
- $73
- #73
- "73
- X'73'

Das Präfix _0x_ und das Suffix _h_ kommt insbesondere in der Programmierung zum Einsatz, während das Dollar-Präfix bei bestimmten Prozessorfamilien in der Assemblersprache verwendet wird.

## Das Verhältnis von Hexadezimalzahlen und Binärzahlen

Werden komplexe Zustände beschrieben, können Bitfolgen bzw. binäre Zeichenketten sehr lang werden. Im alltäglichen Gebrauch des Dezimalzahlensystems verwenden wir hier Gruppen von drei Ziffern, um eine sehr **große Zahl** wie eine Million oder eine Billion lesbarer zu machen. Dasselbe gilt auch für digitale Systeme: Um eine Bitfolge wie 11110101110011112 besser lesen zu können, wird sie üblicherweise in gleichmäßigen Gruppen von vier aufgeteilt. Unser Beispiel würde dann so aussehen: 1111 0101 1100 11112. Noch lesbarer wird es, wenn Sie die Binärziffern in Hexadezimalzahlen umwandeln.

Da 16 im Dezimalsystem die vierte Potenz von 2 (bzw. 24) ist, besteht eine direkte Beziehung zwischen den Zahlen 2 und 16, sodass eine Hexadezimalziffer einen Wert hat, der **4 Binärziffern** entspricht. Aufgrund dieser Beziehung können Sie **4 Ziffern** einer binären Zahl mit **einer einzigen Hexadezimalziffer** darstellen. Dies macht die Konvertierung zwischen Binär- und Hexadezimalzahlen relativ einfach, sodass sich große Binärzahlen dank des Hexadezimalsystems mit weniger Ziffern schreiben lassen.
## Hexadezimal-Tabelle zur Umwandlung in Dezimal- und Binärzahlen

Hexadezimalzahlen gehören einem komplexeren System an als das reine Binär- oder Dezimalsystem und werden oft im Zusammenhang mit Speicheradressen verwendet. Durch die Unterteilung einer binären Zahl in Gruppen von **4 Bits** kann jeder Bit-Satz von 4 Ziffern einen Wert zwischen „0000“ (0) und „1111“ (8+4+2+1 = 15) annehmen. Dies ergibt insgesamt 16 verschiedene Zahlenkombinationen von 0 bis 15. Hier ist zu beachten, dass auch „0“ auch eine gültige Ziffer ist.

|Dezimalzahl|4 Bit-Binärzahl|Hexadezimalzahl|
|---|---|---|
|0|0000|0|
|1|0001|1|
|2|0010|2|
|3|0011|3|
|4|0100|4|
|5|0101|5|
|6|0110|6|
|7|0111|7|
|8|1000|8|
|9|1001|9|
|10|1010|A|
|11|1011|B|
|12|1100|C|
|13|1101|D|
|14|1110|E|
|15|1111|F|
|16|0001 0000|10 (1+0)|
|17|0001 0001|11 (1+1)|
|18|0001 0010|12 (1+2)|
|19|0001 0011|13 (1+3)|
|20|0001 0100|14 (1+4)|

Der Umrechnungstabelle nach sieht unsere vorherige binäre Zahlenreihe 1111 0101 1100 11112 im Hexadezimalsystem wie folgt aus: **F5CF** – diese Zahl ist nun einfacher zu lesen als die lange Bitfolge. Durch die Verwendung der hexadezimalen Notation wird ein digitaler Code also mit weniger Ziffern und einer deutlich **geringeren Fehlerwahrscheinlichkeit** geschrieben. Ebenso ist die Konvertierung von Hexadezimalzahlen zurück in Binärzahlen lediglich der umgekehrte Vorgang.

Um unsere Zahl von eben nun eindeutig als Hexadezimalzahl zu kennzeichnen, können Sie F5CF in der Form von **F5CF16**, **$F5CF** oder **#F5CF** angeben. Letztere Schreibweise, auch Hashwert genannt, kommt in der digitalen Farbkodierung zum Einsatz, denn Designer und Entwickler verwenden HEX-Farben im Webdesign. Eine HEX-Farbe wird als sechsstellige Kombination aus Zahlen und Buchstaben ausgedrückt, die durch ihre Mischung aus Rot, Grün und Blau ([RGB](https://www.ionos.at/digitalguide/websites/webdesign/rgb-farben/ "RGB-Farben")) definiert ist. #000000 beispielsweise steht für die Farbe Schwarz und #FFFFFF für die Farbe Weiß.

## Zählen mit Hexadezimalzahlen

Nun wissen Sie, wie Sie vier binäre Ziffern in eine Hexadezimalzahl konvertieren. Falls Sie mehr als vier binäre Ziffern haben, dann beginnen Sie einfach **von vorne** bzw. fahren mit dem nächsten Satz von 4 Bits fort. Mit zwei Hexadezimalzahlen können Sie bis FF zählen, was dem dezimalen Wert 255 entspricht.

Das Hinzufügen zusätzlicher Hexadezimalziffern, um Binärzahlen in eine Hexadezimalzahl zu konvertieren, ist sehr einfach, wenn Ihnen 4, 8, 12 oder 16 binäre Ziffern vorliegen. Sie können aber auch „0“ oder „00“ links vom höchstwertigen Bit **hinzufügen**, wenn die binäre Bit-Anzahl kein Vielfaches von vier ist. Zum Beispiel ist 1100101101100112 eine 14 Bit lange Binärzahl, die zu groß für nur drei Hexadezimalziffern, aber zu klein für eine vierstellige Hexadezimalzahl ist.

Die Lösung besteht darin, zusätzliche Nullen an das am weitesten links stehende Bit zu addieren, bis Sie einen vollständigen Satz von **4-Bit-Binärzahlen** haben. In unserem Beispiel würde die Reihe von oben nun so aussehen: **00**1100101101100112.

## Fazit

Der große Vorteil des Hexadezimalsystems ist die Kompaktheit seiner Zahlen, da durch die Basis 16 weniger Ziffern zur Darstellung einer bestimmten Zahl benötigt werden als im Binär- oder Dezimalformat. Zudem geht es relativ **schnell und unkompliziert**, Hexadezimalzahlen in Binärzahlen umzuwandeln und umgekehrt.