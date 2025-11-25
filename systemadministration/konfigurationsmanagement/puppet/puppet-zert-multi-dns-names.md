---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet zert multi dns names

Standardmäßig erstellt Puppet ein SSL-Zertifikat für Ihren Puppet-Master, der nur den vollständig qualifizierten Domainnamen des Servers enthält. Je nachdem, wie Ihr Netzwerk konfiguriert ist, kann es sinnvoll sein, dass der Server mit anderen Namen bekannt ist. In diesem Rezept machen wir ein neues Zertifikat für unseren Puppet-Master, der mehrere DNS-Namen hat.

### Fertig werden

Installiere das Puppet-Masterpaket, wenn du das noch nicht getan hast. Sie müssen dann den Puppet-Master-Service mindestens einmal starten, um eine Zertifizierungsstelle (CA) zu erstellen.



Wie es geht...

Die Schritte sind wie folgt:

1. Den Laufenden Puppet Master stoppen
```
# service puppetmaster stop
[ ok ] Stopping puppet master.
```

2. Löche (`clean`) das aktuelle server zertifikat.
```
# puppet cert clean puppet
Notice: Revoked certificate with serial 6
Notice: Removing file Puppet==SSL==Certificate puppet at../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/../puppet/sign../puppet/puppet.pem'
Notice: Removing file Puppet==SSL==Certificate puppet at../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/cer../puppet/puppet.pem'
Notice: Removing file Puppet==SSL==Key puppet at../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/private_ke../puppet/puppet.pem'
```

3. Erstellen Sie ein neues Puppet-Zertifikat mit `puppet cert` mit der Option --dns-alt-names :
```
root@puppet:~# puppet certificate generate puppet --dns-alt-names puppet.example.com,puppet.example.org,puppet.example.net --ca-location local
Notice: puppet has a waiting certificate request
true
```

4. Signiren Sie das neue Zertifikat
```
root@puppet:~# puppet cert --allow-dns-alt-names sign puppet
Notice: Signed certificate request for puppet
Notice: Removing file Puppet==SSL==CertificateRequest puppet at../puppet/v../puppet/l../puppet/pupp../puppet/s../puppet/../puppet/reques../puppet/puppet.pem'
```

5 Restart des Puppet master
```root@puppet:~# service puppetmaster restart
[ ok ] Restarting puppet master.

```
