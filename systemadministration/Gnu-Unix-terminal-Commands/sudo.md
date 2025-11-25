---
tags:
  - bash
  - konsole
  - sudo
  - sicherheit
  - system-administration
  - berechtigungen
  - gnu-tools
  - gpasswd
---
# sudo /etc/sudoers

Grundsätzlicher aufbau

```sh
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
[[Defaults]]       timestamp_timeout = 0   # Immer PW eingeben
Defaults        timestamp_timeout = 540 # PW nach 9 Studen wieder eingeben
Defaults        pwfeedback              # fuer jedes eingegebene Zeichen ein Stern (*) ausgegeben
[[Defaults]] askpass = /PFAD/ZUM/EXTERNEN/PROGRAMM
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Host alias specification
Host_Alias   HOMENET4     = 192.168.4.0/24
Host_Alias   DESKTOP = tobkern-desktop, tobkern-desktop-win10

# User alias specification
Runas_Alias     OP = root, operator
Runas_Alias  DBA = oracle,pgsql
User_Alias    WEBMASTERS = tobkern, www-data
User_Alias   ADMINS  = tobkern
User_Alias   DEVEL   = tobkern

# Cmnd alias specification
Cmnd_Alias      KILL = /bin/kill
Cmnd_Alias      PRINTING = /usr/sbin/lpc, /usr/bin/lprm
Cmnd_Alias      SHUTDOWN = /usr/sbin/shutdown
Cmnd_Alias      HALT = /usr/sbin/halt
Cmnd_Alias      REBOOT = /usr/sbin/reboot
Cmnd_Alias      SHELLS = /sbin/sh, /usr/bin/sh, /usr/bin/csh, /usr/bin/ksh,/usr/local/bin/tcsh, /usr/bin/rsh, /usr/local/bin/zsh
Cmnd_Alias      VIPW = /usr/sbin/vipw, /usr/bin/passwd, /usr/bin/chsh, /usr/bin/chfn
Cmnd_Alias      PAGERS = /usr/bin/more, /usr/bin/pg, /usr/bin/less
Cmnd_Alias      SYSTEM = /sbin/reboot,/usr/bin/kill,/sbin/halt,/sbin/shutdown,/etc/init.d/
Cmnd_Alias   PW      = /usr/bin/passwd [A-z]*, !/usr/bin/passwd root # Not root pwd!
Cmnd_Alias   DEBUG   = /usr/sbin/tcpdump,/usr/bin/wireshark,/usr/bin/nmap

# User privilege specification
root    ALL=(ALL:ALL) ALL
tobkern ALL=(ALL:ALL) NOPASSWD: ALL
tkern   ALL=(ALL:ALL) ALL
%administrator  ALL = (ALL) AL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
%operator ALL = /var/log/messages*
sysadmin     DMZ     = (ALL) NOPASSWD: SYSTEM,PW,DEBUG
sysadmin     ALL,!DMZ = (ALL) NOPASSWD: ALL   # Can do anything outside the DMZ.
%dba         ALL     = (DBA) ALL              # Group dba can run as database user.

# anyone can mount/unmount a cd-rom on the desktop machines
ALL          DESKTOP = NOPASSWD: /sbin/mount /cdrom,/sbin/umount /cdrom

# See sudoers(5) for more information on "#include" directives:
[[includedir]] /etc/sudoers.d
```

# sudo aliases

**User Aliases**
```sh
User_Alias		GROUPONE = abby, brent, carl
User_Alias		GROUPTWO = brent, doris, eric,
User_Alias		GROUPTHREE = doris, felicia, grant
```

**Group Aliases**

Gruppen müssen mit einem Großbuchstaben anfangen !

**Runas_Alias**

```sh
Runas_Alias		WEB = www-data, apache
GROUPONE	ALL = (WEB) ALL
```

**Berechtigungen Prüfen**

```bash
sudo -l -U john
```

**Alle Benutzer Überprüfen**

```bash
getent passwd | cut -d: -f1 | xargs -n 1 sudo -l -U
[sudo] password for joe: 
...
User root may run the following commands on fedora35:
    (ALL) ALL
User bin is not allowed to run sudo on fedora35.
...
User joe may run the following commands on fedora35:
    (ALL) ALL
...
User john is not allowed to run sudo on fedora35.
```

**Abfragen ob user sudo benutzen kann**

```bash
sudo -v
oder mit login test
sudo -l
```
### lternative Method: Using gpasswd

As an alternative to usermod, the `gpasswd` command can also be used to add a user to the sudoers group.

#### Executing the Gpasswd Command

The command format is as follows:

```bash
gpasswd -a <example username> sudo
```

For example:

```bash
gpasswd -a josh sudo
```

_Example output:_

```bash
adding josh to group sudo
```

