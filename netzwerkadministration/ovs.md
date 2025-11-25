---
tags:
  - networking
  - ovs
  - openvswitch
---
# ovs 

```

                                +-------------------------------------+
                                |  Server                             |
                                |                                     |
+-----------------------+       |                                     |
|   Laptop              |       |         Bridge                      |
|                       |       +--------+---------+---------+--------+
+-----------+-----------+       |  vpn   |  vme    |  vme.10 | vnet1  |  +--------------+
|  vme.10   |   vme     |       |        |         |  tag=10 | tag=10 |  |  Virtual     |
192.168.10.2|192.168.0.2+------->        |         |         |        <--+  machine     |
+-----------+-----------+       +--------+---------+---------+--------+  +--------------+

  vme.10: Linux VLAN with id=10     vme:     ovs bridge              [192.168.0.1]
  vme:    VPN tap                   vme.10:  port for VLAN 10 access [192.168.10.200]
                                    vpn:     port used by VPN server []
                                    vnet1:   auto-created by libvirt []

$ ovs-vsctl show
Bridge vme
        Port "vnet1"
            tag: 10
            Interface "vnet1"
        Port vpn
            Interface vpn
                type: internal
        Port "vme.10"
            tag: 10
            Interface "vme.10"
                type: internal
        Port vme
            Interface vme
                type: internal
```

Quellen:

* [ovs](https://mail.openvswitch.org/pipermail/ovs-discuss/2016-October/042723.html)
* [Der Software-Switch Open Vswitch](https://www.linux-magazin.de/ausgaben/2013/02/open-vswitch/)