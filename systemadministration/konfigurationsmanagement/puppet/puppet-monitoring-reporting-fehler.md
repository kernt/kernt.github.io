---
tags:
  - monitoring
  - docker
  - konfigurationsmanagement
---
# puppet-monitoring-reporting-fehler

Puppets Fehlermeldungen können manchmal etwas verwirrend sein. Aktualisierte und zunehmend hilfreiche Fehlermeldungen sind ein Grund für die Aktualisierung Ihrer Puppet-Installation, wenn Sie eine Version vor Version 3 ausführen.

Hier sind einige der häufigsten Fehler, die Sie vielleicht begegnen, und was zu tun ist.

## Wie es geht

Oft ist der erste Schritt einfach, um das Web für die Fehlermeldung Text zu suchen und sehen, welche Erklärungen Sie für den Fehler finden können, zusammen mit jeder hilfreiche Beratung über die Festsetzung es. Hier sind einige der häufigsten rätselhaften Fehler, mit möglichen Erklärungen:
`Could not retrieve file metadata for XXX: getaddrinfo: Name or service not known`

Wo `XXX` ist eine Datei Ressource, können Sie versehentlich eingegeben `puppe../puppet//modules...` in einer Datei-Quelle anstelle von `puppe../puppet///modules...` (beachten Sie die Triple Slash):
`Could not evaluate: Could not retrieve information from environment production source(s) XXX`

Die Quelldatei ist möglicherweise nicht vorhanden oder darf nicht in der richtigen Position im Puppet Repo sein:
Error: Could not set 'file' on ensure: No such file or directory XXX

Der Dateipfad kann ein übergeordnetes Verzeichnis (oder Verzeichnisse) angeben, das nicht vorhanden ist. Sie können in der Puppe separate Dateiressourcen verwenden, um diese zu erstellen:
`change from absent to file failed: Could not set 'file on ensure: No such file or directory`

Dies wird oft durch Puppe verursacht, die versucht, eine Datei in ein Verzeichnis zu schreiben, das nicht existiert. Prüfen Sie, ob das Verzeichnis bereits vorhanden ist oder in der Puppe definiert ist und dass die Dateiressource das Verzeichnis benötigt (damit das Verzeichnis immer zuerst erstellt wird):
`undefined method 'closed?' for nil:NilClass`

Diese nicht hilfreiche Fehlermeldung wird grob übersetzt, da etwas schief gelaufen ist. Es neigt dazu, ein catch-all Fehler durch viele verschiedene Probleme verursacht werden, aber Sie können in der Lage zu bestimmen, was ist falsch aus dem Namen der Ressource, die Klasse oder das Modul. Ein Trick ist, den `--debug` Switch hinzuzufügen, um nützliche Informationen zu erhalten:
`[root@cookbook ~]# puppet agent -t --debug`

Wenn du deine Git-Geschichte überprüftest, um zu sehen, was in der jüngsten Veränderung berührt wurde, kann dies ein anderer Weg sein, um zu identifizieren, was umstößige Puppet ist:
`Could not parse for environment --- "--- production": Syntax error at end of file at line 1`

Dies kann durch Negativ-Befehlszeilenoptionen verursacht werden, z.B wenn Sie `puppet -verbose` statt `puppet --verbose` eingeben. Diese Art von Fehler kann schwer zu sehen sein:
`Duplicate definition: X is already defined in [file] at line Y; cannot redefine at [file] line Y`

Das hat mir in der Vergangenheit ein bisschen verwirrt. Puppet beschwert sich über eine doppelte Definition, und normalerweise, wenn Sie zwei Ressourcen mit dem gleichen Namen haben, wird Puppet Ihnen gerne sagen, wo sie beide definiert sind. Aber in diesem Fall gibt es die gleiche Datei und Zeilennummer für beide. Wie kann eine Ressource ein Duplikat von sich sein?

Die Antwort ist, wenn es sich um einen definierten Typ handelt (eine Ressource, die mit dem Schlüsselwort `define` wurde). Wenn Sie zwei Instanzen eines definierten Typs erstellen, haben Sie auch zwei Instanzen aller Ressourcen, die in der Definition enthalten sind, und sie müssen unterschiedliche Namen haben.
Beispielsweise:

```pp
define check_process() {
  exec { 'is-process-running?':
    command =>../puppet/b../puppet/ps ax../puppet/b../puppet/grep ${name}../puppet/t../puppet/pslist.${name}.txt",
  }
}

check_process { 'exim': }
check_process { 'nagios': }
```

Wenn wir Puppet laufen lassen, wird der gleiche Fehler zweimal ausgegeben:

```s
t@mylaptop ~$ puppet apply duplicate.pp
Error: Duplicate declaration: Exec[is-process-running?] is already declared in file duplicate.pp:4; cannot redeclare at duplicate.pp:4 on node cookbook.example.com
Error: Duplicate declaration: Exec[is-process-running?] is already declared in file duplicate.pp:4; cannot redeclare at duplicate.pp:4 on node cookbook.example.com
```

Weil die `exec`-Ressource heißt `is-process-running?` , Wenn Sie versuchen, mehr als eine Instanz der Definition zu erstellen, wird Puppet verweigern, weil das Ergebnis zwei `exec`-Ressourcen mit dem gleichen Namen wäre. Die Lösung besteht darin, den Namen der Instanz (oder einen anderen eindeutigen Wert) in den Titel jeder Ressource aufzunehmen:

```pp
exec { "is-process-${name}-running?":
  command =>../puppet/b../puppet/ps ax../puppet/b../puppet/grep ${name}../puppet/t../puppet/pslist.${name}.txt",
}
```

Jede Ressource muss einen eindeutigen Namen haben und einen guten Weg, um dies mit einer Definition zu gewährleisten, um die Variable `${name}` in ihrem Titel zu interpolieren. Beachten Sie, dass wir von der Verwendung von Single-to-Double-Anführungszeichen im Ressourcentitel umgestellt haben:
`"is-process-${name}-running?"`

Die doppelten Anführungszeichen sind erforderlich, wenn Puppet den Wert einer Variablen in einen String interpolieren soll.
