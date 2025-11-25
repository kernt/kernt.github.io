---
tags:
  - networking
  - system-administration
  - nmcli
---
This mode can be used for computer script processing as you can see a terse output displaying only the values.

```none
nmcli connection show
```

```
nmcli con up id bond0
nmcli con up id port0
nmcli dev disconnect bond0
nmcli dev disconnect ens3
```

```sh
uuidgen eth0
/etc/sysconfig/network-scripts/ifcfg-eth0
dmidecode  | grep -i uuid
dmesg | grep -i e69bb090-bcc3-485e-b6db-45878b068b63
```

```
cp -r /etc/sysconfig/network-scripts  ~/
```

root@vmd47205:~# lspci -s 00:12.0 -vv
00:12.0 Ethernet controller: Red Hat, Inc. Virtio network device
        Subsystem: Red Hat, Inc. Virtio network device

`nmcli con add type ethernet con-name enp0s3 ifname enp0s3`

       load filename...
           Load/reload one or more connection files from disk. Use this after manually editing a connection file to ensure that NetworkManager is
           aware of its latest state.
       import [--temporary] type type file file
           Import an external/foreign configuration as a NetworkManager connection profile. The type of the input file is specified by type option.
           Only VPN configurations are supported at the moment. The configuration is imported by NetworkManager VPN plugins.  type values are the
           same as for vpn-type option in nmcli connection add. VPN configurations are imported by VPN plugins. Therefore the proper VPN plugin has
           to be installed so that nmcli could import the data.
           The imported connection profile will be saved as persistent unless --temporary option is specified, in which case the new profile won't
           exist after NetworkManager restart.

https://developer-old.gnome.org/NetworkManager/stable/nmcli.html

https://access.redhat.com/documentation/de-de/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_nmcli

nmcli connection show eth0 |grep connection.uuid

```
nmcli -f name,autoconnect c s
nmcli -p -m multiline -f all con showq
nmcli con add type ethernet con-name static2 ifname enp0s3 ip4 192.168.1.50/24 gw4 192.168.1.1
```

**nmcli connection modify _Example_ ipv6.method "disabled"**

https://access.redhat.com/documentation/de-de/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_static_routes_with_ip_commands