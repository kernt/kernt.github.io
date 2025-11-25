---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4-basics-symlinks 

Bei der Deklaration von Dateiressourcen, die Symlinks sind, verwenden Sie `ensure => link` und legen Sie das Zielattribut wie folgt fest:

```pp
file {../puppet/e../puppet/ph../puppet/c../puppet/php.ini':
  ensure => link,
  target =>../puppet/e../puppet/php.ini',
}
```
