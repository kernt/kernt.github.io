---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet Manifests mit git managen

Es ist eine großartige Idee, deine Puppenmanifeste in ein Versionskontrollsystem wie Git oder Subversion zu stellen (Git ist der De-facto-Standard für Puppet). Das gibt Ihnen einige Vorteile:

* Sie können Änderungen rückgängig machen und auf eine vorherige Version Ihres Manifests zurücksetzen

* Sie können mit neuen Features mit einem Zweig experimentieren

* Wenn mehrere Menschen Änderungen an den Manifesten vornehmen müssen, können sie sie selbstständig in ihren eigenen Arbeitskopien machen und dann später ihre Änderungen zusammenführen

* Sie können die git-Log-Funktion verwenden, um zu sehen, was geändert wurde, und wann (und von wem)

## Fertig werden

In diesem Abschnitt importieren wir Ihre vorhandenen Manifestdateien in Git. Wenn Sie ein Puppet-Verzeichnis in einem vorherigen Abschnitt erstellt haben, verwenden Sie bitte das vorhandene Manifest-Verzeichnis.

In diesem Beispiel erstellen wir ein neues Git-Repository auf einem Server, der von allen unseren Knoten zugänglich ist. Es gibt mehrere Schritte, die wir ergreifen müssen, um unseren Code in einem Git-Repository zu halten:

1. Installiere Git auf einem zentralen Server.
2. Erstellen Sie einen Benutzer, um Git auszuführen und das Repository zu besitzen.
3. Erstellen Sie ein Repository, um den Code zu halten.
4. Erstellen Sie SSH-Schlüssel, um den Schlüssel-basierten Zugriff auf das Repository zu ermöglichen.
5. Installiere Git auf einem Knoten und lade die neueste Version aus unserem Git Repository herunter.

## Wie es geht

Folge diesen Schritten:

1. Zuerst installiere Git auf deinem Git-Server (`git.example.com` in unserem Beispiel).
Der einfachste Weg, dies zu tun, ist die Verwendung von Puppet.
Erstellen Sie das folgende Manifest, nennen es `git.pp`:

```ruby
 package {'git':
    ensure => installed }
```

2.Wenden Sie diese manifest mit `puppet apply git.pp` an, wird dies Git installieren.

3.Als nächstes erstellen Sie einen Git-Benutzer, den die Nodes verwenden, um sich anzumelden und den letzten Code abzurufen. Wieder machen wir das mit Puppet. Wir erstellen auch ein Verzeichnis, in dem unser Repository../puppet/ho../puppet/g../puppet/repos) wie im folgenden Code-Snippet angezeigt wird:

```ruby
group { 'git': gid => 1111, }
user {'git': uid => 1111, gid => 1111, comment => 'Git User', home =>../puppet/ho../puppet/git', require => Group['git'], }
file ../puppet/ho../puppet/git': ensure => 'directory', owner => 1111, group => 1111, require => User['git'], }
file ../puppet/ho../puppet/g../puppet/repos': ensure => 'directory', owner => 1111, group => 1111, require => File../puppet/ho../puppet/git'] }
```

4.Nachdem Sie dieses Manifest angewendet haben, melden Sie sich als Git-Benutzer an und erstellen Sie ein leeres Git-Repository mit dem folgenden Befehl:

```s
sudo -iu git git@git
cd repos git@git
git init --bare puppet.git
[[Initialized]] empty Git repository i../puppet/ho../puppet/g../puppet/rep../puppet/puppet.g../puppet/
```

5.Setzen Sie ein Passwort für den Git-Benutzer, wir müssen uns nach dem nächsten Schritt remote anmelden:

