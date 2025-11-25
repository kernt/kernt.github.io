# ssh Server

## SSHD Options

The enablement of [_sshd_](https://linux.die.net/man/8/sshd), the daemon that serves ssh sessions, is done by editing the _[sshd_config](https://www.man7.org/linux/man-pages/man5/sshd_config.5.html)_ file. Its location varies a little but is usually on _/etc/ssh_ or _/etc/openssh_. The relevant configuration keys are:

- _AllowStreamLocalForwarding_: Allows Unix domain sockets to be forwarded. The default, when omitted, is _yes_
- _AllowTcpForwarding_: Allows TCP port forwarding. The default, when omitted, is to allow. It enables single TCP port forwards and socks proxying
- _DisableForwarding_: Disables all kinds of forwarding. Override, if enabled, all other related configurations options
- _GatewayPorts_: Allows other hosts to use the ports forwarded to a client (reverse tunnels). By default, only the hosts running the SSH server can use reverse tunnels. Disabled by default
- _PermitListen_: Specifies the addresses and ports that can be bound to allow port-forwarding to clients. It provides more fine control if we enable _GatewayPorts_. The default is localhost (‘127.0.0.1’ and ‘::1’)
- _PermitOpen_: Specifies the address and ports a TCP forwarding may point to. By default, any destination is enabled
- _PermitTunnel_: Specifies whether _tun_ device forwarding is allowed. Default is no
- _X11Forwarding_: Specifies whether X11 forwarding is allowed. Default is no
- _X11UseLocalhost_: Forces the X11 forwarding to be only allowed from the SSH server host loopback address. If disabled, other hosts on the SSH server network might use it. Default is true
- 
4. [orward TCP Tunnels](https://www.baeldung.com/linux/ssh-tunneling-and-proxying#forward-tcp-tunnels)

4.1. [Single-Port](https://www.baeldung.com/linux/ssh-tunneling-and-proxying#1-single-port)

**A forward or direct TCP tunnel is the one that follows the direction of the SSH connection from the client to the SSH server**. [Our introductory tutorial on SSH](https://www.baeldung.com/linux/secure-shell-ssh) briefly describes this type of forwarding. To create a direct TCP forward tunnel, we have to use the _-L_ option on the command line:

```bash
ssh -L [bind_address:]port:host:hostport [user@]remote_ssh_server
```

The optional _bind_address_ assigns a client local interface to listen for connections. If we omit it, _ssh_ binds on the loopback interfaces only. We can also use _“0.0.0.0”_ or _“::”_ to bind on all interfaces. So, if we issue the following command:

```bash
ssh -L 0.0.0.0:8022:10.1.4.100:22 user@10.1.4.20
```

We would have an SSH connection opened to the host on the 10.1.4.20 IP address and a tunnel, listening on the client port 8022, pointing to the SSH address on host 10.1.4.100.

That way, if a connection goes into client port 8022, it will be forwarded to the destination host and port, using the SSH server IP address, looking exactly like a regular local network between them.

Similarly, to forward local sockets (somewhat less usual), we can use:

```bash
ssh -L local_socket:host:hostport [user@]remote_ssh_server
```

Or we can use:

```bash
ssh -L local_socket:remote_socket [user@]remote_ssh_server
```

### 4.2. Dynamic or Multi-Port[](https://www.baeldung.com/linux/ssh-tunneling-and-proxying#2-dynamic-or-multi-port)

**A special case of the forward TCP tunnels is the Socks proxy capability. Using these options, the SSH client listens on a specified binding port and acts as a SOCKS 4 or 5 proxy server**.

Any connections using SOCKS protocol to the binding port will be forwarded to the SSH server using its own IP address. To do that, we would use:

```bash
ssh -D [bind_address:]port [user@]remote_ssh_server
```

Note that we don’t even need to specify the destination host and port for the forwarding in this case. All SOCKS compliant incoming connections on the specified port will flow through the tunnel.

To use it, we must, of course, configure the application that will use the tunnel to use a proxy server on the bound address and port specified on the command line. For instance, after issuing:

```bash
ssh -D 8080 user@10.1.4.100
```

We can configure a browser on the client host to use our SOCKS proxy server on 127.0.0.1, port 8080. That way, we can navigate the web as if we were using a browser installed on the SSH server located at 10.1.4.100.

And what if the client application does not support SOCKS proxying? We can use solutions such as [_proxychains_](https://github.com/haad/proxychains) or [_tsocks_](https://linux.die.net/man/8/tsocks) that intercept sockets systems calls and force the connections to flow through a SOCKS proxy.

# ssh Zertifikate 

**Erstellen einer CA **

```
ssh-keygen -t rsa -b 4096 -f host_ca -C host_ca
```

**user pub key**

```sh
ssh-keygen -t rsa -b 4096 -f user_ca -C user_ca
```

**Erstellen eines Host Zertifikates**

```sh
ssh-keygen -f ssh_host_rsa_key -N '' -b 4096 -t rsa
```

**SSH anweisen host cert zu nutzen**

```bash
HostCertificate /etc/ssh/ssh_host_rsa_key-cert.pub
```

Your server is now configured to present a certificate to anyone who connects. For your local `ssh` client to make use of this (and automatically trust the host based on the certificate's identity), you will also need to add the CA's public key to your `known_hosts` file.

You can do this by taking the contents of the `host_ca.pub` file, adding `@cert-authority *.example.com` to the beginning, then appending the contents to `~/.ssh/known_hosts`:

```bash
@cert-authority *.example.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDwiOso0Q4W+KKQ4OrZZ1o1X7g3yWcmAJtySILZSwo1GXBKgurV4jmmBN5RsHetl98QiJq64e8oKX1vGR251afalWu0w/iW9jL0isZrPrmDg/p6Cb6yKnreFEaDFocDhoiIcbUiImIWcp9PJXFOK1Lu8afdeKWJA2f6cC4lnAEq4sA/Phg4xfKMQZUFG5sQ/Gj1StjIXi2RYCQBHFDzzNm0Q5uB4hUsAYNqbnaiTI/pRtuknsgl97xK9P+rQiNfBfPQhsGeyJzT6Tup/KKlxarjkMOlFX2MUMaAj/cDrBSzvSrfOwzkqyzYGHzQhST/lWQZr4OddRszGPO4W5bRQzddUG8iC7M6U4llUxrb/H5QOkVyvnx4Dw76MA97tiZItSGzRPblU4S6HMmCVpZTwva4LLmMEEIk1lW5HcbB6AWAc0dFE0KBuusgJp9MlFkt7mZkSqnim8wdQApal+E3p13d0QZSH3b6eB3cbBcbpNmYqnmBFrNSKkEpQ8OwBnFvjjdYB7AXqQqrcqHUqfwkX8B27chDn2dwyWb3AdPMg1+j3wtVrwVqO9caeeQ1310CNHIFhIRTqnp2ECFGCCy+EDSFNZM+JStQoNO5rMOvZmecbp35XH/UJ5IHOkh9wE5TBYIeFRUYoc2jHNAuP2FM4LbEagGtP8L5gSCTXNRM1EX2gQ== host_ca
```

The value `*.example.com` is a pattern match, indicating that this certificate should be trusted for identifying any host which you connect to that has a domain of `*.example.com` — such as `host.example.com` above. This is a comma-separated list of applicable hostnames for the certificate, so if you're using IP addresses or [SSH config](https://goteleport.com/blog/ssh-config/) entries here, you can change this to something like `host1,host2,host3` or `1.2.3.4,1.2.3.5` as appropriate.

Once this is configured, remove any old host key entries for `host.example.com` in your `~/.ssh/known_hosts` file, and start an `ssh` connection. You should be connected straight to the host without needing to trust the host key. You can check that the certificate is being presented correctly with a command like this:

```vbnet
$ ssh -vv host.example.com 2>&1 | grep "Server host certificate"
debug1: Server host certificate: ssh-rsa-cert-v01@openssh.com SHA256:dWi6L8k3Jvf7NAtyzd9LmFuEkygWR69tZC1NaZJ3iF4, serial 0 ID "host.example.com" CA ssh-rsa SHA256:8gVhYAAW9r2BWBwh7uXsx2yHSCjY5OPo/X3erqQi6jg valid from 2020-03-17T11:49:00 to 2021-03-16T11:50:21
debug2: Server host certificate hostname: host.example.com
```

At this point, you could continue by issuing host certificates for all hosts in your estate using your host CA. The benefit of doing this is twofold: you no longer need to rely on the insecure [trust on first use (TOFU)](https://en.wikipedia.org/wiki/Trust_on_first_use) model for new hosts, and if you ever redeploy a server and therefore change the host key for a certain hostname, your new host could automatically present a signed host certificate and avoid the dreaded `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` message.

## Issuing user certificates (to authenticate users to hosts)

Generate user keypair and sign it with our user CA. It's up to you whether you use a passphrase or not.

```sql
$ ssh-keygen -f user-key -b 4096 -t rsa

$ ls -l
-rw-r--r--. 1 honda honda  737 Mar 19 16:33 user-key.pub
-rw-------. 1 honda honda 3369 Mar 19 16:33 user-key

$ ssh-keygen -s user_ca -I honda@goteleport.com -n ec2-user,honda -V +1d user-key.pub
Enter passphrase: # the passphrase used for the user CA
Signed user key user-key-cert.pub: id "honda@goteleport.com" serial 0 for ec2-user,honda valid from 2020-03-19T16:33:00 to 2020-03-20T16:34:54

$ ls -l
-rw-------. 1 honda honda 3369 Mar 19 16:33 user-key
-rw-r--r--. 1 honda honda 2534 Mar 19 16:34 user-key-cert.pub
-rw-r--r--. 1 honda honda  737 Mar 19 16:33 user-key.pub
```

`user-key-cert.pub` contains the signed user certificate. You'll need both this and the private key (`user-key`) for logging in.

Here's an explanation of the flags used:

- `-s user_ca`: specifies the CA private key that should be used for signing
- `-I honda@goteleport.com`: the certificate's identity, an alphanumeric string that will be visible in SSH logs when the user certificate is presented. I recommend using the email address or internal username of the user that the certificate is for — something which will allow you to uniquely identify a user. This value can also be used to revoke a certificate in future if needed.
- `-n ec2-user,honda`: specifies a comma-separated list of principals that the certificate will be valid for authenticating, i.e. the *nix users which this certificate should be allowed to log in as. In our example, we're giving this certificate access to both `ec2-user` and `honda`.
- `-V +1d`: specifies the validity period of the certificate; in this case `+1d` means 1 day. Certificates are valid forever by default, so using an expiry period is a good way to limit access appropriately and ensure that certificates can't be used for access perpetually.

If you need to see the options that a given certificate was signed with, you can use `ssh-keygen -L`:

```yaml
$ ssh-keygen -L -f user-key-cert.pub
user-key-cert.pub:
        Type: ssh-rsa-cert-v01@openssh.com user certificate
        Public key: RSA-CERT SHA256:egWNu5cUZaqwm76zoyTtktac2jxKktj30Oi/ydrOqZ8
        Signing CA: RSA SHA256:tltbnMalWg+skhm+VlGLd2xHiVPozyuOPl34WypdEO0 (using ssh-rsa)
        Key ID: "honda@goteleport.com"
        Serial: 0
        Valid: from 2020-03-19T16:33:00 to 2020-03-20T16:34:54
        Principals:
                ec2-user
                honda
        Critical Options: (none)
        Extensions:
                permit-X11-forwarding
                permit-agent-forwarding
                permit-port-forwarding
                permit-pty
                permit-user-rc
```

## Configuring SSH for user certificate authentication

Once you've signed a certificate, you also need to tell the server that it should trust certificates signed by the user CA. To do this, copy the `user_ca.pub` file to the server and store it under `/etc/ssh`, fix the permissions to match the other public key files in the directory, then add this line to `/etc/ssh/sshd_config`:

```bash
TrustedUserCAKeys /etc/ssh/user_ca.pub
```

Once this is done, restart `sshd` with `systemctl restart sshd`.

Your server is now configured to trust anyone who presents a certificate issued by your user CA when they connect. If you have a certificate in the same directory as your private key (specified with the `-i` flag, for example `ssh -i /home/honda/user-key ec2-user@host.example.com`), it will automatically be used when connecting to servers.

# ssh Reverse Tunnel Single Port

**The reverse or callback proxies allow us to do tricks similar to the one above but in the reverse direction**. We can open services on our own local networks to hosts on the remote side of the SSH session. The command syntax is quite similar to the direct forward:

```bash
ssh -R [bind_address:]port:host:hostport [user@]remote_ssh_server
```

This creates a reverse tunnel. It forwards any connection received on the remote SSH server to the _bind_address:port_ to local client network _host:hostport_. If we omit the _bind_address_ parameter, it binds to the loopback interfaces only.

Similarly, using sockets, we can use three different syntaxes:

```bash
ssh -R remote_socket:host:hostport [user@]remote_ssh_server
```

```bash
ssh -R remote_socket:local_socket [user@]remote_ssh_server
```

```bash
ssh -R [bind_address:]port:local_socket [user@]remote_ssh_server
```

# ssh Reverse Tunnel Dynamic oder Multi-Port

**Finally, we can expose a SOCKS proxy server on the remote host directed to the client’s network as we can do with direct forwarding**. We can do this only by omitting the local destination host and port:

```bash
ssh -R [bind_address:]port [user@]remote_ssh_server
```

This opens a port on the remote SSH server that’ll serve as a SOCKS server to the local client network, potentially piercing any outbound traffic rules that would otherwise apply to the remote SSH server.

# Multiple Tunnels and Multiple Host Hopping

**We can create as many tunnels as we need, mixing types and directions.** We can accomplish this by adding more options to the command line:

```bash
ssh -X -L 5432:<DB server IP>:5432 -R 873:<local RSYNC server>:873 [user@]remote_ssh_server
```

This opens a direct forward to a remote PostgreSQL server, a reverse tunnel to a local _rsync_ server, and allows GUI applications to flow to our local X Server.

And **we can also use SSH tunnels to reach farther SSH servers, creating tunnels to them piercing through as many firewall layers as we need**, creating the tunnel on each connection until we can reach the desired point:

```bash
ssh -L 8022:<server2>:22 user@server1
ssh -L 8023:<server3>:22 -p 8022 user@localhost
ssh -p 8023 user@localhost
```

That sequence creates, on each step, a tunnel to the next server, from _server1_, and _server2_, until the tunnel opened on local port 8023 allows us to reach _server3_.

# ssh Persistente tunnel

y the way, an SSH tunnel only exists as long as the SSH connection holds. Even if we can even configure the frequency and timeout for the session keepalives to facilitate the connection-loss detections, it would be nice to **fully automate the SSH session creation and reconnection.**

**For that, a handy piece of software is [_autossh_](https://linux.die.net/man/1/autossh)**. This utility can automatically create and recreate SSH sessions. **If we add authentication keys, as shown on our SSH [keys](https://www.baeldung.com/linux/generating-ssh-keys-in-linux) tutorial, the tunnels will open without user intervention, as long as _autossh_ is running**. Its syntax is:

```bash
autossh [-V] [-M port[:echo_port]] [-f] [SSH_OPTIONS]
```

- -V: Show _autossh_ version
- -M: Creates a direct tunnel on a _port_, loop-backed to a reverse one, _echo_port_. It provides an alive checking mechanism. However, with recent OpenSSH, we can achieve similar results using _ServerAliveInterval_ and _ServerAliveCountMax_ options in the _sshd_config_ file
- -f: Forces _autossh_ to run in the background before running _ssh_
- _SSH_OPTIONS_: The options we would use to start _ssh_

That way, to start a persistent connection, we can use:

```bash
autossh -X -L 5432:<DB server IP>:5432 -R 873:<local RSYNC server>:873 [user@]remote_ssh_server
```

If we’ve defined a host config, the command line is much simpler:

```bash
autossh -f [host]
```


# Erstellung einer ssh CA

Zuerst legen wir die Zertifikate der CA, mit welchen die Schlüssel später signiert werden, an:

`ssh-keygen -C CA -f ca -o -a 500 -t ed25519`

`-o` legt die Dateien nicht als PEM Format ab, sondern verwendet das OpenSSH Format  
`-a` gibt die Anzahl der `KDF (Key Derivation Function)` Runden/Durchläufe an für die Verifikation des Passwortes zum entschlüsseln des Zertifikates  
`-t` gibt den Algorithmus an, welcher verwendet werden soll  
`-f` ist der Dateiname  
`-C` ein beliebiger Kommentar

Hierbei sollte ein starkes Passwort eingegeben werden!

Ein Blick in die `ca.pub` sollte wie folgt aussehen:

```
$ cat ca.pub 
ssh-ed25519 AAAAC3NzbZwjNNNjjwlqdnwnern211CqZz2hw1b9o6HnCpQzB+Wjrcq1X0opoo CA
```

### Erstellen eine SSH-Keys

Genau auf die gleiche Weise erstellen wir uns auch entsprechende SSH-Keys, welche dann im Anschluss signiert werden.

```
ssh-keygen -o -a 200 -t ed25519 -f ~/.ssh/id_ed25519 -C "info@techgoat.net"
```

### Signieren der Schlüssel

Das signieren ist weiterhin identisch und da braucht eigentlich nichts weiter beachtet werden.

Beispiel:

```
$ ssh-keygen -s ca -I rasputin -n root -z 1 /home/rasputin/.ssh/id_ed25519.pub 
Enter passphrase: 
Signed user key /home/rasputin/.ssh/id_ed25519-cert.pub: id "rasputin" serial 1 for root valid forever
```

Und um das noch einmal zu überprüfen:

```
$ ssh-keygen -Lf /home/rasputin/.ssh/id_ed25519-cert.pub
/home/rasputin/.ssh/id_ed25519-cert.pub:
        Type: ssh-ed25519-cert-v01@openssh.com user certificate
        Public key: ED25519-CERT SHA256:/Sl4lhvc5eR7dtkb2hgWuk0LNs6GDUoFVHdHXo4dO3I
        Signing CA: ED25519 SHA256:Ej4xt6dOmcfYiArMJ5wTveulvI1FgCgKgw9Jkzb5F4I
        Key ID: "rasputin"
        Serial: 1
        Valid: forever
        Principals: 
                root
        Critical Options: (none)
        Extensions: 
```