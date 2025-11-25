---
tags:
  - puppet
  - konfigurationsmanagement
  - templates
---
# puppet4 templates

Vorlagen sind eine leistungsstarke Möglichkeit, Embedded Ruby (ERB) zu verwenden, um Konfigurationsdateien dynamisch zu erstellen. 
Sie können die ERB-Syntax auch direkt verwenden, ohne eine separate Datei verwenden zu müssen, indem Sie die Funktion `inline_template` aufrufen. 
ERB erlaubt Ihnen, eine bedingte Logik zu verwenden, über Arrays zu iterieren und Variablen einzuschließen.

## Praktische Umsetzung 

Hier ist eine Beispiel wie man `inline_template` einsetzt :

Füge deinen Ruby code  als `inline_template` code wie folgt ein :

```
cron { 'chkrootkit':
  command =>../puppet/u../puppet/sb../puppet/chkrootkit >
  ../puppet/v../puppet/l../puppet/chkrootkit.log 2>&1',
  hour    => inline_template('<%= @hostname.sum % 24 %>'),
  minute  => '00',
}
```
## Wie es funktioniert

Alles innerhalb des strings nach 'inline_template' wird ausgeführt als wenn es ein ERB template wäre.
Bei dem verwendetem Beispiel ist der Code zwischen `<%=` und `%>` der Teil der als Template vom Puppet gelesen wird.

Bei dem Beispiel nutzen wir `inline_template` um einen scheduled jof für jeden node zu einen anderen Stunde auszuführen , damit nicht alle gleichzeitig beginnen.
Weiteres ist unter dem begriff `Distributing cron jobs efficiently ` zu finden und unter [Puppet4 Management von Ressourcen und Dateien](puppet4-ressourcen-datein.md) behandelt.

Im ERB-Code, ob innerhalb einer Vorlagendatei oder einem `inline_template` string , kann auf unsere Puppet variablen zugegriffen werden indem man das `@` Zeichen voranstellt.
Zum Beispiel 
`<%= @fqdn %>`

Um Variablen in einem anderen Bereich zu verweisen, verwenden Sie `scope.lookupvar` wie folgt:
```
<%= "The value of something from otherclass is " + scope.lookupvar('otherclass::something') %>
```

Sie sollten inline templates verwenden. 
Wenn Sie wirklich brauchen, um eine komplizierte Logik in Ihrem Manifest zu verwenden, sollten Sie mit einer benutzerdefinierten Funktion statt inline templates zu verwenden ein Grund dafür ist z.B die Performance des Systems. siehe auch [Puppet4 Externe Tools und das Puppet Ecosystem](puppet4-externe-tools-ecosystem.md).





Siehe Auch :
* [Puppet4 Arbeiten mit Dateien und Paketen](puppet4-datein-packete.md)


