---
tags:
  - konsole
  - bash
  - cp
  - gnu-tools
  - system-administration
---
# cp Kopie von Dateien oder Verzeichnissen

**Dateien/Ordner ausschließen**

`shopt -s extglob`

und

`cp -av /QUELLE/!(Datei1.txt|Festplattenabbild*|Ordner/Unterordner1) /ZIEL/`
