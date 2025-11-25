---
tags:
  - bash
  - konsole
  - date
  - system-administration
  - gnu-tools
---
# date Anzeige von Datum und Zeit

Die folgenden Parameter werden unterstützt:

```
%%    ein einfaches %
%n    das Zeilenende
%t    ein Tabulator
%H    Stunden 00 bis 23
%I    Stunden 00 bis 12
%M    Minuten 00 bis 59
%p    AM oder PM
%r    die Zeit, 12 Stunden (hh:mm:ss AM/PM)
%S    Sekunden 00 bis 61
%T    die Zeit, 24 Stunden (hh:mm:ss)
%X    die Zeit, 24 Stunden (%H:%M:%S)
%Z    die Zeitzone; oder nichts, wenn keine Zeitzone feststellbar ist
%a    der abgekürzte Wochentag
%A    der ausgeschriebene Wochentag
%b    der abgekürzte Monat
%B    der ausgeschriebene Monat
%c    das Datum und die Zeit (Standardausgabeformat)
%d    der Tag im Monat 00 bis 31
%D    das Datum (mm/dd/yy)
%h    das gleiche wie %b
%j    der Tag im Jahr
%m    der Monat 00 bis 12
%U    die Nummer der Woche, Sonntag erster Tag
%w    der Tag in der Woche 0 bis 6
%W    die Nummer der Woche, Montag erster Tag
%x    das Datum (mm/dd/yy)
%y    die letzten beiden Stellen der Jahreszahl
%Y    die ganze Jahreszahl

```
Wenn dem date Kommando ein Argument übergeben wird, das nicht mit einem + Zeichen beginnt, wird die Systemzeit diesem Argument entsprechend gesetzt. Das Argument muß vollständig aus Ziffern bestehen. Die einzelnen Stellen der Zahl haben folgende Bedeutung:

```
MM    Monat
DD    Tag im Monat
hh    Stunde
mm    Minute
CC    die ersten beiden Stellen der Jahreszahl (optional)
YY    die letzten beiden Stellen der Jahreszahl (optional)
ss    die Sekunden (optional)
```

Die Form, die das date Kommando erwartet ist genau MMDDhhmmCCYY.ss
Dabei dürfen die optionalen Angaben CC, YY, ss weggelassen werden. date erkennt an dem Punkt zwischen Jahreszahl und Sekunden, ob es sich um Jahr oder Sekunden handelt.

Nur der Superuser (root) darf die Systemzeit verändern.

Durch das Ändern der Systemzeit wird nicht die CMos-Uhr gestellt. Nur die aktuelle Systemzeit ist betroffen. Um die CMos-Uhr zu stellen wird zunächst die Systemzeit gestellt und dann mit dem Befehl clock -s die Systemzeit in die CMos-Uhr eingetragen. Auch das kann nur der Systemverwalter durchführen.

Optionen:
-s Datum
(set) setzt die Zeit auf das Datum; das Datum kann Monatsnamen, Zeitzonen und ähnliches enthalten. 
Das oben dargestellte Format funktioniert hier nicht.
-u
zeigt (oder setzt) die Zeit als Greenwich Main Time

2013-02-07

`date +%F`

02/07/13

`date +%D`