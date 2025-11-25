---
tags:
  - kvm
  - ansible
  - virtuallisierung
  - qemu
  - images
---
# Qemu

**Libvirt Adresse**

`qemu+ssh://$USER@$HOST/system`

Hier wird eine Verbindung mit ssh aufgebaut

# Kleints einrichten

## libguestfs

## Allow Libvirt for standard user accounts

/etc/libvirt/libvirtd.conf

```
unix_sock_group = "libvirt"
unix_sock_rw_perms = "0770"
```

```sh
sudo usermod -a -G libvirt $(whoami)newgrp libvirt
```

Restart libvirt daemon.

```sh
sudo systemctl restart libvirtd.service
```
## Enable Nested Virtualization

```
### Intel Processor ###
sudo modprobe -r kvm_intel
sudo modprobe kvm_intel nested=1

### AMD Processor ###
sudo modprobe -r kvm_amd
sudo modprobe kvm_amd nested=1
```

To make this configuration persistent,run:

```
echo "options kvm-intel nested=1" | sudo tee /etc/modprobe.d/kvm-intel.conf
```
# KVM Einrichtung mit Ansible

**Projekt erstellen und indes Verzeichnis wechseln**

```sh
mkdir -p kvmlab/roles && cd kvmlab/roles
```

**Kvm rollen für Ansible initatliesieren**

```shell
$ ansible-galaxy role init kvm_provision

- Role kvm_provision was created successfully
```

**Wechsel in das erstellten rollen Verzeichnis**

```shell
$ cd kvm_provision
$ ls
defaults files handlers meta README.md tasks templates tests vars
```

> Die rolle nutzt files handlers und vars nicht

**Hinzufügen von Variablen in defaults/main.yml**

```yaml
---
# defaults file for kvm_provision
base_image_name: Fedora-Cloud-Base-34-1.2.x86_64.qcow2
base_image_url: https://download.fedoraproject.org/pub/fedora/linux/releases/34/Cloud/x86_64/images/{{ base_image_name }}
base_image_sha: b9b621b26725ba95442d9a56cbaa054784e0779a9522ec6eafff07c6e6f717ea
libvirt_pool_dir: "/var/lib/libvirt/images"
vm_name: f34-dev
vm_vcpus: 2
vm_ram_mb: 2048
vm_net: default
vm_root_pass: test123
cleanup_tmp: no
ssh_key: /root/.ssh/id_rsa.pub
```

`vm-template.xml.j2`

```xml
<domain type='kvm'>
  <name>{{ vm_name }}</name>
  <memory unit='MiB'>{{ vm_ram_mb }}</memory>
  <vcpu placement='static'>{{ vm_vcpus }}</vcpu>
  <os>
    <type arch='x86_64' machine='pc-q35-5.2'>hvm</type>
    <boot dev='hd'/>
  </os>
  <cpu mode='host-model' check='none'/>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='{{ libvirt_pool_dir }}/{{ vm_name }}.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x05' slot='0x00' function='0x0'/>
    </disk>
    <interface type='network'>
      <source network='{{ vm_net }}'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
    </interface>
    <channel type='unix'>
      <target type='virtio' name='org.qemu.guest_agent.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
    </channel>
    <channel type='spicevmc'>
      <target type='virtio' name='com.redhat.spice.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='2'/>
    </channel>
    <input type='tablet' bus='usb'>
      <address type='usb' bus='0' port='1'/>
    </input>
    <input type='mouse' bus='ps2'/>
    <input type='keyboard' bus='ps2'/>
    <graphics type='spice' autoport='yes'>
      <listen type='address'/>
      <image compression='off'/>
    </graphics>
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x0'/>
    </video>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x06' slot='0x00' function='0x0'/>
    </memballoon>
    <rng model='virtio'>
      <backend model='random'>/dev/urandom</backend>
      <address type='pci' domain='0x0000' bus='0x07' slot='0x00' function='0x0'/>
    </rng>
  </devices>
</domain>
```
## Image Formate

Über die Jahre wurden immer wieder neuer Standards für die Speicherung der Virtuellen Festplatten umgesetzt Heute kann man Sie wie Folgt Klassifizieren

1.Read only Images z.B ISO
2.Raw Daten werden 1 zu 1 Block orientiert auf einen Datenträger geschrieben. Lesen und Schreiben ist möglich.
3.High Level Images sind mit einigen Futures erweitert und haben eine Struktur zur besseren Administration.
### Verschlüsselung

Es kommt immer wieder vor , dass der Auftraggeber die Anforderung hat das die gespeicherten Daten in Verschlüsselter Form vorhanden sein müssen hier gibt es mit libvirt ein System zum Management der Verschlüsselten Images, da andernfalls immer jemand das Secret manuell eingeben müsste
### Gold Image

