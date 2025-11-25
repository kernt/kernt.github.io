---
tags:
  - mariadb
  - datenbanken
---
## Installation mit den Repos vom Hersteller

**Deb basierende Installation**

```sh
sudo apt install dirmngr ca-certificates software-properties-common apt-transport-https curl -y
```

**GPG Key installation**

```bash
curl -fsSL http://mirror.mariadb.org/PublicKey_v2 | sudo gpg --dearmor | sudo tee /usr/share/keyrings/mariadb.gpg > /dev/null
```

**Verschiedene Repos zur Installation**

```sh
Option 1: MariaDB 10.5 (LTS)

End-of-life date: 24th June 2025.

echo "deb [arch=amd64,arm64,ppc64el signed-by=/usr/share/keyrings/mariadb.gpg] http://mirror.mariadb.org/repo/10.5/ubuntu/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/mariadb.list

Option 2: MariaDB 10.6 (LTS)

End-of-life date: July 2026.

echo "deb [arch=amd64,arm64,ppc64el signed-by=/usr/share/keyrings/mariadb.gpg] http://mirror.mariadb.org/repo/10.6/ubuntu/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/mariadb.list

Option 3: MariaDB 10.11 (LTS)

End-of-life date: February 2028.

echo "deb [arch=amd64,arm64,ppc64el signed-by=/usr/share/keyrings/mariadb.gpg] http://mirror.mariadb.org/repo/10.11/ubuntu/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/mariadb.list

Option 4: MariaDB 11.4 (Long-term release)

End-of-life date: February 2030.

echo "deb [arch=amd64,arm64,ppc64el signed-by=/usr/share/keyrings/mariadb.gpg] http://mirror.mariadb.org/repo/11.4/ubuntu/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/mariadb.list
```

**APT Updaten**

```bash
sudo apt update
```

**Installation vom MariaDB Server und Client**

```sh
sudo apt install mariadb-server mariadb-client -y
```

# MariaDB gesicherte Installation einrichten

```bash
sudo mysql_secure_installation
```

```sh
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] Y <---- Type Y then press the ENTER KEY.
Enabled successfully!
Reloading privilege tables..
 ... Success!

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] Y <---- Type Y then press the ENTER KEY.
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y <---- Type Y then press the ENTER KEY.
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y <---- Type Y then press the ENTER KEY.
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y <---- Type Y then press the ENTER KEY.
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y <---- Type Y then press the ENTER KEY.
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

# Managen der Maridb

Wenn Sie MariaDB auf eine neuere Version aktualisiert haben (z. B. von 10.5 auf 10.10), ist es wichtig, dass Ihre Datenbanktabellen mit der neuen Version kompatibel sind. Das Tool „mariadb-upgrade“ ist für diesen Zweck gedacht. Es überprüft und aktualisiert die Tabellen, um sie an die Anforderungen der aktualisierten Version anzupassen.

MariaDB-Datenbanktabellen-Upgrade-Tool ausführen

```bash
sudo mariadb-upgrade
```

[Maridb Doc](https://mariadb.com/kb/en/)