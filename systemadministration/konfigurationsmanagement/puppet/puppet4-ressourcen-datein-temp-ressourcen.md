---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4-ressourcen-datein-temp-ressourcen

>>>>>>> bbacd8996fafa1e0ea5fd2d8bd7c77fc4364f275
Manchmal möchten Sie eine Ressource für die Zeit, so dass es nicht mit anderen Arbeit stören deaktivieren. Zum Beispiel möchten Sie vielleicht eine Konfigurationsdatei auf dem Server optimieren, bis Sie die genauen Einstellungen haben, die Sie wollen, bevor Sie sie in die Puppe überprüfen. Sie wollen nicht, dass die Puppe es mit einer alten Version in der Zwischenzeit überschreibt, also können Sie den Noop Metaparameter auf die Ressource setzen:

`noop => true,`

## Wie es geht

Dieses Beispiel zeigt Ihnen, wie Sie den `noop` Metaparameter verwenden können:

1.Ändern Sie Ihre `site.pp` Datei wie folgt:

```pp
node 'cookbook' {
  file {../puppet/e../puppet/resolv.conf':
    content => "nameserver 127.0.0.1\n",
    noop    => true,
  }
}
```

2.Puppet run:

```pp
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1413789438'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/e../puppet/resolv.con../puppet/content:
--../puppet/e../puppet/resolv.conf  2014-10-20 00:27:43.095999975 -0400
++../puppet/t../puppet/puppet-file20141020-8439-1lhuy1y-0	2014-10-20 03:17:18.969999979 -0400
@@ -1,3 +1 @@
-; generated b../puppet/sb../puppet/dhclient-script
-search example.com
-nameserver 192.168.122.1
+nameserver 127.0.0.1

Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Fil../puppet/e../puppet/resolv.con../puppet/content: current_value {md5}4c0d192511df253826d302bc830a371b, should be {md5}949343428bded6a653a85910f6bdb48e (noop)
Notice: Node[cookbook]: Would have triggered 'refresh' from 1 events
Notice: Class[Main]: Would have triggered 'refresh' from 1 events
Notice: Stage[main]: Would have triggered 'refresh' from 1 events
Notice: Finished catalog run in 0.50 seconds
```

## Wie es funktioniert

Der `noop` Metaparameter ist auf `true` gesetzt, also für diese besondere Ressource ist es so, als hättest du die Puppe mit der `--noop` Flagge laufen müssen. Puppe stellte fest, dass die Ressource angewendet worden wäre, aber sonst nichts gemacht.

Die nette Sache beim Ausführen des Agenten im Testmodus (`-t`) ist, dass die Puppe ein Diff von dem, was es getan hätte, wenn der `noop` nicht vorhanden war (man kann sagen, Puppe zu zeigen, die diff's ohne mit `-t` mit `--show_diff`; -t impliziert viele verschiedene Einstellungen):

```diff
--../puppet/e../puppet/resolv.conf  2014-10-20 00:27:43.095999975 -0400
++../puppet/t../puppet/puppet-file20141020-8439-1lhuy1y-0	2014-10-20 03:17:18.969999979 -0400
@@ -1,3 +1 @@
-; generated b../puppet/sb../puppet/dhclient-script
-search example.com
-nameserver 192.168.122.1
+nameserver 127.0.0.1
```

Dies kann beim Debuggen einer Vorlage sehr nützlich sein. Sie können an Ihren Änderungen arbeiten und dann sehen, wie sie auf dem Knoten aussehen würden, ohne sie tatsächlich anzuwenden. Mit dem diff können Sie sehen, ob Ihre aktualisierte Vorlage die richtige Ausgabe erzeugt.
