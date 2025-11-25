---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4-ressourcen-datein-term

So weit, haben wir gesehen, was Puppet für uns tun kann, und die Ordnung, dass es Dinge umsetzt, aber nicht, wenn es sie durchführt.
Eine Möglichkeit, dies zu kontrollieren, ist, den `schedule` Metaparameter zu verwenden. Wenn Sie die Anzahl der Zeiten, in denen eine Ressource innerhalb eines bestimmten Zeitraums angewendet wird, begrenzen, kann `schedule` helfen. Beispielsweise:

```pp
>>>>>>> bbacd8996fafa1e0ea5fd2d8bd7c77fc4364f275
exec {../puppet/u../puppet/b../puppet/apt-get update":
    schedule => daily,
}
```

Die wichtigste Sache, um `schedule` zu verstehen ist, dass es  nur eine Ressource stoppen kann die auch benutzt wird. Es garantiert nicht, dass die Ressource mit einer bestimmten Frequenz angewendet wird. Zum Beispiel hat die `exec` Ressource, die in dem vorherigen Code-Snippet gezeigt wird, `schedule => daily`, aber dies stellt nur eine obere Grenze für die Anzahl von wiederhollungen dar, die die `exec` Ressource pro Tag ausführt werden kann.
Es wird nicht mehr als einmal am Tag angewendet. Wenn du überhaupt kein Puppet betreibst, wird die Ressource überhaupt nicht angewendet.

Die Verwendung des Stundenplans ist beispielsweise auf einer Maschine sinnlos, die für die Ausführung des Agenten alle 4 Stunden konfiguriert ist ( `runinterval` über die Konfigurationseinstellung).

Das heißt, der `schedule` wird am besten verwendet, um die Ressourcen vom Laufen zu beschränken, wenn sie nicht laufen oder laufen nicht müssen; Zum Beispiel möchten Sie vielleicht sicherstellen, dass `apt-get update` nicht mehr als einmal pro Stunde ausgeführt wird. Es gibt einige eingebaute Zeitpläne um Sie  zu verwenden:

* hourly
* daily
* weekly
* monthly
* never

Allerdings können Sie diese ändern und erstellen Sie Ihre eigenen benutzerdefinierten `schedule`(Studen Pläne), mit der Zeitplan Ressource. Wir werden sehen, wie dies im folgenden Beispiel zu tun ist. Nehmen wir an, wir wollen sicherstellen, dass eine `exec` Ressource, die einen Wartungsauftrag darstellt, nicht während der Sprechstunden(User müssen da etwas Zeitbegrenzt nutzen meist einen Dienst) laufen wird,die die Produktion beeinträchtigen könnte.

## Wie es geht

In diesem Beispiel erstellen wir eine benutzerdefinierte `schedule` ressource und weisen diese der Ressource zu:

1.Ändern Sie Ihre `site.pp` Datei wie folgt:

```pp
>>>>>>> bbacd8996fafa1e0ea5fd2d8bd7c77fc4364f275
schedule { 'outside-office-hours':
  period => daily,
  range  => ['17:00-23:59','00:00-09:00'],
  repeat => 1,
}
node 'cookbook' {
  notify { 'Doing some maintenance':
    schedule => 'outside-office-hours',
  }
}
```


