Erfahren Sie, wie Sie die Infrastrukturkonfiguration effizient und automatisiert verwalten, indem Sie Ansible AWX in GitLab integrieren.

Bevor wir die Konfigurationen eingeben, ist es wichtig, verschiedene Gründe aufzuzeigen, die die Vorteile der Integration zwischen AWX und GitLab unterstützen.

1. **Zentralisierte Versionsverwaltung:** GitLab bietet ein zentrales Versionskontrollsystem, mit dem Sie Ihre Spielbücher und anderen Konfigurationsdateien verwalten und abbilden können.
2. **Geschichte** ändern: Sie können die Geschichte der Änderungen sehen, Vergleiche zwischen Versionen vornehmen und Änderungen leicht umkehren, was die Verwaltung der Ansible-Konfiguration erleichtert.
3. **Kontinuierliche Integration und kontinuierliche Lieferung (CI/CD):** Sie können GitLab CI/CD konfigurieren, um die Ausführung von Spielbüchern zu automatisieren, wenn Änderungen am Projektarchiv vorgenommen werden.
4. **Security and Controlled Access:** AWX bietet rollenbasierte Zugriffskontrollfunktionen. Wenn Sie es in GitLab integrieren, können Sie Benutzerverwaltung und Berechtigungen verwenden, um den Zugriff auf Spielbücher und Ausführungsdateien zu kontrollieren.
5. **Benachrichtigungen und Berichterstattung:** Sie können Benachrichtigungen auf GitLab einrichten, um Teams auf Änderungen in der Konfiguration oder im Status von Ansible AWX-Ausführungen aufmerksam zu machen.

Umgebung

```sh
Betriebssystem: Rocky Linux 9.3  
AWX: 17.0.1  
Gitlab: 16.6
```

Für dieses Tutorial werde ich das folgende GitLab-Projekt als Beispiel verwenden:

![](ocfFeDacTmd7tFi7fUy-fg.webp)

![](ocfFeDacTmd7tFi7fUy-fg.webp)

![](1_ocfFeDacTmd7tFi7fUy-fg.webp)

![](2ROdXe32u059w6tNswSwfNQ.webp)

Sobald wir die Anmeldeinformationen konfiguriert haben, müssen wir die Projekt-URL kopieren, um sie auf AWX zu bringen.

![](1_ZLinSJWnoNNngAnUD7i2Ew.webp)

![](1_P1jPulIvTCrOMd0WjEeroA.webp)

![](1_KA9Ftic2XfvvFmLoQoqOmg.webp)

Im Abschnitt "Source Control Credential" müssen wir die Anmeldeinformationen auswählen, die wir im vorherigen Schritt erstellt hatten.

![](1_c04DG68tkxb1zhFfXFEzug.webp)

![](1_-UCaoCX1EYo-v0Bk9jFYwQ.webp)

![](2_-UCaoCX1EYo-v0Bk9jFYwQ.webp)

Und bereit, lasst uns unser Repository synchronisieren.

![](1__9j9lqsid4W3UE7tD5KvzQ.webp)

![](1_x1zneC7ud8wvVd2lUm94-A.webp)

Es sind noch 2 Schritte übrig, um das Inventar des Projekts und die Vorlage für die Ausführung des Spielbuchs zu erstellen.

![](1_wIhD0RIlJJenPx2jlFzA3A.webp)

![](1_SZIOOBlvDSDXMN98Ypezug.webp)

![](1_LDsJVB2eukA6Y7a2mrrxJA.webp)

In diesem Konfigurationsbereich müssen wir das in AWX erstellte Projekt und die Bestandsdatei angeben.

![](1_2gvoMLZYObNznkOb3nVqmw.webp)

![](1_SQFJXGZdZ8dIbpJUehfYCA.webp)

Sobald die Synchronisierung vorbei ist, können wir die Details unseres Inventars im Bereich Hotels sehen.

![](1_rurEHWstgXoL5BI9C5YHQw.webp)

![](1_SG_DejkHTrv8iskrOz4WaQ.webp)

Und jetzt müssen wir unsere Vorlage für die Ausführung des Spielbuchs aufstellen.

![](1_Qhrk4RheqkpPI9jbW2b-aQ.webp)

![](1_5v65gfoCCKJafTNiWyEBOQ.webp)

![](1_pQ2M-j4mowLlQ1zBikojmA.webp)

![](1_NUscKiHSVoFEUO2TKnVvqw.webp)

Bereit. Das Spielbuch endete korrekt und so schließen wir dieses grundlegende Integrations-Tutorial zwischen AWX und GitLab. *