```s
[root@git ~]# passwd git
Changing password for user git.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

6.Jetzt wieder auf Ihrem lokalen Rechner, erstellen Sie einen `ssh` Schlüssel für unsere Nodes zu verwenden, um das Repository zu aktualisieren:

7.Kopiere nun den neu erstellten öffentlichen Schlüssel in die Datei `authorized_keys`. Damit können wir , mit diesem neuen Schlüssel eine Verbindung zum Git-Server herstellen:

```s
t@mylaptop../puppet/.ssh $ ssh-copy-id -i git_rsa git@git.example.com
git@git.example.com's password:
Number of key(s) added: 1
```

8.Jetzt probieren Sie sich in die Maschine, mit: "ssh 'git@git.example.com'" anzumelden  und überprüfen Sie, um sicherzustellen, dass nur die Schlüssel, die Sie wollten hinzugefügt wurden.

9.Als nächstes konfigurieren Sie ssh, um Ihren Schlüssel zu verwenden, wenn Sie auf den Git-Server zugreifen und fügen Sie der Datei ../puppet/.s../puppet/config` Folgendes hinzu:

```s
Host git git.example.com
  User git
  IdentityFil../puppet/ho../puppet/thom../puppet/.s../puppet/git_rsa
```

10.Klonen Sie das Repo auf Ihre Maschine in ein Verzeichnis mit dem Namen Puppet (ersetzen Sie Ihren Servernamen, wenn Sie `git.example.com` nicht verwendet haben):

```s
t@mylaptop ~$ git clone git@git.example.com:rep../puppet/puppet.git
Cloning into 'puppet'...
warning: You appear to have cloned an empty repository.
Checking connectivity... done.
```

Wir haben ein Git-Repository erstellt.
Bevor wir irgendwelche Änderungen an dem Repository vornehmen, ist es eine gute Idee, Ihren Namen und E-Mail in Git zu setzen.
Ihr Name und Ihre E-Mail werden an jedes Commit angehängt, das Sie machen.

11.Wenn Sie in einem großen Team arbeiten, wissen, wer eine Veränderung gemacht hat, ist sehr wichtig; Verwenden Sie dazu folgendes Code-Snippet:

```s
t@mylaptop puppet$ git config --global user.email"thomas@narrabilis.com"
t@mylaptop puppet$ git config --global user.name "ThomasUphill"
```

12.Sie können Ihre Git-Einstellungen mit folgendem Snippet überprüfen:

```s
t@mylaptop ~$ git config --global --list
user.name=Thomas Uphill
user.email=thomas@narrabilis.com
core.editor=vim
merge.tool=vimdiff
color.ui=true
push.default=simple
```

13.Nun, da wir Git richtig konfiguriert haben, wechseln Sie das Verzeichnis in Ihrem Repository-Verzeichnis und erstellen ein neues Site-Manifest wie im folgenden Snippet gezeigt:

```s
t@mylaptop ~$ cd puppet
t@mylaptop puppet$ mkdir manifests
t@mylaptop puppet$ vim manifes../puppet/site.pp
node default {
  include base
}

```

14.Dieses Site-Manifest wird unsere Basisklasse auf jedem Knoten installieren.
Wir werden die Basisklasse mit dem Puppet-Modul erstellen, wie wir es gemacht haben in der Einleitung gemacht haben [Puppet4 Sprache und Syntax](puppet4-basics.md).

```s
t@mylaptop puppet$ mkdir modules
t@mylaptop puppet$ cd modules
t@mylaptop modules$ puppet module generate thomas-base
Notice: Generating module a../puppet/ho../puppet/tuphi../puppet/pupp../puppet/modul../puppet/thomas-base
thomas-base
thomas-ba../puppet/Modulefile
thomas-ba../puppet/README
thomas-ba../puppet/manifests
thomas-ba../puppet/manifes../puppet/init.pp
thomas-ba../puppet/spec
thomas-ba../puppet/sp../puppet/spec_helper.rb
thomas-ba../puppet/tests
thomas-ba../puppet/tes../puppet/init.pp
t@mylaptop modules$ ln -s thomas-base base
```

15.Als letzter Schritt schaffen wir eine symbolische Verknüpfung zwischen dem `Thomas-Base` Verzeichnis und dem `base` verzeichnis.
Um sicherzustellen, dass unser Modul etwas Nützliches macht, fügen Sie den Inhalt der `base` klasse hinzu, die in `thomas-ba../puppet/manifes../puppet/init.pp` definiert ist:

