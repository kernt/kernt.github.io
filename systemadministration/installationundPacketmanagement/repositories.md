---
tags:
  - repository
  - system-administration
---
# Repositories

## Centos7

**ceph.repo**

```sh
[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
```

**CentOS-Gluster-3.8.repo**

```sh
[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
```

```sh
# cat CentOS-Gluster-3.8.repo
# CentOS-Gluster-3.8.repo
#
# Please see http://wiki.centos.org/SpecialInterestGroup/Storage for more
# information

[centos-gluster38]
name=CentOS-$releasever - Gluster 3.8
baseurl=http://mirror.centos.org/centos/$releasever/storage/$basearch/gluster-3.8/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Storage

[centos-gluster38-test]
name=CentOS-$releasever - Gluster 3.8 Testing
baseurl=http://buildlogs.centos.org/centos/$releasever/storage/$basearch/gluster-3.8/
gpgcheck=0
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Storage
```

**crmsh.repo**

```sh
[network_ha-clustering_Stable]
name=Stable High Availability/Clustering packages (CentOS_CentOS-7)
type=rpm-md
baseurl=http://download.opensuse.org/repositories/network:/ha-clustering:/Stable/CentOS_CentOS-7/
gpgcheck=1
gpgkey=http://download.opensuse.org/repositories/network:/ha-clustering:/Stable/CentOS_CentOS-7//repodata/repomd.xml.key
enabled=1
```

**epel.repo**

```sh
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
[[baseurl]]=http://download.fedoraproject.org/pub/epel/7/$basearch
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
[[baseurl]]=http://download.fedoraproject.org/pub/epel/7/$basearch/debug
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
[[baseurl]]=http://download.fedoraproject.org/pub/epel/7/SRPMS
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1
```

**Pakete auf allen Systemen**

| OS     | Packet |
| :------: | :------: |
| beide  | vim    |
| beide  | mc     |
| beide  | tree   |
| beide  | tmux   |
| beide  | wget   |
| cell 3 | cell 4 |
| cell 1 | cell 2 |
| cell 3 | cell 4 |
| cell 1 | cell 2 |
| cell 3 | cell 4 |

Pakete für CentOS 7, DRBD und NFS

Wichtig für [drbd](http://elrepo.org/tiki/tiki-index.php)

| Service | Packet |
|:-------:|:------:|
| beide   | vim    |
| beide   | mc     |
| beide   | tree   |
| beide   | tmux   |
| beide   | cell 2 |

**Pakete für Centos 7, drbd , nfs und GlusterFS Server**

| Service | Packet |
|:-------:|:------:|
| beide   | vim    |
| beide   | mc     |
| beide   | tree   |
| beide   | tmux   |
| beide   | cell 2 |

**Quellen**

* [pulp](https://pulpproject.org/)
* [Sammlung von Packetquellen](https://pkgs.org/)
* [RPM Remis PHP Repo](https://rpms.remirepo.net/)
