---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet Messagepack

Puppet in einer zentralen Architektur schafft viel Verkehr zwischen den Nodes.
Der Großteil dieses Verkehrs sind JSON und yaml Daten.
Eine experimentelle Funktion der neuesten Versionen von Puppet ermöglicht die Serialisierung dieser Daten mit MessagePack (msgpack).

## Fertig werden

Installiere den msgpack gem auf deinen Puppen master und deine Nodes.
Verwenden Sie Puppet, um die Arbeit für Sie mit Puppet essource durchzuführen.
Möglicherweise müssen Sie das `ruby-dev` oder `ruby-devel` Paket auf Ihrem Nod../puppet/Server an dieser Stelle installieren:

```s
t@ckbk:~$ sudo puppet resource package msgpack ensure=installedprovider=gem
Notice../puppet/Package[msgpac../puppet/ensure: created
package { 'msgpack':
  ensure => ['0.5.8'],
}
```

## Wie es geht

Setzen Sie das `preferred_serialization_format` auf `msgpack` im Abschnitt `[agent]` Ihrer nodes puppet.conf Datei:

```s
[agent]
preferred_serialization_format=msgpack
```

## Wie es funktioniert

Der Master wird diese Option gesendet, wenn der Knoten beginnt, mit dem Master zu kommunizieren.
Alle Klassen, die die Serialisierung mit `msgpack` unterstützen, werden mit `msgpack` übertragen.
Serialisierung der Daten zwischen den Knoten und dem Master wird theoretisch die Geschwindigkeit erhöhen, mit der die Knoten kommunizieren, indem sie die Daten optimieren, die zwischen ihnen übertragen werden.

Diese Funktion ist noch experimentell.