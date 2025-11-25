---
tags:
  - konsole
  - bash
  - grep
  - gnu-tools
  - system-administration
---
# Grep examples

`egrep -vi "suche"`

`egrep -i "suche"`

**most usible in Unix/Linux**

`grep -Ev '^(#|$)'`

**sudoers . check rights**

`grep -E '/usr/bin/docker (ps|inspect|exec) \*'`

**new-line-separator-for-each-grep-result-sh-script**

`grep "pattern" /path/to/file | awk '{print $0,"\n"}'`

**Eintrag suchen**

`grep -Ri SSLPassPhraseDialog /etc/httpd/`

**User ids suchen**

`getent passwd | grep -P ":x:(1005|1006)`

**Lines ausgeben**

`grep -ir "text" dir_name`

**Nur Datein zeigen die einen bestimmten text binhalten**

