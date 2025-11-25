---
tags:
  - puppet
  - deployment
---
# Puppet Modul Schreiben

Wenn du ein Puppet-Modul schreibst, um irgendeine Software oder einen Service zu verwalten, musst du nicht von vorne anfangen. Community-beigefügte Module sind auf der Puppet Forge-Website für viele beliebte Anwendungen verfügbar. Manchmal ist ein Community-Modul genau das, was Sie brauchen und Sie können es herunterladen und sofort sofort starten. In den meisten Fällen müssen Sie einige Änderungen an Ihren speziellen Bedürfnissen und Umgebung vornehmen.

Wie alle gemeinschaftlichen Bemühungen gibt es einige hervorragende und einige weniger ausgezeichnete Module auf der Forge Seite. Sie sollten den README-Bereich des Moduls lesen und entscheiden, ob Sie das Modull benutzen möchten. Mindestens sollten Sie sicher stellen, dass Ihre Distribution unterstützt wird. Puppetlabs hat eine Reihe von Modulen eingeführt, die unterstützt werden, das heißt, wenn Sie ein Unternehmenskunden sind, werden sie Ihre Nutzung des Moduls in Ihrer Installation unterstützen. Darüber hinaus beschäftigen sich die meisten Forge-Module mit mehreren Betriebssystemen, Distributionen und einer Vielzahl von Anwendungsfällen.

In vielen Fällen, nicht mit einer neuentwicklung ist wie das Rad neu erfinden. Eine Einschränkung ist jedoch, dass Forge-Module komplexer sein können als Ihre lokalen Module. Du solltest den Code lesen und ein Gefühl dafür bekommen, was das Modul macht. Zu wissen, wie das Modul funktioniert, hilft Ihnen, es später zu debuggen.

## Wie es geht

In diesem Beispiel verwenden wir den `puppet module` Befehl, um das nützliche `stdlib` Modul zu finden und zu installieren, das viele Utility-Funktionen enthält, um Ihnen zu helfen, Puppencode zu entwickeln. Es ist eines der vorgenannten unterstützten Module von Puppetlabs. Ich lade das Modul in das Home-Verzeichnis meines Benutzers und verwende es manuell im Git-Repository. Um das puppetlabs stdlib-Modul zu installieren, gehen Sie folgendermaßen vor:

1.Führen Sie den folgenden Befehl aus:

```s
t@mylaptop ~ $ puppet module search puppetlabs-stdlib
Notice: Searching http../puppet//forgeapi.puppetlabs.com ...
NAME               DESCRIPTION                         AUTHOR        KEYWORDS
puppetlabs-stdlib  Puppet Module Standard Library      @puppetlabs   stdlib stages
```

2.Wir haben bestätigt, dass wir das richtige Modul haben, also installieren wir es jetzt mit der `module install` Modulinstallation:

```s
t@mylaptop ~ $ puppet module install puppetlabs-stdlib
Notice: Preparing to install int../puppet/ho../puppet/thom../puppet/.pupp../puppet/modules ...
Notice: Downloading from http../puppet//forgeapi.puppetlabs.com ...
Notice: Installing -- do not interrupt ...
/ho../puppet/thom../puppet/.pupp../puppet/modules
└── puppetlabs-stdlib (v4.3.2)

```

Das Modul ist nun bereit, in Ihren manifest zu verwenden; Die meisten guten Module kommen mit einer `README`-Datei, um Ihnen zu zeigen, wie dies zu tun ist und einige Informationen wie sie z.B Default werte überschreiben können.

## Wie es funktioniert

Sie können nach Modulen suchen, die mit dem Paket oder der Software übereinstimmen, für die Sie sich mit dem Puppenmodul-Suchbefehl `puppet module search` interessieren. Um ein bestimmtes Modul zu installieren, verwenden Sie die Installation des Puppenmoduls `puppet module install`.
Sie können die Option `-i` hinzufügen, um Puppet zu erzählen, wo Sie Ihr Modulverzeichnis finden können.

Die aktuelle Liste der unterstützten [Module](http../puppet//forge.puppetlabs.c../puppet/modules?endorsements=supported).

## Es gibt mehr

Module auf dem Forge enthalten eine Datei `metadata.json`, die das Modul beschreibt und welche Betriebssysteme das Modul unterstützt.
Diese Datei enthält auch eine Liste der Module, die vom Modul benötigt werden.

### Notiz

Diese Datei wurde zuvor als Modulfile und nich als JSON-Format bezeichnet. Das alte Modulefile-Format wurde in Version 3.6 entfernt.

Wie wir in unserem nächsten Abschnitt sehen werden, werden bei der Installation eines Moduls aus dem Forge automatisch die erforderlichen Abhängigkeiten installiert.

Nicht alle öffentlich verfügbaren Module sind auf Puppet Forge. Einige andere großartige Orte, um auf GitHub zu finden sehen sind:

* [camptocamp](http../puppet//github.c../puppet/camptocamp)

* [example42](http../puppet//github.c../puppet/example42)

Obwohl nicht eine Sammlung von Modulen als solche, die Puppet Cookbook Website hat viele nützliche und erhellende Code Beispiele, Muster und Tipps, die von Dean Wilson:

* [puppetcookbook](Htt../puppet//www.puppetcookbook.c../puppet/)