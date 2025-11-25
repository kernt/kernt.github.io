---
tags:
  - jenkins
  - pipeline
---
# Jenkins pipeline als Code

Als Standard-Interaktionsmodell für Jenkins, historisch gesehen, wurde sehr web-UI lastig entwickelt, was es Benutzern ermöglicht, manuelle Workspaces zu erstellen und dann manuell die Details über einen Webbrowser auszufüllen.
Dies erfordert zusätzliche Anstrengungen, um jobs zu erstellen und zu verwalten, um mehrere Projekte zu testen und zu bauen.
Außerdem hält sie die Konfiguration eines Auftrags zum Erstellen/Testen/Bereitstellen von dem eigentlichen Code, der gebaut/getestet/implementiert wird.
Dies verhindert, dass Benutzer ihre vorhandenen CI/CD-Best Practices an die Job-Konfigurationen selbst anwenden kann.

## [Pipeline](https://jenkins.io/doc/book/pipeline/getting-started/#global-variable-reference#)

Mit der Einführung des Pipeline-Plugins können Anwender nun den gesamte Build/Test/Deploy-Pipeline eines Projekts in einer `Jenkinsfile` implementieren und diese neben ihrem Code speichern und ihre Pipeline als ein weiteres Codepaket überprüfen, das in die Quellcodeverwaltung eingecheckt wird.

Das Pipeline-Plugin wurde von dem Build Flow Plugin inspiriert, zielt aber darauf ab, einige Konzepte zu erforschen, die von Build Flow mit Features wie:

* Die Möglichkeit, Aufträge auszusetzen/fortzusetzen.
* Überprüfen der Pipeline-Definition in die Quellcodeverwaltung [Jenkinsfile](../jenkins-jenkinsfile)
* Unterstützung für die Erweiterung der Domain-spezifischen Sprache mit zusätzlichen, organisationsbezogenen Schritten über die Funktion "Shared Libraries".

In einem angrenzenden Thema ist das [Job-DSL-Plugin](../jenkins-plugin-job-dsl), das auch erwähnenswert ist.

### Quelle

* [jenkins pipline as code Original](https://jenkins.io/solutions/pipeline/)
* [Pipeline Examples](https://jenkins.io/doc/pipeline/examples/)
