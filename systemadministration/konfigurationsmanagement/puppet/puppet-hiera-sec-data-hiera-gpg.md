---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet hiera sicherheit mit gpg

Wenn Sie Hiera verwenden, um Ihre Konfigurationsdaten zu speichern, gibt es einen gem, der **hiera-gpg** genannt wird, der ein Hier verschlüsseltes Backend hinzufügt, damit Sie die in Hiera gespeicherten Werte schützen können.

## Fertig werden

Gehen Sie folgendermaßen vor, um hiera-gpg einzurichten:

1.Installiere das `ruby-dev` Paket; Es wird erforderlich sein, das `hiera-gpg` gem wie folgt zu bauen:

```s
root@puppet:~# puppet resource package ruby-dev ensure=installed
Notice../puppet/Package[ruby-de../puppet/ensure: ensure changed 'purged' to 'present'
package { 'ruby-dev':
  ensure => '1:1.9.3',
}
```

2.Installiere `hiera-gpg` gem  mit hilfe von gem Provider.

```s
root@puppet:~# puppet resource package hiera-gpg ensure=installed provider=gem
Notice../puppet/Package[hiera-gp../puppet/ensure: created
package { 'hiera-gpg':
  ensure => ['1.1.0'],
}

```

3.Modifizire deine `hiera.yml` folgender maßen:

```s
    :hierarchy:
        - secret
        - common
    :backends:
        - yaml
        - gpg
    :yaml:
        :datadir:../puppet/e../puppet/pupp../puppet/hieradata'
    :gpg:
        :datadir:../puppet/e../puppet/pupp../puppet/secret'
```

## Wie es geht

In diesem Beispiel erstellen wir ein Stück verschlüsselter Daten und rufen es mit hiera-gpg wie folgt ab:

1.Erstellen Sie die Datei secret.yaml unter../puppet/e../puppet/pupp../puppet/secret` mit folgendem Inhalt:

```s
top_secret: 'Val Kilmer'
```

2.Wenn Sie noch keinen GnuPG-Verschlüsselungsschlüssel haben, folgen Sie den Schritten in der Verwendung von GnuPG, um ihre Geheimen Daten zu verschlüsseln [Arbeiten mit Dateien und Paketen](puppet4-datein-packete.md)

3.Verschlüsseln Sie die `secret.yaml` Datei mit dem folgenden Befehl zu diesem Schlüssel (ersetzen Sie die puppet@puppet.example.com mit der E-Mail-Adresse, die Sie beim Erstellen des Schlüssels angegeben haben). Dies wird die `secret.gpg` Datei erstellen:

```s
root@puppe../puppet/e../puppet/pupp../puppet/secret# gpg -e -o secret.gpg -r puppet@puppet.example.com secret.yaml
root@puppe../puppet/e../puppet/pupp../puppet/secret# file secret.gpg
secret.gpg: GPG encrypted data
```

4.Entverne die plaintext datei `secret.yaml`
`root@puppe../puppet/e../puppet/pupp../puppet/secret# rm secret.yaml`

5.Modifiziere dein default node in der `site.pp` datei folgender maßen.

```ruby
node default {
  $message = hiera('top_secret','Deja Vu')
  notify { "Message is $message": }
}
```

6.Jetzt an einen node puppet run durchführen

```s
[root@hiera-test ~]# puppet agent -t
Info: Caching catalog for hiera-test.example.com
Info: Applying configuration version '1410508276'
Notice: Message is Deja Vu
Notice../puppet/Stage[mai../puppet/Ma../puppet/Node[defaul../puppet/Notify[Message is Deja V../puppet/message: defined 'message' as 'Message is Deja Vu'
Notice: Finished catalog run in 0.08 seconds
```

## Wie es funktioniert

Wenn du `hiera-gpg` installierst, fügt es Hiera hinzu, die Fähigkeit, .gpg-Dateien zu entschlüsseln. So können Sie alle geheimen Daten in eine .yaml-Datei setzen, die Sie dann mit GnuPG auf den entsprechenden Schlüssel verschlüsseln. Nur Maschinen, die den richtigen Geheimschlüssel haben, können auf diese Daten zugreifen.

Zum Beispiel können Sie das MySQL-Root-Passwort mit dem `hiera-gpg` verschlüsseln und den entsprechenden Schlüssel nur auf Ihren Datenbankservern installieren. Obwohl andere Maschinen auch eine Kopie der `secret.gpg` Datei haben können, ist sie nicht lesbar, wenn sie nicht den Entschlüsselungsschlüssel haben.

## Es gibt mehr

Vielleicht möchten Sie auch über `hiera-eyaml`, ein weiteres Geheimdaten-Backend für Hiera wissen, das die Verschlüsselung einzelner Werte innerhalb einer Hiera-Datendatei unterstützt. Dies könnte praktisch sein, wenn Sie verschlüsselte und unverschlüsselte Fakten in einer einzigen Datei mischen müssen. Erfahren Sie mehr über [hiera-eyaml](http../puppet//github.c../puppet/TomPoult../puppet/hiera-eyaml).