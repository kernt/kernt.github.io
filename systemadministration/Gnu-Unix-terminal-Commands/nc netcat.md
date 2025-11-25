---
tags:
  - system-administration
  - gnu-tools
  - networking
---
| option | Beschreibung |
| ------ | ------------ |
|        |              |


**Port-Bereich scannen**

```sh
nc -zv localhost 22-25 

nc: connect to localhost port 22 (tcp) failed: Connection refused
nc: connect to localhost port 23 (tcp) failed: Connection refused
nc: connect to localhost port 24 (tcp) failed: Connection refused
Connection to localhost 25 port [tcp/smtp] succeeded!
```

