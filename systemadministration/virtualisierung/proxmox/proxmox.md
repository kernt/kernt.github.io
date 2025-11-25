---
tags:
  - proxmox
  - virtualisierung
---
# Proxmox

Proxmox VE (Proxmox Virtual Environment; kurz PVE) ist eine auf Debian basierende Open-Source-Virtualisierungsplattform zum Betrieb von virtuellen Maschinen mit einer Web-Oberfläche zur Einrichtung und Steuerung von x86-Virtualisierungen. Die Umgebung basiert auf QEMU mit der Kernel-based Virtual Machine (KVM).
PVE bietet neben dem Betrieb von klassischen virtuellen Maschinen (Gastsystemen), die auch den Einsatz von Virtual Appliances erlauben, auch Linux Containers (LXC) an.
## Proxmox Administration

_No Subscription Warnmeldung entfernen_

```sh
sed -i.bak 's/NotFound/Active/g' /usr/share/perl5/PVE/API2/Subscription.pm && systemctl restart pveproxy.service
```

_Versionen ausgeben_

`pveversion -v`
## Quellen

* [Proxmox Installation](/proxmox/proxmox-installation)
* [Proxmox API](/proxmox/proxmox-api)
* [Proxmox Open vSwitsh](/proxmox/proxmox-openvswitch)