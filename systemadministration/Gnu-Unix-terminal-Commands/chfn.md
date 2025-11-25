---
tags:
  - bash
  - konsole
  - chfn
  - gnu-tools
  - system-administration
---
# chfn erweiterte Benutzerinformationen anpassen

Der Befehl chfn ändert den vollen Namen, die Büronummer sowie die Telefonnummern für ein Benutzerkonto.

```
-u, --help     zeigt die Hilfe an und beendet das Programm
-f, --full-name     Vollständigen Benutzernamen ändern
-r, --room     Zimmernummer ändern
-w, --work-phone     dienstliche Telefonnummer ändern
-h, --home-phone     private Telefonnummer ändern
-R, --root     führt die Veränderungen in dem Verzeichnis CHROOT_VERZ durch und verwendet die Konfigurationsdateien aus dem Verzeichnis CHROOT_VERZ
```

Der Benutzer marco hat geheiratet und hat jetzt einen anderen Nachnamen bekommen. Er heißt nicht mehr Marco Beispiel sondern Marco Mustermann:

`sudo chfn -f "Marco Mustermann" marco`

**Die Benutzerin sandra hat das Bürozimmer gewechselt, weil sie befördert wurde:**

`sudo chfn -r ZIMMERNUMMER sandra`

**Die Benutzerin emma hat ein neues Firmenhandy mit neuer Telefonnummer bekommen:**

`sudo chfn -w DIENST_TELEFONNUMMER emma`

**Der Benutzer ben hat einen neuen Telefonanbieter und hat eine neue private Telefonnummer:**

`sudo chfn -h PRIVATE_TELEFONNUMMER ben`
