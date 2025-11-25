---
tags:
  - jenkins
  - devops
---
# Jenkins

Jenkins ist ein Programm das bei der Umsetzung einer CI/CD Umgebung hilft.
Im Gegensatz zu vielen anderen Alternativen wird ein allgemeiner Ansatz eingesetzt was die einarbeitungzeit hier etwas Verändern kann.
Für Jenkins gibt viele Plugins die die Funktionalität start erweitern können.

Inhaltesverzeichnis

* [jenkins-jenkinsfile](../jenkins-jenkinsfile)
* [jenkins Pipelins als Code](../jenkins-pipline-as-code)
* [jenkins-plugin-job-dsl](../jenkins-plugin-job-dsl)

## Installation

Einrichtung der Proxys

### mod_proxy mit Apache 2.4

```sh
ProxyPass         /jenkins  http://localhost.local:8081/jenkins nocanon
ProxyPassReverse  /jenkins  http://localhost.local:8081/jenkins
ProxyRequests     Off
AllowEncodedSlashes NoDecode

# Local reverse proxy authorization override
# Most unix distribution deny proxy by default (ie /etc/apache2/mods-enabled/proxy.conf in Ubuntu)
<Proxy http://localhost.local:8081/jenkins*>
  Order deny,allow
  Allow from all
</Proxy>
```

### mod_proxy mit HTTPS

```sh
ProxyRequests     Off
ProxyPreserveHost On
AllowEncodedSlashes NoDecode

<Proxy http://localhost:8081/jenkins*>
  Order deny,allow
  Allow from all
</Proxy>

ProxyPass         /jenkins  http://localhost:8081/jenkins nocanon
ProxyPassReverse  /jenkins  http://localhost:8081/jenkins
ProxyPassReverse  /jenkins  http://your.host.com/jenkins
```

```s
ProxyRequests     Off
ProxyPreserveHost On
ProxyPass /jenkins/ http://localhost:8081/jenkins/ nocanon
AllowEncodedSlashes NoDecode

<Location /jenkins/>
  ProxyPassReverse /
  Order deny,allow
  Allow from all
</Location>

Header edit Location ^http://www.example.com/jenkins/ https://www.example.com/jenkins/
```

## Proxy mit ha proxy

Ausgangslage:
Domain: vmd36612.contaboserver.net
Jenkins: ist local installiert
https: letsencrypt wird noch über die Domain ausgeliefert

haproxy.conf