Ein Gold Image oder auch im Englischen als Base Image bekannt, ist spezielles Image welches als Basis für andre Images dient.
In der Praxis entsteht als erstes ein 1:1-Klone das sogenannte Clone Image, da es nach dem Klonen genau wie das Gold Image ist können Änderungen am Gold Image in das Delta Image gespeichert werden
### Secret mit virsh anlegen

Das einrichten des sectres ist recht einfach mit:

`virsh secret-define secret.xml`

Haben wir schon das Secret hinterlegt nun muss [virsh](../virsh) noch wissen welche Images es mit dem Erstellten secret öffnen/Starten kann.
Wenn man sich in der Konsole befunden hat wird sich [virsh](../virsh) schon automatisch ein Image aussuchen das meist auch Richtig ist, kann aber mit
`virsh secret-list` kontrolliert werden.
Insbesondere wenn ein Image nachträglich verschlüsselt wurde wird es nicht immer automatisch auch vom [virsh](../virsh) festgestellt, wenn einen Nachbearbeitung notwendig wird muss die XML Datei Manuel angepasst werden
### Images Builder

**Vm Image vergrößern:**

`qemu-img resize ubuntu-server.qcow2 +5GB`

wenn die Platte nicht gefunden werden kann muss noch `partprobe` ausgeführt werden auf dem Gast .

danach ist das Image größer muss aber noch auf dem Gast  angepasst werden .

**Größe des Images Überprüfen**

Es kommt manchmal vor, dass die Kapazität noch nicht genutzt werden kann also am besten immer prüfen !
`sudo qemu-img check -r all /var/lib/libvirt/images/web.qcow2`

**Gast Partition vergrößern**
Die erweiterung des Images wird als device interpritiert und kann zum Beispiel bei LVM als PV hinzugefügt werden .

Device  /dev/hdc1 zur my_volume_group Volume Gruppe hinzufügen

`vgextend my_volume_group /dev/hdc1`

Logisches Volume maximal erweitern

`lvextend -l +100%FREE /dev/mapper/cl-root`

Dateisystem Prüfen

`mount`

Ausgabe:

`# /dev/vda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)`

Dateisystem erweitern bei xfs Dateisystemen

`xfs_growfs /dev/mapper/cl-root`

## Text Konsolen

**Textkonsole bei Centos ermöglichen**
`grubby --update-kernel=ALL --args="console=ttyS0"`

## Netzwerk

Quelle:

* [rhel7-access-virtual-machines-console/](https://www.certdepot.net/rhel7-access-virtual-machines-console/)

# qemu tools

qemu-img

qemu-io

qemu-storage-daemon

qemu-system-x86_64

grub-firmware-qemu - GRUB firmware image for QEMU
ipxe-qemu  - PXE-Boot-Firmware - ROM-Images für Qemu
qemu-kvm 
qemu-keymaps
qemu-web-desktop -Remote desktop service with virtual machines in a browser.

virt-p2v - physical-to-virtual machine converter
virt-v2v - virtual-to-virtual machine converter
virtnbdbackup - Backup utility for libvirt

## KVM CPU Pinning

CPU Pining auf Dual Socket CPU Board einrichten

Ohne CPU Pining verliert man in der Regel ca 50% CPU Power in der VM sollte die VM mehr als einen VCPU konfiguriert haben  


Vorarbeit : ( herausfinden Welches Cores zu welchem Socket gehören )

```
root@ws07a:~# lscpu|grep NUMA
NUMA node(s):          2
NUMA node0 CPU(s):     0,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38
NUMA node1 CPU(s):     1,3,5,7,9,11,13,15,17,19,21,23,25,27,29,31,33,35,37,39
```

Im Beispiel gehören die CPUs 0,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38 zur ersten Echten CPU Core ( node 0 )

Will man z.b 4 VCores einer VM auf einen Socket binden sieh das Setup so aus:

```
root@ws07a:~# virsh vcpupin ws06 0 0 --live 
root@ws07a:~# virsh vcpupin ws06 1 2 --live 
root@ws07a:~# virsh vcpupin ws06 2 4 --live 
root@ws07a:~# virsh vcpupin ws06 3 6 --live 
```
## KVM VM IO Drosselung einrichten

Im Live Betrieb ein VM Limitieren ( Das kann hilfreich sein wenn z.b. eine VM sämtlich Disk IO Performance verbraucht > z.b. DDOS Attacke ) 
  
```sh
root@ws07a:~# virsh blkdeviotune ws06 sda --read_bytes_sec 10485760
root@ws07a:~# virsh blkdeviotune ws06 sda --write_bytes_sec 10485760 
```

Limitier Disk Read und Write auf 10MB/sek

`root@ws07a:~# vvirsh blkdeviotune ws06  sda --total_iops_sec 200`

Limitiert die VM auf Max 200IOPs/Sek   ( 200IOPs entspricht ca einer 1-3TB Sata Platte )

[[qemu]]
[[virt-install]]

**Installation Prüfen**

`virt-host-validate`

