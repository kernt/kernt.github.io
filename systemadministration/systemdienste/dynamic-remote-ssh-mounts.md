---
tags:
  - ssh
  - sicherheit
  - sicherheits-tools
  - system-administration
---
Mount a remote Directory dnamicley.

Requirements:

 - afuse
 - ssh
 - autofs
 - sshfs
##  Install Requirements

**On Debian/Ubuntu like Systems**

`apt install sshfs fuseutils autofs `

**On CentOS7 systems: Add fuse group if not exist**

`sudo groupadd fuse`

**Richte deinen Benutzer ein**

`usermod -a -G fuse $(whoami)`

**Oser als sudo user**

`sudo usermod -a -G fuse $(whoami)`

**AutoFS Master file _/etc/auto.master_**

```sh
/mnt/sshfs /etc/auto.sshfs uid=1000,gid=1000,--timeout=30,--ghost
```

Execute a command before login via ssh

`ssh -t mymachine.example.com 'mount /home ; /bin/bash'`

add ssh specific config in you bash

```sh
if [ $SSH_TTY ]; then
    ...
fi
```

Use _~/.ssh/rc_ file
  
```sh
# ~/.ssh/rc, assuming your login shell is the C shell

if ( ! $?SSH_AUTH_SOCK ) then
  eval `ssh-agent`
  /usr/bin/tty | grep 'not a tty' > /dev/null
  if ( ! $status ) then
    ssh-add
  endif
 endif
```

**AutoFS ssh datei _/etc/auto.sshfs_**

```sh
bar -fstype=fuse,rw,nodev,nonempty,noatime,allow_other,max_read=65536 :sshfs\#$(whoami)@bar.com\:
```

**afuse Konfiguration**

`afuse -o mount_template="sshfs %r:/ %m" -o unmount_template="fusermount -u -z %m" ~/sshfs/`

**afuse basic mount**

```sh
  afuse -o mount_template="sshfs %r:/ %m" \
        -o unmount_template="fusermount -u -z %m" \
             mountpoint/
```

**afuse unmount**

Use _config_ User file to add a command for example with port knoking features.

`ProxyCommand bash -c 'knock-script %h; sleep 1; ssh 127.0.0.1 -W %h:%p'`
### Sources:

* [autofs-and-sshfs-the-perfect-couple](http://www.tjansson.dk/2008/01/autofs-and-sshfs-the-perfect-couple/)
* [Scripts](https://github.com/lahwaacz/Scripts)
* [sshfs arch linux](https://wiki.archlinux.org/index.php/SSHFS)
* [orelly ssh rc files](https://docstore.mik.ua/orelly/networking_2ndEd/ssh/ch08_04.htm)
* [ssh rc](https://unix.stackexchange.com/questions/240874/ssh-rc-not-working)