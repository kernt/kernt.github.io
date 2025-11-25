Installation auf Debian 12

`sudo apt update && install samba samba-client cifs-utils` 

Samba Status Prüfen

sudo systemctl status smbd
smbstatus --version


**Samba smbauser share**

```ini
[share]
path = /home/sambauser/share
valid users = sambauser
read only = no
browseable = yes

[SharedDocs]
   path = /srv/samba/sharedocs
   writable = yes
   guest ok = no
   valid users = @sambashare

sudo mkdir -p /srv/samba/sharedocs
sudo chown nobody:nogroup /srv/samba/sharedocs
sudo chmod 0775 /srv/samba/sharedocs




```

`sudo systemctl restart smbd`

**User account hinzufügen**

`sudo adduser sambauser`

**User sambauser passwurd setzen**

`sudo smbpasswd -a sambauser`

**Verzeichnis auf dem Server anlegen**

`sudo mkdir /home/sambauser/share`

**Berechtigungen anpassen**

```sh
sudo chown sambauser:sambauser /home/sambauser/share
sudo chmod 700 /home/sambauser/share
```

**Verbindung Prüfen**

`smbclient '\\localhost\share' -U sambauser`

**Konfigurationsdateo Prüfen**

`testparm`

