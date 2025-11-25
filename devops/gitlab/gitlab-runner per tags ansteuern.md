##### Vorbereitungen

Ich verwenden hierbei einen gitlab-runner mit **Shell executor** und einen weiteren mit **Docker executor**.  
Hierbei ist vor allem darauf zu achten, dass entweder direkt bei der Einrichtung des gitlab-runners oder im Nachhinein für jeden der runner ein oder mehrere _tags_ gesetzt werden.  
Ich verwende hierbei das tag `docker` für den **Docker executor** und das tag `shell` für den **Shell executor**.  
![](https://techgoat.net/images/267.png)

##### Wozu braucht man das überhaupt?

Mit dieser Unterteilung ist es möglich innerhalb einer Gitlab-CI Pipeline stages auf den entsprechenden gitlab-runnern auszuführen und somit sowohl Shell Befehle auszuführen als auch in docker Images zu arbeiten.

einfachstes Beispiel:  
Innerhalb einer Pipeline ist es möglich sich per **Shell executor** und **Dockerfile** ein entsprechendes docker Image zu erstellen und dieses in der nächsten stage per **Docker executor** für Tests zu verwenden.

##### tags in der .gitlab-ci.yml

Die offizielle Dokumentation zur Verwendung von tags in der gitlab-CI findet sich hier: [Using tags](https://docs.gitlab.com/ee/ci/runners/#using-tags)

Das von mir angesprochene Beispiel würde als `.gitlab-ci.yml` wie folgt aussehen:

**1.)** Als erstes die stage zur Erstellung des docker images

```

build image:
  stage: build image
  script:
    - docker build --no-cache -t <docker image name>:${CI_COMMIT_REF_SLUG}_${CI_PIPELINE_ID}
  tags:
    - shell
```

**2.)** Nun die stage um z.B. Tests in dem neu erstellten docker image durchzuführen:

```

testing:
  stage: testing
  image: <docker image name>:${CI_COMMIT_REF_SLUG}_${CI_PIPELINE_ID}
  script:
    - /bin/test_tool.sh
    ...
  tags:
    - docker
```

Beim durchlaufen der Pipeline erkennt man in der Augabe welcher gitlab-runner für die entsprechende stage verwendet.

bei der `build image` stage:

```
Running with gitlab-runner 12.1.0 (de7731dd)
  on Shell executor AUxbxyQE
```

und entsprechend bei der `testing` stage:

```
Running with gitlab-runner 12.1.0 (de7731dd)
  on Docker executor jSyHnqkS
```

Somit kann man selbst bei gitlab-runnern, welche dediziert für ein bestimmtes Projekt verwendet werden, diese noch z.B. für verschiedene Umgebungen per _tag_ einrichten. Dies ist zum Beispiel praltisch wenn man sicher gehen will, dass man immer einen gitlab-runner für die `production` Umgebung verfügbar hat, sofern einer damit getaggt ist.