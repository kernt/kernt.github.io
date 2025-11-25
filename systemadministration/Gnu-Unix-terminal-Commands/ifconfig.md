---
tags:
  - gnu-tools
  - system-administration
---


**Den Internen IP-Traffic auflisten:**

```sh
|   |
|---|
|`INTERNALIPS=$(``ifconfig` `\|` `grep` `-i` `"inet addr"` `\|` `cut` `-d` `":"` `-f2 \|` `awk` `'{ print $1 }'``);``for` `entry` `in` `$INTERNALIPS;` `do` `for` `eintrag` `in` `$INTERNALIPS;` `do` `netstat` `-tapen \|` `awk` `'{ print $4 "  " $5 "  " $6}'` `\|` `grep` `-i $entry \|` `grep` `-i $eintrag;` `done``;` `done`|
```

