---
tags:
  - gitlab
  - gitlab-runner
---
# gitlab install runner

GitLab ist ein wunderbares Werkzeug für CI und Einsatz für Docker Bilder und Dienstleistungen. Es kombiniert Git mit Issue Tracking, ein Docker Registry und CI, um eine einheitliche Erfahrung zu bieten. Allerdings ist dieser Prozess nicht einzigartig für GitLab. Die in diesem Abschnitt erlernten Lektionen sollten unabhängig von den verwendeten Systemen gelten.

## Einrichten von GitLab für CI

GitLab CI ist kostenlos im Rahmen der Angebote bei GitLab.com erhältlich. Es ist auch möglich, für eine Unternehmenslizenz zu zahlen, die mehr Ressourcen zur Verfügung stellt. Die kostenlosen Community- und kommerziellen Firmenausgaben von GitLab können auch lokal installiert werden. In Kapitel 3, Cluster-Bausteine ​​- Registry, Overlay Networks und Shared Storage haben wir gezeigt, wie man GitLab installiert und wie man die Docker Registry aktiviert.

Ein Registry ist ein wichtiger Teil der Herstellung von CI Arbeit. Es bietet einen Platz für fertige Bilder für die Verwendung im Cluster gespeichert werden. Ein anderes Register könnte anstelle von GitLab verwendet werden. Beispielsweise kann eine Organisation den Docker Hub oder die sichere Registrierung im Docker Datacenter verwenden.

Ab GitLab 8.0 ist GitLab CI vollständig in das Hauptpaket integriert. Es gibt nichts anderes zu installieren oder zu aktivieren in GitLab, um diese Funktionen zur Verfügung zu stellen, mit einer Ausnahme. Die letzte Komponente, die benötigt wird, ist ein GitLab Runner.

## Installieren des GitLab Runner auf Ubuntu

Es ist die Aufgabe des GitLab Runners, die eigentlichen Bildaufbauten auszuführen. Jeder Build wird in die Warteschlange gestellt und verarbeitet. Es ist möglich, mehrere Läufer zu verwenden, um mehrere Builds parallel durchzuführen. Läufer können markiert werden, um bestimmte Aufgaben wie das Erstellen von Docker-Bildern oder das Erstellen einer Anwendung für Microsoft Windows auszuführen.

Die GitLab.com bietet einen gemeinsamen Läufer, der zum Erstellen von Images verwendet werden kann. Dies ist ideal für kleinere Projekte oder zum Testen. Größere Organisationen sollten sich für eine Unternehmenslizenz bewerben oder ihren eigenen Läufer lokal installieren. Ein lokaler Läufer kann auch erwünscht sein, wenn die Bildtests sensible Informationen enthalten oder wenn sie mit Ressourcen kommunizieren müssen, die nicht aus dem Internet zugänglich sind.

Die bevorzugte Methode, den GitLab Runner zu betreiben, besteht darin, es nativ auf Debian, Ubuntu, RHEL oder CentOS zu installieren. Allerdings gibt es ein offizielles Docker-Bild. Weitere Informationen zur Verwendung des Bildes finden Sie unter https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/blob/master/docs/install/docker.md.

Der erste Schritt bei der Installation eines Läufers, der Docker-Bilder erstellen wird, ist, dass Docker installiert ist. Der Build-Prozess muss in der Lage sein, Bilder zu ziehen, neue zu bauen und den fertigen Build in das Repository zu schieben.

GitLab bietet ein Skript, um ihr `apt `Repository zum Server hinzuzufügen. Laden Sie das Skript herunter, überprüfen Sie, ob es das tun wird, was erwartet wird, und führen Sie es dann aus. Aus Sicherheitsgründen nicht den Download direkt an `sudo `bash:
```
$ wget -O script.deb.sh https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh
$ sudo bash script.deb.sh
```
Das Skript wird den GPG-Schlüssel von GitLab dem apt-Schlüsselbund hinzufügen, das Repository hinzufügen und das `apt-get update` ausführen. Wenn es fertig ist, kann der Läufer installiert werden:
`$ apt-get install gitlab-ci-multi-runner`

Der Läufer muss nicht auf dem GitLab Server sein. Wenn load ein Problem ist, können Läufer auf mehreren Servern oder auf Cloud-Providern installiert werden. Es ist auch möglich, dass GitLab automatisch die Nummernläufer auf jedem von Docker Machine unterstützten Anbieter skaliert. Weitere Informationen finden Sie unter https://docs.gitlab.com/runner/install/autoscaling.html.

## Registrierung eines runners

