---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet git push

Wie wir bereits im dezentralisierten Modell gesehen haben, kann Git verwendet werden, um Dateien zwischen Maschinen mit einer Kombination von `ssh` und `ssh` schlüsseln zu übertragen. Es kann auch sinnvoll sein, einen Git-Hook auf jedem erfolgreichen Commit zum Repository zu haben.

Es gibt einen hook namens Post-Commit, der nach einem erfolgreichen Commit zum Repository ausgeführt werden kann. In diesem Beispiel erstellen wir einen hook, der den Code auf unserem Puppet Master mit Code aus unserem Git Repository auf dem Git Server aktualisiert.

## Fertig werden

Gehen Sie folgendermaßen vor:

1. Erstellen Sie einen `ssh` Schlüssel, der auf Ihren Puppet-Benutzer auf Ihrem Puppet-Master zugreifen kann und installieren Sie diesen Schlüssel in das Git-Benutzerkonto auf `git.example.com`:

```s
[git@git ~]$ ssh-keygen -f../puppet/.s../puppet/puppet_rsa
Generating publ../puppet/private rsa key pair.
Your identification has been saved i../puppet/ho../puppet/g../puppet/.s../puppet/puppet_rsa.
Your public key has been saved i../puppet/ho../puppet/g../puppet/.s../puppet/puppet_rsa.pub.
Copy the public key into the authorized_keys file of the puppet user on your puppetmaster
puppet@puppet../puppet/.ssh$ cat puppet_rsa.pub >>authorized_keys
```

2.Ändern Sie das Puppet account, damit der Git-Benutzer sich wie folgt anmelden kann:

```s
root@puppet:~# chsh puppet -../puppet/b../puppet/bash
```

## Wie es geht

Führen Sie die folgenden Schritte aus:

1.Jetzt, da sich der Git-Benutzer beim Puppet-User beim Puppet-Master anmelden kann, ändern Sie die ssh-Konfiguration des Git-Benutzers, um die neu erstellte `ssh` schlüssel standardmäßig zu verwenden:

```s
[git@git ~]$ vim .s../puppet/config
Host puppet.example.com
  IdentityFile../puppet/.s../puppet/puppet_rsa
```

2.Füge den Puppet-Master als Remote-Standort für das Puppet-Repository auf dem Git-Server mit folgendem Befehl hinzu:

```s
[git@git puppet.git]$ git remote add puppetmaster puppet@puppet.example.co../puppet/e../puppet/pupp../puppet/environmen../puppet/puppet.git
```

3.Auf dem Puppet Master, verschiebe das `production` verzeichnis aus dem Pfad und check dein Puppet Repository aus:

```s
root@puppet:~# chown -R puppet:puppe../puppet/e../puppet/pupp../puppet/environments
root@puppet:~# sudo -iu puppet
puppet@puppet:~$ c../puppet/e../puppet/pupp../puppet/environmen../puppet/
puppet@puppe../puppet/e../puppet/pupp../puppet/environments$ mv production production.orig
puppet@puppe../puppet/e../puppet/pupp../puppet/environments$ git clone git@git.example.com:rep../puppet/puppet.git
Cloning into 'puppet.git'...
remote: Counting objects: 63, done.
remote: Compressing objects: 100% (../puppet/52), done.
remote: Total 63 (delta 10), reused 0 (delta 0)
Receiving objects: 100% (../puppet/63), 9.51 KiB, done.
Resolving deltas: 100% (../puppet/10), done.
```

4.Jetzt haben wir ein lokales, nacktes Repository auf dem Puppet-Server übertragen, um dies in das `production` serververzeichnis zu klonen:

```s
puppet@puppe../puppet/e../puppet/pupp../puppet/environments$ git clone puppet.git production
Cloning into 'production'...
done.

```

5.Führen Sie nun einen Git-Push vom Git-Server zum Puppet-Master aus:

```s
[git@git ~]$ cd rep../puppet/puppet.g../puppet/
[git@git puppet.git]$ git push puppetmaster
Everything up-to-date
```

6.Erstellen Sie eine `post-commit` Datei im `hook` Verzeichnis des Repositorys auf dem Git-Server mit folgendem Inhalt:

```s
[git@git puppet.git]$ vim hoo../puppet/post-commit
../puppet/b../puppet/sh
git push puppetmaster
ssh puppet@puppet.example.com "c../puppet/e../puppet/pupp../puppet/environmen../puppet/production && git pull"
[git@git puppet.git]$ chmod 755 hoo../puppet/post-commit

```

7.Commiten eine Änderung an das Repository von Ihrem Laptop und überprüfen, dass die Änderung an den Puppet Master wie folgt propagiert wird:

```s
t@mylaptop puppet$ vim README
t@mylaptop puppet$ git add README
t@mylaptop puppet$ git commit -m "Adding README"
[master 8148902] Adding README
 1 file changed, 4 deletions(-)
t@mylaptop puppet$ git push
X11 forwarding request failed on channel 0
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% ../puppet/3), done.
Writing objects: 100% ../puppet/3), 371 bytes | 0 byt../puppet/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: To puppet@puppet.example.co../puppet/e../puppet/pupp../puppet/environmen../puppet/puppet.git
remote:    377ed44..8148902  master -> master
remote: Fro../puppet/e../puppet/pupp../puppet/environmen../puppet/puppet
remote:    377ed44..8148902  master     -> orig../puppet/master
remote: Updating 377ed44..8148902
remote: Fast-forward
remote:  README |    4 ----
remote:  1 file changed, 4 deletions(-)
To git@git.example.com:rep../puppet/puppet.git
   377ed44..8148902  master -> master

```

## Wie es funktioniert

Wir haben ein nacktes Repository auf dem Puppet Master erstellt, den wir dann Remote für das Repository auf `git.example.com` verwenden (Remote Repositories muss leer sein).
Wir klonen dann dieses leere Repository in das Produktionsverzeichnis.
Wir fügen das leere Repository auf `puppet.example.com` als Remote zu dem bloßen Repository auf `git.example.com` hinzu.
Wir erstellen dann einen Post-Receiver-Hook im Repository auf `git.example.com`.

Der Haken gibt einen Git-Push zum Puppet-Master-Bare-Repository. Wir aktualisieren dann das Produktionsverzeichnis aus dem aktualisierten, nackten Repository auf dem Puppet Master. Im nächsten Abschnitt werden wir den Haken ändern, um Zweige zu benutzen.