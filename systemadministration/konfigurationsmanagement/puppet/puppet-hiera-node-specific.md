---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet Hiera node spezifikatioen

In unserer Hierarchie, die in `hiera.yaml` definiert ist, haben wir einen Eintrag erstellt, der auf der `hosts` gacts basiert.
In diesem Abschnitt erstellen wir yaml Dateien im Host Unterverzeichnis von Hiera mit Informationen, die für einen bestimmten spezifisch Host sind.

## Fertig werden

Installieren und konfigurieren Sie Hiera wie im letzten Abschnitt und verwenden Sie die im vorherigen Rezept definierte Hierarchie, die einen Eintrag `hos../puppet/%{hostname}` enthält.

## Wie es geht

Im Folgenden sind die Schritte beschrieben:

1.Erstellen Sie eine Datei unter../puppet/e../puppet/pupp../puppet/hierada../puppet/hosts`, die den Hostnamen Ihres Test Nodes endhelt. Zum Beispiel, wenn Ihr Host `cookbook-test` genannt wird, dann würde die Datei `cookbook-test.yaml` genannt werden.

2.Füge einen spezifische Nachricht indiese datei ein :

```s
message: 'This is the test node for the cookbook'
```

3.Führe puppet auf zwei unterschiedlichen nodes aus um den Unterschied zu sehen:

```s
t@ckbk:~$ sudo puppet agent -t
Info: Caching catalog for cookbook-test
Notice: Message is This is the test node for the cookbook
[root@hiera-test ~]# puppet agent -t
Info: Caching catalog for hiera-test.example.com
Notice: Message is Default Message
```

## Wie es funktioniert

Hiera sucht inder Hierarchie nach Dateien, die mit den von facter zurückgegebenen Werten übereinstimmen. In diesem Fall wird die Datei `cookbook-test.yaml` gefunden, indem man den Hostnamen des Knotens in den Suchpfad../puppet/e../puppet/pupp../puppet/hierada../puppet/hos../puppet/%{hostname}.yaml` einfügt.

Mit Hiera ist es möglich, die Komplexität Ihres Puppencodes stark zu reduzieren. Wir verwenden Yaml-Dateien für separate Werte, zuvor hatten Sie große Case-Anweisungen oder verschachtelte if-Anweisungen.
