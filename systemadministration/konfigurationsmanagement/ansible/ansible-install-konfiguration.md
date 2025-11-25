---
tags:
  - konfigurationsmanagement
  - ansible
---
# Installation von Ansible

Da es nicht immer auf Anhieb möglich ist mit der node auf Anhieb zuzugreifen und alle notwendigen Voraussetzungen automatisiert einzurichten kann man mit :

`ansible myhost --sudo -m raw -a "yum install -y python2 python-simplejson"`

alle Voraussetzungen um Ansible einrichten durchführen.
**myhost** ist hier der Zielhost und **python-simplejson** ein Playbook das alle Abhängigkeiten einrichtet.

## Installation Ubuntu

Für Ubuntu kann man [PPA](https://launchpad.net/~ansible/+archive/ansible) verwenden um auf ein aktuelles Release zu kommen

**Repo für Ubuntu xenial:**

```sh
deb http://ppa.launchpad.net/ansible/ansible/ubuntu xenial main
deb-src http://ppa.launchpad.net/ansible/ansible/ubuntu xenial main
```

**Repo einrichten und update durchführen**

```sh
sudo add-apt-repository ppa:ansible/ansible
sudo apt-get update
```

**Add Repository für Ubuntu**

```sh
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```

Da hier auch die quellen vorhanden sind kann man auch sich das Packet wie folgt bauen
make deb

```sh

## Installation RHEL/Centos

## Installation mit [Pip](https://pypi.python.org/pypi/pip)

Die [pip](https://pypi.python.org/pypi/pip) Installation ist recht einfach aber  `easy_install`  muss vorhanden sein so haben die meisten Distributionen in der bootstrap Variante nicht dabei und auch noch nicht in der Netinstall, gerade seit dem die Pakete größer werden werden hier kaum mehr Extras eingerichtet.

Einfach mal mit dem [Packetmanager](../packetmanager) nach `easy_install` suchen .

Wenn dann installiert ist kann man wie folgt [Ansible](../ansible) installieren.
```

```sh
sudo easy_install pip
sudo pip install ansible
```
