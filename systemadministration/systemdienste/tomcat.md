---
tags:
  - java
  - tomcat
---
# Tomcat

Einfache grundlagen zur einrichtung.

Installation auf centos

`yum install httpd-devel apr apr-devel apr-util apr-util-devel gcc make libtool autoconf`

Get the latest source tarball

```sh
# wget http://www.eu.apache.org/dist/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.42-src.tar.gz
# wget http://www.eu.apache.org/dist/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.42-src.tar.gz.sha1
```

Check the integrity of the file:

```sh
sha1sum -c tomcat-connectors-1.2.42-src.tar.gz.sha1
tomcat-connectors-1.2.42-src.tar.gz: OK
```

Extract the archive

```sh
tar xf tomcat-connectors-1.2.42-src.tar.gz
# cd tomcat-connectors-1.2.42-src/native/
```

Get the path for apxs

```sh
# which apxs
/usr/sbin/apxs
```

Configure, compile and install:

```
# ./configure --with-apxs=/usr/sbin/apxs
# make
# libtool --finish /usr/lib64/httpd/modules
# make install
```
### Configuration

Open the file /etc/httpd/conf/workers.properties and add the following to reflect your application details

```ini
worker.list=app1,app2

worker.app1.type=ajp13
worker.app1.host=app1.example.com
worker.app1.port=8201
worker.app1.socket_timeout=10

worker.app2.type=ajp13
worker.app2.host=app2.example.com
worker.app2.port=8201
worker.app1.socket_timeout=10
```

Open the file /etc/httpd/conf/httpd.conf and add the following

```sh
LoadModule jk_module modules/mod_jk.so

JkWorkersFile "/etc/httpd/conf/workers.properties"
JkLogFile     "/var/log/mod_jk.log"
JkLogLevel  info
JkLogStampFormat "[%a %b %d %H:%M:%S %Y] "
JkOptions     +ForwardKeySize +ForwardURICompat -ForwardDirectories
JkRequestLogFormat     "%w %V %T"
```

Restart Apache. Check the log file mod_jk.log for any issues

`[Sun Dec 03 10:08:57 2017] [7005:140288381306848] [info] init_jk::mod_jk.c (3595): mod_jk/1.2.42 initialized`

## Tomcat-Cluster

Cluster einrichten bei Tomcat

## Tomcat-Reverse-Proxy

Tomcat als setup mit reverse proxy nutzen

## Tomcat [Logging](../logging)

Mit tomcat logs Praktikabel einrichten

Zu anpassende Datei für Änderungen am logging ist immer die `server.xml` im Verzeichnis.

```xml
 <Valve className="org.apache.catalina.valves.AccessLogValve"
                 directory="logs"  prefix="access_log" suffix=".log"
                 pattern="%h %l %u %t %{HOST}i &quot;%r&quot; %s %b %I %S %D" />
                <!--
                    %a - Remote IP address
                        %A - Local IP address
                        %b - Bytes sent, excluding HTTP headers, or '-' if zero
                        %B - Bytes sent, excluding HTTP headers
                        %h - Remote host name (or IP address if enableLookups for the connector is false)
                        %H - Request protocol
                        %l - Remote logical username from identd (always returns '-')
                        %m - Request method (GET, POST, etc.)
                        %p - Local port on which this request was received
                        %q - Query string (prepended with a '?' if it exists)
                        %r - First line of the request (method and request URI)
                        %s - HTTP status code of the response
                        %S - User session ID
                        %t - Date and time, in Common Log Format
                        %u - Remote user that was authenticated (if any), else '-'
                        %U - Requested URL path
                        %v - Local server name
                        %D - Time taken to process the request, in millis
                        %T - Time taken to process the request, in seconds
                        %I - current request thread name (can compare later with stacktraces)
                -->
```

## Tomcat Tools

* [webjars](https://www.webjars.org/) WebJars are client-side web libraries (e.g. jQuery & Bootstrap) packaged into JAR (Java Archive) files.
* [tomcatmanager](https://pypi.org/project/tomcatmanager/)
* [easy-war-deployment](http://emmanuelrosa.com/articles/easy-war-deployment/)
* [apache-http-server-mod_proxy_ajp](https://confluence.atlassian.com/kb/proxying-atlassian-server-applications-with-apache-http-server-mod_proxy_ajp-830284354.html)
