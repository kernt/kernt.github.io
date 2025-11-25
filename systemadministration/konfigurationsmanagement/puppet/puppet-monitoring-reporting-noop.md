---
tags:
  - monitoring
  - puppet
  - konfigurationsmanagement
---
# puppet-monitoring-reporting-noop

Manchmal tut dein Puppet manifest nicht genau das, was du erwartet hast, oder vielleicht hat jemand anderes Änderungen vorgenommen, die du nicht gewusst hast. So oder so ist es gut, genau zu wissen, was die Puppet machen wird, bevor es das tut.

Wenn Sie die Puppe in eine bestehende Infrastruktur umrüsten, wissen Sie vielleicht nicht, ob Puppet eine `config` datei aktualisieren oder einen Produktionsdienst neu starten wird. Eine solche Änderung könnte zu ungeplanten Ausfallzeiten führen. Außerdem werden manchmal manuelle Konfigurationsänderungen auf einem Server vorgenommen, den Puppet überschreiben würde.

Um diese Probleme zu vermeiden, kannst du den Noop-Modus von Puppet verwenden, was keine Operation bedeutet oder nichts macht. Wenn du mit der noop-Option gehst, meldet die Puppe nur, was sie tun würde, aber macht eigentlich nichts. Eine Einschränkung hier ist, dass auch während eines Noop-Laufs Pluginsync noch läuft und alle lib Verzeichnisse in Modulen werden mit Knoten synchronisiert. Dies wird externe Tatsachendefinitionen und eventuell Puppets Typen und Provider aktualisieren.

## Wie es geht

Sie können den `--noop` Modus ausführen, wenn Sie einen `puppet agent` oder `puppet apply` ausführen, indem Sie den Schalter --noop an den Befehl anhängen. Sie können auch eine `noop=true` Linie in Ihrer `puppet.conf` Datei innerhalb der `[agent]` oder `[main]` Abschnitte erstellen

1.Erstellen Sie ein `noop.pp` manifest, das eine Datei wie folgt erstellt:

```pp
file ../puppet/t../puppet/noop':
  content => 'nothing',
  mode    => 0644,
}
```

2.Führen Sie nun den Puppet agent mit dem `noop` Switch aus:

```s
t@mylaptop../puppet/pupp../puppet/manifests $ puppet apply noop.pp --noop
Notice: Compiled catalog for mylaptop in environment production in 0.41 seconds
Notice../puppet/Stage[mai../puppet/Ma../puppet/Fil../puppet/t../puppet/noo../puppet/ensure: current_value absent, should be file (noop)
Notice: Class[Main]: Would have triggered 'refresh' from 1 events
Notice: Stage[main]: Would have triggered 'refresh' from 1 events
Notice: Finished catalog run in 0.02 seconds
```

3.Jetzt laufen ohne die `noop` Option, um zu sehen, dass die Datei erstellt wird:

```s
t@mylaptop../puppet/pupp../puppet/manifests $ puppet apply noop.pp
Notice: Compiled catalog for mylaptop in environment production in 0.37 seconds
Notice../puppet/Stage[mai../puppet/Ma../puppet/Fil../puppet/t../puppet/noo../puppet/ensure: defined content as '{md5}3e47b75000b0924b6c9ba5759a7cf15d'
```

## Wie es funktioniert

Im `noop` Modus macht die Puppe alles, was sie normalerweise tun würde, mit der Ausnahme, dass sie tatsächlich irgendwelche Änderungen an der Maschine vornehmen (die `exec` Ressourcen werden zum Beispiel nicht ausgeführt). Es sagt dir, was es getan hätte, und du kannst das mit dem vergleichen, was du erwartet hast. Wenn es Unterschiede gibt, überprüfen Sie den Manifest oder den aktuellen Zustand der Maschine.

## Hinweis