```ruby
class base {
  file ../puppet/e../puppet/motd':
    content => "${==fqdn}\nManaged by puppet ${==puppetversion}\n"
  }
}

```

16.Fügen Sie nun das neue Basismodul und das site manifest zu Git hinzu, indem Sie `git add` und `git commit` wie folgt verwenden:

```s
t@mylaptop modules$ cd ..
t@mylaptop puppet$ git add modules manifests
t@mylaptop puppet$ git status
On branch master
Initial commit
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
new file:   manifes../puppet/site.pp
new file:   modul../puppet/base
new file:   modul../puppet/thomas-ba../puppet/Modulefile
new file:   modul../puppet/thomas-ba../puppet/README
new file:   modul../puppet/thomas-ba../puppet/manifes../puppet/init.pp
new file:   modul../puppet/thomas-ba../puppet/sp../puppet/spec_helper.rb
new file:   modul../puppet/thomas-ba../puppet/tes../puppet/init.pp
t@mylaptop puppet$ git commit -m "Initial commit with simple base module"
[master (root-commit) 3e1f837] Initial commit with simple base module
 7 files changed, 102 insertions(+)
 create mode 100644 manifes../puppet/site.pp
 create mode 120000 modul../puppet/base
 create mode 100644 modul../puppet/thomas-ba../puppet/Modulefile
 create mode 100644 modul../puppet/thomas-ba../puppet/README
 create mode 100644 modul../puppet/thomas-ba../puppet/manifes../puppet/init.pp
 create mode 100644 modul../puppet/thomas-ba../puppet/sp../puppet/spec_helper.rb
 create mode 100644 modul../puppet/thomas-ba../puppet/tes../puppet/init.pp
```

17.Zu diesem Zeitpunkt wurden Ihre Änderungen an dem Git-Repository lokal begangen; Sie müssen jetzt diese Änderungen zurück zu git.example.com drücken, damit andere Knoten die aktualisierten Dateien abrufen können:

```s
t@mylaptop puppet$ git push origin master
Counting objects: 15, done.
Delta compression using up to 4 threads.
Compressing objects: 100% ../puppet/9), done.
Writing objects: 100% (../puppet/15), 2.15 KiB | 0 byt../puppet/s, done.
Total 15 (delta 0), reused 0 (delta 0)
To git@git.example.com:rep../puppet/puppet.git
 * [new branch]      master -> master

```

## Wie es funktioniert

Git-Tracks wechselt zu Dateien und speichert einen vollständigen Verlauf aller Änderungen.
Die Geschichte des Repo besteht aus Commits.
Ein Commit repräsentiert den Zustand des Repo zu einem bestimmten Zeitpunkt, den Sie mit dem Befehl `git commit` erstellen und mit einer Nachricht kommentieren.

Sie haben jetzt Ihre Puppet Manifest-Dateien zum Repo hinzugefügt und Ihr erstes Commit erstellt. Dies aktualisiert die Geschichte des Repo, aber nur in Ihrer lokalen Arbeitskopie. Um die Änderungen mit der `git.example.com` Kopie zu synchronisieren, drückt der Befehl `git push` alle Änderungen seit der letzten Synchronisierung.

## Es gibt mehr

Nun, da Sie ein zentrales Git-Repository für Ihre Puppet Manifeste haben, können Sie mehrere Kopien davon an verschiedenen Orten ausprobieren und an ihnen arbeiten, bevor Sie Ihre Änderungen begehen. Zum Beispiel, wenn Sie in einem Team arbeiten, kann jedes Mitglied eine eigene lokale Kopie des Repo und synchronisieren Änderungen mit den anderen über den zentralen Server. Sie können auch GitHub als Ihren zentralen Git-Repository-Server verwenden. GitHub bietet kostenlose Git-Repository-Hosting für öffentliche Repositories, und Sie können für GitHub's Premium-Service bezahlen, wenn Sie nicht möchten, dass Ihr Puppet-Code öffentlich zugänglich ist.

Im nächsten Abschnitt werden wir unser Git-Repository sowohl für zentrale als auch dezentrale Puppen-Konfigurationen nutzen.
