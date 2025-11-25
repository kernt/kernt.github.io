---
tags:
  - udev
  - gnu-tools
  - system-administration
  - udevadm
  - ectest
---
# Udev Infos anzeigen lassen

**Displaying Device Properties**

```bash
udevadm info -a -p /block/sr0
```

**Interfaces auflisten**

`ip link show`

**Nameschema ausgeben lassen**

`udevadm info --query=property --property=ID_NET_NAMING_SCHEME /sys/class/net/eno1'`

**Hinzufügen des name schema auf alles kernel**

`grubby --update-kernel=ALL --args=net.naming-scheme=_rhel-8.4_`

**NetworkManager connection profile ausgeben**

`nmcli connection modify example_profile connection.interface-name "eno1np0"`

his udev rule will tell linux to remove the pci device when a network device which has the ID_NET_NAME_ONBOARD of eno2 is added. Add it to e.g.  _/etc/udev/rules.d/90-disable-eno2.rules_ :

```sh
ACTION=="add", SUBSYSTEM=="net", ENV{ID_NET_NAME_ONBOARD}=="eno2", RUN+="/bin/sh -c 'echo 1 > /sys$DEVPATH/device/remove'"
```

**Find the path marked "P" with this command.**

`udevadm info --path=/sys/class/net/eno2`

**Test with this command with the path from above**

`udevadm test --action="add" /devices/pci0000:00/0000:00:1c.4/0000:03:00.0/net/eno2 2>&1 | less`

* [configuring_and_managing_networking/consistent-network-interface-device-naming_configuring-and-managing-networking](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/consistent-network-interface-device-naming_configuring-and-managing-networking)

**Input änderungen ausgeben**

`udevadm trigger --subsystem-match=input --action=change`

# evtest

```bash
evtest /dev/input/event4
```

[evtest](https://manpages.ubuntu.com/manpages/trusty/man1/evtest.1.html)