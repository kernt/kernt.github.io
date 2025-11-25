---
tags:
  - konfigurationsmanagement
  - ansible
---
# Puppet für fortgeschrittene: Rollen Profile

Gut organisierte Puppet manifestes sind leicht zu lesen; Der Zweck eines Moduls sollte in seinem Namen offensichtlich sein auch ohne besondere Sprachkenntnisse also keine Slangs .
Der Zweck einer Node sollte in einer einzigen Klasse definiert werden.
Diese Einzelklasse sollte alle Klassen enthalten, die erforderlich sind, um diesen Zweck auszuführen zu können.
Craig Dunn schrieb einen Beitrag über ein solches Klassifikationssystem, das er ["Rollen und Profile" nannte](htt../puppet//www.craigdunn.o../puppet/20../puppet/../puppet/2../puppet/).
In diesem Modell sind Rollen der einzige Zweck einer Node, eine Node kann nur eine Rolle haben, eine Rolle kann mehr als ein Profil enthalten, und ein Profil enthält alle Ressourcen, die sich auf einen einzelnen Dienst beziehen. In diesem Beispiel erstellen wir eine Web-Server-Rolle, die mehrere Profile verwendet.

Hier nochmal das Schema

```s
 Rolle 1-N Profile N Ressourcen

```

## Wie es geht

Wir erstellen zwei Module, um unsere Rollen und Profile zu speichern. Rollen enthalten ein oder mehrere Profile. Jede Rolle oder jedes Profil wird als Unterklasse definiert, wie z.B. `profile::base`.

1.Entscheiden Sie sich für eine Namensstrategie für Ihre `roles` und `profiles`.
In unserem Beispiel erstellen wir zwei Module, Rollen und Profile, die unsere `roles` und `profiles` enthalten:

```s
puppet module generate thomas-profiles
ln -s thomas-profiles profiles
puppet module generate thomas-roles
ln -s thomas-roles roles
```

2.Beginnen Sie mit der Festlegung der Bestandteile unserer `webserver` rolle als Profil. Um dieses Beispiel einfach zu halten, erstellen wir zwei Profile. Zuerst ein `base` profil, um unsere grundlegenden Server-Konfigurationsklassen darin einzuschließen. Zweitens eine `apache` Klasse zum Installieren und Konfigurieren des Apache Webservers (`httpd`) wie folgt:

```s
$ vim profil../puppet/manifes../puppet/base.pp
class profiles::base {
  include base
}
$ vim profil../puppet/manifes../puppet/apache.pp
class profiles::apache {
  $apache = $::osfamily ? {
    'RedHat' => 'httpd',
    'Debian' => 'apache2',
    }
  service { "$apache":
    enable => true,
    ensure => true,
  }
  package { "$apache":
    ensure => 'installed',
  }
}
```

3.Definiere eine `rols::webserver` Klasse für unsere `webserver` Rolle wie folgt:

```s
$ vim rol../puppet/manifes../puppet/webserver.pp
class roles::webserver {
  include profiles::apache
  include profiles::base
}
```

4.Wenden Sie die `roles::webserver` Klasse auf einen Node an.
In einer zentralen Installation würden Sie entweder einen externen Node klassiefizierer ( External Node Classifier ENC) verwenden, um die Klasse auf den Node anzuwenden, oder Sie würden Hiera verwenden, um die Rolle zu definieren:

```ruby
 node 'webtest' {
    include roles::webserver
  }
```

## Wie es funktioniert

Wenn Sie die Teile der Web-Server-Konfiguration in verschiedene Profile zerlegen, können wir diese Teile selbstständig anwenden.
Wir haben ein Basisprofil erstellt, das wir erweitern können, um alle Ressourcen zu nutzen, die wir auf alle Nodes angewendet haben.
Unsere `roles::webserver` Klasse enthält einfach die Basis- und Apache-Klassen.

## Es gibt mehr

Wie wir im nächsten Abschnitt sehen werden, können wir Parameter an Klassen übergeben, um zu ändern, wie sie funktionieren.
In unserer `roles::webserver` Klasse können wir die Klasse Instanziierungssyntax anstelle von `include` verwenden und sie mit Parametern in den Klassen überschreiben.
Zum Beispiel, um einen Parameter an die Basisklasse zu übergeben, würden wir verwenden:

```ruby
  class {'profiles::base':
    parameter => 'newvalue'
  }
```

Wo wir früher verwendet haben:

`include profiles::base`

### Tip

In früheren Versionen dieses Buches wurden Knoten- und Klassenvererbung verwendet, um ein ähnliches Ziel zu erreichen, also zur Code-Wiederverwendung.
Die Nodevererbung ist in Puppet Version 3.7 und höher veraltet.
Node- und Klassenvererbung sollten vermieden werden.
Mit Rollen und Profilen erreicht man das gleiche Maß an Lesbarkeit und ist viel einfacher nachzuverfolgen.
