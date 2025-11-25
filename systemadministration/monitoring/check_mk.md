# Checkmk installieren

3.1 Voraussetzungen

Für die Ausführung der hier vorgestellten Befehle benötigen Sie eine funktionierende Installation der Docker Engine und Grundkenntnisse in deren Nutzung.

3.2 Checkmk als Docker-Container installieren

Sie können das Image mit einem einzigen Befehl herunterladen und einen Container mit einer laufenden Checkmk Instanz erzeugen.

```sh
docker container run -dit -p 8084:5000 -p 8001:8000 --tmpfs /opt/omd/sites/cmk/tmp:uid=1000,gid=1000 --name monitoring -v /etc/localtime:/etc/localtime:ro --restart always checkmk/check-mk-raw:2.3.0p19
```

Mit dem Herunterladen von Checkmk erklären Sie sich mit der Endbenutzer-Lizenzvereinbarung und den Allgemeinen Vertragsbedingungen einverstanden, die für Ihre Checkmk-Edition gelten und unter https://checkmk.com/de/legal abrufbar sind. Sie stimmen auch den Checkmk-Exportkontrollbestimmungen zu, insbesondere werden Sie die Software nicht an Personen oder Unternehmen mit Sitz im Iran, auf der Krim, in Kuba, Nordkorea, Russland, Sudan oder Syrien weitergeben.

Wenn Sie sich sicher sind, dass die Daten in der Checkmk Container-Instanz nur in diesem speziellen Container verfügbar sein sollen, finden Sie im Handbuch entsprechende Befehle.

Sobald das Image heruntergeladen und der Container gestartet wurde, sollten Sie eine ähnliche Ausgabe wie diese erhalten:

```
Unable to find image 'checkmk/check-mk-cloud:2.3.0p11' locally
2.3.0p11: Pulling from checkmk/check-mk-cloud
6552179c3509: Already exists
241a5e43b545: Pull complete
c953736bfe96: Pull complete
19e779b2b0d9: Pull complete
1f3636baf252: Pull complete
0f0bcf1a0726: Pull complete
Digest: sha256:448e9e9de08b389bda8561d4df240a919a09d7360518d345e38323d644706b5a
Status: Downloaded newer image for checkmk/check-mk-cloud:2.3.0p11
84f7ba45c1227d30257469daa6199ef0a64e259db2c0fb7c4ab14e0434a24d23
```

Sie können sich nun zum ersten Mal einloggen und Checkmk ausprobieren. Sie finden das initiale Passwort für den Account cmkadmin in den Logs des Containers:

`docker container logs monitoring`

Die Ausgabe in diesem Beispiel ist auf die wesentlichen Informationen gekürzt.

Created new site monitoring with version 2.3.0p11.cce.

    The site can be started with omd start monitoring.
    The default web UI is available at http://your_server/monitoring/
    The admin user for the web applications is cmkadmin with password: generated-password
    (It can be changed with 'htpasswd -m ~/etc/htpasswd cmkadmin' as site user.)
    Please do a su - monitoring for administration of this site. 

Hinweis: Die im Log angezeigte URL für den Zugriff auf die Weboberfläche mit der ID des Containers ist nur innerhalb des Containers bekannt und für den Zugriff von außen über den Webbrowser nicht geeignet.

Stattdessen können Sie über http://localhost:8080/cmk/check_mk/ auf die Benutzeroberfläche von Checkmk zugreifen.

Hinweis: Wenn Ihr Checkmk Server keinen Internetzugang hat, können Sie das Docker-Image als .tar.gz-Datei herunterladen. Befolgen Sie die Anleitung, um das Docker-Image zu installieren.

https://download.checkmk.com/checkmk/2.3.0p11/check-mk-cloud-docker-2.3.0p11.tar.gz

Weitere Informationen darüber, wie Sie Checkmk als Docker-Container betreiben, finden Sie hier.