```s
global
  log /dev/log  local0
  log /dev/log  local1 notice
  stats socket /var/lib/haproxy/stats level admin
  chroot /var/lib/haproxy
 tune.ssl.default-dh-param 2048
 user haproxy
 group haproxy
 daemon

defaults
   log global
   mode  http
   option  httplog
   option  dontlognull
        timeout connect 5000
        timeout client 50000
        timeout server 50000

frontend www-https
    # Bind 443 with the generated letsencrypt cert.
    bind *:443 ssl crt /etc/haproxy/certs/vmd36612.contaboserver.net.pem
    # set x-forward to https
    # reqadd X-Forwarded-Proto:\ https
    # set X-SSL in case of ssl_fc <- explained below
    http-request set-header X-SSL %[ssl_fc]
    # Select a Challenge
    acl letsencrypt-acl path_beg /.well-known/acme-challenge/
    [[acl]] url_jenkins path_beg /jenkins
    [[use_backend]] jenkins if url_jenkins
    acl jenkins path_dir -i /jenkins
    use_backend jenkins if jenkins
    # Use the challenge backend if the challenge is set
    use_backend letsencrypt-backend if letsencrypt-acl
    default_backend https-backend

frontend www-http
    bind *:80
    mode http
    acl letsencrypt-acl path_beg /.well-known/acme-challenge/
    use_backend letsencrypt-backend if letsencrypt-acl
    acl url_jenkins path_beg /jenkins/
    acl host-is-jenkins-vmd366 hdr(host) eq vmd36612.contaboserver.net
    acl jenkins path_dir -i /jenkins
    use_backend jenkins if url_jenkins host-is-jenkins-vmd366 jenkins
    default_backend www-backend

backend www-backend
    mode http
    balance roundrobin
    option forwardfor
    option httpchk HEAD / HTTP/1.1\r\nHost:localhost
    cookie SERVERID insert indirect
    redirect scheme https code 301 if !{ ssl_fc }

backend https-backend
    mode http
    option forwardfor
    option httpchk HEAD / HTTP/1.1\r\nHost:localhost
    cookie SERVERID insert indirect
    redirect scheme https code 301 if !{ ssl_fc }
    server ghost 127.0.0.1:81
backend letsencrypt-backend
    server letsencrypt 127.0.0.1:54321

    mode http
    option forwardfor
    [[option]] httpchk HEAD :8080/jenkins HTTP/1.1rnHost:localhost:8080/jenkins
    [[http-request]] set-header X-Forwarded-Host vmd36612.contaboserver.net
    ##http-request set-header X-Forwarded-Port 8443 if { ssl_fc }
    [[http-request]] set-header X-Forwarded-Port 8080 if !{ ssl_fc }
    [[http-request]] set-header X-Forwarded-Proto https if { ssl_fc }
    server jenkins01 127.0.0.1:8080
    reqrep ^([^\ :]*)\ /(.*) \1\ /\2
   acl response-is-redirect res.hdr(Location) -m found
   rspirep ^Location:\ (http|https)://127.0.0.1:8080/jenkins/(.*) Location:\ \1://vmd36612.contaboserver.net/jenkins/\2 if response-is-redirect
```

> Damit es auch geht muss der Jenkins Dienst mit `JENKINS_ARGS="$JENKINS_ARGS --prefix=/jenkins"` gestartet sein.

## Sicherheit

Certifikat für jenkins umwandeln

Zuerst das  certificate in das DER format umwandeln :

`openssl x509 -outform der -in certificate.pem -out certificate.der`

Danach, import  in den keystore :

`keytool -import -alias my-domain-com -keystore /root/.keystore -file certificate.der`

## Nach der Installation

So sollte er nicht eingerichtet sein

```sh
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

c30df8355aee45cbbcadceb2ad1cb0ab

This may also be found at: /root/.jenkins/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```

Er wird nämlich als User root ausgeführt.

Wenn man ein Packet der Distribution ausgeführt sollte es dazu den Benutzer jenkins geben.
Folgender massen kann es Prüfen.

`getent  passwd jenkins`

### Jenkins Node Konfiguration

