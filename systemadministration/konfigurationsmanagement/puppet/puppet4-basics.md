---
tags:
  - puppet
  - konfigurationsmanagement
---
# Benutzen der Puppet community version

Orginal Qelle zum Thema [style_guide](htt../puppet//docs.puppetlabs.c../puppet/guid../puppet/style_guide.html)

Wie wird es nun umgesetzt...

In diesem Kapitel werde ich mit einigen wichtigen Beispielen wie dein Code style compliant wird.


# Vertiefung

Stell sicher, das du in deiner manifests Datei nur zwei Leerzeichen benutzt  (keine tabs), wie folgt:

```
service {'httpd':
  ensure  => running,
}
```

* [False](puppet4-basics-false.md)
* [Qouting](puppet4-basics-qouting.md)
* [Variablen](puppet4-basics-variablen.md)
* [Parameter](puppet4-basics-parameter.md)
* [Symlinks](puppet4-basics-symlinks.md)
* [manifests Erstellen](puppet4-basics-manitests.md)
* [Überprüfen deiner manifests mit Puppet-lint](puppet4-basics-lint.md)
* [Puppet4 Module erstellen](puppet4-basics-modules.md)
* [Puppet4 Standard zur Benennung einhalten](puppet4-standart-bezeichnung.md)
* [Puppet4 Einsetzen von Templates](puppet4-templates.md)
* [Puppet4 irritieren über mehrere Items](puppet4-basics-irritieren-multi-items.md)
* [Puppet4 IF anweisung einsetzen](puppet4-if.md)
* [Puppet4 Reguläre Ausdrücke in IF Anweisungen einsetzen](puppet4-if-regex.md)
* [Puppet4 Verwenden von Selektoren und Case-Anweisungen](puppet-selektoren-case.md)
* [Puppet4 den in Operator verwenden](puppet4-basics-in-operator.md)
* [Puppet4 Verwenden von regulären Ausdrucksersetzungen](puppet4-basics-regex-substitutions.md)
* [Puppet4 Arbeiten mit dem future Parser](puppet4-basics-future-parser.md)
