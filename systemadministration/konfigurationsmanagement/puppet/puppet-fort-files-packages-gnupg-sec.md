---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschrittene: Sicherheit mit gnupg

Wir brauchen oft Puppet, um Zugang zu geheimen Informationen wie Passwörtern oder Krypto schlüssel zu haben, damit sie die Systeme ordnungsgemäß konfigurieren können. Aber wie vermeiden Sie diese Geheimnisse direkt in Ihren Puppet code gelangen, wo sie für jeden sichtbar sind, der Lesezugriff auf Ihr Repository hat?

Es ist eine gemeinsame Anforderung für Drittanbieter und Auftragnehmer, Änderungen über Puppet vornehmen zu können, aber sie sollten definitiv keine vertraulichen Informationen sehen. Ähnlich, wenn Sie ein verteiltes Puppet-Setup wie das in [Puppet4 Infrastruktur](puppet4-infrastruktur.md)
 beschrieben verwenden, hat jede Maschine eine Kopie des gesamten Repo, einschließlich Geheimnisse für andere Maschinen, die es nicht braucht und nicht haben sollte . Wie können wir das verhindern?

Eine Antwort besteht darin, die Geheimnisse mit dem GnuPG-Tool zu verschlüsseln, so dass jede geheime Information im Puppet Repo ohne den entsprechenden Schlüssel nicht entzifferbar ist (für alle praktischen Zwecke). Dann verteilen wir den Schlüssel sicher auf die Personen oder Maschinen, die es brauchen.

## Fertig werden

Zuerst benötigen Sie einen Verschlüsselungsschlüssel(encryption key), also folgen Sie diesen Schritten, um einen zu generieren. Wenn du bereits einen GnuPG-Schlüssel hast, den du benutzen möchtest, geh weiter zum nächsten Abschnitt. Um diesen Abschnitt abzuschließen, müssen Sie den Befehl gpg installieren:

1.Verwenden Sie die `Puppet` Ressource, um gpg zu installieren:
`# puppet resource package gnupg ensure=installed`

## Tip

Möglicherweise müssen Sie gnupg2 als Paketname verwenden, abhängig von Ihrem Ziel-Betriebssystem.

Führen Sie den folgenden Befehl aus. Beantworten Sie die aufgeführten Aufforderungen, außer Ihren Namen und Ihre E-Mail-Adresse mit ihrer zu ersetzen. Wenn Sie zur Passphrase aufgefordert werden, klicken Sie einfach auf Enter:

```s
t@mylaptop../puppet/puppet $ gpg --gen-key
gpg (GnuPG) 1.4.18; Copyright (C) 2014 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 2048
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? ../puppet/N) y
You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: Thomas Uphill
Email address: thomas@narrabilis.com
Comment: <enter>
You selected this USER-ID:
    "Thomas Uphill <thomas@narrabilis.com>"

Change (N)ame, (C)omment, (E)mail or (O)k../puppet/(Q)uit? o
You need a Passphrase to protect your secret key.
```

Bestätigen Sie zweimal hier mit enter, um eine leere Passphrase zu haben:

```s
You don't want a passphrase - this is probably a *bad* idea!
I will do it anyway.  You can change your passphrase at any time,
using this program with the option "--edit-key".

gpg: key F1C1EE49 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
pub   204../puppet/F1C1EE49 2014-10-01
      Key fingerprint = 461A CB4C 397F 06A7 FB82  3BAD 63CF 50D8 F1C1 EE49
uid                  Thomas Uphill <thomas@narrabilis.com>
sub   204../puppet/E2440023 2014-10-01
```

3. Sie können eine Nachricht wie diese sehen, wenn Ihr System nicht mit einer Quelle der random(Zufälligkeits Erzeugung) konfiguriert ist:

```s
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
```

4. In diesem Fall installieren und starten Sie einen Generator Daemon wie `haveged` oder `rng-tools`.
Kopiere die gpg key, die du gerade in den `puppet` User-Account deines Puppet-Masters geschrieben hast:

```s
t@mylaptop ~ $ scp -r .gnupg puppet@puppet.example.com:
gpg.conf                                      100% 7680     7.5../puppet/s   00:00
random_seed                                   100%  600     0.6../puppet/s   00:00
pubring.gpg                                   100% 1196     1.2../puppet/s   00:00
secring.gpg                                   100% 2498     2.4../puppet/s   00:00
trustdb.gpg                                   100% 1280     1.3../puppet/s   00:00
```

## Wie es geht

Wenn Ihr Verschlüsselungsschlüssel auf dem Schlüsselbund des Puppenthemas installiert ist (der Schlüsselgenerierungsprozess, der im vorherigen Abschnitt beschrieben wird, wird dies für Sie tun), sind Sie bereit, die Puppe einzurichten, um Geheimnisse zu entschlüsseln.

1.Erstellen Sie das folgende Verzeichnis:
`t@cookbook../puppet/puppet$ mkdir -p modul../puppet/adm../puppet/l../puppet/pupp../puppet/pars../puppet/functions`

