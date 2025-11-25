---
tags:
  - cockpit
  - system-administration
  - firewall-cmd
  - systemctl
---
# Cockpit Installation

```sh
dnf -y install cockpit
systemctl enable --now cockpit.socket
```

**Eingehend Port 9090/tcp freigeben**

```sh
firewall-cmd --permanent --add-service=cockpit
firewall-cmd --reload
```

**Performance-Metriken:**

```sh
dnf -y install cockpit-pcp # Collecting performance metrics
systemctl restart pmlogger
systemctl enable --now pmlogger.service pmproxy.service
```

**Session-Recording einzurichten**

```sh
dnf -y install cockpit-session-recording
systemctl enable --now sssd
```

**auf der Konsole einzelne Sessions aufzunehmen und wieder abzuspielen**

```sh
tlog-rec --file-path=tlog.log

# do your work

tlog-play --file-path=tlog.log
```
# cockpit Konfiguration 

**Systemctl aktivieren**

```sh
sudo systemctl start cockpit
​sudo systemctl enable cockpit.socket
```

**firewalld**

```sh
sudo firewall-cmd --add-service=cockpit
​sudo firewall-cmd --add-service=cockpit --permanent
​sudo firewall-cmd --reload
```

[resolv proxy](https://cockpit-project.org/guide/172/cockpit.conf.5.html)

```ini
[WebService]
Origins = https://somedomain1.com https://somedomain2.com:9090
ProtocolHeader = X-Forwarded-Proto
LoginTitle = 
LoginTo = 
MaxStartups = 
AllowUnencrypted = 
UrlRoot=/secret
```

Port anpassen
Hier mal mit dem port 90 als Ziel Port

*/etc/systemd/system/cockpit.socket.d/listen.conf*

```sh
sudo systemctl daemon-reload
sudo systemctl restart cockpit.socket

sudo firewall-cmd [--zone=ZONE] --add-port=90/tcp
sudo firewall-cmd --permanent [--zone=ZONE] --add-port=90/tcp
firewall-cmd --runtime-to-permanent
```

**SELinux Port freigeben**

```sh
sudo semanage port -a -t websm_port_t -p tcp 90
```