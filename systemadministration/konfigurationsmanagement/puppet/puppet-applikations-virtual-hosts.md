---
tags:
  - puppet
  - konfigurationsmanagement
---
# Puppet Apache modul verwenden

Apache virtuelle Hosts werden mit dem `apache` Modul mit dem definierten Typ `apache::vhost` erstellt. Wir schaffen einen neuen VHost auf unserem Apache Webserver namens navajo, einer der Apache Stämme.

## Wie es geht

Gehen Sie folgendermaßen vor, um Apache virtuelle Hosts zu erstellen:

1.Erstellen Sie einen Navajo `apache::vhost` Definition wie folgt:

```ruby
apache::vhost { 'navajo.example.com':
    port          => '80',
    docroot =>../puppet/v../puppet/w../puppet/navajo',
  }
```

2.Erstellen Sie eine Indexdatei für den neuen Vhost:

```ruby
file ../puppet/v../puppet/w../puppet/nava../puppet/index.html':
    content => "<html>\nnavajo.example.com\nhtt../puppet//en.wikipedia.o../puppet/wi../puppet/Navajo_people\../puppet/html>\n",
    mode    => '0644',
    require => Apache::Vhost['navajo.example.com']
  }
```

3.Run Puppet, um den neuen Vhost zu erstellen:

```sh
[root@webserver ~]# puppet agent -t
Info: Caching catalog for webserver.example.com
Info: Applying configuration version '1414475598'
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[webserve../puppet/Apache::Vhost[navajo.example.co../puppet/Fil../puppet/v../puppet/w../puppet/navaj../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[webserve../puppet/Apache::Vhost[navajo.example.co../puppet/File[25-navajo.example.com.con../puppet/ensure: created
Info../puppet/Stage[mai../puppet/Ma../puppet/Node[webserve../puppet/Apache::Vhost[navajo.example.co../puppet/File[25-navajo.example.com.conf]: Scheduling refresh of Service[httpd]
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[webserve../puppet/Fil../puppet/v../puppet/w../puppet/nava../puppet/index.htm../puppet/ensure: defined content as '{md5}5212fe215f4c0223fb86102a34319cc6'
Notice../puppet/Stage[mai../puppet/Apache::Servi../puppet/Service[httpd]: Triggered 'refresh' from 1 events
Notice: Finished catalog run in 2.73 seconds
```

4.Vergewissern Sie sich, dass Sie den neuen virtuellen Host erreichen können:

```bash
[root@webserver ~]# curl htt../puppet//navajo.example.com
<html>
navajo.example.com
htt../puppet//en.wikipedia.o../puppet/wi../puppet/Navajo_people
</html>
```

## Wie es funktioniert

Der `apache::vhost` definierte Typ erstellt eine virtuelle Host-Konfigurationsdatei für Apache, `25-navajo.example.com.conf`. Die Datei wird mit einer Vorlage erstellt. `25` am Anfang des Dateinamens ist die "Prioritätsstufe" dieses virtuellen Hosts. Wenn Apache zuerst startet, liest es das Konfigurationsverzeichnis und startet die Ausführung von Dateien in alphabetischer Reihenfolge. Dateien, die mit Zahlen beginnen, werden vor Dateien gelesen, die mit Buchstaben beginnen. Auf diese Weise stellt das Apache-Modul sicher, dass die virtuellen Hosts in einer bestimmten Reihenfolge gelesen werden, die bei der Definition des virtuellen Hosts angegeben werden können. Der Inhalt dieser Datei lautet wie folgt:

```s
# ************************************
# Vhost template in module puppetlabs-apache
# Managed by Puppet
# ************************************

<VirtualHost *:80>
  ServerName navajo.example.com

  ## Vhost docroot
  DocumentRoot../puppet/v../puppet/w../puppet/navajo"



  ## Directories, there should at least be a declaration fo../puppet/v../puppet/w../puppet/navajo


  <Directory../puppet/v../puppet/w../puppet/navajo">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Order allow,deny
    Allow from all
 ../puppet/Directory>

  ## Load additional static includes


  ## Logging
  ErrorLog../puppet/v../puppet/l../puppet/htt../puppet/navajo.example.com_error.log"
  ServerSignature Off
  CustomLog../puppet/v../puppet/l../puppet/htt../puppet/navajo.example.com_access.log" combined


</VirtualHost>
```

