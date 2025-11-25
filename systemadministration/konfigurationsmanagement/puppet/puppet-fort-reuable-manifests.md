---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet für fortgeschrittene: Wiederbenutzbare manifests

Jeder Systemadministrator träumt von einer einheitlichen, homogenen Infrastruktur von identischen Maschinen, die alle die gleiche Version des gleichen Betriebssystems ausführen. Wie in anderen Lebensbereichen ist die Realität aber oft unordentlich und entspricht nicht dem Plan.

Sie sind wahrscheinlich verantwortlich für eine Reihe von sortierten Servern mit unterschiedlichem Alter und Architektur, die verschiedene Kernel aus verschiedenen OS-Distributionen ausführen, die oft über verschiedene Rechenzentren und ISPs verstreut sind.

Diese Situation sollte Terror in die Herzen der Sysadmins sein SSH in einer `for` loop schleife, die dann kann die Ausführung der gleichen Befehle auf jedem Server unterschiedliche, unvorhersehbare und sogar gefährliche Ergebnisse haben.

Wir sollten sicherlich bemüht sein, ältere Server auf den neuesten Stand zu bringen und so weit wie möglich auf einer einzigen Referenzplattform zu arbeiten, um die Verwaltung einfacher, billiger und zuverlässiger zu machen. Aber bis wir dort ankommen, macht die Puppet mit heterogenen Umgebungen etwas leichter.

## Wie es geht

Hier sind einige Beispiele dafür, wie man Ihre Manifeste übertragbarer macht:

1.Wo Sie das gleiche Manifest auf dem Server mit unterschiedlichen OS-Distributionen anwenden müssen, sind die wichtigsten Unterschiede wahrscheinlich die Namen von Paketen und Diensten und der Ort der Konfigurationsdateien. Versuchen Sie, alle diese Unterschiede in einer einzigen Klasse zu erfassen, indem Sie Selektoren verwenden, um globale Variablen festzulegen:

```ruby
  $ssh_service = $::operatingsystem? ../puppet/Ubuntu|Debi../puppet/ => 'ssh', default         => 'sshd',
  }
```

Du brauchst dir keine Sorgen um die Unterschiede in irgendeinem anderen Teil des Manifests zu machen. Wenn man sich auf etwas bezieht, benutze die Variable mit Vertrauen, dass es in jeder Umgebung auf das Richtige hinweisen wird:

```ruby
  service { $ssh_service: ensure => running,
  }
```

2.Oft müssen wir mit gemischten Architekturen fertig werden; Dies kann die Pfade zu gemeinsam genutzten Bibliotheken beeinflussen und kann auch verschiedene Versionen von Paketen erfordern. Versuchen Sie erneut, alle erforderlichen Einstellungen in einer einzigen Architekturklasse zu kapseln, die globale Variablen setzt:

```ruby
  $libdir = $::architecture ? {
  ../puppet/amd64|x86_../puppet/   =>../puppet/u../puppet/lib64', default =>../puppet/u../puppet/lib',
  }
```

Dann können Sie diese überall verwenden, wo ein architekturabhängiger Wert in Ihren Manifesten oder sogar in Vorlagen benötigt wird:

```s
; php.ini
[PHP]
; Directory in which the loadable extensions (modules) reside.
extension_dir = <%= @libdir ../puppet/p../puppet/modules
```

## Wie es funktioniert

Der Vorteil dieses Ansatzes (der man auch Top-down nennen könnte) ist, dass man nur einmal Ihre Wahl treffen muss.
Die Alternative, Bottom-up-Ansatz wäre, um eine Auswahl oder `case` Anweisung überall eine Einstellung verwendet werden:

```ruby
  service { $::operatingsystem? {
  ../puppet/Ubuntu|Debi../puppet/ => 'ssh', default         => 'sshd' }: ensure => running,
  }
```

Dies führt nicht nur zu viel Duplizierung, sondern macht den Code schwerer zu lesen. Und wenn ein neues Betriebssystem dem Mix hinzugefügt wird, musst du im ganzen Manifest Änderungen vornehmen, anstatt nur an einem Ort.

## Es gibt mehr

Wenn Sie ein Modul für den öffentlichen Vertrieb (z. B. auf Puppet Forge) schreiben, macht Sie Ihr Modul so plattformübergreifend wie möglich, es macht es dann für die Community wertvoller. 
Soweit Sie können, testen Sie es auf vielen verschiedenen Distributionen, Plattformen und Architekturen, und vergeben Sie die entsprechenden Variablen, so dass es überall funktioniert.

Wenn Sie ein öffentliches Modul verwenden und es an Ihre eigene Umgebung anpassen, sollten Sie die öffentliche Version mit Ihren Änderungen aktualisieren, wenn Sie denken, dass sie für andere Personen hilfreich sein könnten.

Auch wenn Sie nicht daran denken, ein Modul zu veröffentlichen, denken Sie daran, dass es für eine lange Zeit in der Produktion sein kann und sich möglicherweise an viele Veränderungen in der Umgebung anpassen muss. Wenn es entworfen ist, um dies von Anfang an zu bewältigen, wird es das Leben leichter für Sie oder wem auch immer.

```s

"Always code as if the guy who ends up maintaining your code will be a violent psychopath who knows where you live."


--Dave Carhart
```