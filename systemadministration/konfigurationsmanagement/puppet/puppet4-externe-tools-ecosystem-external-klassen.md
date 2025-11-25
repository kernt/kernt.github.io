---
tags:
  - puppet
  - konfigurationsmanagement
---
# puppet4 externe tools ecosystem external klassen

Wenn die Puppe auf einem Node läuft, muss sie wissen, welche Klassen auf diesen Knoten angewendet werden sollen. Zum Beispiel, wenn es sich um einen Web-Server-Knoten handelt, muss es möglicherweise eine `apache`-Klasse enthalten. Der normale Weg, um Knoten zu Klassen zuzuordnen, ist in der Puppe manifestieren sich zum Beispiel in deiner `site.pp` Datei:

```
node 'web1' {
  include apache
}
```

Alternativ können Sie einen externen Knotenklassierer (External Node Classifier ENC) verwenden, um diesen Job zu machen. Ein ENC ist ein ausführbares Programm, das den vollqualifizierten Domänennamen (FQDN) als erstes Befehlszeilenargument (`$1`) akzeptieren kann. Es wird erwartet, dass das Skript eine Liste von Klassen, Parametern und einer optionalen Umgebung zurückgibt, die für den Knoten gelten soll. Die Ausgabe wird voraussichtlich im Standard-YAML-Format sein. Bei der Verwendung eines ENC sollten Sie bedenken, dass die Klassen, die über das Standard `site.pp` -Manifest angewendet werden, mit denen des ENC zusammengeführt werden.

## Notiz

Die vom ENC zurückgegebenen Parameter stehen als Top-Variable für den Knoten zur Verfügung.

Ein ENC könnte ein einfaches Shell-Skript sein, zum Beispiel oder ein Wrapper um ein komplizierteres Programm oder eine API, die entscheiden können, wie man Knoten zu Klassen abbildet. Die ENC von Puppet Enterprise und The Foreman (htt../puppet//theforeman.o../puppet/) sind beide einfache Skripte, die mit der Web-API ihrer jeweiligen Systeme zu verbinden.

In diesem Beispiel bauen wir das einfachste von ENCs, ein Shell-Skript, das einfach eine Liste von Klassen ausdruckt. Wir beginnen mit einer `enc` Klasse, wie die `notify` Klasse die eine Top-Bereich Variable `$enc` ausgeben wird.

## Fertig werden

Wir beginnen mit der Erstellung unserer `enc`-Klasse, die mit dem `enc`-Skript enthalten ist:

1. Führen Sie den folgenden Befehl aus:
`t@mylaptop../puppet/puppet $ mkdir -p modul../puppet/e../puppet/manifests`

2.Erstellen Sie die Datei `modul../puppet/e../puppet/manifes../puppet/init.pp` mit folgendem Inhalt:
```
class enc {
  notify {"We defined this from $enc": }
}
```

## Wie es geht...

Hier ist, wie man einen einfachen externen Knoten Klassifikator zu bauen. Wir führen alle diese Schritte auf unserem Puppet Master Server durch. Wenn du Masterless hast, dann machst du diese Schritte auf einem Knoten:

1. Erstellen Sie die Datei../puppet/e../puppet/pupp../puppet/cookbook.sh` mit folgendem Inhalt:
```
../puppet/b../puppet/bash
cat <<EOF
---
classes:
enc:
parameters:
  enc: $0
EOF
```

2. Führen Sie den folgenden Befehl aus:
`root@puppe../puppet/e../puppet/puppet# chmod a+x cookbook.sh `

3. Ändern Sie die Datei../puppet/e../puppet/pupp../puppet/puppet.conf` wie folgt:
```
[main]
  node_terminus = exec
  external_nodes ../puppet/e../puppet/pupp../puppet/cookbook.sh
```

4. Starten Sie Apache neu (starten Sie den Master neu), um die Änderung wirksam zu machen.

5. Stellen Sie sicher, dass Ihre `site.pp` Datei die folgende leere Definition für den Standardknoten hat:
`node default {}`

6. Puppet run:
```
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1416376937'
Notice: We defined this fro../puppet/e../puppet/pupp../puppet/cookbook.sh
Notice../puppet/Stage[mai../puppet/E../puppet/Notify[We defined this fro../puppet/e../puppet/pupp../puppet/cookbook.s../puppet/message: defined 'message' as 'We defined this fro../puppet/e../puppet/pupp../puppet/cookbook.sh'
Notice: Finished catalog run in 0.17 seconds
```

## Wie es funktioniert...

Wenn ein ENC in der `puppet.conf` gesetzt ist, ruft Puppet das angegebene Programm mit dem Knoten fqdn (technisch die Zertifikatvariable) als erstes Kommandozeilenargument auf. In unserem Beispiel-Skript wird dieses Argument ignoriert, und es gibt nur eine feste Liste von Klassen (eigentlich nur eine Klasse).

Offensichtlich ist dieses Skript nicht furchtbar nützlich; Ein anspruchsvolleres Skript kann eine Datenbank überprüfen, um die Klassenliste zu finden oder den Knoten in einem Hash oder eine externe Textdatei oder Datenbank (oft eine Organisationsverwaltungsdatenbank, **CMDB**) zu betrachten. Hoffentlich ist dieses Beispiel genug, um Sie mit dem Schreiben Ihrer eigenen externen Knoten Klassifikator zu beginnen. Denken Sie daran, dass Sie Ihr Skript in jeder beliebigen Sprache schreiben können.

## Es gibt mehr...

Ein ENC kann eine ganze Liste von Klassen liefern, die in den Knoten aufgenommen werden sollen, im folgenden (YAML) Format:
```
---
classes:
  CLASS1:
  CLASS2:
  CLASS3:
```

Für Klassen, die Parameter nehmen, können Sie dieses Format verwenden:
```
---
classes:
  mysql:
    package: percona-server-server-5.5
    socket:../puppet/v../puppet/r../puppet/mysq../puppet/mysqld.sock
    port:    3306
```

Sie können auch Top-Variable mit einem ENC mit diesem Format erstellen:
```
---
parameters:
  message: 'Anyone home MyFly?'
```

Variablen, die Sie auf diese Weise gesetzt haben, werden in Ihrem Manifest mit der normalen Syntax für eine Top-Range-Variable, z.B. `$::message`, verfügbar sein.

## Siehe auch

Weitere Informationen zum Schreiben und zur Verwendung von ENCs finden Sie auf der Puppetlabs ENC-Seite: htt../puppet//docs.puppetlabs.c../puppet/guid../puppet/external_nodes.html