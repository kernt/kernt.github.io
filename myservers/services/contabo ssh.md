
**ssh config**

```bash
Host *
  #ForwardAgent yes
  #Compression yes
  ConnectionAttempts 3
  #UserKnownHostsFile ~/.ssh/known_hosts.d/%k
  IdentityFile ~/.ssh/id_ed25519
  # ProxyCommand /usr/bin/nc -X connect -x IP:PORT %h %p 
  #
  # ProxyJump user@host:port
  # SendEnv SHELL='/bin/usr/zsh' 
  # http://man.openbsd.org/OpenBSD-current/man5/ssh_config.5#TOKENS

Host srv1-tobkern
  HostName 192.168.4.17
  User tobkern

Host srv2-tobkern
 HostName 192.168.4.93
 User tobkern

#Host proxy-tobkern
#  HostName 192.168.4.13
#  User tobkern
#  ForwardAgent yes
#  SendEnv yes
#LocalCommand sudo su ansible
#ProxyJump ansible@127.0.0.1:22
#ProxyCommand ssh rp3.fritz.box -W %h:%p

Host depoyments-tobkern
  HostName 192.168.4.43
  User tobkern
  #Password R72#lh45
  ForwardAgent yes
  SendEnv yes
  #LocalCommand sudo su ansible
  #ProxyJump ansible@127.0.0.1:22
  #ProxyCommand ssh rp3.fritz.box -W %h:%p

Host vmd47205-tobkern
  HostName vmd47205.contaboserver.net
  User tobkern
  Port 22

# 62.171.183.4
Host vmd132643-root
  HostName 2a02:c207:3013:2643:0000:0000:0000:0001
  User root
  Port 22
```

ssh only 

```sh
ChallengeResponseAuthentication no
PasswordAuthentication no
PermitRootLogin no
PermitRootLogin prohibit-password
```