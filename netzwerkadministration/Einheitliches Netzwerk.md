# SUSE

* [sec-basicnet-nm](https://documentation.suse.com/sles/12-SP5/html/SLES-all/cha-basicnet.html#sec-basicnet-nm)

**start NetworkManager**

`systemctl start network`

**restart NetworkManager**

`systemctl restart network`

**stop NetworkManager**

`systemctl stop network`

# Red Hat like Systeme

Major Version 9

/etc/sysconfig/network-scripts/readme-ifcfg-rh.txt

```sh
NetworkManager stores new network profiles in keyfile format in the
/etc/NetworkManager/system-connections/ directory.

Previously, NetworkManager stored network profiles in ifcfg format
in this directory (/etc/sysconfig/network-scripts/). However, the ifcfg
format is deprecated. By default, NetworkManager no longer creates
new profiles in this format.

Connection profiles in keyfile format have many benefits. For example,
this format is INI file-based and can easily be parsed and generated.

Each section in NetworkManager keyfiles corresponds to a NetworkManager
setting name as described in the nm-settings(5) and nm-settings-keyfile(5)
man pages. Each key-value-pair in a section is one of the properties
listed in the settings specification of the man page.

If you still use network profiles in ifcfg format, consider migrating
them to keyfile format. To migrate all profiles at once, enter:

# nmcli connection migrate

This command migrates all profiles from ifcfg format to keyfile
format and stores them in /etc/NetworkManager/system-connections/.

Alternatively, to migrate only a specific profile, enter:

# nmcli connection migrate <profile_name|UUID|D-Bus_path>

For further details, see:
* nm-settings-keyfile(5)
* nmcli(1)
```

# Ubuntu Netplan umstellen

```sh
# cat /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: NetworkManager
```

**Start network manager**

```sh
sudo systemctl start NetworkManager
sudo systemctl enable NetworkManager
<<<<<<< HEAD
```

Verzeichnis für die Zentrale Konfiguration _/etc/NetworkManager/_

- conf.d  
- dispatcher.d
- dnsmasq.d
- dnsmasq-shared.d
- NetworkManager.conf
- system-connections

Im Unterordner `/etc/NetworkManager/system-connections/*` ist dann die Konfiguration der Interfaces zu finden und sollte auch dort rein geschrieben werden

**Interface der Verwaltung durch nmcli nach `/etc/NetworkManager/system-connections/*` schreiben**

`nmcli connection add type ethernet ifname eth0`

**Ereignisse im lokalem Netzerk anzeigen**

`nmcli monitor`

# Anpassungen für VmWare

Udev rules

`/etc/udev/rules.d/70-persistent-net.rules`

```
ACTION=="add", SUBSYSTEM=="net", DEVTYPE!="?*", ATTR{address}=="00:24:9b:0d:6d:54", NAME="startech_en0"
```

https://serverfault.com/questions/610967/network-adapter-to-eth-number-mapping-for-vmware

https://www.suse.com/support/kb/doc/?id=000019043

```sh
nmcli monitor
```

udev rules einheitlich definieren

https://unix.stackexchange.com/questions/91085/udev-renaming-my-network-interface

https://forums.veeam.com/vmware-vsphere-f24/linux-vm-lose-ip-mac-networking-after-replication-backup-t4836.html
https://github.com/coreos/bugs/issues/2437

https://unix.stackexchange.com/questions/91085/udev-renaming-my-network-interface

https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/consistent-network-interface-device-naming_configuring-and-managing-networking#consistent-network-interface-device-naming_configuring-and-managing-networking