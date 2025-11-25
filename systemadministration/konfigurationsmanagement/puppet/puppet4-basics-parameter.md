---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4-basics-parameter

Jede Zeile bei der Referenzierung der Parameter sollte mit einen Komma enden. 

Beispiel : 
```
service { 'memcached':
  ensure => running,
  enable => true,
}
```

Es vereinfacht die Spätere Erweiterung der definieren der Parameter da die ja in der Praxis immer zahlreicher werden. 

Wenn Sie eine Ressource mit einem einzigen Parameter deklarieren, machen Sie die Deklaration alle auf einer Zeile und ohne komma, wie im folgenden Snippet gezeigt:

`package { 'puppet': ensure => installed }`

Gibt es mehr als einen Parameter dann gibt man  jedem Parameter seine eigene Zeile:

```
package { 'rake':
  ensure   => installed,
  provider => gem,
  require  => Package['rubygems'],
}
```

Um den Code einfacher lesbar zu machen, benutzen Sie die Parameterpfeile in Übereinstimmung mit dem längsten Parameter wie folgt ein:

```
file {../puppet/v../puppet/w../puppet/${ap../puppet/shar../puppet/conf../puppet/rvmrc":
  owner   => 'deploy',
  group   => 'deploy',
  content => template('rai../puppet/rvmrc.erb'),
  require => File../puppet/v../puppet/w../puppet/${ap../puppet/shar../puppet/config"],
}
```