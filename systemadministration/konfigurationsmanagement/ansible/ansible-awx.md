---
tags:
  - konfigurationsmanagement
  - awx
  - ansible
---
# Ansible AWX

UI Für Ansible
## Ansible AWX Konfiguration

```sh
pip install -r requirements/requirements.txt -r requirements/requirements_dev.txt -r requirements/requirements_git.txt || pip3  install -r requirements/requirements.txt -r requirements/requirements_dev.txt -r requirements/requirements_git.txt 
pip install -r requirements.txt
install virtualenv
make virtualenv
make virtualenv_ansible
```
## Apache2 Ansible AWX Reverse Proxy

Apache2 Ansible AWX Reverse Proxy
obivalt/devops/konfigurationsmanagement/ansible/ansible-amx.md

```sh
<VirtualHost *:80>
    RewriteEngine On
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>

<VirtualHost *:443>
	ServerAdmin webmaster@tuxcoach.de
    ServerName awx.tuxcoach.de

	ErrorLog ${APACHE_LOG_DIR}/error-awx-tuxcoach-de.log
	CustomLog ${APACHE_LOG_DIR}/access-awx-tuxcoach-de.log combined

	SSLEngine on
	SSLCertificateFile	/opt/ssl/awx.tuxcoach.de/cert.pem
	SSLCertificateKeyFile /opt/ssl/awx.tuxcoach.de/privkey.pem

    SSLProxyEngine On
    SSLProxyVerify none
    SSLProxyCheckPeerCN off
    SSLProxyCheckPeerName off
    SSLProxyCheckPeerExpire off

    ProxyPreserveHost On
[[RemoteIPHeader]] X-Forwarded-For
[[RemoteIPHeader]] 
    ProxyPass / http://awx.tuxcoach.de:8052/
    ProxyPassReverse / http://awx.tuxcoach.de:8052/

    RewriteEngine on
    RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC]
    RewriteCond %{HTTP:CONNECTION} ^Upgrade$ [NC]
    RewriteRule .* ws://localhost:8052%{REQUEST_URI} [P]
# Protocols h2 http/1.1
</VirtualHost>
  [[Header]] always set Strict-Transport-Security "max-age=63072000"

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```

**Systemctl service**

```ini
[Unit]
Description=Ansible AMX docker service
After=network.target docker.service

[Service]
Type=simple
[User]=awx
User=root
[Group]=ansible
Group=root
# Load env vars from /etc/default/ and /etc/sysconfig/ if they exist.
# Prefixing the path with '-' makes it try to load, but if the file doesn't
# exist, it continues onward.
[EnvironmentFile]=-/etc/default/awx
[EnvironmentFile]=-/etc/sysconfig/awx
# sollte noch als user ansible gemachet werden
# Die gruppe ansible ist vorhanden nur der user nicht
# 
# /home/amx/
ExecStart=/usr/local/bin/docker-compose -f /root/.awx/awxcompose/docker-compose.yml up 
# /home/amx/
ExecStop=/usr/local/bin/docker-compose -f /root/.awx/awxcompose/docker-compose.yml down
Restart=on-failure
RestartSec=3
[[StartLimitBurst]]=3
[[StartLimitInterval]]=60
WorkingDirectory=/opt/netbox-docker

[Install]
WantedBy=multi-user.target
```

## Ansible AMX Administration

`docker-compose -f /root/.awx/awxcompose/docker-compose.yml down`

**Damit läuft folgende Container**

`awx_web /usr/bin/tini -- /bin/sh - ... Up 0.0.0.0:80→8052/tcp`

 Daran wurde der Port auf 8052 in der Datei /root/.awx/awxcompose/docker-compose.yml  geändert, damit er nicht mit Port 80 lokal kollidiert.

Systemctl Skript "awx" ist vorhanden.

## Dateien und Verzeichnisse

Basis für das docker-composefile.yml ist */root/.awx/awxcompose/docker-compose.yml*

Binds

Default bind ist /root/.awx/

und die folgenden

Default bind ist /root/.awx/ und die folgenden

```sh
"/root/.awx/awxcompose/redis_socket:/var/run/redis:rw",
"awxcompose_rsyslog-socket:/var/run/awx-rsyslog:rw",
"awxcompose_supervisor-socket:/var/run/supervisor:rw",
"awxcompose_rsyslog-config:/var/lib/awx/rsyslog:rw",
"/root/.awx/awxcompose/environment.sh:/etc/tower/conf.d/environment.sh:rw",
"/root/.awx/awxcompose/credentials.py:/etc/tower/conf.d/credentials.py:rw",
"/root/.awx/awxcompose/SECRET_KEY:/etc/tower/SECRET_KEY:rw",
"/root/.awx/awxcompose/nginx.conf:/etc/nginx/nginx.conf:ro"
```

Volumes

```sh
"/etc/nginx/nginx.conf": {},
"/etc/tower/SECRET_KEY": {},
"/etc/tower/conf.d/credentials.py": {},
"/etc/tower/conf.d/environment.sh": {},
"/var/lib/awx/rsyslog": {},
"/var/lib/nginx": {},
"/var/run/awx-rsyslog": {},
"/var/run/redis": {},
"/var/run/supervisor": {}
```

Secrets

`/root/.awx/awxcompose/SECRET_KEY`

Step 1: Update and Upgrade System Packages

```sh
sudo apt update -y
sudo apt upgrade -y
```

Step 2: Install Ansible and Required Packages

```sh
sudo apt install python-setuptools -y
sudo apt install python3-pip -y
sudo pip3 install ansible
ansible --version
pip3 install docker==6.1.3
sudo pip3 install docker-compose
docker-compose version
```

Step 3: Grant Docker Access to the Current User

`sudo usermod -aG docker $USER`

Step 4: Install Required Packages for AWX Setup

`sudo apt install git vim pwgen -y`

Step 5: Clone the Ansible AWX Repository

`sudo git clone https://github.com/ansible/awx.git --branch 17.0.1 --depth 1`

Step 6: Generate a Secret Key and Modify the Inventory File

```sh
cd awx/installer
pwgen -N 1 -s 30
sudo vi inventory
```

In the inventory file, replace the admin_password and secret_key with your desired values. Save the file by pressing Esc, then :wq.

Step 7: Run the Ansible Playbook to Install AWX

`ansible-playbook -i inventory install.yml`

Step 8: Access the AWX Web Interface

After the installation is complete, you can access the AWX web interface by navigating to http://your-server-ip in your web browser.

Use the user name admin and the admin password you specified in the inventory file during installation. Once you login you should be greeted with the following dashboard.

Quellen:

https://vpsie.com/knowledge-base/how-to-install-ansible-awx-on-debian-12/