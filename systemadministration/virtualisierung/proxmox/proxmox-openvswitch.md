---
tags:
  - proxmox
  - openvswitch
---
# Proxmox und Open vSwitch

Open vSwitch-Pakete installieren

```sh
apt-get -y install openvswitch-switch ethtool
```

Open vSwitch mit gre Tunnel

/etc/network/interfaces

```sh
auto vmbr0
allow-ovs vmbr0
iface vmbr0 inet manual
  ovs_type OVSBridge
  ovs_ports eth0 admin0 tep0

auto vmbr1
allow-ovs vmbr1
iface vmbr1 inet manual
  ovs_type OVSBridge
  post-up /root/ovs/gre1.sh

auto eth0
allow-vmbr0 eth0
iface eth0 inet manual
  ovs_bridge vmbr0
  ovs_type OVSPort

allow-vmbr0 admin0
iface admin0 inet static
  ovs_type OVSIntPort
  ovs_bridge vmbr0
  address 192.168.122.11
  netmask 255.255.255.0
  gateway 192.168.122.1

allow-vmbr0 tep0
iface tep0 inet static
  address 10.0.2.11
  netmask 255.255.255.0
  ovs_type OVSIntPort
  ovs_bridge vmbr0

```

gre1.sh

```sh
#!/bin/bashovs-vsctl set bridge vmbr1 stp_enable=trueovs-vsctl add-port vmbr1 gre1112 -- set interface gre1112 type=gre options:remote_ip=10.0.2.12ovs-vsctl add-port vmbr1 gre1113 -- set interface gre1113 type=gre options:remote_ip=10.0.2.13
```