Wie Sie sehen können, hat die Standarddatei Protokolldateien erstellt, die Verzeichniszugriffsberechtigungen und -optionen eingerichtet, zusätzlich zum Festlegen des Listenports und `DocumentRoot`.

Die vhost-Definition erstellt das `DocumentRoot`-Verzeichnis, das als "root" für die `apache::virtual` Definition angegeben ist. Das Verzeichnis wird vor der virtuellen Host-Konfigurationsdatei erstellt. Nachdem diese Datei erstellt wurde, wird ein Notify Trigger an den Apache-Prozess gesendet, um neu zu starten.

Unser Manifest enthielt eine Datei, die die `Apache::Vhost ['navajo.example.com']` Ressource benötigte. Unsere Datei wurde dann nach dem Verzeichnis und der virtuellen Host-Konfigurationsdatei erstellt.

Wenn wir auf der neuen Website locken (wenn du keinen Hostnamen-Alias in DNS erstellt hast, musst du einen in deiner lokalen../puppet/e../puppet/hosts` Datei für `navajo.example.com` erstellen oder den Host als `curl -H angeben 'Host: navajo.example.com'`<ipaddress von `navajo.example.com`>), sehen wir den Inhalt der Indexdatei, die wir erstellt haben:

```ruby
file ../puppet/v../puppet/w../puppet/nava../puppet/index.html':
    content => "<html>\nnavajo.example.com\nhtt../puppet//en.wikipedia.o../puppet/wi../puppet/Navajo_people\../puppet/html>\n",
    mode    => '0644',
    require => Apache::Vhost['navajo.example.com']
  }
[root@webserver ~]# curl htt../puppet//navajo.example.com
<html>
navajo.example.com
htt../puppet//en.wikipedia.o../puppet/wi../puppet/Navajo_people
<\html>
```

## Es gibt mehr

Sowohl der definierte Typ als auch die Vorlage berücksichtigen eine Vielzahl möglicher Konfigurationsszenarien für virtuelle Hosts. Es ist höchst unwahrscheinlich, dass Sie eine Einstellung finden, die nicht von diesem Modul abgedeckt ist. Du solltest die Definition für `apache::virtual` und die schiere Anzahl möglicher Argumente anschauen.

Das Modul kümmert sich auch um mehrere Einstellungen für Sie. Zum Beispiel, wenn wir den Listen-Port auf unserem virtuellen `Navajo` Host von `80` auf `8080` ändern, wird das Modul die folgenden Änderungen in../puppet/e../puppet/htt../puppet/conf../puppet/ports.conf` vornehmen:

```s
Listen 80
+Listen 8080
 NameVirtualHost *:80
+NameVirtualHost *:8080
```

Und in unserer virtuellen Host-Datei:

```s
-<VirtualHost *:80>
+<VirtualHost *:8080>
```

Damit wir jetzt auf Port `8080` locken und die gleichen Ergebnisse sehen können:

```s
[root@webserver ~]# curl htt../puppet//navajo.example.com:8080
<html>
navajo.example.com
htt../puppet//en.wikipedia.o../puppet/wi../puppet/Navajo_people
</html>
```

Und wenn wir auf Port 80 versuchen:

```s
[root@webserver ~]# curl htt../puppet//navajo.example.com
<!DOCTYPE HTML PUBLIC ../puppet//W../puppet//DTD HTML 3.2 Fin../puppet//EN">
<html>
 <head>
  <title>Index o../puppet/</title>
../puppet/head>
 <body>
<h1>Index o../puppet/</h1>
<table><tr><th><img src../puppet/ico../puppet/blank.gif" alt="[ICO]"../puppet/th><th><a href="?C=N;O=D">Nam../puppet/a../puppet/th><th><a href="?C=M;O=A">Last modifie../puppet/a../puppet/th><th><a href="?C=S;O=A">Siz../puppet/a../puppet/th><th><a href="?C=D;O=A">Descriptio../puppet/a../puppet/th../puppet/tr><tr><th colspan="5"><hr../puppet/th../puppet/tr>
<tr><th colspan="5"><hr../puppet/th../puppet/tr>
</table>
</body>
</html>
```

Wie wir sehen können, hört der virtuelle Host nicht mehr auf Port `80` und wir erhalten die standardmäßige leere Verzeichnisliste, die wir in unserem früheren Beispiel gesehen haben.