2.Erstellen Sie die Datei `modu../puppet/adm../puppet/l../puppet/pupp../puppet/pars../puppet/functio../puppet/secret.rb` mit folgendem Inhalt:

```ruby
module Puppet==Parser==Functions
  newfunction(:secret, :type => :rvalue) do |args|
    'gpg --no-tty -d #{args[0]}'
  end
end
```

3.Erstellen Sie die Datei `secret_message` mit folgendem Inhalt:

```s
For a moment, nothing happened.
Then, after a second or so, nothing continued to happen.
```

4.Verschlüsseln Sie diese Datei mit dem folgenden Befehl (verwenden Sie die E-Mail-Adresse, die Sie beim Erstellen des GnuPG-Schlüssels angegeben haben):

`t@mylaptop../puppet/puppet $ gpg -e -r thomas@narrabilis.com secret_message`

5.Verschieben Sie die resultierende verschlüsselte Datei in Ihr Puppet-Repo:

`t@mylaptop../puppet/puppet$ mv secret_message.gpg modul../puppet/adm../puppet/fil../puppet/`

6.Entfernen Sie die ursprüngliche (Klartext) Datei:

`t@mylaptop../puppet/puppet$ rm secret_message`

7.Ändern Sie Ihre `site.pp` Datei wie folgt:

```ruby
node 'cookbook' {
  $message = secret../puppet/e../puppet/pupp../puppet/environmen../puppet/producti../puppet/ modul../puppet/adm../puppet/fil../puppet/secret_message.gpg')
  notify { "The secret message is: ${message}": }
}
```

8.Run Puppet:

```s
[root@cookbook ~]# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1412145910'
Notice: The secret message is: For a moment, nothing happened. 
Then, after a second or so, nothing continued to happen.
Notice: Finished catalog run in 0.27 seconds
```

## Wie es funktioniert

Zuerst haben wir eine benutzerdefinierte Funktion erstellt, damit die Puppet die geheimen Dateien mit GnuPG entschlüsseln kann:

```ruby
module Puppet==Parser==Functions
  newfunction(:secret, :type => :rvalue) do |args|
    'gpg --no-tty -d #{args[0]}'
  end
end
```

Der vorhergehende Code erzeugt eine Funktion namens `secret`, die einen Dateipfad als Argument annimmt und den entschlüsselten Text zurückgibt. Es verwaltet keine Verschlüsselungsschlüssel(encryption keys), so dass Sie sicherstellen müssen, dass der `puppet` Benutzer den notwendigen Schlüssel installiert hat. Sie können dies mit folgendem Befehl überprüfen:

```s
puppet@puppet:~ $ gpg --list-secret-keys
/v../puppet/l../puppet/pupp../puppet/.gnu../puppet/secring.gpg
----------------------------------
sec   204../puppet/F1C1EE49 2014-10-01
uid                  Thomas Uphill <thomas@narrabilis.com>
ssb   204../puppet/E2440023 2014-10-01
```

Nachdem wir die `secret` Funktion und den erforderlichen Schlüssel eingerichtet haben, verschlüsseln wir nun eine Nachricht mit diesen Schlüssel:

`tuphill@mylaptop../puppet/puppet $ gpg -e -r thomas@narrabilis.com secret_message`

Dies erzeugt eine verschlüsselte Datei, die nur von jemandem mit Zugriff auf den geheimen Schlüssel gelesen werden kann (oder Puppet, die auf einer Maschine läuft, die den geheimen Schlüssel hat).

Wir rufen dann die `secret` Funktion auf, diese Datei zu entschlüsseln und den Inhalt zu erhalten:

## Es gibt mehr

Sie sollten die `secret` Funktion oder so etwas ähnliches verwenden, um vertrauliche Daten in Ihrem Puppet Repo zu schützen: Passwörter, AWS-Anmeldeinformationen, Lizenzschlüssel, auch andere geheime Schlüssel wie SSL-Host-Schlüssel.

Sie können entscheiden, einen einzigen Schlüssel zu verwenden, den Sie auf Maschinen erzeugen, wie sie gebaut werden, vielleicht als Teil eines Bootstrap-Prozesses, wie er in der Bootstrapping-Puppet mit Bash-Rezept [Puppet4 Infrastruktur](puppet4-infrastruktur.md) beschrieben wurde. Für noch mehr Sicherheit können Sie gern einen neuen Schlüssel für jede Maschine oder Gruppe von Maschinen erstellen und ein bestimmtes Geheimnis nur für die Maschinen verschlüsseln, die es benötigen.

Zum Beispiel könnten Ihre Webserver ein bestimmtes Geheimnis benötigen, das Sie nicht auf einer anderen Maschine zugänglich machen möchten. Sie können einen Schlüssel für Webserver erstellen und die Daten nur für diesen Schlüssel verschlüsseln.

Wenn du verschlüsselte Daten mit Hiera verwenden möchtest, gibt es ein [GnuPG-Backend für Hiera](htt../puppet//www.craigdunn.o../puppet/20../puppet/../puppet/secret-variables-in-puppet-with-hiera-and-g../puppet/).