Sobald der runner installiert ist, muss er bei GitLab registriert sein. Es gibt zwei Möglichkeiten, es zu tun. Die erste ist, sie für ein einziges Projekt zu registrieren. In diesem Fall wird der Läufer nur für die Builds des spezifischen Projekts laufen. Die andere Möglichkeit ist, sie global zu registrieren. Damit wird der runner für jedes Projekt in GitLab zur Verfügung gestellt. Es gibt Vor- und Nachteile für beide Ansätze, aber sie schließen sich nicht gegenseitig aus. Es ist möglich, einen oder mehrere globale runner für den allgemeinen Gebrauch und andere speziell für ein Projekt zu konfigurieren.

Ein Registrierungs-Token wird benötigt, bevor der runner mit GitLab registriert werden kann. Für projektspezifische runner gehen Sie auf die Seite **Runners** des Projekts. Im Abschnitt **Specific Runners** wird die URL und das Token angezeigt, die bei der Registrierung eines Läufers verwendet werden sollen:

Bei gemeinsamen Läufern befindet sich das Registrierungs-Token auf der Runners-Seite in  **Admin Area** (das Schraubenschlüssel-Symbol), wenn es als Administrator angemeldet ist. Die URL ist die gleiche wie für den projektspezifischen runner:

Es gibt mehrere Möglichkeiten, einen Läufer zu erstellen, mit dem Docker Image erstellt werden können. Das einfachste ist, den Shell-Executor zu benutzen. Es ist entworfen, um alle Shell-Befehle auszuführen, um das Projekt zu erstellen. Es funktioniert gut für fast jede Art von Projekt einschließlich `Docker`. Der `gitlab-runner` Benutzer muss in der `docker` Gruppe sein oder auf andere Weise Rechte zum Docker laufen lassen.

Mit dem folgenden Befehl wird der Läufer registriert. Ersetzen Sie $GITLAB_URL durch die CI-URL für die GitLab-Installation, die auf der Runners-Seite des Projekts aufgeführt ist. Wenn Sie GitLab.com verwenden, wäre die URL https://gitlab.com/ci. Das `$REGISTRATION_TOKEN` Token ist das Token aus den projektspezifischen oder globalen Runner-Seiten:

```sh
$ sudo gitlab-ci-multi-runner register -n \
--url $GITLAB_URL \
--registration-token $REGISTRATION_TOKEN \
--executor shell \
--description "Shell Runner"
```
Die andere Möglichkeit besteht darin, den Docker-in-Docker-Executor zu betreiben. Dieser Executor baut das Projekt in einem Docker-Container auf, der Docker und Docker Compose enthält:

```sh
$ sudo gitlab-ci-multi-runner register -n \
--url $GITLAB_URL \
--registration-token $REGISTRATION_TOKEN \
--executor docker \
--description "Docker Runner" \
--docker-image "docker:latest" \
--docker-privileged
```

Der Läufercontainer muss im privilegierten Modus laufen, um die Buildcontainer auszuführen.

## Aktivieren eines Docker-Repositorys

Die nächste Sache, die vor dem Beginn des Build-Prozesses eingerichtet wird, besteht darin, eine Docker-Registry zu aktivieren. GitLab bietet einen Registrierungsdienst an, der pro Projektbasis aktiviert werden kann. Wenn es bereits eingerichtet ist oder Sie GitLab.com verwenden, gehen Sie zu Projekt bearbeiten und stellen Sie sicher, dass das Feld neben Container Registry überprüft wird.

Die GitLab-Registry ist eine gute Wahl wegen der Integration mit GitLab und dem CI-Prozess. Andere Register können, falls gewünscht, verwendet werden. Wenn ein anderes Registry verwendet wird, richte ein Konto für den Läufer ein, der Rechte hat, neue Bilder in die Registry zu schieben.

## Hinzufügen des Projekts zum Repository

Schließlich muss das Projekt dem Git-Repository in GitLab hinzugefügt werden. Dies geschieht mit Standard `git` Befehlen. Im Folgenden ist das grundlegende Skript. Wenn die Docker-Konfiguration bereits in Git ist, werden nur die letzten beiden Befehle benötigt. Achten Sie darauf, die URL der Fernbedienung zu ändern, die mit Ihrem Repository übereinstimmt:
```
$ git init
$ git add .
$ git commit -m 'initial commit'
$ git remote add origin http://gitlab.example.com/project/repo
$ git push -u origin master
```

Die folgenden Beispiele verwenden ein sehr einfaches Docker-Bild. Es ist ein Nginx Webserver mit einer einzigen Seite Website. Im Folgenden ist die Dockerfile:

```
FROM nginx:1.10 
COPY index.html /usr/share/nginx/html/
```

