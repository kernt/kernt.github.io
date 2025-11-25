---
tags:
  - apache
---
# Apache starten als unprivilegierter Benutzer

```sh
sudo setcap 'cap_net_bind_service=+ep' /usr/sbin/apache2
sudo /etc/init.d/apache2 stop
sudo chown -R www-data: /var/{log,run}/apache2/
sudo -u www-data apache2ctl start
```
## Apache Reverse Proxy Server

**Quelle**
`https://askubuntu.com/questions/694036/apache-as-non-root`
