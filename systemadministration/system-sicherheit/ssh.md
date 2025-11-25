---
tags:
  - sicherheit
  - ssh
---
# SSH Server Einrichten

ssh agent für Passphrase benutzen

1. ssh agent im Hintergrund starten

    `eval "$(ssh-agent -s)"`

2. SSH Private Key zum SSH Agent hinzufügen

   `ssh-add ~/.ssh/id_rsa`

danch prüfen ob das Zielsystem funktioniert.


Publish SSH Host Key Fingerprints in DNS

die .pub-Schlüsselpaare

    /etc/ssh/ssh_host_rsa_key.pub
    /etc/ssh/ssh_host_dsa_key.pub
    /etc/ssh/ssh_host_ecdsa_key.pub

erstellt.

Beispielsweise, aber typisch, sieht ein SSHFP Resource Record (für BIND) etwa wie

    server.hq.c3d2.de. 86400 IN SSHFP 2 1 2492656260c5452d5c5452c6d21ea770f79bb9c8

aus.

Details dazu sind bei rfc:6594 zu entnehmen.

Die 2, erste Zahl nach SSHFP, gibt den Typ vom SSH Key an. Dabei stehen die Angaben für:

Value 	Algorithm name
0 	reserved
1 	RSA
2 	DSA
3 	ECDSA

Die 1, zweite Zahl nach SSHFP, gibt den Typ vom Hash Algorithmus an. Dabei stehen die Angaben für:
Value 	Algorithm name
1 	SHA1
2 	SHA256
Die Berechnung des Fingerprintes erfolgt mit

awk '{print $2}' /etc/ssh/ssh_host_dsa_key.pub | openssl base64 -d -A | openssl sha1

. Mit

ssh -o "VerifyHostKeyDNS yes" user@hostname

kann SSHFP getestet werden. 

## ssh PKI benutzen

https://goteleport.com/blog/how-to-configure-ssh-certificate-based-authentication/
https://www.ssh.com/academy/pki
https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Certificate-based_Authentication
https://www.strongdm.com/blog/configure-ssh-certificate-based-authentication
https://developer.hashicorp.com/vault/docs/secrets/ssh/signed-ssh-certificates
https://smallstep.com/docs/tutorials/ssh-certificate-login/


## Dynamic Port Forwarding (SOCKS Proxy)

You can set up a SOCKS proxy using SSH to route your browser traffic securely through the remote machine.

`ssh -D 8080 user@remote-server`

This sets up a dynamic proxy on port 8080. In your browser, set the SOCKS proxy to `localhost:8080`, and your traffic will be tunneled through the SSH connection.

## SSH Multiplexing (Speed Up Multiple SSH Connections)

SSH multiplexing allows you to reuse a single SSH connection for multiple sessions, reducing connection time.

1. Add this to your `~/.ssh/config` file:

```
Host *     
ControlMaster auto     
ControlPath ~/.ssh/controlmasters/%r@%h:%p     
ControlPersist 10m
```

After connecting once, any further SSH connections to the same server will be much faster.

## ProxyJump (Accessing a Server Through a Proxy/Jump Host)

If you need to connect to a remote server through a jump server (bastion host), you can use `-J`:

```
ssh -J jump-server user@final-server
```

This will first connect to `jump-server` and then to `final-server` via that connection.

## SSH Jump Host with a Specific Identity File

If you need to specify different identity files for a jump server and the final destination:

```sh
ssh -J user1@jump-server -i ~/.ssh/jump-server-key -i ~/.ssh/final-server-key user2@final-server
```

## SSH Force Command (Limit User Commands)

You can force a specific command to run every time a user logs in via SSH (useful for restricting what a user can do).

Add a rule in the remote server’s `~/.ssh/authorized_keys` file:

```sh
command="/usr/bin/uptime" ssh-rsa AAAA... user@local
```

This will force the user to run `/usr/bin/uptime` every time they SSH into the server.

**Screen-Session per SSH attachen**

`ssh -t <SERVER> screen -r`

**Anzeigen der bekannten Fingerprints samt ASCII-Art**

`ssh-keygen -lv -f ~/.ssh/known_hosts`

**eine SSH-Session beenden**

`<Enter> ~ .`