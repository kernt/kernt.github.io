n this article, we will cover how to set up NFS servers on Ceph clusters. Ceph uses NFS-Ganesha NFS server. It only supports NFS v4.0+.
## 1. Deploying a single NFS server

A single command will deploying a NFS server.

`sudo ceph nfs cluster create [nfs-cluster-name]`

We can check the status of the creation with these commands.

```sh
sudo ceph orch ls --service_name=nfs.[nfs-cluster-name]  
sudo ceph orch ps --service_name=nfs.[nfs-cluster-name]  
sudo ceph nfs cluster info [nfs-cluster-name]
```

Here we first created a nfs cluster named _singletest_. The following screen shot shows its status and information.

![](XPmbQC8vaWTd5Fp9dWm-hw.webp)

## 2. Exporting a Ceph File System

First, let’s create a file system named _cephfs-nfs_.

`sudo ceph fs volume create cephfs-nfs`

Now, we create a user named nfsclient to expose this CephFS through the nfs server that was set up in the first step.

```sh
sudo ceph auth get-or-create client.nfsclient \  
   mon 'allow r' \  
   osd 'allow rw pool=.nfs namespace=singletest, allow rw tag cephfs data=cephfs-nfs' \  
   mds 'allow rw path=/' > nfsclient.key
```

Now, we are ready to create a configuration file and apply it to the nfs cluster.

`sudo ceph nfs cluster config set singletest -i nfs.conf`

We can also query the configuration using _get_ option. The output is almost identical, meaning the output can be used as the input file, _nfs.conf_.

![](Zs51WK7dQp6bx30AgwXudw.webp)

singletest nfs server configuration

## 3. Setting up NFS clients

In order to mount this NFS share on a client node, we need to install _nfs-common_ package.

`sudo apt install nfs-common`

Then, create a mount point and mount the share on the mount directory. An example is shown below.

```sh
sudo mkdir -p /nfs/ceph  
sudo mount 172.29.65.17:/ /nfs/ceph
```

The export shows up as _/nfs/ceph/ceph_. The ceph directory came from _Pseudo_ seting in the configuration.

## Setting up a HA NFS servers with ingress

Now, we will set up another NFS server with high availability. There will be one (virtual) ip address with two backing NFS servers.

An overall architecture is shown below.

![](XkYBCdXVglzNJdf1G56aw.webp)

The following command will create an HA NFS server.

```sh
# ceph nfs cluster create [nfs-cluster-name] [placement] --ingress --virtual-ip [ip]  
sudo ceph nfs cluster create hatest "experiment-ceph-mon1 experiment-ceph-mon2" \  
   --ingress --virtual-ip 172.29.65.22 
```

![](BQ-uwOhCChnL7HAcvnBbA.webp)

Internally, Ceph uses _keepalived_ and _haproxy_ to implement this. We can check this using _ceph orch ps_ command.

![](EdmLwC9ml5pYouS3S7Jtrg.webp)
hatest nfs cluster process information

Please note that _haproxy_ and _keepalived_ processes are launched at different nodes than specified nodes for backend nfs service.

The remaining steps to use this nfs service is the same as a single NFS server with the new virtual ip and nfs cluster/user name changes if necessary.

## Deleting NFS servers

To delete a NFS server, use this command.

```sh
# sudo ceph nfs cluster rm [nfs cluster name]  
sudo ceph nfs cluster rm singletest  
sudo ceph nfs cluster rm hatest
```

Users and Ceph file systems that are created for the nfs servers are not tied to the nfs servers. They are not affected by this remove command.