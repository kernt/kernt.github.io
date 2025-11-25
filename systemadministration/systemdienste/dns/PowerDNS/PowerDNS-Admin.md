Voraussitzungen

- Ein Laufender PowerDNS Server

**Abhängigkeiten zur PowerDNS Admin installation installieren**

```sh
sudo apt -y install python3-dev python3-venv python3-pip
sudo apt -y install default-libmysqlclient-dev python3-mysqldb libsasl2-dev libffi-dev libldap2-dev libssl-dev libxml2-dev libxslt1-dev libxmlsec1-dev pkg-config libpq-dev
```

**Berechtigungen an der Datenbank definieren**

```sql
CREATE DATABASE powerdns;
GRANT ALL ON powerdns.* TO 'powerdns_user'@'%' IDENTIFIED BY 'Strongpassword';
FLUSH PRIVILEGES;
EXIT
```

**Datenbanken schema importieren**

```bash
mysql -u powerdns_user -p powerdns < /usr/share/pdns-backend-mysql/schema/schema.mysql.sql
```

**PDNS Start prüfen**

`pdns_server --daemon=no --guardian=no --loglevel=9`

**Port listen testen**

`ss -alnp4 | grep pdns`

**erstelle Tabelen Prüfen**

```sql
sudo mysql -u root
use powerdns;
show tables;
```

## Vorbereitungen des resolv systems

**dnsmasq deaktivieren**

```
sudo sed -i 's/^dns=dnsmasq/#&/' /etc/NetworkManager/NetworkManager.conf && \
sudo service networking restart
```

**Stop und disable _systemd-resolved_**

```sh
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

**SymLink Entvernen**

```sh
sudo unlink /etc/resolv.conf
```

> Wie beschreiben unter Ubuntu .

**/etc/resolv.conf Anpassen**

```
nameserver 8.8.8.8
```

**PowerDNSAdmin Konfiguration anlegen**

*powerdnsadmin/default_config.py*

```python
SQLA_DB_USER = 'powerdns'
SQLA_DB_PASSWORD = '_**strongpassword**_'
SQLA_DB_HOST = 'localhost'
SQLA_DB_NAME = 'powerdns'

### DATABASE - MySQL  
SQLALCHEMY_DATABASE_URI = 'mysql://{}:{}@{}/{}'.format(  
    urllib.parse.quote_plus(SQLA_DB_USER),  
    urllib.parse.quote_plus(SQLA_DB_PASSWORD),  
    SQLA_DB_HOST,  
    SQLA_DB_NAME  
)
### DATABASE - SQLite  
#SQLALCHEMY_DATABASE_URI = 'sqlite:///' + os.path.join(basedir, 'pdns.db')
```
