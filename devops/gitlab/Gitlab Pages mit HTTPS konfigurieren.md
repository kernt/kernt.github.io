Gitlab bietet die Möglichkeit Webseiten zu hosten, hierfür muss das Feature „Pages“ konfiguriert werden.  
Hier am Beispiel werden die Pages auf dem selben Server wie Gitlab ausgeliefert, installiert ist Gitlab in der „Omnibus Installation“ Version.

Für die Verwendung von Gitlab Pages sind CI Runner notwendig. Ohne diese können die Jobs nicht durchgeführt werden und die Seite wird nicht ausgeliefert.

#### Gitlab Pages konfigurieren

Um das Feature zu aktivieren, muss man die `/etc/gitlab/gitlab.rb` bearbeiten.

In dieser befinden sich der Parameter `pages_external_url`. In dieser muss die URL unter der Pages laufen soll angegeben werden.  
Diese bestimmt auch, unter welcher Subdomain die Seiten dann standardmäßig erreichbar sind.

zb:  
`pages_external_url 'http://gitlab.<domain>.<tld>'`

#### DNS Einstellungen

Damit die Webseiten auch direkt erreicht werden können bietet sich ein Wildcardeintrag im DNS an.

`*.gitlab.<domain>.<tld> A <IP des Gitlab Servers>`

#### HTTPS Verschlüsselung

Um den Zugriff auf die Pages direkt per HTTPS zu verschlüsseln muss man bei `pages_external_url` die Domain mit `https` angeben.  
`pages_external_url 'https://gitlab.<domain>.<tld>'`

Zudem im Pages NGINX Bereich das entsprechende Zertifikat. Es empfiehlt sich zusätzlich noch die HTTP zu HTTPS Weiterleitung zu aktivieren.

```
################################################################################
## GitLab Pages NGINX
################################################################################
pages_nginx['redirect_http_to_https'] = true
pages_nginx['ssl_certificate'] = "<Pfad>"
pages_nginx['ssl_certificate_key'] = "<Pfad>"
```

#### Änderung übernehmen

Damit die Änderungen übernommen werden. Muss `gitlab-ctl reconfigure` ausgeführt werden.

#### Eigene Seitenerstellung einschränken

Es ist möglich, den Zugriff zum Pages Feature zu reglimentieren. Damit kann man verhindern, dass User sich den Domainname und das Zertifikat selbst wählen. Sondern ein vorgegebenes Zertifikat benutzen und eine URL nach gleichem Standard.

Hierfür muss man in den Admin Area -> Settings -> Pages, hier kann man nun das Häkchen bei „Require users to prove ownership of custom domains“ setzen oder nicht.

Die Gitlab Pages Seite wird mit dem SSL Zertifikat von Pages selbst ausgeliefert und im Format  
`<User bzw Gruppenname>.<FQDN von Gitlab Pages>/<Repositoryname>`