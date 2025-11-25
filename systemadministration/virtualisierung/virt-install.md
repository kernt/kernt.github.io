---
tags:
  - kvm
  - virtuall install
---
# virtuall Install

## Debian 12

```sh
virt-install \
--name debian12 \
--ram 1024 \
--disk path=./debian12.qcow2,size=8 \
--vcpus 1 \
--os-type linux \
--os-variant generic \
--network bridge=virbr0 \
--graphics none \
--console pty,target_type=serial \
--location 'http://ftp.nl.debian.org/debian/dists/Debian12.5/main/installer-amd64/' \
--extra-args 'console=ttyS0,115200n8 serial'
```

## CentOS 9

```sh
virt-install \
--name centos9 \
--ram 2048 \
--disk path=./centos9.qcow2,size=8 \
--vcpus 1 \
--os-type linux \
--os-variant centos9 \
--network bridge=virbr0 \
--graphics none \
--console pty,target_type=serial \
--location 'http://mirror.i3d.net/pub/centos/9/os/x86_64/' \
--extra-args 'console=ttyS0,115200n8 serial'
```

## Ubuntu 24.04

```sh
virt-install \
--name ubuntu2404 \
--ram 2048 \
--disk path=./ubuntu2404.qcow2,size=8 \
--vcpus 1 \
--os-type linux \
--os-variant generic \
--network bridge=virbr0 \
--graphics none \
--console pty,target_type=serial \
--location 'http://archive.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/' \
--extra-args 'console=ttyS0,115200n8 serial'
```

## OpenSUSE 13

```s
virt-install \
--name opensuse13 \
--ram 1024 \
--disk path=./opensuse13.qcow2,size=8 \
--vcpus 1 \
--os-type linux \
--os-variant generic \
--network bridge=virbr0 \
--graphics none \
--console pty,target_type=serial \
--location 'http://download.opensuse.org/distribution/openSUSE-current/repo/oss/' \
--extra-args 'console=ttyS0,115200n8 serial'
```

### Generic ISO

Download an ISO file and give the filename to the --cdrom= parameter. This is used instead of --location. A VNC console is available on localhost, port 5999 for you to use.

**An example for FreeBSD 10. First download the ISO:**

```sh
wget http://ftp.freebsd.org/pub/FreeBSD/releases/ISO-IMAGES/10.1/FreeBSD-10.1-RELEASE-amd64-dvd1.iso
```

**Then start virt-install:**

```sh
 virt-install \
--name freebsd10 \
--ram 1024 \
--disk path=./freebsd10.qcow2,size=8 \
--vcpus 1 \
--os-type generic \
--os-variant generic \
--network bridge=virbr0 \
--graphics vnc,port=5999 \
--console pty,target_type=serial \
--cdrom ./FreeBSD-10.1-RELEASE-amd64-dvd1.iso \
```

**os-variant**

For install use the folloing

```sh
sudo apt install libosinfo-bin
```

You can get a list of supported operating system variants with the osinfo-query os command. Below you'll find an example output:

```sh
osinfo-query os
 Short ID             | Name                                               | Version  | ID                                      
----------------------+----------------------------------------------------+----------+-----------------------------------------
 debian7              | Debian Wheezy                                      | 7        | http://debian.org/debian/7              
 freebsd10.0          | FreeBSD 10.0                                       | 10.0     | http://freebsd.org/freebsd/10.0         
 openbsd5.5           | OpenBSD 5.5                                        | 5.5      | http://openbsd.org/openbsd/5.5          
 rhel6.5              | Red Hat Enterprise Linux 6.5                       | 6.5      | http://redhat.com/rhel/6.5              
 rhel7.0              | Red Hat Enterprise Linux 7.0                       | 7.0      | http://redhat.com/rhel/7.0              
 ubuntu12.04          | Ubuntu Precise Pangolin LTS                        | 12.04    | http://ubuntu.com/ubuntu/12.04          
 win3.1               | Microsoft Windows 3.1                              | 3.1      | http://microsoft.com/win/3.1            
 win7                 | Microsoft Windows 7                                | 6.1      | http://microsoft.com/win/7              
 winxp                | Microsoft Windows XP                               | 5.1      | http://microsoft.com/win/xp  
```

## Kickstart and debootstrap

If you have a kickstart file set up you can give it directly to the vm using the `--extra-args` parameter:

```sh
 --extra-args "ks=http://server/vm.ks"
```

If you don't have a server set up you can inject a file into the initrd and use that for kickstarting:

```sh
--initrd-inject=vm.ks --extra-args "ks=file:/vm.ks"
```

preseed.cfg is a regular preseed file (as described in the Debian Wiki) in your local filesystem. It must be named preseed.cfg in order for d-i to pick it up from the initrd.

Here is another, rather boring, image of a Debian install via virt-install:

Starting a VM

To start a VM you've just created after the installation, use the virsh start NAME command:

virsh start centos7

Use the virsh list --all to list all available virtual machines, including powered off ones:

```s
virsh list --all
 Id    Name                           State
----------------------------------------------------
 4     centos7                        running
 -     debian7                        shut off
 -     win7                           shut off
 -     win98                          shut off
 -     winxp                          shut off
```

Stopping and removing

To stop a VM, you give the (unintuitive) command virsh destroy NAME:

virsh destroy centos7

It will not remove any data, just stop the VM by pulling the virtual power cable.
If you want to remove the VM from the virsh list, you need to undefine it:

virsh undefine centos7

This will remove the configuration. If you don't undefine the VM and want to try the virt-install again it will give an error like this:

ERROR  Guest name 'centos7' is already in use.
You do manually need to remove the virtual disk after undefining a vm.

[[qemu]]
[[kvm qemu]]