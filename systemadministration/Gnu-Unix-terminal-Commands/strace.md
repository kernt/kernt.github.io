---
tags:
  - konsole
  - bash
  - strace
  - system-administration
  - gnu-tools
---
# strace

startet programm und gibt dessen Systemaufrufe auf dem Bildschirm aus

`strace programm`

startet programm und gibt dessen Systemaufrufe auf dem Bildschirm aus mit dem user mustermann

`strace -u mustermann programm`

Ausgabe in prog.log Logdatei

`strace -o prog.log programm`

verfolgt auch Kindprozesse

`strace -f -o prog.log programm`

verfolge die Aufrufe des laufenden Prozesses mit Prozess-ID pid

`strace -p pid`

gibt nur Systemaufrufe aus, die das Dateimanagement betreffen

`strace -e trace=open,close,read,write`

### File activity

Strace can monitor file related activity. There are two useful parts. The first is **file**, which shows file interactions. The other one allows tracing **file descriptors**. Both can be used to monitor for actions like opening files, reading/writing and closing. Usually using “trace=file” provides enough insights. If you really need more insights in the way a program deals with file descriptors, then use the second one.

- Monitor opening of files: strace -e open -p 1234
- See all file activity: strace -e trace=file -p 1234 **or** strace -e trace=desc -p 1234

If you want to track specific paths, use 1 or more times the **-P** parameter, following by the path.

```fallback
# strace -P /etc/cups -p 2261
```

# Network-related actions

Strace definitely can be useful for revealing more details about network traffic. Very useful to determine what network related connections are used, like when building your Docker image.

`strace -e trace=network`

### Memory calls

To get better insights on the memory usage and system calls, strace can monitor for these as well. They are nicely grouped in the memory group.

`strace -e trace=memory`

Quellen:

* [mit-strace-prufen-was-ein-programm-so-treibt](http://www.effinger.org/blog/2010/05/08/mit-strace-prufen-was-ein-programm-so-treibt/)
* [strace](https://strace.io/)