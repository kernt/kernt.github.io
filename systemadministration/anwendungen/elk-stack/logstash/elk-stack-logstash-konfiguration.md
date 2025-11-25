---
tags:
  - elk
  - elk-stack
  - logstash
---
# elk stack logstash konfiguration

Eine allgemeine Logstash-Plugin-Konfiguration wie folgt aus:

```c
input {
  
}

filter {
  
}

output {
  
}
```

Eine Logstash-Konfiguration besteht aus einer Reihe von `input`-, `filter`- und `output`-Plugins und deren entsprechenden Eigenschaften. Jedes Plugin spielt eine wichtige Rolle beim Analysieren, Verarbeiten und schließlich beim Einlegen der Daten in das erforderliche Format. Input-Plugins generieren das Ereignis, Filter modifizieren sie, und die Ausgabe liefert sie an andere Systeme.

![Logstash plugins](https://www.packtpub.com/graphics/9781787288546/graphics/Ch03_01.jpg)