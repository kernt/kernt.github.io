---
tags:
  - ip
  - vlan
  - tcpdump
---

**Vlan module Prüfen**

```sh
modprobe --first-time 8021q
```

**Module Infos**

```sh
modinfo 8021q
```

Im Verzeichnis _/proc/net/vlan_ werden die Infos zur Konfiguration des vlan's angelegt.

**Konfiguration des Vlans ausgeben

```sh
ls -l /proc/net/vlan/
```

**Ausgeben des konfig Files**

```sh
cat /proc/net/vlan/config
```

**Inhalt der konfiguration ausgeben**

```sh
cat /proc/net/vlan/vlan10-if
```

****

**Vlan mit #ip anlegen**

`ip link add link eth0 name eth0.100 type vlan id 100`

**Aufzeichnen von tagged frammes die das Phykalische device erreichen mit #tcpdump**

`tcpdump -nnei enp1s0 -vvv`

VLAN tagging is a method of handling more than one VLAN on a network port. VLAN tagging is used to tell which packet belongs to which VLAN as packets traverse a network medium. In this guide, we will configure 802.1q VLAN Tagging in a Network interface on RHEL / CentOS and Fedora system.

To create a VLAN, an interface is created on top of another interface referred to as the _parent interface_. The VLAN interface will tag packets with the VLAN ID as they pass through the interface, and returning packets will be untagged.

Before doing any configuration, ensure 8021q module is loaded.

```
sudo modprobe --first-time 8021q
modinfo 8021q
```

In this example, I’ll configure an **enp6s0** interface on the server.

```
$ ip link  show  dev enp6s0
2: enp6s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 38:90:a5:14:96:54 brd ff:ff:ff:ff:ff:ff
```

You can use the network manager command line tool – **nmcli** to accomplish this or directly edit network configuration files.

## Manually Edit Configuration files

Edit the parent interface configuration file and set like below.

```
$ sudo vim /etc/sysconfig/network-scripts/ifcfg-enp6s0
TYPE=Ethernet
NAME=enp6s0
DEVICE=enp6s0
BOOTPROTO=none
ONBOOT=yes
```

As seen above, we’ve set the interface to come up on boot and we don’t assign IP information here.

Now configure the VLAN interface. The configuration file name should be the parent interface plus a `.` character plus the VLAN ID number. In my setup, the VLAN ID is **21**, and the parent interface is _enp6s0_, so the configuration file name should be:

```
sudo vim /etc/sysconfig/network-scripts/ifcfg-enp6s0.21
```

All network configuration information is added into this file.

```
PHYSDEV=enp6s0
NAME=enp6s0.21
BOOTPROTO=none
ONBOOT=yes
IPADDR=172.10.10.11
GATEWAY=172.10.10.1
DNS1=172.10.10.1
DNS2=8.8.8.8
PREFIX=24
VLAN=yes
TYPE=Vlan
VLAN_ID=21
```

After making the change, restart the networking service in order for the changes to take effect.

```
sudo systemctl restart network
```

Alternatively, manually bring up the interface.

```
sudo ifdown enp6s0 && sudo ifup enp6s0
sudo ifup enp6s0.21
```

Confirm IP address information for the interface.

```
$ ip ad | grep enp6s0
2: enp6s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
49: enp6s0.21@enp6s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 172.10.10.11/24 brd 172.10.10.255 scope global noprefixroute enp6s0.21
```

## Using NMCLI Tool

The same configurations can be done purely from the command-line interface. For this method, the _NetworkManager_ service should be running.

```
systemctl status NetworkManager
```

Check current network configurations.

```
nmcli con show
```

To create an 802.1Q VLAN interface on Ethernet interface **enp6s0**, with VLAN interface **VLAN21** and ID **21**, issue a command as follows:

```
nmcli con add type vlan con-name enp6s0.21 dev enp6s0 id 21 connection.autoconnect yes
```

You can then assign an IP address to the VLAN Interface.

```
nmcli connection modify enp6s0.21 ipv4.addresses 172.10.10.11/24 \
  ipv4.method manual ipv4.gateway 172.10.10.1 \
  ipv4.dns 172.10.10.1 +ipv4.dns 8.8.8.8
```

To view all the parameters associated with the VLAN created above, issue a command as follows.

```
nmcli connection show enp6s0.21
```

You have successfully configured VLAN tagging on an interface in RHEL / CentOS or Fedora server.

### Creating Bridge on VLAN Interface

Creation of VLAN interface **_enp89s0.30_**:

```
nmcli connection add type vlan con-name enp89s0.30 dev enp89s0 id 30 master br-vlan30 connection.autoconnect yes
```

Bridge creation **_br-vlan30_**:

```
nmcli connection add type bridge con-name br-vlan30 ifname br-vlan30 connection.autoconnect yes \
     ipv4.method  manual  ipv4.addresses  172.20.30.7/24 ipv4.gateway  172.20.30.1 ipv4.dns 172.20.20.252
```

Similar articles:

- [Add Secondary IP Address to CentOS / RHEL Network Interface](https://computingforgeeks.com/adding-a-secondary-ip-address-to-rhel-centos-8-network-interface/)
- [ifconfig vs ip usage guide on Linux](https://computingforgeeks.com/ifconfig-vs-ip-usage-guide-on-linux/)
- [Creating Openstack Network and Subnets](https://computingforgeeks.com/creating-openstack-network-and-subnets/)