###### Hinweis

Vollständige Beispiele für das in diesem Abschnitt beschriebene CI-Verfahren sind auf GitLab verfügbar. Jeder Zweig ist ein komplettes Beispiel für jeden Schritt. Laden Sie die Beispiele aus https://gitlab.com/perlstalker/ci-demo herunter.

## Erstellen von .gitlab-ci.yml

CI wird über eine Datei mit dem Namen `gitlab-ci.yml` im Stammverzeichnis des Repositorys konfiguriert. Diese YAML-Datei definiert die Build-Stufen, Variablen und Jobs, die zum Erstellen, Testen und Bereitstellen des Bildes benötigt werden. Es können beliebig viele Aufträge definiert und Aufträge parallel ausgeführt werden. Ausführliche Informationen zu den Optionen, die in `.gitlab-ci.yml` verfügbar sind, finden Sie unter https://docs.gitlab.com/ce/ci/yaml/README.html.

Das folgende Beispiel `.gitlab-ci.yml` Beispiel ist eine einfache Konfiguration, die das Bild aufbauen und in die Registry schieben kann. In diesem Beispiel wird der Docker-in-Docker-Executor verwendet. Die ersten drei Zeilen können bei der Verwendung des Shell-Executors mit einem lokalen Läufer weggelassen werden:

```yml
image: docker:latest 
services: 
  - docker:dind 
 
stages: 
  - build 
  - test 
  - release 
  - deploy 
 
variables: 
  DOCKER_CI_IMAGE: registry.gitlab.com/perlstalker/ci-demo:$CI_BUILD_REF_NAME 
 
before_script: 
  - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN registry.gitlab.com 
 
build-image: 
  stage: build 
  tags: 
    - doc
```

Die `stage` Option definiert die Liste der Stufen, mit denen Aufträge konfiguriert werden können. Eine beliebige Anzahl von Stufen kann so definiert werden, dass sie den Anforderungen des Projekts entspricht. Die vier hier aufgeführten, `build`, `test`, `release` und `deploy`, sind typisch. Jede Stufe wird in der Reihenfolge aufgeführt, in der sie aufgeführt ist. Jobs für die `build ` stage werden zuerst ausgeführt, dann `test` und so weiter. Wenn es in jeder Phase mehrere Aufträge gibt, werden sie parallel ausgeführt.

Mit der `variablen` Option können Sie Variablen für den Rest des Buildprozesses definieren. Im Beispiel wird eine Variable namens `$DOCKER_CI_IMAGE` auf den Namen des Bildes gesetzt, das gebaut wird. Der Bildname enthält die Variable `$CI_BUILD_REF_NAME`, die eine eingebaute Variable ist, die auf den Zweig- oder Tag-Namen gesetzt ist. Der Wert von `$DOCKER_CI_IMAGE` sollte für das jeweilige Projekt geändert werden.

Die Verwendung von `$CI_BUILD_REF_NAME` kann sehr hilfreich sein, besonders beim Testen eines neuen Bildes oder beim Vorbereiten einer neuen Version. Während des Testens bedeutet das Erstellen eines neuen Zweigs, dass das erzeugte Bild mit dem Zweignamen versehen wird. Zum Beispiel, wenn ein Zweig namens `foo` erstellt wird, wird das neue Iamge`image:foo`.

###### Hinweis

GitLab bietet eine Reihe nützlicher Variablen, die in der CI-Konfiguration verwendet werden können. Eine vollständige Liste finden Sie unter https://docs.gitlab.com/ce/ci/variables/README.html.

Git-Tags erzeugen auch ein Bild mit diesem Tag. Wenn zum Beispiel das `1.1`-Tag zu einem Commit hinzugefügt wird, wird ein neues Bild namens `image:1.1` erstellt. Dies macht es einfach, versionierte Bilder für die Bereitstellung zu erstellen. Denken Sie daran, dass mit `image:version` statt `image:latest` macht es einfacher, genau zu verfolgen, was auf dem Cluster läuft und macht es einfach, zurück zu der vorherigen Bildversion im Falle eines Fehlers zurückzukehren.

Die Option `before_script` enthält eine Liste der Befehle, die ausgeführt werden sollen, um die Umgebung vor jedem Job vorzubereiten. In diesem Beispiel meldet es sich in die GitLab Docker-Registrierung mit einem speziellen CI-Account und baut Token. Das Build-Token gibt dem Builder vorübergehend die Rechte des Benutzers, der den Build ausgelöst hat. Wenn eine andere Registrierung verwendet wird, werden stattdessen die Anmeldeinformationen für diese Registrierung verwendet.

