One of the big problems I quickly had to solve when I wanted to set up Proxmox VE server clusters at physical server providers is the way clustering is managed in PVE.

![The offical logo of tinc vpn which is used in this tutorial to create a distributed full mesh vpn](https://miro.medium.com/v2/resize:fit:450/1*Guf3LCnq5MVWH00su_g7gQ.jpeg)

`tinc-vpn-logo`

PVE designers don’t want you to set up clusters on the Internet, and they deliberately put as many obstacles in your way as possible to prevent you from doing so. I’m barely exaggerating.

## OpenVPN vs Tinc

One solution I’ve tried is to set up an OpenVPN server on one of the servers and connect the others to it. I’ve actually done a tutorial on this for PVE.

This approach is not optimal: if you have 2 machines, it’s fine, but if you have 3 or more, you end up either with a SPOF or having to set up a complex mesh of server-client pairs. Beyond 3, it’s practically unmanageable.

On the contrary, Tinc is a free software (GPLv2) allowing to set up multipoint VPNs (full mesh routing) between machines on the Internet. Tinc is designed for our use case!

Like any “open” service on the Internet, it is obviously necessary to do firewalling to block malicious traffic. For your information, Tinc uses port 655 in TCP and UDP, which you will therefore have to open in both directions between all of your servers, but block for the rest of the Internet.

# Install and Configure

We will have to install the package/binary on all the VPN machines. No worries on this side, tinc is [multiplatform (Windows too) and is packaged in many Linux distributions](https://www.tinc-vpn.org/download/) . On the Debian repositories, tinc is present without problem and I imagine that it will be the case for other distributions too.

`apt install tinc`

Once installed, we can create our VPN. To declare a VPN, we must add its name in a global configuration file:

`echo "vpntalhad" >> /etc/tinc/nets.boot`

We then create a directory of the same name, itself containing a _hosts_ folder .

`mkdir -p /etc/tinc/talhad/hosts`

In these folders, we will create a configuration file, /etc/tinc/talhad/tinc.conf. This file will allow us to describe the current node and who should connect to whom.

```
Name = node01  
AddressFamily = ipv4  
Address = 1.1.1.1  
Device = /dev/net/tun  
Mode = switch  
ConnectTo = node02  
ConnectTo = node03  
ConnectTo = node04  
[...]

```
In the important information, **Name** will be the (arbitrary) name of the node for the whole VPN, **Address** represents the public IP address of your node, and **ConnectTo**, the list of other **Names** of the other nodes.

From there, we can ask tinc to generate public key/private key pairs for all our servers. On all VPN nodes, run:

`echo -e '\n\n' | tincd -n vpntalhad -K4096`

If all goes well and all conf files and folders have been created correctly, _tinc_ should create a private key/public key pair, then create a file /etc/tinc/vpntalhad/hosts/node01 (on the one with Name node01).

Send (by scp for example) all the files to the hosts folder on all servers. Now all VPNs will know which public IP to contact and with which key to authenticate the traffic.

# Alternate method: tinc invite / tinc join

A nice feature of _tinc_ , but only in 1.1 (the latest version), is the ability to add nodes to an existing VPN via a simple tinc prompt command ( [see the docs](https://www.tinc-vpn.org/documentation-1.1/How-invitations-work.html#How-invitations-work) ).

I haven’t tested it, but apparently tinc creates a URL that allows you to invite anyone and configure your node with a tinc join.

The whole URL is around 80 characters long and looks like this: **_server.example.org:12345/cW1NhLHS-1WPFlcFio8ztYHvewTTKYZp8BjEKg3vbMtDz7w4_**

# OS Network Configuration

Now that we have all the prerequisites, we can configure the network aspect of our VPN. It depends on your distribution of course (or in the case of PVE, if you use OpenVSwitch or not for example). In the case of a recent Debian, we will manage all this with ip.

To handle this part, Tinc will ask you to create 2 scripts. A tinc-up.sh and a tinc-down.sh, respectively for starting and shutting down the VPN interface.

```sh
cat /etc/tinc/vpntalhad/tinc-up  
#!/bin/bash  
/sbin/ip link set vpntalhad up  
/sbin/ip addr add 10.0.0.1/24 dev vpntalhad  
/sbin/ip route add 10.0.0.0/24 dev vpntalhad  
cat /etc/tinc/vpntalhad/tinc-down  
#!/bin/bash  
/sbin/ip link set vpntalhad down  
chmod +x tinc-*
```

Nothing too complicated here, we ask the ip binary to create a vpntalhad interface. Then we assign it the virtual IP 10.0.0.1 (our node01). Finally, add a route so that all VPN traffic goes through this interface.

# We start it

Last step, we will ask systemd to start the VPN that we have just created as soon as the machine starts.

```sh
systemctl daemon-reload  
systemctl enable tinc@vpntalhad  
systemctl start tinc@vpntalhad
```

And there you have it!!! You now have a simple and effective multipoint VPN!

```
root@node01 ~ # ping node02  
PING node02 (10.0.0.2) 56(84) bytes of data.  
64 bytes from node02 (10.0.0.2): icmp_seq=1 ttl=64 time=17.1 ms
```

https://medium.com/@talhakhalid101/create-a-distributed-vpn-with-tinc-fb36dcf5f76a