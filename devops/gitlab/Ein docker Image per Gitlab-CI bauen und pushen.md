ch habe ein nächtliches Backup einer MySQL-Datenbank und mein Ziel ist es, nachdem das MySQL-Backup in ein Repo in meinem Gitlab gepusht wurde ein **docker** -Images zu erstellen, welches dieses Backup gleich einbindet und anschließend in die Gitlab-eigene Registry pusht.  
Damit bin ich in der Lage auf jedem beliebigen **docker** -Host einen **docker** MySQL-Container zu starten, welcher direkt durch ein entsprechendes Webfrontend angesprochen werden kann.
##### Das MySQL-Backup

Ich erstelle per `cron` nächtlich ein MySQL-Backup, welches nach dem erstellen, neben dem regulären georedundanten Backup, in ein Repo in meinem persönlichen Gitlab gepusht wird. Dieses Repo hat dabei den Namen `techgoat.net`.  
![](https://techgoat.net/images/238.png)

Dieses Backup soll nun in ein entsprechendes **docker** eingebunden werden.
##### Der MySQL-Container

In einem anderen Repo habe ich nun das entsprechende `Dockerfile` liegen, aus welchem das Image erstellt werden soll. Dafür verwende ich die neuste Version des offiziellen [mysql **docker** Images](https://hub.docker.com/_/mysql) als Basis.

Das `Dockerfile`:

```
FROM mysql:latest
ENV MYSQL_ROOT_PASSWORD 123
ENV MYSQL_DATABASE techgoat_db
ENV MYSQL_USER dieOberziege
ENV MYSQL_PASSWORD geheimesZiegenbockPasswort
ADD techgoat_db.sql /docker-entrypoint-initdb.d
EXPOSE 3306
```

Wie man sieht wird beim bauen des Images gleich die richtige Datenbank mit entsprechendem Benutzer angelegt und anschließend sorgt der Befehl  
`ADD techgoat_db.sql /docker-entrypoint-initdb.d` dafür, dass der Dump auch direkt beim bauen in die Datenbank eingespielt wird.
### Automatisiert das Image bauen und pushen

Der Ablauf soll wie folgt sein:

- das nächtliche Backup wird in das Gitlab gepusht
- die Gitlab-CI:
    - klont sich das Repo mit dem `Dockerfile` des MySQL-Containers
    - baut das **docker** Image
    - loggt sich in die entsprechende Registry ein
    - pusht das Image in die Registry
##### Bereitstellungstoken

Damit der `gitlab-runner` das `Dockerfile` auch per `git` klonen kann benötigt dieser Leserechte auf das Repo. Hierfür erzeugen und verwenden wir einen entsprechenden [deploy-token (Bereitstellungstoken),](https://docs.gitlab.com/ee/user/project/deploy_tokens/) welcher Leserechte auf das Repository und/oder die Registry erlaubt.

Das findet man unter: **Repo** —> **Einstellungen/Settings** —> **Repository** —> **Deploy Token/Bereitstellungstoken**

Hier vergibt man einen Namen, setzt ein Ablaufdatum und die „Scopes“ (auf was darf zugegriffen werden) und klickt den markant grünen Knopf **Bereitstellungstoken erstellen**

Nun erscheint direkt oberhalb des Punktes **Bereitstellungstoken hinzufügen** ein neuer Kasten **Dein neuer Bereitstellungstoken**.  
Da findet sich im ersten Feld der „Benutzername“ und darunter der Token. Diesen sollte man sich sicher notieren, da dieser später nicht noch einmal angezeigt wird.

Wie viele Bereitstellungstoken ein Repo bereits in Verwendung hat wird etwas weiter unten beim Punkt **Aktive Bereitstellungstoken** angezeigt.

![](https://techgoat.net/images/239.png)
##### Registry Zugangsdaten geschützt ablegen

Um den `gitlab-runner` zu gestatten ein Image in eine Registry der Wahl zu pushen muss dieser sich vorher in diese einloggen. Damit die entsprechenden Zugangsdaten nicht als Klartext in der `.gitlab-ci.yml` eintragen zu müssen, kann man diese auch als geschützte Variablen in Gitlab hinterlegen.

Diese kann man unter: **Repo** —> **Einstellungen/Settings** —> **CI/CD** —> **Variables** eintragen.

Hier setzt man als `Key` den Namen der Variable und in `Value` den Wert. Um uns in unsere **docker** Registry einloggen zu können benötigen wird drei Variablen:

- `REGISTRY_USER` —> Der Benutzername für die Registry
- `REGISTRY_PASSWORD` —> Das entsprechende Passwort für die Registry
- `REGISTRY_NAME` —> Name/URL der Registry

Diese Variablen kann noch auf `geschützt` stellen, was bedeutet das der Inhalt der Variablen beim Verwenden nicht angezeigt wird und zusätzlich auf `maskiert` stellen. Dies verhindert das der Inhalt der Variable in den Logs auftaucht.

Welche Vorrausetzungen notwendig sind um eine Variable zu maskieren findet man in der [Dokumentation.](https://gitlab.katzen-in-ketchup-tauchen.de/help/ci/variables/README#masked-variables)

![](https://techgoat.net/images/241.png)

##### .gitlab-ci.yml

Nun sollten wir alles zusammen haben um einen entsprechenden Automatismus per Gitlab-CI zu erstellen. Mein Aufbau ist hierbei wie folgt:

- Es gibt drei Stages
- Stage 1 (`build image`):
    - Dockerfile per `git` + `Bereitstellungstoken` klonen
    - den MySQL-Dump zum `Dockerfile` verschieben
    - das **docker** Image erstellen
- Stage 2 (`push image`):
    - in die **docker** Registry einloggen
    - das Image in die Registry pushen
- Stage 3 (`cleanup`):
    - enfernen des erstellten Images vom Gitlab-Host

`.gitlab-ci.yml`:

```yaml
stages:
  - git_checkout
  - docker_push
  - image_cleanup

build image:
  stage: git_checkout
  script:
    - git clone https://gitlab+deploy-token-4:iaymFJ5UqSDRcUPw18xd@gitlab.domain.tld/docker/techgoat_mysql.git
    - mv techgoat_db.sql techgoat_mysql/
    - docker build -t registry.domain.tld/privat/techgoat.net/techgoat_mysql ./techgoat_mysql

push image:
  stage: docker_push
  script:
    - echo "${REGISTRY_PASSWORD}" | docker login --username "${REGISTRY_USER}" --password-stdin "${REGISTRY_NAME}"
    - docker push registry.domain.tld/privat/techgoat.net/techgoat_mysql

cleanup:
  stage: image_cleanup
  script:
    - docker image rm registry.domain.tld/privat/techgoat.net/techgoat_mysql
```

**Stage 1 (`build image`)**:

Der Aufbau des `git clone` Befehls unter Verwendung des vorhin erstellten Bereitstellungstoken sieht folgendermaßen aus:

```sh
git clone https://<Bereitstellungstoken-Benutzername>:<Bereitstellungstoken-Passwort>@<Gitlab-URL>/Repo/Name.git
```

Nach dem klonen wird der Dump per `mv` in den Ornder des eben geklonten Repos verschoben und anschließend wird das Image per `docker build` erstellt.

**Stage 2 (`push image`)**:

per `echo "${REGISTRY_PASSWORD}" | docker login --username "${REGISTRY_USER}" --password-stdin "${REGISTRY_NAME}"` loggt sich der `gitlab-runner` in die Registry ein. Wie man sieht kommen hier die vorher erstellten geschützten Variablen ins Spiel.  
Weitere Möglichkeiten zur Verwaltung der Zugangsdaten findet man in der entsprechenden [docker login Dokumentation.](https://docs.docker.com/engine/reference/commandline/login/)  
Nach dem Login erfolgt der per `docker push ...` der Push des neuen Images in die Registry.

**Stage 3 (`cleanup`)**:

Da die `gitlab-runner` auf dem gleichen System wie das Gitlab selber laufen, möchte ich es möglichst sauber halten und deshalb entfernnt die letzte Stage das frisch erstellte Image nach dem Push in die Registry vom lokalen System.
##### Das Ergebnis

Wenn alles klappt sollten wir eine entsprechende Pipeline haben, welche alle Aufgaben erfüllt:

![](https://techgoat.net/images/242.png)

und in der Registry sollte stets die aktuellste Version verfügbar sein:

![](https://techgoat.net/images/243.png)