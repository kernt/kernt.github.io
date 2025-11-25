# NFS

**Network File System** (**NFS**) is a distributed file system protocol, it allowing a user on a client computer to access files over a computer network much like local storage is accessed. Using NFS we can store our file without add more disk or local storage on our computer. NFS protocol originally developed by Sun Microsystems (Sun) in 1984.

## GlusterFS

GlusterFS is free and open source software scalable, distributed file system that aggregates disk storage resources from multiple servers into a single global namespace. You can create large, distributed storage solutions for media streaming, data analysis, and other data- and bandwidth-intensive tasks.

**Advantages**

- Scales to several petabytes
- Handles thousands of clients
- POSIX compatible
- Uses commodity hardware
- Can use any ondisk filesystem that supports extended attributes
- Accessible using industry standard protocols like NFS and SMB
- Provides replication, quotas, geo-replication, snapshots and bitrot detection
- Allows optimization for different workloads
- Open Source

Gluster file system supports different types of volumes based on the requirements. Some volumes are good for scaling storage size, some for improving performance and some for both.

## NFS Ganesha

NFS-Ganesha is a user-space file server for the NFS protocol with support for NFSv3, v4, v4.1, pNFS. It provides a FUSE-compatible File System Abstraction Layer(FSAL) to allow the file-system developers to plug in their storage mechanism and access it from any NFS client. NFS-Ganesha can access the FUSE filesystems directly through its FSAL without copying any data to or from the kernel, thus potentially improving response times.

— Gluster Docs

# Practical

## **Topology**

![](SEPjzEC6DYELxuexeoh2hg.webp)

Before setup NFS Ganesha, we must setup GlusterFS

Before setup NFS Ganesha, we must setup GlusterFS

## Setup GlusterFS

**Environment**

- Spec: 8Core, 16GB RAM
- OS : Ubuntu 20.04
- Root Disk: /dev/vda 30GB
- Additional disks: /dev/vdb 20GB
- Gluster mode: replicated

**Edit /etc/hosts on all node**

```sh
nano /etc/hosts  
...  
172.20.12.13 gluster01  
172.20.12.14 gluster02  
172.20.12.15 gluster03  
...
```

**Create partition for Gluster**

```sh
# Execute on all node  
mkfs.xfs /dev/vdb  
mkdir /glusterfs  
mount /dev/vdb /glusterfs  
mkdir -p /glusterfs/replicated
```

**Installation glusterFS**

```sh
# Execute on all node  
apt update  
apt install software-properties-common  
add-apt-repository ppa:gluster/glusterfs-10  
apt update  
apt install glusterfs-server
```

**Start glusterFS service on all node**

`systemctl enable --now glusterd`

**Create Peering between node (Execute on node gluster01)**

```sh
gluster peer probe gluster02  
gluster peer probe gluster03
```

Check peering status

`gluster peer status`

**Create gluster volume**

```sh
# Exceute on node gluster01  
# Create volume  
gluster volume create vol_replica replica 3 transport tcp \  
gluster01:/glusterfs/replicated \  
gluster02:/glusterfs/replicated \  
gluster03:/glusterfs/replicated  
  
# Start volume  
gluster volume start vol_replica  
  
# Check volume  
gluster volume info
```

## Setup Ganesha

**prerequisite**

make sure no similar nfs service is running. For example the service nfs-server.service

```sh
systemctl status nfs-server  
systemctl disable nfs-server  
systemctl stop nfs-server
```

**Setup nfs-ganesha-gluster**

```sh
# Exectue on all node  
apt update  
apt install nfs-ganesha-gluster glusterfs-ganesha
```

**Disable NFS feature on Gluster**

By default, if we create a new volume on glusterfs the nfs.disable setting is on.

```sh
gluster volume get vol_replica nfs.disable  
...Output  
Option                             Value  
--------                           -------  
nfs.disable                        on
```

If value is off, we need to change

```sh
gluster volume set vol_replica nfs.disable on  
...Output  
volume set: success
```

**Create Configuration NFS-Gluster**

```sh
# Backup original configuration  
mv /etc/ganesha/ganesha.conf /etc/ganesha/ganesha.conf.orig  
  
# Create new configuration file  
nano /etc/ganesha/ganesha.conf
```

Edit new configuration files on all node

```c
NFS_CORE_PARAM {  
    # possible to mount with NFSv3 to NFSv4 Pseudo path  
    mount_path_pseudo = true;  
    # NFS protocol  
    Protocols = 3,4;  
}  
EXPORT_DEFAULTS {  
    # default access mode  
    Access_Type = RW;  
}  
  
LOG {  
    # default log level  
    Default_Log_Level = WARN;  
}  
  
%include "/etc/ganesha/gluster.conf"  
  
EXPORT {  
    # uniq ID  
    Export_Id = 101;  
    # mount path of Gluster Volume  
    Path = "/vol_replica"; #mesti disamain sama nama volume  
    FSAL {  
        name = "GLUSTER"; # jangan diubah, ini adalah nama lib ganesha yang diimpor (libfsalgluster.so)  
        # hostname or IP address of this Node  
        hostname="gluster01"; # sesuaikan dengan masing-masing hostname  
        # Gluster volume name  
        volume="vol_replica";  
    }  
    # rconfig for root Squash  
    Squash="No_root_squash";  
    # NFSv4 Pseudo path  
    Pseudo="/vol_replica"; # mesti disamain  
    # allowed security options  
    SecType = "sys";  
}
```

**Activated service NFS-Ganesha**

```sh
systemctl restart nfs-ganesha  
systemctl enable nfs-ganesha  
systemctl status nfs-ganesha  
# if failed, try to check log at /var/log/ganesha/ganesha.log
```

**Verify Mount Point Ganesha**

```sh
showmount -e localhost  
  
...Output  
Export list for ss-ganesha-compute1:  
/vol_replica (everyone)

```
# Whats next?

In this article, we have done to setup glusterFS with NFS Ganesha. As endpoint, we can connect to the NFS server to access file-system. For the next step, we can add High Availability for our NFS Ganesha to Prevent loss of data when one node is not ready.