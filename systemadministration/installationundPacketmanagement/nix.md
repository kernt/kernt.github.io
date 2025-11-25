# den Paketmanager "nix" separat installieren

Wenn man gerne die Vorteile des Paketmanagers `nix` nutzen möchte ohne extra `nixos` installieren, kann man diesen auch neben dem bereits vorhandenen Paketmanager (egal ob `apt`, `yum`, etc.) im Userspace installieren.
### Installation

Folgende Tools müssen installiert sein:

- `curl`
- `tar`
- `rsync`
- `sudo`

Die Installation selber erfolgt dann mit dem Befehl:  
`sh <(curl -L https://nixos.org/nix/install) --daemon`

```sh
rasputin@debian:~$ sh <(curl -L https://nixos.org/nix/install) --daemon
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
Dload  Upload   Total   Spent    Left  Speed      
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0     
100  2699  100  2699    0     0   5958      0 --:--:-- --:--:-- --:--:--  5958             
downloading Nix 2.3.10 binary tarball for x86_64-linux from
'https://releases.nixos.org/nix/nix-2.3.10/nix-2.3.10-x86_64-linux.tar.xz' to '/tmp/nix-binary-tarball-unpack.75MNYE1Sd2'...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
Dload  Upload   Total   Spent    Left  Speed
100 16.4M  100 16.4M    0     0  9796k      0  0:00:01  0:00:01 --:--:-- 9796k
Note: a multi-user installation is possible. See https://nixos.org/nix/manual/#sect-multi-user-installation
Switching to the Daemon-based Installer                              
Welcome to the Multi-User Nix Installation
This installation tool will set up your computer with the Nix package
manager. This will happen in a few stages:
1. Make sure your computer doesn't already have Nix. If it does, I
   will show you instructions on how to clean up your old one.
...
```

Hierbei werden dann die entsprechenden Benutzer, Gruppen und Verzeichnisse angelegt.  
Anschließend wird die Umgebung für `nix` generiert und aufgebaut und der zum Schluß wird der `nix` Daemon gestartet.
### Verwenden des `nix` Paketmanagers

Nach der Installation kann man ein neues Terminal öffnen (oder loggt sich neu ein) um den Paketmanager zu nutzen…

```sh
rasputin@debian:~$ nix-shell -p nix-info --run "nix-info -m"
these paths will be fetched (63.79 MiB download, 272.83 MiB unpacked):
  /nix/store/16n426g6jbcwwdwlq7h7qmbv1v22p2v5-zlib-1.2.11
  /nix/store/2hkhh0fhzdj6iibvs1krnfwiwfvdk28j-linux-headers-5.11
...
copying path '/nix/store/6xss8bksa5hsjl981y9as0adrcyjfgkl-stdenv-linux' from 'https://cache.nixos.org'...
 - system: `"x86_64-linux"`
 - host os: `Linux 4.19.0-16-amd64, Debian GNU/Linux, 10 (buster)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.10`
 - channels(root): `"nixpkgs-21.05pre285495.55fafd9e23a"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixpkgs`
```
Beispiel:
```sh
rasputin@debian:~$ nix-env -iA nixpkgs.ansible_2_9
installing 'ansible-2.9.12'
these paths will be fetched (31.82 MiB download, 199.81 MiB unpacked):
  /nix/store/03xyzk6kvbzx823d7i6gasw7jm9m7lf0-python3.8-bcrypt-3.2.0
  /nix/store/08p57lqsydb0ybf9g25r4prfhwflfbv0-python3.8-Jinja2-2.11.3
...
building '/nix/store/ypyhm35v137m5k508xzab8md7dkk71k9-user-environment.drv'...
created 2 symlinks in user environment
```

```sh
rasputin@debian:~$ ansible --version
ansible 2.9.12
  config file = None
  configured module search path = ['/home/rasputin/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /nix/store/xahlwrbapx7v741f3p4mj7r64qnqd8qq-python3.8-ansible-2.9.12/lib/python3.8/site-packages/ansible
  executable location = /nix/store/xahlwrbapx7v741f3p4mj7r64qnqd8qq-python3.8-ansible-2.9.12/bin/ansible
  python version = 3.8.9 (default, Apr  2 2021, 11:20:07) [GCC 10.2.0]
```

Weitere Beispiele zur Verwendung des Paketmanagers `nix` finden sich in dem Artikel [Verwendung von NixOS [Grundlagen]](https://techgoat.net/index.php?id=199)

https://nixos.org/download/

After installation 

```
I am executing:

    $ sudo systemctl restart nix-daemon.service

to start the nix-daemon.service

Alright! We're done!
Try it! Open a new terminal, and type:

  $ nix-shell -p nix-info --run "nix-info -m"

Thank you for using this installer. If you have any feedback or need
help, don't hesitate:

You can open an issue at
https://github.com/NixOS/nix/issues/new?labels=installer&template=installer.md

Or get in touch with the community: https://nixos.org/community

---- Reminders -----------------------------------------------------------------
[ 1 ]
Nix won't work in active shell sessions until you restart them.

Press enter/return to acknowledge.
```