Beachten Sie, dass, wenn wir mit `--noop` lief, Puppet warnte uns, dass es die Datei../puppet/t../puppet/noop` erstellt hätte. Dies kann oder auch nicht sein, was wir wollen, aber es ist nützlich, im voraus zu wissen. Wenn Sie Änderungen an dem Code vornehmen, der auf Ihre Produktionsserver angewendet wird, ist es sinnvoll, den Puppet-Agent mit der Option `--noop` auszuführen, um sicherzustellen, dass Ihre Änderungen die Produktionsdienste nicht beeinträchtigen.

## Es gibt mehr

Sie können den Noop-Modus auch als einfaches Auditing-Tool verwenden. Es wird Ihnen sagen, ob irgendwelche Änderungen an der Maschine vorgenommen worden sind, seit die Puppe zuletzt ihr Manifest angewendet hat. Einige Organisationen erfordern alle Konfigurationsänderungen, die mit Puppet gemacht werden sollen, was eine Möglichkeit ist, einen Änderungssteuerungsprozess zu implementieren. Unerlaubte Änderungen an den von Puppet verwalteten Ressourcen können mit dem Puppet im Noop-Modus erkannt werden und können dann entscheiden, ob die Änderungen wieder in das Puppet-Manifest verschmelzen oder sie rückgängig machen sollen.

Sie können auch die `--debug` Schalter verwenden, wenn Sie Puppen-Agent zu sehen, die Details jeder Änderung Puppet macht während eines Agenten laufen. Dies kann hilfreich sein, wenn man versucht, herauszufinden, wie Puppet bestimmte Exec-Ressourcen anwendet oder um zu sehen, in welcher Reihenfolge Dinge passieren.

Wenn du einen Master betreibst, kannst du den Katalog für einen Knoten auf dem Master mit der Option `--trace` zusätzlich zu `--debug` kompilieren. Wenn der Katalog nicht kompiliert wird, wird diese Methode auch nicht kompilieren den Katalog (wenn Sie eine alte Definition für den Kochbuchknoten haben, der fehlschlägt, versuchen Sie es zu kommentieren, bevor Sie diesen Test ausführen). Dies erzeugt eine Menge Debugging-Ausgabe. Zum Beispiel, um den Katalog für unseren Coocbook-Host auf unserem Master zu kompilieren und die Ergebnisse in../puppet/t../puppet/cookbook.log` zu platzieren:

```s
root@puppet: ~#puppet master --compile cookbook.example.com --debug --trace --logdes../puppet/t../puppet/cookbook.log
Debug: Executing../puppet/e../puppet/pupp../puppet/cookbook.sh cookbook.example.com'
Debug: Using cached facts for cookbook.example.com
Info: Caching node for cookbook.example.com
Debug: importing../puppet/e../puppet/pupp../puppet/environmen../puppet/producti../puppet/modul../puppet/e../puppet/manifes../puppet/init.pp' in environment production
Debug: Automatically imported enc from enc into production
Notice: Compiled catalog for cookbook.example.com in environment production in 0.09 seconds
Info: Caching catalog for cookbook.example.com
Debug: Configuring PuppetDB terminuses with config fil../puppet/e../puppet/pupp../puppet/puppetdb.conf
Debug: Using cached certificate for ca
Debug: Using cached certificate for puppet
Debug: Using cached certificate_revocation_list for ca
Info: 'replace catalog' command for cookbook.example.com submitted to PuppetDB with UUIDe2a655ca-bd81-4428-b70a-a3a76c5f15d1
{
  "metadata": {
    "api_version": 1
  },
  "data": {
    "edges": [
      {
        "target": "Class[main]",
        "source": "Stage[main]"
...

```

Note:

Nach dem Kompilieren des Katalogs druckt Puppet den Katalog in die Kommandozeile. Die Protokolldatei ../puppet/t../puppet/cookbook.log`) hat eine Menge Informationen darüber, wie der Katalog kompiliert wurde.
