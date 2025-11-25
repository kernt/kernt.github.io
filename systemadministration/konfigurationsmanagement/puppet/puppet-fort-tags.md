---
tags:
  - konfigurationsmanagement
  - puppet
---
# Puppet für fortgeschrittene: tags benutzen

Manchmal muss eine Puppetklasse über einen andere wissen oder zumindest wissen, ob diese vorhanden ist oder nicht. Zum Beispiel muss eine Klasse, die die Firewall verwaltet, möglicherweise wissen, ob der Node ein Webserver ist oder nicht.

Die `tagged` Funktion wird Puppet sagen, ob eine benannte Klasse oder Ressource im Katalog für diesen Knoten vorhanden ist.
Sie können auch beliebige Tags an einen Node oder eine Klasse anwenden und auf das Vorhandensein dieser Tags prüfen.
Tags sind ein weiterer Metaparameter, ähnlich wie `require` und  `notify` wie in [Puppet4 Sprache und Syntax](puppet4-basics.md) eingeführt. Metaparameter werden in der Zusammenstellung des Puppet-Katalogs verwendet, sind aber kein Attribute der Ressource, an die sie angehängt sind.

## Wie es geht

Um Ihnen zu helfen herauszufinden, ob Sie auf einem bestimmten Knoten oder einer Klasse von Knoten laufen, werden alle Knoten automatisch mit dem Knotennamen und den Namen der Klassen, die sie enthalten, markiert. Hier ist ein Beispiel, das Ihnen zeigt, wie man mit dem Tag versehen hat, um diese Informationen zu erhalten:

1.Füge folgenden Code deiner `site.pp` Datei (ersetze cookbook mit dem Namen deiner machine):

```ruby
  node 'cookbook' {
    if tagged('cookbook') {
      notify { 'tagged cookbook': }
    }
  }
```

2.Run Puppet:

```ruby
root@cookbook:~# puppet agent -vt
Info: Caching catalog for cookbook
Info: Applying configuration version '1410848350'
Notice: tagged cookbook
Notice: Finished catalog run in 1.00 seconds
```

Nodes werden auch automatisch mit den Namen aller Klassen versehen, die sie zusätzlich zu mehreren anderen automatischen Tags enthalten.
Sie können `tagged` verwenden, um herauszufinden, welche Klassen auf dem Knoten enthalten sind.

Du bist nicht nur darauf beschränkt, die von Puppet automatisch angewandten Tags zu überprüfen. Sie können auch Ihre eigenen hinzufügen. Um ein beliebiges tag auf einem Knoten zu setzen, verwenden Sie die `tag` Funktion wie im folgenden Beispiel:

3.Passe deine `site.pp` Datei folgender maßen an:

```ruby
  node 'cookbook' {
    tag('tagging')
    class {'tag_test': }
  }
```

4.Fügen Sie ein `tag_test` Modul mit dem folgenden init.pp hinzu (oder seien Sie faul und fügen Sie die folgende Definition zu Ihrem site.pp hinzu):

```ruby
  class tag_test {
    if tagged('tagging') {
      notify { 'containing no../puppet/class was tagged.': }
    }
  }
```

5.Run Puppet

```s
root@cookbook:~# puppet agent -vt
Info: Caching catalog for cookbook
Info: Applying configuration version '1410851300'
Notice: containing no../puppet/class was tagged.
Notice: Finished catalog run in 0.22 seconds
```

6.Sie können auch Tags verwenden, um festzustellen, welche Teile des Manifests gelten sollen.
Wenn Sie die Option `--tags` auf der Puppet-Befehlszeile verwenden, wird Puppet nur diejenigen Klassen oder Ressourcen angewenden, die mit den einzelnen Tags versehen sind, die Sie enthalten. Zum Beispiel können wir unsere `cookbook` mit zwei Klassen definieren:

```ruby
  node cookbook {
    class {'first_class': }
    class {'second_class': }
  }
  class first_class {
    notify { 'First Class': }
  }
  class second_class {
    notify {'Second Class': }
  }
```

7.Jetzt, wenn wir `puppet run` auf dem `cookbook` Node laufen lassen, sehen wir beide wurden benachrichtigt:

```s
root@cookbook:~# puppet agent -t
Notice: Second Class
Notice: First Class
Notice: Finished catalog run in 0.22 seconds
```

8.Jetzt die `first_class` und add -tags-Funktion auf die Befehlszeile anwenden:

```s
root@cookbook:~# puppet agent -t --tags first_class
Notice: First Class
Notice: Finished catalog run in 0.07 seconds
```

## Es gibt mehr

Sie können Tags verwenden, um eine Sammlung von Ressourcen zu erstellen und dann die Sammlung als eine Abhängigkeit für eine andere Ressource zu nutzen.
Zum Beispiel, einige Services hangten von einer Konfigurationsdatei, die aus einer Reihe von Datei-Snippets gebaut wird ab , wie im folgenden Beispiel:

```ruby
  class firewall::service {
    service { 'firewall': ...
    }
    File <| tag == 'firewall-snippet' |> ~> Service['firewall']
  }
  class myapp {
    file {../puppet/e../puppet/firewall../puppet/myapp.conf': tag => 'firewall-snippet', ...
    }
  }
```

Hier haben wir angegeben, dass der `firewall` Service benachrichtigt werden soll, wenn eine Datei-Resource den Tag `firewall-snippet` aktualisiert wird.
Alles was wir tun müssen, um ein `firewall` Konfigurations-Snippet für eine bestimmte Anwendung oder einen Dienst hinzuzufügen, ist, es mit `firewall-snippet` zu markieren, und Puppet wird den Rest erledigen.

Obwohl wir eine `notify => Service ["firewall"]` Funktion zu jeder Snippet-Ressource hinzufügen könnten, wenn unsere Definition des Firewall-Dienstes jemals geändert werden müsste, müssten wir nacharbeiten und alle snippets entsprechend aktualisieren. Das tag lässt uns die Logik an einem Ort kapseln, was die zukünftige Wartung und das Refactoring wesentlich erleichtert.

### Hinweiss

Was ist `<| Tag == 'firewall-snippet' |> syntax` Dies wird als Resource Collector bezeichnet, und es ist eine Möglichkeit, eine Gruppe von Ressourcen zu spezifizieren, indem sie nach einem Stück Code über sie suchen; In diesem Fall der Wert eines Tags.
Sie können mehr über Ressourcensammler und die `<| |>` Puppet Betreiber (manchmal als Raumschiff-Operator bekannt) auf der [Puppet Labs Website](htt../puppet//docs.puppetlabs.c../puppet/pupp../puppet/3/referen../puppet/lang_collectors.html) finden.