Der letzte Block ist ein Job namens `build-image`. Die `stage` option gibt die Bühne, auf der dieser Job läuft. In diesem Fall läuft der `build-image` Job im `build` stage. Wenn mehrere Jobs in derselben Stufe definiert sind, werden sie parallel ausgeführt.

Jobs können eine Liste von `tags` definieren, die verwendet werden, um zu definieren, welcher runner verwendet wird. Im vorigen Beispiel wird das Image von einem Läufer mit dem Docker-Tag gebaut. Wenn mehrere Tags gesetzt sind, muss der runner mit allen zusammenhängen. Dies macht es möglich, runner zu schaffen, die bestimmte Arten von Projekten und nicht andere bauen können.

Schließlich ist die `script` option eine Liste von Befehlen, die ausgeführt werden sollen, um das Projekt zu erstellen. Jeder Befehl muss erfolgreich sein, bevor der nächste Befehl ausgeführt wird. Wenn einer von ihnen scheitert, bricht der Build auf und es wird als fehlgeschlagen markiert. Die Befehle können regelmäßige Systembefehle wie im Beispiel oder Skripte aus dem Repository sein.

In diesem Beispiel baut der Läufer zuerst das Bild auf. Das `--pull` Flag wird mit dem `docker build` verwendet, um sicherzustellen, dass die neueste Kopie des Basisbildes verwendet wird, um das neue Bild zu erstellen. Wenn der Build gelingt, wird das neue Image wieder in das Repository geschoben. Eine Liste der verfügbaren Images finden Sie im Registry-Bereich auf der Projektseite:
![gitlap-container-reg](https://www.packtpub.com/graphics/9781787122123/graphics/image_09_003.jpg)

Der Status des aktuellen Builds sowie der vorherigen Builds steht im **Pipelines** Bereich des Projekts zur Verfügung. Klicken Sie auf irgendeinen Build, um zu bohren und den Status jeder Build-Bühne zu sehen. Wenn Sie auf die Bühne klicken, wird das Build-Log für die Bühne angezeigt. Irgendwelche Fehler, die durch einen fehlgeschlagenen Build erzeugt werden, erscheinen dort:
![gitlap-build-status](https://www.packtpub.com/graphics/9781787122123/graphics/image_09_004.jpg)

## Testen des Bildes

Einer der Gründe, dass CI ist so beliebt bei Software-Entwickler ist, dass es eine Möglichkeit, automatisch eine Test-Suite gegen den neuen Build, um sicherzustellen, dass es richtig funktioniert. Die Tests bieten ein Maß an Sicherheit zu wissen, dass, was auch immer Änderungen der Entwickler gemacht, diese nicht brechen die Funktionalität der Software. Dies ist genauso wertvoll für Docker-Imges, die auf potenziell Hunderte oder Tausende von Servern eingesetzt werden sollen.

Es gibt mehrere Möglichkeiten, die Tests durchzuführen:

* Die erste ist, die Tests in das Git-Repository aufzunehmen, aber nicht das Bild hinzuzufügen. Dies hält das Bild klein, erfordert aber, dass alle Werkzeuge, die zum Testen benötigt werden, auf dem Läufer installiert sind. Einige Läufer, wie der Docker-in-Docker-Executor, dürfen nicht die Werkzeuge enthalten, die zum Ausführen der Tests benötigt werden.

* Der zweite Weg ist, die Tests im Docker-Bild aufzunehmen. Die Tests können dann innerhalb des Containers mit entweder Docker Run oder Docker ausgeführt werden. Dies führt zu einem größeren Bild, sondern sorgt dafür, dass alle Werkzeuge zum Testen des Bildes zur Verfügung stehen. Es macht es auch möglich, die Tests auf einem laufenden Bild jederzeit auszuführen. Dies könnte nützlich sein, wenn es Probleme gibt, die nur auftauchen, wenn das Bild in der Produktion läuft.

* Eine dritte Möglichkeit besteht darin, ein separates Bild zu erstellen, das alle zur Validierung des Bildes benötigten Tests enthält. Dieses Bild kann während der Testphase von GitLab heruntergeladen und ausgeführt werden, um das Anwendungsbild zu testen. Das Schöne an diesem Ansatz ist, dass die Tests und Testbild unabhängig aktualisiert werden können, wie neue Testfälle kommen.

Werfen wir einen Blick auf die sehr einfache Webseite, die in diesem Beispiel verwendet wird:

```
<html> 
  <head><title>Docker Orchestration</title></head> 
  <body> 
    <h1>Docker Orchestration is fun!</h1> 
    <p>Automate all the things!</p> 
  </body> 
</html> 
```

Der Chef hat beschlossen, dass die Web-Seite muss immer verkünden, wie viel Spaß Docker Orchestrierung ist. Zu diesem Zweck wird ein Test erstellt, um sicherzustellen, dass es Spaß macht. In diesem Beispiel wird es in einem `tests` verzeichnis platziert und heißt `run-tests.sh`:

```
#!/bin/sh
curl -s http://localhost | grep 'fun!</h1>' > /dev/null
```

Dieser Test ist sehr einfach. Es greift auf die Webseite zu und prüft, ob `fun!` Ist das letzte in der Überschrift. Um die Dinge zu vereinfachen, wird der Test dem Bild hinzugefügt. Die aktualisierte `Dockerfile` ist hier aufgelistet. Es macht zwei Dinge. Zuerst fügt es curl hinzu, das für den Test benötigt wird. Zweitens fügt es die Tests zum `/tests` Verzeichnis im Image hinzu:

```
FROM nginx:1.11 
RUN apt-get update && apt-get install -y curl && apt-get clean 
COPY index.html /usr/share/nginx/html/ 
COPY tests /tests 
``` 

Um den Test auszuführen, muss ein neuer Job zu `.gitlab-ci.yml` hinzugefügt werden. Das folgende Snippet zeigt den Job. Sobald `.gitlab-ci.yml` im Repository aktualisiert wird, werden zukünftige Commits die Tests ausführen:

```
test-image: 
  stage: test 
  tags: 
    - docker 
  script: 
    - docker pull $DOCKER_CI_IMAGE 
    - docker run -d -P --name $CI_BUILD_ID $DOCKER_CI_IMAGE 
    - docker run --rm --network container:$CI_BUILD_ID --name $CI_BUILD_ID-test 
    --entrypoint /tests/run-tests.sh $DOCKER_CI_IMAGE 
    - docker stop $CI_BUILD_ID; docker rm -f $CI_BUILD_ID 
```

Die `stage` ist so eingestellt, dass der Job in der `test` läuft. Dieser Job wird nicht ausgeführt, wenn die `build` Stufe fehlschlägt. Es enthält auch die `Docker`-Tag, so dass GitLab wird ein Läufer, der auch mit Docker getaggt wird.

Der Skriptabschnitt listet die Befehle auf, die ausgeführt werden, um das Bild zu testen. Die ersten beiden Befehle laden das Bild herunter und führen es im Hintergrund aus. Der Containername wird auf $ CI_BUILD_ID gesetzt, welches die eindeutige Build-ID ist, die dem Build von GitLab zugewiesen wurde. Es ist wichtig, den Namen zu setzen, weil es benötigt wird, wenn der Test ausgeführt wird.

Der dritte Eintrag im Scriptblock führt den eigentlichen Test aus. Es startet einen neuen Container, der das Netzwerk des Zielcontainers teilt. Dies ermöglicht es den Tests, sich mit dem Webserver auf localhost zu verbinden. Der Testcontainer führt das Test-Skript aus, das früher gezeigt wurde. Das Grep im Skript wird fehlschlagen, wenn das Muster nicht übereinstimmt und der Test einen Nicht-Null-Exit-Code zurückgibt, den GitLab als Fehler interpretiert. Wenn das Muster im Skript gefunden wird, ist der Test erfolgreich und der Prozess  geht weiter.

Die letzte Zeile stoppt den Zielcontainer und entfernt sie. Der Container, der die Tests ausführt, wird automatisch entfernt, wenn der Test durchgeführt wird, weil --rm verwendet wurde, als es gestartet wurde.
Bereinigungsarbeiten

Der vorangehende Testjob funktioniert super, wenn die Tests erfolgreich sind, aber wenn es fehlschlägt, kann er den Testcontainer laufen lassen. Dies ist kein Problem bei der Verwendung der Shared Runner auf GitLab.com, weil eine neue VM für jeden Test gestartet und sofort entfernt wird, nachdem der Build-Prozess abgeschlossen ist. Allerdings könnte es ein Problem sein, wenn man einen lokalen Läufer benutzt. In diesem Fall kann nach dem Test ein Job für die Bereinigung benötigt werden.

Um zu starten, erstellen Sie eine neue `cleanup_test` Stage und ein Job namens `cleanup_tests` ist definiert, um in der `cleanup_test` Stufe zu  laufen. Normalerweise, wenn ein Job in der `test` stage ausfällt, würde nichts danach laufen. Der Cleanup `test` Job ruft um, dass mit der `when` Option. Wenn man es immer aufsetzt, läuft der Job, egal was in der Testphase passiert. Wenn die Option auch auf `on_success` oder `on_failure` gesetzt werden kann, um den Job nur dann auszuführen, wenn die Aufträge erfolgreich sind oder fehlschlagen. Es gibt eine vierte Option, `manuell`, die später im Kapitel behandelt wird. Das folgende Snippet zeigt, wie ein Job erstellt wird, der nach dem CI-Prozess abschließt:

```
stages: 
  - build 
  - test 
  - cleanup_test 
  - release 
  - deploy 
... 
test-image: 
  stage: test 
  tags: 
    - docker 
  script: 
    - docker pull $DOCKER_CI_IMAGE 
    - docker run -d -P --name $CI_BUILD_ID $DOCKER_CI_IMAGE 
    - sh tests/run-tests.sh $CI_BUILD_ID 
 
cleanup_tests: 
  stage: cleanup_test 
  script: 
    - docker stop $CI_BUILD_ID; docker rm -f $CI_BUILD_ID 
  when: always 
```

### Das Bild freigeben

Der `build-image` Job hat jedes Mal ein aktualisiertes Image erstellt, wenn Änderungen an das Git-Repository gedrückt werden. Schauen wir uns genau an, was entsteht. Der Befehl `docker build` im Beispiel erstellt ein Bild mit dem Namen `registry.gitlab.com/perlstalker/ci-demo:$CI_BUILD_REF_NAME`. Die Variable `$CI_BUILD_REF_NAME` ist der Name des Zweigs oder Tags. Dies bedeutet, dass an diesem Punkt das einzige Image, das erstellt wird, `registry.gitlab.com/perlstalker/ci-demo:master` ist. Es gibt keine: neues Bild. Wenn jemand zu versuchen und zu laufen Docker laufen `registry.gitlab.com/perlstalker/ci-demo`, würde der Befehl fehlschlagen, weil das Bild mit seinem implizierten neuesten Tag nicht vorhanden ist.

Die Tatsache, dass GitLab das Bild nicht automatisch mit dem `latest` Tag markiert, ist ein Feature. Die Idee ist für den CI-Plan in `.gitlab-ci.yml`, um das Iamge explizit zu markieren. Dies geschieht durch die Schaffung eines neuen Jobs. Ein Snippet von `.gitlab-ci.yml` wird hier gezeigt:

```
variables: 
  DOCKER_CI_IMAGE: registry.gitlab.com/perlstalker/ci-demo:$CI_BUILD_REF_NAME 
  DOCKER_RELEASE_IMAGE: registry.gitlab.com/perlstalker/ci-demo:latest 
... 
release-image: 
  stage: release 
  tags: 
    - docker 
  script: 
    - docker pull $DOCKER_CI_IMAGE 
    - docker tag $DOCKER_CI_IMAGE $DOCKER_RELEASE_IMAGE 
    - docker push $DOCKER_RELEASE_IMAGE 
  only: 
    - master
```

Der erste Unterschied ist, dass eine neue Variable, `$DOCKER_RELEASE_IMAGE`, mit dem Namen des Release-Bildes hinzugefügt wurde.
###### Hinweis

Der image Tag ist `latest`.

Wie immer ist die Arbeit im `script` block ausgeführt. Der erste Schritt ist, die neueste Kopie des Iamges für diesen Zweig zu Downloaden(pull). Als nächstes, `docker` tag-Tags, die Iamge mit dem Release-Image Name. Speziell wird der `latest` Tag gegeben. Schließlich schiebt der Docker `push` Befehl das neu markierte Image wieder in die Registry.

Der `only` Block ist wichtig. Es sagt GitLab, dass dieser Job nur beim Aufbau eines der aufgeführten Zweige ausgeführt werden muss. In diesem Fall, `master´. Ohne den einzigen Block würde jedes neu gebildete Bild aus irgendeinem Zweig das `latest` Image ersetzen.

Das Schöne an dieser Konfiguration ist, dass neue Bilder in verschiedenen Zweigen bearbeitet werden können und Bilder für diese Zweige erstellt werden. Diese Bilder können einzeln ausgeführt, getestet und debugged werden, ohne das freigegebene Bild zu verändern. Wenn die Änderungen bereit sind, kann der Entwicklungszweig wieder in den Masterzweig zusammengeführt und ein neues neues Bild erstellt werden.
### Bereitstellung des Bildes

Nehmen wir einen Moment und sehen, wo die Dinge stehen. An diesem Punkt werden neue Docker-Bilder für ein Projekt automatisch erstellt, sobald Änderungen in das Git-Repository in GitLab geschoben werden. Eine Test-Suite wird gegen das neue Bild ausgeführt, um sicherzustellen, dass es wie erwartet funktionieren wird. Wenn die Änderungen an den `master` zweig(branch) gedrückt werden, wird ein neues Release-Image erstellt. Das einzige, was noch zu tun ist, ist, das Image in den Cluster einzusetzen.

Wie das Bild eingesetzt wird, hängt von der gewählten Orchestrierungs-Suite ab. In den meisten Fällen ist alles, was benötigt wird, um den Befehl auszuführen, der eine rollende Aktualisierung des Dienstes durchführt. Ein Snippet eines automatisierten Einsatzauftrags ist wie folgt:

```
deploy-to-dev: 
  stage: deploy 
  tags: 
    - docker 
  environment: development 
  script: 
    - ./deploy.sh $DOCKER_CI_IMAGE development 
  only: 
    - master 
```

Hier sind ein paar Dinge zu beachten. Zuerst wird die `stage` eingestellt auf `deploy`. Der Job wird nur ausgeführt, wenn die `build`-, `test`- und `release`-Stufen erfolgreich abgeschlossen sind. `ènvironment` ist optional. Die Seite `environment` im Abschnitt **Pipelines** im Projekt macht es einfach, Bereitstellungen in verschiedenen Umgebungen zu verfolgen:

![gutlab-ci-environments](https://www.packtpub.com/graphics/9781787122123/graphics
/image_09_005.jpg)

Das `deploy.sh` Skript im Snippet ist ein Platzhalter. Das Skript kann enthalten, was benötigt wird, um das Bild in Ihrer Umgebung bereitzustellen. Das Skript konnte auch direkt in den `script` block aufgenommen werden. 

###### Tip

Verwenden Sie das Stichwort `tags` anstelle eines Zweignamens im einzigen Abschnitt für Bereitstellungen. Dies begrenzt die Bereitstellung nur auf `only`, die markiert wurden. Es wird sichergestellt, dass alle Bilder, die bereitgestellt werden, das Tag verwenden. Es wird leichter zu sagen, welche Version des Bildes ein Container ausgeführt wird, basierend auf dem Tag auf dem Image.

Manuelles Bereitstellen des Bildes

Die automatische Bereitstellung eines Bildes zum Cluster ist spannend. Vielleicht zu aufregend. Nicht jeder möchte, dass Dienste automatisch aktualisiert werden. Das bedeutet nicht, dass GitLab nicht für Bereitstellungen verwendet werden kann. GitLab bietet eine Möglichkeit, einen Job manuell zu konfigurieren. Der Build-Prozess wird wie es normalerweise funktioniert, aber wenn es zu einem manuellen Job kommt, wird es warten, bis jemand GitLab sagt, dass es okay ist, um fortzufahren. Dies sorgt für die Konsistenz einer automatisierten Bereitstellung, während ein Mensch immer noch entscheiden kann, ob die Bereitstellung geschieht oder nicht.

Das folgende `.gitlab-ci.yml` Snippet zeigt, wie man einen Job mit einem `manual` Trigger erstellt. Es ist grundsätzlich das gleiche wie die vorangegangene automatisierte Bereitstellung. Der einzige funktionale änderung ist das ``when` tag:
```
deploy-to-prod: 
  stage: deploy 
  tags: 
    - docker 
  environment: production 
  script: 
    - ./deploy.sh $DOCKER_CI_IMAGE production 
  only: 
    - master 
  when: manual 
```

Wenn Sie das Handbuch einstellen, wird der Build- und Bereitstellungsprozess beendet, bis jemand zu **Pipelines** geht, klickt auf den Build und klickt auf die Bereitstellung. Der folgende Screenshot zeigt die Build-Pipeline:

![gitlab-piplines](https://www.packtpub.com/graphics/9781787122123/graphics/image_09_006.jpg)

###### Hinweis

Beachten Sie, dass alle Aufträge gelungen sind. Nur der `deploy-to-prod` job hat nicht. Die Dreieck-Play-Taste zeigt an, dass der `deploy-to-prod` auf eine `manual` Aktion wartet. Wenn diese Schaltfläche gedrückt wird, beginnt der Einsatz.


Eine beliebige Anzahl von Bereitstellungsaufträgen kann definiert werden, um eine Vielzahl von Anforderungen zu erfüllen. Beispielsweise könnte ein vollautomatischer Job neue Bilder in eine Entwicklungsumgebung implementieren. Ein manueller Job könnte dann den Einsatz in eine QA- oder Kanarische Umgebung auslösen. Schließlich könnte ein anderer manueller Job das Bild vollständig in die Produktion bringen. Dies könnte mit parallelen Jobs wie hier gezeigt oder als separate Build-Stufen erfolgen. Die Verwendung von separaten Stufen stellt sicher, dass jeder vorherige Einsatz abgeschlossen ist, bevor er schließlich in die Produktion geschoben wird.

### Bereitstellung nach Kubernetes

Die vorangehenden Beispiele verwendeten ein Dummy-Deployment-Skript. Lassen Sie uns einen Blick darauf werfen, wie man GitLab CI nutzen könnte, um ein neues Image nach Kubernetes einzusetzen.

Das folgende .gitlab-ci.yml-Snippet macht ein paar Annahmen:

* Zuerst wird `kubectl` auf der Läufer-Maschine installiert und der `gitlab-runner` Benutzer hat Berechtigungen, Änderungen am Kubernetes-Cluster vorzunehmen

* Zweitens wurde bereits eine Bereitstellung erstellt, die durch den Buildprozess aktualisiert wird:

```
    variables: 
      DOCKER_CI_IMAGE: registry.gitlab.com/perlstalker
        /ci-demo:$CI_BUILD_REF_NAME 
      DOCKER_RELEASE_IMAGE: registry.gitlab.com/perlstalker
        /ci-demo:latest 
      DEPLOYMENT: ci-demo 
      CONTAINER: web 
    ... 
    deploy-to-kube: 
      stage: deploy 
      tags: 
        - kubectl 
      environment: production 
      script: 
        - kubectl set image deployment/$DEPLOYMENT     
        $CONTAINER=$DOCKER_CI_IMAGE 
      only: 
        - tags 
      when: manual 
```

Zwei neue Variablen, `DEPLOYMENT` und `CONTAINER` werden der Konfiguration hinzugefügt. Sie enthalten den Namen der Bereitstellung und den Container innerhalb dieser Bereitstellung, die aktualisiert werden soll.

Als Beispiel wird das `kubectl`-Tag auf den Job gesetzt. Die Erwartung ist, dass ein Läufer mit dem `kubectl`-Tag zur Verfügung steht, der kubectl konfiguriert hat, um den Cluster zu verwalten. Dies macht es einfach, einen Läufer zu erstellen, der nur für Bereitstellungen verwendet wird und nicht für andere Build-Jobs verwendet wird.

Das Skript enthält nur den Befehl, um ein Bild in der Bereitstellung zu aktualisieren. Es ist nicht nötig, mit dem Bild im Läufer etwas zu tun. Die Kubernetes-Knoten, die die Bereitstellung ausführen, führen das Update auf dem Container basierend auf der Bereitstellungsstrategie des Pods durch.

Schließlich sagt der `only` Block GitLab, um diesen Job nur auszuführen, wenn der Build einen `git`-Tag-Satz hat. Dadurch wird sichergestellt, dass das gebildete Bild einen anderen Tag hat als das, was vorher ausgeführt wurde. Wenn der Bildname genauso ist wie das, was die Bereitstellung derzeit verwendet, wird Kubernetes nicht wissen, dass das Bild aktualisiert werden muss.

### DAB-Dateien erstellen

Docker Swarm bewegt sich zu **Distributed Application Bundles (DAB)**, um Stacks für die Bereitstellung zu definieren. Mit GitLab CI können Sie automatisch DAB-Dateien für ein Projekt erstellen. Das folgende Snippet geht davon aus, dass sich `docker-compose.yml` im Stammverzeichnis des Projekts befindet:

```
build-dab: 
  stage: release 
  tags: 
    - docker 
  script: 
    - docker-compose bundle 
  artifacts: 
    paths: 
      - *.dab 
    name: "${PROJECT_NAME}-${CI_BUILD_REF_NAME}" 
```

Der `script`-Block lädt das `docker-compose` -Bündel, das die `.dab`-Datei erzeugt. Zusätzliche Optionen können bei Bedarf dem Befehl hinzugefügt werden. Der `artifacts` block erzählt GitLab, dass der Buildprozess Dateien erzeugt, die wichtig sind. In diesem Beispiel werden die `.dab`-Dateien gespeichert, wenn der Build erfolgreich ist. Alle Dateien, die den Einträgen in der Pfadliste entsprechen, werden der Artefaktarchivdatei hinzugefügt.

Schließlich gibt die Namensoption den Namen des Artefaktarchivs an. In diesem Beispiel ist es der Name des Archivs und der Build-Referenzname, der durch einen Bindestrich getrennt ist. Zum Beispiel, wenn das Projekt heißt ci-demo und der Master-Zweig gebaut wird, wäre der Name ci-Demo-Master. Die Artefaktarchivdatei heißt ci-demo-master.zip.