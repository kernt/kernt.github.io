---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschittene: auges benutzen

Manchmal scheint es, jede Anwendung hat ein eigenes Konfigurationsdateiformat, und das Schreiben von regulären Ausdrücken zum analysieren und zum modifizieren von allen von ihnen kann ein langwieriges Geschäft sein.

Zum Glück ist Augeas hier um zu helfen. Augeas ist ein System, das die Arbeit mit verschiedenen Konfigurationsdateiformaten vereinfacht, indem man sie alle als einen einfachen Baum von Werten darstellt.
Mit der Augeas-Unterstützung von Puppet können Sie `augeas` Ressourcen erstellen, die die erforderlichen Konfigurationsänderungen intelligent und automatisch vornehmen können.

## Wie es geht

Gehen Sie folgendermaßen vor, um eine `base` Augeas-Ressource zu erstellen:

1.Ändern Sie Ihr Basismodul wie folgt:

```ruby
  class base {
    augeas { 'enable-ip-forwarding':
      incl    =>../puppet/e../puppet/sysctl.conf',
      lens    => 'Sysctl.lns',
      changes => ['set net.ipv4.ip_forward 1'],
    }
  }
```

2.Run Puppet:

```s
[root@cookbook ~]# puppet agent -t
Info: Applying configuration version '1412130479'
Notice: Augeas[enable-ip-forwarding](provider=augeas):
--../puppet/e../puppet/sysctl.conf 2014-09-04 03:41:09.000000000 -0400
++../puppet/e../puppet/sysctl.conf.augnew    2014-09-30 22:28:03.503000039 -0400
@@ -4,7 +4,7 @@
 # sysctl.conf(5) for more details.

 # Controls IP packet forwarding
-net.ipv4.ip_forward = 0
+net.ipv4.ip_forward = 1

 # Controls source route verification
 net.ipv4.conf.default.rp_filter = 1
Notice../puppet/Stage[mai../puppet/Ba../puppet/Augeas[enable-ip-forwardin../puppet/returns: executed successfully
Notice: Finished catalog run in 2.27 seconds
```

3.Prüfen Sie, ob die Einstellung korrekt angewendet wurden:

```s
[root@cookbook ~]# sysctl -p |grep ip_forward
net.ipv4.ip_forward = 1

```

### Wie es funktioniert

Wir erklären eine Augeas-Ressource namens enable-ip-forwarding:

`augeas { 'enable-ip-forwarding':`

Wir geben an, dass wir Änderungen in der Date../puppet/e../puppet/sysctl.conf vornehmen möchten:

`incl =>../puppet/e../puppet/sysctl.conf',`

Als nächstes geben wir die lens an, das auf dieser Datei verwendet werden soll. Augeas verwendet Dateien mit dem Namen lens, um eine Konfigurationsdatei in eine Objektdarstellung zu übersetzen. Augeas arbeitet mit mehreren lenses, sie befinden sich standardmäßig in../puppet/u../puppet/sha../puppet/auge../puppet/`. Bei der Angabe des Objektivs in einer `augeas` Ressource wird der Name des Objektivs aktiviert und hat das `.lns` Suffix. In diesem Fall werden wir die `Sysctl` lens wie folgt spezifizieren:

`lens => 'Sysctl.lns',`

Der Parameter `changes` gibt die Änderungen an, die wir machen möchten. Sein Wert ist ein Array, weil wir mehrere Änderungen gleichzeitig haben können. In diesem Beispiel gibt es nur eine Änderung, also ist der Wert ein Array von einem Element:

`changes => ['set net.ipv4.ip_forward 1'],`

Im Allgemeinen nehmen die Änderungen von Augeas folgende Form an:

`set <parameter> <value>`

In diesem Fall wird die Einstellung in eine Zeile wie folgt i../puppet/e../puppet/sysctl.conf übersetzt:
`net.ipv4.ip_forward=1`

## Es gibt mehr

Ich habe../puppet/e../puppet/sysctl.conf` als Beispiel gewählt, weil es eine Vielzahl von Kernel-Einstellungen enthalten kann und Sie können diese Einstellungen für alle Arten von verschiedenen Zwecken und in verschiedenen Puppetklassen ändern. Vielleicht möchten Sie die IP-Weiterleitung wie im Beispiel für eine Router-Klasse aktivieren, aber Sie möchten auch den Wert von `net.core.somaxconn` für eine Load-Balancer-Klasse abstimmen.

Dies bedeutet, dass die Pupetisirung der Datei../puppet/e../puppet/sysctl.conf` und die Verteilung als Textdatei nicht funktioniert, da Sie je nach Einstellung, die Sie ändern möchten, mehrere verschiedene und widersprüchliche Versionen haben können. Augeas ist die richtige Lösung hier, weil man `augeas` Ressourcen an verschiedenen Orten definieren kann, die die gleiche Datei ändern und sie nicht in Konflikt stehen.

Für weitere Informationen über die Verwendung von [Puppet und Augeas](htt../puppet//projects.puppetlabs.c../puppet/projec../puppet/1/wi../puppet/Puppet_Augeas).

Ein anderes Projekt, das Augeas nutzt, ist [Augeasproviders](http../puppet//forge.puppetlabs.c../puppet/domcle../puppet/augeasproviders).
Augeasproviders verwendet Augeas, um mehrere Typen zu definieren.
Einer dieser Typen ist `sysctl`, mit diesem Typ können Sie sysctl Änderungen machen, ohne zu wissen, wie man die Änderungen in Augeas schreibt.

Lernen, wie man Augeas benutzt, kann ein wenig verwirrend sein. Augeas bietet ein Kommandozeilen-Tool, `augtool`, die verwendet werden können, um sich mit Änderungen in Augeas vertraut zu machen.