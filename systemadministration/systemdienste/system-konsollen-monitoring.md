---
tags:
  - monitoring
  - sar
  - iostat
  - mpstat
  - vmstat
---
# Monitoring

## SAR

SAR System CPU I/O Statistiken 3 mal mit einem 1 sec Intervall

`sar -b 1 3`
## iostst

IO Geräte Statistiken

`iostat -p sda`
## mpstat

Processor Statistiken

Alle Infos

`mpstat -A`

Cpu ode cores

`mpstat -P ALL`
## vmstat

Virtual Memory Statistiken

Alle 2 sec 10 mal

`vmstat 2 10`

### Quellen

* [monitoring-das-maechtigste-werkzeug](https://www.informatik-aktuell.de/entwicklung/methoden/monitoring-das-maechtigste-werkzeug-fuer-cloud-microservices-und-business.html)
* [cachethq The open source status page system](https://cachethq.io/)
* [ubuntu-server-with-prometheus](https://www.howtoforge.com/tutorial/monitor-ubuntu-server-with-prometheus/)
* [prometheus](../prometheus)