1. Puppet run ausführen. Was du sehen wirst, hängt von der Tageszeit ab. Wenn es derzeit außerhalb der Öffnungszeiten ist, die Sie definiert haben, wird Puppet die Ressource wie folgt anwenden:
```

2.Puppet run ausführen. Was du sehen wirst, hängt von der Tageszeit ab. Wenn es derzeit außerhalb der Öffnungszeiten ist, die Sie definiert haben, wird Puppet die Ressource wie folgt anwenden:

```pp
[root@cookbook ~]# date
Fri Jan  2 23:59:01 PST 2015
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1413734477'
Notice: Doing some maintenance
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[cookboo../puppet/Notify[Doing some maintenanc../puppet/message: defined 'message' as 'Doing some maintenance'
Notice: Finished catalog run in 0.07 seconds
```

3. Wenn die Zeit innerhalb der Sprechstunden ist, wird die Puppet nichts tun:

```pp
[root@cookbook ~]# date
Fri Jan  2 09:59:01 PST 2015
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1413734289'
Notice: Finished catalog run in 0.09 seconds
```

## Wie es funktioniert

Ein Zeitplan besteht aus drei Bits:

* Der Zeitraum(`period`) (stündlich(hourly), täglich(daily), wöchentlich(weekly) oder monatlich(monthly))

* Der Bereich(`range`) (standardmäßig auf den ganzen Zeitraum, kann aber ein kleinerer Teil davon sein)

* Die Wiederholungszählung(`repeat`) zählt(wie oft die Ressource darf innerhalb des Bereichs angewendet werden, die Voreinstellung ist 1 oder einmal pro Periode)

Unser kundenspezifischer Zeitplan, der `outside-office-hours` genannt wird, liefert diese drei Parameter:

```pp
>>>>>>> bbacd8996fafa1e0ea5fd2d8bd7c77fc4364f275
schedule { 'outside-office-hours':
  period => daily,
  range  => ['17:00-23:59','00:00-09:00'],
  repeat => 1,
}
```

Die `period`(Periode) ist `daily`(täglich), und der `range`(Bereich) ist definiert als ein Array von zwei Zeitintervallen:

```s
17:00-23:59
00:00-09:00
```

Der `schedule`, der mit `outside-office-hours` benannt ist, steht nun für uns zur Verfügung, um mit jeder Ressource zu arbeiten, so als ob es in Puppet wie die `daily`(Tages) oder `hourly`(Stunden) Pläne eingebaut wurde. In unserem Beispiel vergeben wir diesen Zeitplan der `exec` Ressource mit dem `schedule`-Metaparameter:

```pp
notify { 'Doing some maintenance':
  schedule => 'outside-office-hours',
}
```

Ohne diesen `schedule` würde die Ressource jedes Mal angewendet, wenn die Puppe läuft. Mit ihm wird Puppet die folgenden Parameter überprüfen, um zu entscheiden, ob die Ressource angewendet werden soll oder nicht:

* Ob die Zeit im zulässigen Bereich liegt

* Ob die Ressource bereits in dieser Periode die maximal zulässige Anzahl der Zeiten durchgeführt hat

Zum Beispiel, betrachten wir, was passiert, wenn Puppet um 4 Uhr, 5 Uhr und 6 Uhr an einem bestimmten Tag läuft:

* 4 Uhr .: Es ist außerhalb der erlaubten Zeitspanne, so dass Puppet nichts tun wird

* 5 Uhr .: Es ist innerhalb der erlaubten Zeitspanne, und die Ressource wurde noch nicht in diesem Zeitraum ausgeführt, so wird Puppet die Ressource anwenden

* 6 Uhr .: Es ist innerhalb der erlaubten Zeitspanne, aber die Ressource wurde bereits die maximale Anzahl von Zeiten in diesem Zeitraum ausgeführt, so Puppet wird nichts tun

Und so weiter bis zum nächsten Tag.

## Es gibt mehr

Der `repeat`(Wiederholungs) parameter regelt, wie oft die Ressource bei den anderen Einschränkungen des Zeitplans angewendet wird. Zum Beispiel, um eine Ressource nicht mehr als sechs Mal pro Stunde anzuwenden, verwenden Sie einen Zeitplan wie folgt:

```s
period => hourly,
repeat => 6,
```

Denken Sie daran, dass dies nicht garantiert, dass der Job sechs Mal pro Stunde läuft. Es setzt einfach eine obere Grenze; Egal wie oft Puppe läuft oder sonst etwas passiert, wird der Job nicht laufen, wenn es schon sechsmal in dieser Stunde gelaufen ist. Wenn Puppet nur einmal am Tag läuft, wird der Job nur einmal ausgeführt. So ein `schedule`(Zeitplan) wird am besten verwendet, um sicherzustellen, dass Dinge nicht zu bestimmten Zeiten passieren (oder nicht eine gegebene Frequenz überschreiten ).