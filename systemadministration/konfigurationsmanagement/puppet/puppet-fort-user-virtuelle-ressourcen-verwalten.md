---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschrittene: 

Benutzer sind ein großartiges Beispiel für eine Ressource, die möglicherweise durch mehrere Klassen realisiert werden muss. Betrachten Sie die folgende Situation. Um die Verwaltung einer großen Anzahl von Maschinen zu vereinfachen, haben Sie Klassen für zwei Arten von Benutzern definiert: `developers` und `sysadmins`. Alle Maschinen müssen `sysadmins` einschließen, aber nur einige Maschinen benötigen `developers`:

```ruby
node 'server' {
  include user::sysadmins
}

node 'webserver' {
  include user::sysadmins
  include user::developers
}
```

Allerdings können einige Benutzer Mitglieder beider Gruppen sein. Wenn jede Gruppe ihre Mitglieder einfach als reguläre Benutzerressourcen deklariert, führt dies zu einem Konflikt, wenn ein Knoten sowohl Entwickler als auch `sysadmins` enthält, wie im `webserver` Beispiel.

Um diesen Konflikt zu vermeiden, besteht ein gemeinsames Muster darin, für alle Benutzer virtuelle Ressourcen zu erstellen, die in einem einzelnen Klassenbenutzer definiert sind `user::virtual`, dass jede Maschine enthält und dann die Benutzer verwirklicht, wo sie benötigt werden. Auf diese Weise wird es keinen Konflikt geben, wenn ein Benutzer Mitglied in mehreren Gruppen ist.

## Wie es geht

Gehen Sie folgendermaßen vor, um eine `user::virtual` Klasse zu erstellen:

1.Erstellen Sie die Datei `modu../puppet/us../puppet/manifes../puppet/virtual.pp` mit folgendem Inhalt:

```ruby
class user::virtual {
  @user { 'thomas':  ensure => present }
  @user { 'theresa': ensure => present }
  @user { 'josko':   ensure => present }
  @user { 'nate':    ensure => present }
}
```

2.Erstelle die Datei `modul../puppet/us../puppet/manifes../puppet/developers.pp` mit folgendem Inhalt:

```ruby
class user::developers {
  realize(User['theresa'])
  realize(User['nate'])
}
```

3.Erstelle die Datei `modul../puppet/us../puppet/manifes../puppet/sysadmins.pp` mit folgendem Inhalt:

```ruby
class user::sysadmins {
  realize(User['thomas'])
  realize(User['theresa'])
  realize(User['josko'])
}
```

4.Modifiziere deine `nodes.pp` Datei folgernder maßen:

```ruby
node 'cookbook' {
  include user::virtual
  include user::sysadmins
  include user::developers
}
```

5.Puppet Run

```s
cookbook# puppet agent -t
Info: Caching catalog for cookbook.example.com
Info: Applying configuration version '1413180590'
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/User[theres../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/User[nat../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/User[thoma../puppet/ensure: created
Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/User[josk../puppet/ensure: created
Notice: Finished catalog run in 0.47 seconds
```

## Wie es funktioniert

Wenn wir Klasse  `user::virtuall` includiren , werden alle Benutzer als virtuelle Ressourcen deklariert (weil es das `@` Zeichen enthalt):

```ruby
  @user { 'thomas':  ensure => present }
  @user { 'theresa': ensure => present }
  @user { 'josko':   ensure => present }
  @user { 'nate':    ensure => present }
```

Das heißt, die Ressourcen existieren im Puppets Katalog; Sie können von anderen Ressourcen verwiesen und verknüpft werden, und sie sind in jeder Hinsicht identisch mit regelmäßigen Ressourcen, außer dass Puppet nicht wirklich die entsprechenden Benutzer auf der Maschine erstellt.

Um dies zu erreichen, müssen wir die virtuellen Ressourcen mit `realize` umsetzen. Wenn wir `user::sysadmins` Klasse einschließen, erhalten wir den folgenden Code:

```ruby
  realize(User['thomas'])
  realize(User['theresa'])
  realize(User['josko'])
```

Der aufruf von `realize` auf einer virtuellen Ressource erzählt Puppet, "Ich möchte diese Ressource jetzt verwenden". Dies ist, was es tut, wie wir aus der Run-Ausgabe sehen können:
`Notice../puppet/Stage[mai../puppet/User::Virtu../puppet/User[theres../puppet/ensure: created`

Allerdings ist da ein in den `developers` und `sysadmins` in beiden Klassen! Wir  sollten `realize` nicht zwei mal auf der gleichen resource aufrufen.

```ruby
realize(User['theresa'])
...
realize(User['theresa'])
```

Ja, sind es, und das ist gut Sie sind explizit erlaubt, Ressourcen mehrmals zu verwirklichen, und es wird kein Konflikt geben. So lange, wir eine Klasse, irgendwo, aufrufe zu da ist ein Konto erkennen, wird es funktioniren. Unrealisierte Ressourcen werden bei der Katalogzusammenstellung einfach verworfen.

## Es gibt mehr

Wenn Sie dieses Muster verwenden, um Ihre eigenen Benutzer zu verwalten, sollte jeder Knoten die Klasse `user::virtual` als Teil Ihrer grundlegenden Haushaltungskonfiguration enthalten. Diese Klasse deklariert alle Benutzer (als virtuell) in Ihrer Organisation oder Website. Dazu gehören auch alle Benutzer, die nur zum Ausführen von Anwendungen oder Diensten (z. B. `apache`, `www-daten` oder `deploy`) existieren. Dann können Sie sie nach Bedarf auf einzelnen Knoten oder in bestimmten Klassen realisieren.

Für die Produktion verwenden Sie wahrscheinlich auch eine UID und GID für jeden Benutzer oder jede Gruppe, so dass diese numerischen Identifikatoren über Ihr Netzwerk synchronisiert werden. Sie können dies mit den `uid` - und `gid`-Parametern für die Benutzerressource tun.

## Hinweis

Wenn Sie die UID des Benutzers nicht angeben, erhalten Sie beispielsweise nur die nächste ID-Nummer, die auf einer bestimmten Maschine verfügbar ist, so dass derselbe Benutzer auf verschiedenen Rechnern eine andere UID hat. Dies kann zu Berechtigungsproblemen bei der Verwendung von gemeinsam genutztem Speicher oder zum Verschieben von Dateien zwischen Maschinen führen.

Um Ein gemeinsames Muster ist es eine möglichkeit bei der Definition von Benutzern als virtuelle Ressourcen ,und  den Benutzern anhand der zugewiesenen Rollen innerhalb Ihrer Organisation Tags zuzuordnen. Sie können dann die 
`collector` Syntax verwenden, anstatt zu `realize`, um Benutzer mit bestimmten Tags zu sammeln.

Zum Beispiel sehen Sie das folgende Code-Snippet:

```ruby
@user { 'thomas':  ensure => present, tag => 'sysadmin' }
@user { 'theresa': ensure => present, tag => 'sysadmin' }
@user { 'josko':   ensure => present, tag => 'dev' }
User <| tag == 'sysadmin' |>
```

Im vorherigen Beispiel würden nur Benutzer `thomas` und `theresa` aufgenommen.