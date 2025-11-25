---
tags:
  - bash
  - konsole
  - berechtigungen
  - adduser
  - groupdel
  - usermod
  - system-administration
  - gnu-tools
  - deluser
  - user
---
# User und Gruppen

## Standart User

```
adm-kvm
docker
adm-libvirt
sshmgr
ansible
rsshadm
```

## Standart Gruppen

```
adm-kvm
webvirtmgr
gitlab-runner
ssh-opt
libvirtd
```

User für ssh verbindungen ohne password rsshadm

```s
sudo mkdir -p  /home/rsshadm/.ssh
sudo chmod 700 /home/rsshadm/.ssh
sudo chmod 0600 /home/rsshadm/.ssh/authorized_keys
sudo chown -R rsshadm:rsshadm /home/rsshadm/.ssh
```