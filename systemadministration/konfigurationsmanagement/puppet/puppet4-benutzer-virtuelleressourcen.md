---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4-benutzer-virtuelleressource

Benutzer können ein echter Schmerz sein. Ich meine nicht die Leute, obwohl das zweifellos wahr ist. Aber das Beibehalten von UNIX-Benutzerkonten und Dateizugriffsrechnungen in einem Netzwerk von Maschinen, von denen einige verschiedene Betriebssysteme betreiben, kann ohne eine Art zentralisiertes Konfigurationsmanagement sehr schwierig sein.

Jeder neue Entwickler, der der Organisation beitritt, braucht ein Konto auf jeder Maschine, zusammen mit sudo Privilegien und Gruppenmitgliedschaften und braucht ihre SSH-Schlüssel für eine Reihe von verschiedenen Konten zugelassen. Der Systemadministrator, der sich um diese manuell kümmern muss, wird den ganzen Tag am Arbeitsplatz sein, während der Systemadministrator, der Puppet benutzt, in wenigen Minuten fertig ist und für ein frühes Mittagessen aussteigen wird.

In diesem Kapitel werden wir einige praktische Muster und Techniken betrachten, um Benutzer und ihre damit verbundenen Ressourcen zu verwalten. Benutzer sind auch eine der häufigsten Anwendungen für virtuelle Ressourcen, so dass wir alles über diese erfahren werden. Im letzten Abschnitt werden wir exportierte Ressourcen vorstellen, die mit virtuellen Ressourcen zusammenhängen.

In diesem Kapitel werden wir folgende Rezepte abdecken:

## [Virtuelle Ressourcen verwenden](puppet-fort-user-virtuelle-ressourcen-benutzen.md)

## [Verwalten von Benutzern mit virtuellen Ressourcen](puppet-fort-user-virtuelle-ressourcen-verwalten.md)

## [Verwalten des SSH-Zugriffs der Benutzer](puppet-fort-user-virtuelle-ressourcen-ssh.md)

## [Verwalten der Benutzeranpassungsdateien](puppet-fort-user-virtuelle-ressourcen-benutzer-anpassen.md)

## [Verwenden von exportierten Ressourcen](puppet-fort-user-virtuelle-ressourcen-export.md)
