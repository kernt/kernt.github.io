---
tags:
  - monitoring
  - puppet
  - konfigurationsmanagement
---
# puppet-monitoring-reporting-berichte

Wenn Sie eine Menge von Maschinen verwalten, kann die Berichterstattung von Puppet Ihnen einige wertvolle Informationen darüber geben, was tatsächlich dort passiert ist.

## Wie es geht

Um Berichte zu aktivieren, füge dies einfach zu einer Client-Puppet.conf hinzu: in den Abschnitten [main] oder [agent]:

`report = true`

> Tip
>> In den letzten Versionen (größer als 3,0) der Puppet ist `report=true` die Standardeinstellung.

## Wie es funktioniert

Wenn das Reporting aktiviert ist, erzeugt Puppet eine Reportdatei mit Daten wie:

* Datum und Uhrzeit der Ausführung

* Gesamtzeit für den Lauf

* Protokollmeldungen ausgegeben während des Laufs

* Liste aller Ressourcen im Manifest des Clients

* Ob Puppet irgendwelche Ressourcen verändert hat und wie viele

* Ob der Run erfolgreich war oder nicht

Standardmäßig werden diese Berichte auf dem Node unter../puppet/v../puppet/l../puppet/pupp../puppet/reports` in einem Verzeichnis gespeichert, das nach dem Hostnamen benannt ist, aber Sie können ein anderes Ziel mit der Option `reportdir` angeben. Sie können Ihre eigenen Skripts erstellen, um diese Berichte zu verarbeiten (die sich im Standard-YAML-Format befinden). Wenn wir Puppet Agent auf `cookbook.example.com` ausführen, wird die folgende Datei auf dem Master erstellt:
`/v../puppet/l../puppet/pupp../puppet/repor../puppet/cookbook.example.c../puppet/201411230717.yaml`

## Es gibt mehr

Wenn Sie mehr als einen Master-Server haben, können Sie alle Ihre Berichte an denselben Server senden, indem Sie `report_server` im Abschnitt `[agent]` von `puppet.conf` angeben.

Wenn Sie nur einen Bericht wünschen, oder wenn Sie die Berichterstattung nicht immer aktivieren möchten, können Sie den `--report` Schalter zur Befehlszeile hinzufügen, wenn Sie den Puppet-Agent manuell ausführen:

```s
[root@cookbook ~]# puppet agent -t --report
Notice: Finished catalog run in 0.34 seconds
```

Sie sehen keine zusätzliche Ausgabe, aber eine Berichtsdatei wird im `report` verzeichnis generiert.

Sie können auch einige Gesamtstatistiken über einen Puppet-Run sehen, indem Sie den `--summarising` switch:

```s
[root@cookbook ~]# puppet agent -t --report --summarize
Notice: Finished catalog run in 0.35 seconds
Changes:
            Total: 2
Events:
            Total: 2
          Success: 2
Resources:
            Total: 10
          Changed: 2
      Out of sync: 2
Time:
Filebucket: 0.00
         Schedule: 0.00
           Notify: 0.00
Config retrieval: 0.94
            Total: 0.95
         Last run: 1416727357
Version:
Config: 1416727291
           Puppet: 3.7.3

```

## Weitere Berichtsarten

Puppet kann verschiedene Arten von Berichten mit der Berichtsoption im Abschnitt `[main]` oder `[master]` von `puppet.conf` auf deinen Puppet Master Server generieren. Es gibt mehrere eingebaute Berichtsarten, die unter [Puppet Reports](http../puppet//docs.puppetlabs.c../puppet/referenc../puppet/late../puppet/report.html) aufgeführt sind. Zusätzlich zu den eingebauten Berichtsarten gibt es einige gemeinschaftlich entwickelte Berichte, die sehr nützlich sind. Der [Foreman](htt../puppet//theforeman.org) bietet beispielsweise einen Foreman-Berichtstyp an, den Sie aktivieren können, um Ihre Node berichte an den Foreman weiterzuleiten.
