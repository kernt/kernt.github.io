---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschrittene: Arrays und Ressourcen

Alles, was eine Ressource tun können , können Sie mit auch einen array tun , um eine Reihe von Ressourcen darzustellen.
Benutze Sie diese  Idee, um ihre Manifestes umzugestalten, um sie kürzer und klarer zu machen.

## Wie es geht

Hier sind die Schritte zum Refactor mit Arrays von Ressourcen:

1.Identifizieren Sie eine Klasse in Ihrem Manifest, wo Sie mehrere Instanzen der gleichen Art von Ressource haben, zum Beispiel Pakete:

```ruby
  package { 'sudo' : ensure => installed }
  package { 'unzip' : ensure => installed }
  package { 'locate' : ensure => installed }
  package { 'lsof' : ensure => installed }
  package { 'cron' : ensure => installed }
  package { 'rubygems' : ensure => installed }
```

2.Gruppieren Sie sie zusammen und ersetzen Sie sie mit einer einzigen Paketressource mit einem Array:

```ruby
 package
  {
    [ 'cron',
    'locate',
    'lsof',
    'rubygems',
    'sudo',
    'unzip' ]:
    ensure => installed,
  }
```

## Wie es funktioniert…

Die meisten Puppet-Ressourcentypen können ein Array anstelle eines einzelnen Namens annehmen und eine Instanz für jedes der Elemente im Array erstellen.
Alle Parameter, die Sie für die Ressource bereitstellen (z. B. `ensure => installed`), werden jedem der neuen Ressourceninstanzen zugeordnet.
Diese Kurze variante wird nur funktionieren, wenn alle Ressourcen die gleichen Attribute haben.

## Siehe auch

Das Iterating über mehrere Artikel Rezept in[Puppet4 Sprache und Syntax](puppet4-basics.md)