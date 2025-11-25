---
tags:
  - konsole
  - bash
  - passwd
  - system-administration
  - gnu-tools
---
# passwd Änderung des Passworts eines Benutzers

Infos zu allen Benutzen

`passwd -S $USER`

> Info:**L** = gesperrtes Passwort **NP** = kein Passwort **P** =  gültiges Passwort

**Nur abgelaufene tokensanpassen**

`passswd -k`

**Pw Änderung via stin**

`passwd --stin | echo $PW`