* [jenkins-more-than-just-target-practice](https://www.fortynorthsecurity.com/jenkins-more-than-just-target-practice/)

## Jenkins Fehlersuche

`java -jar jenkins.war  --httpsPort=8443 --httpPort=-1 --httpsCertificate=fullchain.pem --httpsPrivateKey=privkey-rsa.pem`

## Materialien zum Testen

* [Github Centos Pipline Service](https://github.com/CentOS/container-pipeline-service)

## Dateien und Verzeichnisse

Home verzeichnis : `/var/lib/jenkins`

## Jenkins Plugins

Die Wichtisten:

* Pipeline
* Ant Plugin

* [rhnpush](https://wiki.jenkins-ci.org/display/JENKINS/rhnpush+Plugin)
adds a post build step to push RPMs to Spacewalk or RHN satelite servers. It requires rhnpush to be installed on the slave
* [SCP](https://wiki.jenkins-ci.org/display/JENKINS/SCP+plugin)
ploads build artifacts to repository sites using SCP (SSH) protocol. First you should define SCP hosts on hudson global config page.
* [workflow-aggregator](https://wiki.jenkins-ci.org/display/JENKINS/Pipeline+Plugin)

## Jenkins Jobs

* [jenkins-php](http://jenkins-php.org/installation.html)

## Jenkins Sicherheit

Tools:

* [keystore-explorer](http://keystore-explorer.org/features.html)

Benutzer Zugangsdaten:

* **User** gelten für bestimmte Benutzer
* **System** gelten Systemweit

### Https mit Jenkins

## Jenkins Pipelines

* [COMPATIBILITY mit pipeline-plugin](https://github.com/jenkinsci/pipeline-plugin/blob/master/COMPATIBILITY.md)
* [Jenkins pipeline Buch](https://jenkins.io/doc/book/pipeline/)
* [Continuous Delivery mit Jenkins Workflow](https://dzone.com/refcardz/continuous-delivery-with-jenkins-workflow)
* [Pipeline syntax](https://jenkins.io/doc/book/pipeline/syntax/)
* [Pipeline schnitte Refarens](https://jenkins.io/doc/pipeline/steps/)
* [pipeline shared-libraries](https://jenkins.io/doc/book/pipeline/shared-libraries/)
* [Docker Piplien Demo](https://github.com/jenkinsci/workflow-aggregator-plugin/blob/master/demo/README.md)
* [continuous_delivery_with_jenkins_pipeline](https://go.cloudbees.com/docs/cloudbees-documentation/cookbook/book.html#ch13__continuous_delivery_with_jenkins_pipeline)
* [TUTORIAL](https://github.com/jenkinsci/pipeline-plugin/blob/master/TUTORIAL.md)
* [top-10-best-practices-jenkins-pipeline-plugin](https://www.cloudbees.com/blog/top-10-best-practices-jenkins-pipeline-plugin)
* [hello-world](https://jenkins.io/doc/pipeline/tour/hello-world/)
* [https mit Jenkins und Let's Encrypt](https://github.com/hughperkins/howto-jenkins-ssl/blob/master/letsencrypt.md)
* [Jenkins tutorialspoint](https://www.tutorialspoint.com/jenkins/index.htm)

## Jenkins Berechtigungen

Berechtigungen für Github
Dazu wird ein _New personal access token_ unter -> USER -> Settings -> Developer Settings -> Personel access tokens mit den Berechtigungen _admin:repo_hook_ erstellt.

## Jenkins Toturial Jenkins Docukentation

* [tutorialspoint](https://www.tutorialspoint.com/jenkins/jenkins_installation.htm)
* [jenkins-2-0-von-ci-nach-cd-mit-pipelines](https://blog.openknowledge.de/2016/05/jenkins-2-0-von-ci-nach-cd-mit-pipelines/)
* [Jenkins als CI Werkzeug](http://home.edvsz.fh-osnabrueck.de/skleuker/CSI/Werkzeuge/Jenkins/)
* [Use+Jenkins](https://wiki.jenkins-ci.org/display/JENKINS/Use+Jenkins)

## ToDo

* [von-continuous-integration-zu-continuous-delivery-mit-jenkins-pipeline-teil1](https://www.informatik-aktuell.de/entwicklung/methoden/von-continuous-integration-zu-continuous-delivery-mit-jenkins-pipeline-teil-1.html)
* [von-continuous-integration-zu-continuous-delivery-mit-jenkins-pipeline-teil2](https://www.informatik-aktuell.de/entwicklung/methoden/von-continuous-integration-zu-continuous-delivery-mit-jenkins-pipeline-teil-2.html)
* [von-continuous-integration-zu-continuous-delivery-mit-jenkins-pipeline-teil3](https://www.informatik-aktuell.de/entwicklung/methoden/von-continuous-integration-zu-continuous-delivery-mit-jenkins-pipeline-teil-3.html)

## Pipeline

* [continuous-delivery](https://www.cloudbees.com/continuous-delivery/pipeline)

## [Jenkins Cluster](../jenkins-cluster)

## Doku Übersicht

* [kubernetes-cd-pipline-jenkins-integration](../kubernetes-cd-pipline-jenkins-integration)
* [selenium tests mit jenkins](../selenium-tests-jenkins)
* [kubernetes-cd-pipline-erstellen](../kubernetes-cd-pipline-erstellen)
* [Übersetzung von Pipeline as Code with Jenkins](../jenkins-pipline-as-code)
