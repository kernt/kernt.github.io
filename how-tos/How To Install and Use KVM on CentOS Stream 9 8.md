---
tags:
  - kvm
  - virtualisierung
  - libvirt
---
In today’s guide we will be installing KVM on CentOS Stream Linux operating system. KVM (Kernel-based Virtual Machine) is an open source virtualization technology built into Linux Kernel. KVM can be used on any x86 hardware with virtualization extensions (Intel VT or AMD-V). The role of KVM as hypervisor is specification of the host resources – Memory, CPUs, and virtual devices availed to the virtual machine instances being defined.

KVM consists of a loadable kernel module, `kvm.ko`, that provides the core virtualization infrastructure and a processor specific module, kvm-intel.ko or kvm-amd.ko. Below diagram illustrates how the KVM hypervisor virtualizes the compute resources for Linux on KVM.

[![kvm virtualization technology](https://computingforgeeks.com/wp-content/uploads/2021/06/kvm-virtualization-technology.jpg?ezimgfmt=ng%3Awebp%2Fngcb23%2Frs%3Adevice%2Frscb23-1 "How To Install and Use KVM on CentOS Stream 9|8 1")](https://computingforgeeks.com/wp-content/uploads/2021/06/kvm-virtualization-technology.jpg?ezimgfmt=ng%3Awebp%2Fngcb23%2Frs%3Adevice%2Frscb23-1)

## Step 1: Confirm CPU Virtualization support

Your hardware needs to have CPU virtualization extensions; Intel VT for Intel or AMD-V for AMD processor. In some systems, this is disabled on BIOS and you may need to enable it.

```sh
cat /proc/cpuinfo | egrep --color "vmx|svm"
```

The `lscpu` command can also be used to check for virtualization CPU extensions:

```sh
lscpu | grep Virtualization
Virtualization: VT-x
```

This confirms I have Intel processor and VT-x extension
## Step 2: Install KVM Virtualization Tools

Let’s start by performing system upgrades to use Kernel updates that could be available.

```sh
sudo dnf -y update
```

If there are Kernel related updates consider performing system reboot then install KVM Virtualization Tools on CentOS Stream:

```sh
sudo dnf install @virt
```

Accept installation by pressing the **y** key:

```sh
Transaction Summary
==================================================================================
Install  166 Packages

Total download size: 99 M
Installed size: 352 M
Is this ok [y/N]: y
```

Check if Kernel modules are loaded:

```sh
$ lsmod | grep kvm
kvm_intel             315392  0
kvm                   847872  1 kvm_intel
irqbypass              16384  1 kvm
```

## Step 3: Install other KVM Tools

Let’s perform installation of other tools that helps with the management of Virtual Machines on KVM:

```sh
sudo dnf -y install bridge-utils virt-top libvirt-devel libguestfs-tools
```

We have an example guide showing how `libguestfs-tools` can be used: [How to mount VM virtual disk on KVM hypervisor](https://computingforgeeks.com/how-to-mount-vm-virtual-disk-on-kvm-hypervisor/)

## Step 4: Start and enable KVM service

By default, KVM daemon `libvirtd` is not started, start the service using the command:

```sh
sudo systemctl start libvirtd
```

Also enable the service to be started at system boot:

```sh
sudo systemctl enable libvirtd
```

Check if service is started successfully:

```sh
$ systemctl status libvirtd
● libvirtd.service - Virtualization daemon
   Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2024-07-24 20:29:16 CEST; 59s ago
     Docs: man:libvirtd(8)
           https://libvirt.org
 Main PID: 23126 (libvirtd)
    Tasks: 19 (limit: 32768)
   Memory: 15.3M
   CGroup: /system.slice/libvirtd.service
           ├─23126 /usr/sbin/libvirtd --timeout 120
           ├─23251 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
           └─23252 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
....
```
## Step 5: Install Virt Manager GUI – Optional

If running a Desktop Environment on your CentOS Stream, you can install the `virt-manager` package which provides Desktop Management application for your KVM Virtual Machines.

```sh
sudo dnf install virt-manager
```
## Step 6: Network Bridge (Optional)

The Linux bridge **virbr0** is created at the time of installation and can be used to create Virtual Machines that doesn’t need external IP connectivity. It uses NAT to give VMs internet access.

```sh
brctl show
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.525400876bb5	yes		virbr0-nic
```

If you need a bridge with external connections support, refer to the guide below on creation:

- [How to Create a Linux Network Bridge on RHEL / CentOS 8](https://computingforgeeks.com/how-to-create-a-linux-network-bridge-on-rhel-centos-8/)
## Step 7: Creating Virtual Machines

You can use virt-install command to create a Linux Virtual Machine on KVM.

I’ll download [CentOS Stream DVD installation ISO](https://www.centos.org/centos-stream/) file

```sh
cd ~/
wget https://ftp.iij.ad.jp/pub/linux/centos-stream/9-stream/BaseOS/x86_64/iso/CentOS-Stream-9-latest-x86_64-dvd1.iso
```

VM installation using _virt-install_:

```sh
virt-install \
--name centos-stream-9 \
--ram 2048 \
--vcpus 2 \
--disk path=/var/lib/libvirt/images/centos-stream-9.img,size=20 \
--os-variant centos-stream9 \
--os-type linux \
--network bridge=virbr0 \
--graphics none \
--console pty,target_type=serial \
--location ~/CentOS-Stream-9-latest-x86_64-dvd1.iso \
--extra-args 'console=ttyS0,115200n8 serial'
```

The installation is on text mode but the procedure of installation is similar to GUI. After finishing the installation, reboot the instance and login

```sh
CentOS Stream 9
..
Activate the web console with: systemctl enable --now cockpit.socket

localhost login:
```

You can also login through console:

```sh
virsh console centos-stream-9
```

Press **ENTER** key on getting:

```sh
Escape character is ^]
```

More guides on KVM:

- [How to Create CentOS / Fedora / RHEL VM Templates on KVM](https://computingforgeeks.com/how-to-create-centos-fedora-rhel-vm-templates-on-kvm/)
- [Virsh Commands cheat sheet](https://computingforgeeks.com/virsh-commands-cheatsheet/)
- [How to Provision VMs on KVM with Terraform](https://computingforgeeks.com/how-to-provision-vms-on-kvm-with-terraform/)
- [Configure virt-manager as a non-root user on Linux](https://computingforgeeks.com/use-virt-manager-as-non-root-user/)