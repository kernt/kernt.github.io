---
tags:
  - bash
  - konsole
  - vim
  - system-administration
---
# VIM  

**Starten**

|Komando | Beschreibung |
| :---: | :---: |
|vi |Aufruf von vi mit leerem Text-Puffer.|
|vi -|Liest den Inhalt der Standardausgabe ein. Beispiel: ls | vi -|
|vi Dateiname | Datei wird geladen und der Cursor bei der ersten Zeile platziert.|
|vi + Dateiname | Datei wird geladen und der Cursor bei der letzten Zeile platziert.|
|vi +n Dateiname | Dateiname Datei wird geladen und der Cursor bei der n-ten Zeile platziert.|
|vi +/Zeichenkette Dateiname | Datei wird geladen, der Cursor bei der Zeile mit Zeichenkette plaziert.|

**Beenden**

|Komando | Beschreibung |
| :---: | :---: |
|:wq |Speichern und vi verlassen.|
|ZZ |Ebenfalls speichern und vi verlassen.|
|:q |vi verlassen, falls Datei unverändert.|
|:q! |vi verlassen, egal ob Datei verändert oder nicht.|
|:w |Datei speichern.|

**Dateien laden**

|Komando | Beschreibung |
| :---: | :---: |
|e Datei |Datei wird geladen, wenn sie existiert, ansonsten erzeugt.|
|:next | Die nächste Datei wird geladen, falls vi mit mehreren Dateien aufgerufen wurde.|
|:prev |Die vorherige Datei wird geladen, falls vi mit mehreren Dateien aufgerufen wurde.|

**Cursorbewegungen**

|Komando | Beschreibung |
| :---: | :---: |
|^|Springe zum Anfang der aktuellen Zeile.|
|$|Springe zum Ende der aktuellen Zeile.|
|g|Ans Ende der Datei Springen|

**Text eingeben**

|Komando | Beschreibung |
| :---: | :---: |
|i|(insert), Eingabe vor dem aktuellen Zeichen.|
|a|(append), Eingabe nach dem aktuellen Zeichen.|
|I|(Insert), Eingabe am Anfang der aktuellen Zeile.|
|A|(Append), Eingabe am Ende der aktuellen Zeile.|
|o|neue Zeile und Eingabe nach der aktuellen Zeile.|
|O|neue Zeile und Eingabe vor der aktuellen Zeile.|
|Ctrl-v|Eingabe eines Steuerzeichens.|

**Text ändern**

|Komando | Beschreibung |
| :---: | :---: |
|[Count]rZeichen|(replace), Änderung des aktuellen Buchstaben in Zeichen.|
|R|(Replace), Überschreibmodus vom aktuellen Buchstaben aus.|
|cwWort|ersetzt den Text ab der Cursorposition bis zum Wortende durch Wort.|
|ccZeichenkette|ersetzt die aktuelle Zeile durch Zeichenkette.|
|J|hängt die der aktuellen folgende Zeile an die aktuelle an.|

**Text löschen**

|Komando | Beschreibung |
| :---: | :---: |
|[Count]x|1 (bzw. Count) Zeichen unter dem Cursor (rechts) wird gelöscht.|
|[Count]x|1 (bzw. Count) Zeichen links vom dem Cursor wird gelöscht.|
|D|löscht von der Cursorposition bis zum Zeilenende.|
|[Count]dd|1 (bzw. Count) Zeilen werden gelöscht.|
|[Count]d[Richtung]|1 (bzw. Count) mal wird in Richtung [Richtung] gelöscht.|
|d$|Löschen bis zum zeilen Ende|

**Die Zwischenablagen**

|Komando | Beschreibung |
| :---: | :---: |
|"1..0, a..z|Die Ablage 1..0 bzw. a..z für die nächste Aktion auswählen.|
|[Count]y[Richtung]|1 (bzw. Count) Bewegungen in [Richtung].|
|[Count]yy|1 (bzw. Count) Zeilen werden in die aktuelle Zwischenablage kopiert. |
|Beliebige Löschaktion|Gelöschter Text wird in die aktuelle Zwischenablage kopiert.|
|P|Inhalt der Zwischenablage wird hinter dem Cursor eingefügt.|
|p|Inhalt der Zwischenablage wird vor dem Cursor eingefügt.|

**Suchen und Ersetzen**

|Komando | Beschreibung |
| :---: | :---: |
|/Regex|Suche vorwärts nach dem regulären Ausdruck Regex.|
|?Regex|Suche vorwärts nach dem regulären Ausdruck Regex.|
|n|Wiederholt das letzte Suchkommando.|
|N|Wiederholt das letzte Suchkommando in die jeweils andere Richtung.|
|fZeichen|Sucht nach Zeichen in der aktuellen Zeile vorwärts.|
|FZeichen|Sucht nach Zeichen in der aktuellen Zeile rückwärts.|
|:%s/Quelle/Ziel/|Ersetzt Quelle textweit beim 1. Vorkommen in der Zeile durch Ziel.|
|:%s/Quelle/Ziel/g|Ersetzt Quelle im Text überall durch Ziel.|
|:%s/Quelle/Ziel/gc|Ersetzt Quelle im Text überall durchZiel.|
|yy|zeile Kopiren|

**Sonstige Goodies**

| Komando | Beschreibung |
| :-----: | :----------: |
|    -    |              |
|    %    |              |
|  :u u   |              |
|         |              |

multiple windows

If you want, you can probably do everything from one vim session! :) Here are some commands to turn one vim session (inside one xterm) into multiple windows.

```sh
 :e filename      - edit another file
 :split filename  - split window and load another file
 ctrl-w up arrow  - move cursor up a window
 ctrl-w ctrl-w    - move cursor to another window (cycle)
 ctrl-w_          - maximize current window
 ctrl-w=          - make all equal size
 10 ctrl-w+       - increase window size by 10 lines
 :vsplit file     - vertical split
 :sview file      - same as split, but readonly
 :hide            - close current window
 :only            - keep only this window open
 :ls              - show current buffers
 :b 2             - open buffer #2 in this window
```

* [vim windows](https://www.cs.oberlin.edu/~kuperman/help/vim/windows.html)

`vim - danach :w Dateiname`
### Quellen

* [Vim_Shortcuts](https://shortcutworld.com/Vim/linux/Vim_Shortcuts)
* [Disable automatic comment insertion](https://vim.fandom.com/wiki/Disable_automatic_comment_insertion)
