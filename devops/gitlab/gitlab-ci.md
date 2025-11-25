---
tags:
  - gitlab
  - gitlab-ci
---
# Erste Schritte mit GitLab CI / CD

> **Hinweis:** Beachten Sie, dass nur Projektbetreuer und Administratoren die Berechtigung haben, auf die Einstellungen eines Projekts zuzugreifen.

> **Hinweis:** Kommen Sie von Jenkins zu GitLab? Sehen Sie sich unsere Referenz an, um Ihre vorhandenen Pipelines in unser Format umzuwandeln.

GitLab bietet einen kontinuierlichen Integrationsservice. 
Wenn Sie eine `.gitlab-ci.yml` Datei zum Stammverzeichnis Ihres Repositorys hinzufügen und Ihr GitLab-Projekt so konfigurieren, dass ein Runner verwendet wird , wird bei jedem Commit oder Push die CI- Pipeline ausgelöst.

Die `.gitlab-ci.yml` Datei teilt dem GitLab Runner mit, was zu tun ist. Standardmäßig läuft es eine Pipeline mit drei Stufen: `build`, `test`, und `deploy`.
Sie müssen nicht alle drei Stufen verwenden. Stufen ohne Jobs werden einfach ignoriert.
Wenn alles in Ordnung ist (keine Rückgabewerte ungleich Null), wird dem Commit ein schönes grünes Häkchen zugeordnet. Auf diese Weise können Sie leicht feststellen, ob durch ein Commit ein Test fehlgeschlagen ist, bevor Sie sich den Code überhaupt ansehen.
Die meisten Projekte verwenden den CI-Service von GitLab, um die Testsuite auszuführen, sodass Entwickler sofort Feedback erhalten, wenn sie etwas kaputt machen.
Es gibt einen wachsenden Trend, die kontinuierliche Bereitstellung und Bereitstellung zu verwenden, um getesteten Code automatisch in Staging- und Produktionsumgebungen bereitzustellen.

Kurz gesagt, die Schritte, die für ein funktionierendes CI erforderlich sind, lassen sich wie folgt zusammenfassen:

1. Fügen Sie dies .gitlab-ci.ymlzum Stammverzeichnis Ihres Repository hinzu
2. Konfigurieren Sie einen Runner

Von da an startet der Runner bei jedem Push in Ihr Git-Repository automatisch die Pipeline und die Pipeline wird auf der Pipelines- Seite des Projekts angezeigt .

In diesem Handbuch wird vorausgesetzt, dass Sie über Folgendes verfügen:

* Eine funktionierende GitLab-Instanz ab Version 8.0 oder GitLab.com.
* Ein Projekt in GitLab, für das Sie CI verwenden möchten.
* Betreuer- oder Eigentümerzugriff auf das Projekt

Zerlegen wir es in Teile und arbeiten an der Lösung des GitLab CI-Puzzles.

## Eine `.gitlab-ci.yml` Datei erstellen

Bevor Sie erstellen, .gitlab-ci.ymllassen Sie uns zunächst kurz erklären, worum es geht.

### Was ist `.gitlab-ci.yml`

In der `.gitlab-ci.yml` Datei konfigurieren Sie, was CI mit Ihrem Projekt macht. Es befindet sich im Stammverzeichnis Ihres Repositorys.
Bei jedem Push in Ihr Repository sucht GitLab nach der `.gitlab-ci.yml`
Datei und startet Jobs auf Runners entsprechend dem Inhalt der Datei für dieses Commit.
Da `.gitlab-ci.yml` es sich im Repository befindet und versionskontrolliert ist, werden alte Versionen immer noch erfolgreich erstellt, Forks können CI problemlos verwenden, Zweigstellen können unterschiedliche Pipelines und Jobs haben und Sie haben eine einzige Informationsquelle für CI. `.gitlab-ci.yml` In unserem Blog erfahren Sie mehr über die Gründe, warum wir sie verwenden .

### Eine einfache `.gitlab-ci.yml` Datei erstellen

> **Hinweis:** Ist `.gitlab-ci.yml` eine YAML- Datei, so dass Sie besonders auf Einrückungen achten müssen. Verwenden Sie immer Leerzeichen, keine Tabulatoren.

Sie müssen eine Datei mit dem Namen `.gitlab-ci.yml` im Stammverzeichnis Ihres Repositorys erstellen . Unten sehen Sie ein Beispiel für ein Ruby on Rails-Projekt.

```yml
image: "ruby:2.5"

before_script:
  - apt-get update -qq && apt-get install -y -qq sqlite3 libsqlite3-dev nodejs
  - ruby -v
  - which ruby
  - gem install bundler --no-document
  - bundle install --jobs $(nproc)  "${FLAGS[@]}"

rspec:
  script:
    - bundle exec rspec

rubocop:
  script:
    - bundle exec rubocop

```

Dies ist die einfachste Konfiguration, die für die meisten Ruby-Anwendungen funktioniert:

1. Definieren Sie zwei Jobs `rspe` cund `rubocop` (die Namen sind beliebig) mit unterschiedlichen auszuführenden Befehlen.
2. Vor jedem Auftrag werden die von definierten Befehle `before_script` ausgeführt.

