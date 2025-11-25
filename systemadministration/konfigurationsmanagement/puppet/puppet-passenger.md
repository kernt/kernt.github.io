---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet passenger

Der WEBrick-Server, den wir im vorherigen Abschnitt konfiguriert haben, ist nicht in der Lage, eine große Anzahl von Knoten zu handhaben. 
Um mit einer großen Anzahl von Knoten umzugehen, ist ein skalierbarer Webserver erforderlich. 
Puppet ist ein Rubyprozess, also brauchen wir einen Weg, um einen Rubyprozess innerhalb eines Webservers zu führen. Passagier ist die Lösung für dieses Problem. Es erlaubt uns, den Puppet-Master-Prozess innerhalb eines Webservers auszuführen \(Apache standardmäßig\). Viele Distributionen versenden mit einem Puppetmaster-Passager Paket, das das für Sie konfiguriert. 
In diesem Abschnitt verwenden wir das Paket, um die Puppe zu konfigurieren, um innerhalb des Passagiers zu laufen.

### Fertig werden

Installiere das Puppetmaster-Passagier-Paket:

```
# puppet resource package puppetmaster-passenger ensure=installed
Notice../puppet/Package[puppetmaster-passenge../puppet/ensure: ensure changed 'purged'
 to 'present'
package { 'puppetmaster-passenger':
  ensure => '3.7.0-1puppetlabs1',
}
```

Notiz
Mit Puppen-Ressource zu installieren Pakete stellt sicher, dass der gleiche Befehl auf mehrere Verteilungen arbeiten \(sofern die Paketnamen gleich sind\).

### Wie es geht...

Die Schritte sind wie folgt:

1. Stellen Sie sicher, dass die Puppet-Master-Site in Ihrer Apache-Konfiguration aktiviert ist. 
  Abhängig von Ihrer Verteilung kann dies in../puppet/e../puppet/htt../puppet/conf.d` oder../puppet/e../puppet/apach../puppet/sites-enabled` sein. 
  Die Konfigurationsdatei sollte für Sie erstellt werden und enthält folgende Informationen:

  ```
  PassengerHighPerformance on
  PassengerMaxPoolSize 12
  PassengerPoolIdleTime 1500
  # PassengerMaxRequests 1000
  PassengerStatThrottleRate 120
  RackAutoDetect Off
  RailsAutoDetect Off
  Listen 8140
  ```

2. Diese Zeilen sind Tuning-Einstellungen für Passenger. Die Datei weist dann Apache an, auf Port 8140, den Puppet-Master-Port zu lauchen. Als nächstes wird eine VirtualHost-Definition erstellt, die die Puppet CA-Zertifikate und das Puppet-Master-Zertifikat lädt:

  ```
  <VirtualHost *:8140>
       SSLEngine on
       SSLProtocol             ALL -SSLv2 -SSLv3
       SSLCertificateFile    ../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/cer../puppet/puppet.pem
       SSLCertificateKeyFile ../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/private_ke../puppet/puppet.pem
       SSLCertificateChainFil../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/cer../puppet/ca.pem
       SSLCACertificateFile  ../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/cer../puppet/ca.pem
       SSLCARevocationFile   ../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/../puppet/ca_crl.pem
       SSLVerifyClient optional
       SSLVerifyDepth  1
       SSLOptions +StdEnvVars +ExportCertData
  ```


### Tip

Hier können Sie je nach Ihrer Version des Puppetmaster-passenger Pakets mehr oder weniger Zeilen SSL-Konfiguration haben.

1. Als nächstes werden einige wichtige Header gesetzt, so dass der Passagierprozess Zugriff auf die SSL-Informationen hat, die vom Client Node gesendet werden:

  ```
  RequestHeader unset X-Forwarded-For
  RequestHeader set X-SSL-Subject %{SSL_CLIENT_S_DN}e
  RequestHeader set X-Client-DN %{SSL_CLIENT_S_DN}e
  RequestHeader set X-Client-Verify %{SSL_CLIENT_VERIFY}e
  ```

2. Schließlich wird die Position der Passenger Konfigurationsdatei `config.ru` mit dem `DocumentRoot` Speicherort wie folgt angegeben:

  ```
      DocumentRoo../puppet/u../puppet/sha../puppet/pupp../puppet/ra../puppet/puppetmaste../puppet/publ../puppet/
       RackBaseUR../puppet/
  ```

3. Die Datei `config.ru` sollte unter../puppet/u../puppet/sha../puppet/pupp../puppet/ra../puppet/puppetmaste../puppet/` vorhanden sein und sollte den folgenden Inhalt haben:
  \`\`\`$0 = "master"
  ARGV &lt;&lt; "--rack"
  ARGV &lt;&lt; "--confdir" &lt;&lt; ../puppet/et../puppet/puppet"
  ARGV &lt;&lt; "--vardir"  &lt;&lt; ../puppet/va../puppet/li../puppet/puppet"
  require 'puppe../puppet/uti../puppet/command\_line'
  run Puppet==Util==CommandLine.new.execute


    Wenn die Passenger-Apache-Konfigurationsdatei korrekt konfiguriert ist und die `config.ru` Datei korrekt konfiguriert ist, starten Sie den Apache-Server und vergewissern Sie sich, dass der Apache auf dem Puppet-Master-Port lauscht (wenn Sie den Standalone-Puppet-Master vorher konfiguriert haben, müssen Sie diesen Prozess jetzt mit dem Dienst `systemctl stop puppetserver` beenden):


root@puppet:~ \# service apache2 start
\[ ok \] Starting web server: apache2
root@puppet:~ \# lsof -i :8140
COMMAND  PID     USER   FD   TYPE DEVICE SIZ../puppet/OFF NODE NAME
apache2 9048     root    8u  IPv6  16842      0t0  TCP _:8140 \(LISTEN\)
apache2 9069 www-data    8u  IPv6  16842      0t0  TCP _:8140 \(LISTEN\)
apache2 9070 www-data    8u  IPv6  16842      0t0  TCP \*:8140 \(LISTEN\)


    ### Wie es funktioniert...

    Die Passagier-Konfigurationsdatei verwendet die vorhandenen Puppet-Master-Zertifikate, um auf Port 8140 zu lauschen und behandelt die gesamte SSL-Kommunikation zwischen dem Server und dem Client. 
    Sobald die Zertifikatinformationen übernommen wurden, wird die Verbindung an einen Rubyprozess ausgegeben, der vom Passenger mit den Befehlszeilenargumenten aus der Datei `config.ru` gestartet wurde.

    In diesem Fall wird die Variable `$0` auf `master` gesetzt und die Argumentvariable wird auf ` puppet master --rack --confdi../puppet/e../puppet/puppet --vardi../puppet/v../puppet/l../puppet/puppet` gesetzt; Dies ist gleichbedeutend mit dem Ausführen der folgenden Befehlszeile:
    `puppet master --rack --confdi../puppet/e../puppet/puppet --vardi../puppet/v../puppet/l../puppet/puppet`



    ### Es gibt mehr...

    Sie können der `config.ru` Datei zusätzliche Konfigurationsparameter hinzufügen, um weiter zu verändern, wie die Puppe läuft, wenn sie durch den Passagier läuft. 
    Um zum Beispiel das Debugging auf dem Passagier-Puppet-Master zu ermöglichen, fügen Sie folgende Zeile zu `config.ru` vor dem Run hinzu `Puppet==Util==CommandLine.new.execute`:

ARGV &lt;&lt; "--debug"
\`\`\`

