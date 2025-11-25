---
tags:
  - vagrant
  - Plugin
  - libvirt
---
# Vagrant Plugin libvirt

[Libvirt](https://wiki.debian.org/libvirt) is a good provider for Vagrant because it's faster than [VirtualBox](https://wiki.debian.org/VirtualBox) and it's in the main repository. Install it as follow:

```sh
sudo apt install vagrant-libvirt libvirt-daemon-system
```

Ensure your user is a member of the libvirt group (this is needed to manage libvirt via the qemu:///system uri, which is the default for vagrant-libvirt)

`sudo usermod --append --groups libvirt $USER`

logout and login so that your group membership applies

```sh
# should return libvirt
groups | grep -o libvirt
```

Quellen und Beispiele

* [vagrant-and-libvirt-kvm-qemu-setting-up-boxes-the-easy-way](http://www.lucainvernizzi.net/blog/2014/12/03/vagrant-and-libvirt-kvm-qemu-setting-up-boxes-the-easy-way/)

* [Vagrant::Hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater)
* [vagrant-linode](https://github.com/displague/vagrant-linode)