Die `.gitlab-ci.yml` Datei definiert Auftragssätze mit Einschränkungen, wie und wann sie ausgeführt werden sollen.
Die Jobs sind als Elemente der obersten Ebene mit einem Namen (in unserem Fall rspecund rubocop) definiert und müssen immer das scriptSchlüsselwort enthalten.
Jobs werden zum Erstellen von Jobs verwendet, die dann von Läufern ausgewählt und in der Umgebung des Läufers ausgeführt werden.
Wichtig ist, dass jeder Job unabhängig voneinander ausgeführt wird.
Wenn Sie überprüfen möchten, ob das `.gitlab-ci.yml` Ihres Projekts gültig ist, befindet sich ein Lint-Tool unter der Seite `/-/ci/lint` Ihres Projektnamensraums.
Unter **CI / CD ➔ Pipelines** und **Pipelines ➔ Jobs** in Ihrem Projekt finden Sie auch eine Schaltfläche "CI Lint", um zu dieser Seite zu gelangen .
Weitere Informationen und eine vollständige `.gitlab-ci.yml` Syntax finden Sie in der [Referenzdokumentation unter `.gitlab-ci.yml`](https://gitlab.com/help/ci/yaml/README.md).

#### Push `.gitlab-ci.yml` to GitLab

Sobald Sie erstellt haben `.gitlab-ci.yml`, sollten Sie es zu Ihrem Git-Repository hinzufügen und an GitLab senden.

```
git add .gitlab-ci.yml
git commit -m "Add .gitlab-ci.yml"
git push origin master

```

Wenn Sie jetzt zur Seite Pipelines gehen, sehen Sie, dass die Pipeline aussteht.

## Einen Runner konfigurieren

In GitLab führen Runners die Jobs aus, in denen Sie definiert haben `.gitlab-ci.yml`. 
Ein Runner kann eine virtuelle Maschine, ein VPS, eine Bare-Metal-Maschine, ein Docker-Container oder sogar ein Cluster von Containern sein. 
GitLab und die Runners kommunizieren über eine API. 
Die einzige Voraussetzung ist, dass der Runner-Computer über Netzwerkzugriff auf den GitLab-Server verfügt.
Ein Runner kann für ein bestimmtes Projekt spezifisch sein oder mehrere Projekte in GitLab bedienen.

Wenn es allen Projekten dient, wird es als _Shared Runner_ bezeichnet .

Weitere Informationen zu verschiedenen Läufern finden Sie in der Dokumentation zu den [Läufern](https://gitlab.com/help/ci/runners/README.md) .
Sie können feststellen, ob Ihrem Projekt Läufer zugewiesen sind, indem Sie auf Einstellungen **assigned CI / CD** klicken.
Das Einrichten eines Läufers ist einfach und unkompliziert.
Um einen funktionierenden Runner zu haben, müssen Sie zwei Schritte ausführen:

* [Es installieren](https://docs.gitlab.com/runner/install/)
* [Konfigurieren Sie es](https://gitlab.com/help/ci/runners/README.md#registering-a-specific-runner)

Folgen Sie den obigen Links, um Ihren eigenen Läufer einzurichten oder einen freigegebenen Läufer zu verwenden, wie im nächsten Abschnitt beschrieben.
Sobald der Runner eingerichtet wurde, sollte er auf der Seite Runners Ihres Projekts unter **Settings following CI / CD** angezeigt werden .

### Shared Runners

enn Sie GitLab.com verwenden , können Sie die von GitLab Inc. bereitgestellten Shared Runners verwenden.
Hierbei handelt es sich um spezielle virtuelle Maschinen, die auf der GitLab-Infrastruktur ausgeführt werden und ein beliebiges Projekt erstellen können.
So aktivieren Sie die freigegebenen Runners Sie gehen müssen , um Ihr Projekt **Einstellungen ➔ CI / CD** und klicken Sie auf **Aktivieren Shared Runner**.
Lesen Sie mehr über Shared Runners.

#### Anzeigen des Status Ihrer Pipeline und Ihrer Aufträge

Nachdem Sie den Runner erfolgreich konfiguriert haben, sollte der Status Ihrer letzten Festschreibungsänderung von ausstehend zu ausgeführt , erfolgreich oder fehlgeschlagen angezeigt werden .
Sie können alle Pipelines anzeigen, indem Sie die Seite Pipelines in Ihrem Projekt aufrufen.

Sie können auch alle Jobs anzeigen, indem Sie die Seite **Pipelines ➔ Jobs** aufrufen.

Wenn Sie auf den Status eines Jobs klicken, können Sie das Protokoll dieses Jobs anzeigen. Dies ist wichtig, um zu diagnostizieren, warum ein Job fehlgeschlagen ist oder anders als erwartet reagiert hat.

Sie können den Status von Commits auch auf den verschiedenen Seiten in GitLab anzeigen, z. B. **Commits** und **Merge-Anforderungen** .

## Beispiele

In der [README-Datei](https://gitlab.com/help/ci/examples/README.md) mit [Beispielen](https://gitlab.com/help/ci/examples/README.md) finden Sie eine Liste von Beispielen, die GitLab CI in verschiedenen Sprachen verwenden.

