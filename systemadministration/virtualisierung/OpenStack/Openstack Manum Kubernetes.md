---
tags:
  - openstack
  - manum
---
Magnum is an OpenStack API service that allows you to deploy Kubernetes Clusters on OpenStack Cloud Platform in minutes. After [bootstrapping your cluster from a Template](https://computingforgeeks.com/create-kubernetes-cluster-on-openstack-magnum-with-fedora-coreos/), there could be a need from business workloads demanding addition of more Worker nodes into the Kubernetes Cluster. In this guide we explore the steps that are essential in scaling the cluster by adding extra minions. I guarantee the process is as easy as A-B-C.

Confirm your cluster is in Healthy state before expanding the number of nodes:

```
$ openstack coe cluster list -f json
  {
    "uuid": "f99fc7df-b5f3-4621-92a4-cc7a695acb60",
    "name": "k8s-cluster-02",
    "keypair": "admin",
    "node_count": 1,
    "master_count": 1,
    "status": "CREATE_COMPLETE",
    "health_status": "HEALTHY"
  }
]
```

o expand your Kubernetes cluster deployed using Magnum, use the **‘cluster update**‘ command to modify:

```
$ openstack coe cluster update <cluster> replace node_count=<desiredcount>
```

Where applied parameters are:

- <**cluster**>: This is the UUID or name of the cluster to be updated.
- **replace**: This second parameter specify the desired change to be made to the cluster attributes. This can be ‘**add**’, ‘**replace**’ and ‘**remove**’. In our update we’re using replace.
- **<desiredcount>**: Is the desired number of nodes after the change.

You can scale a cluster by adding servers to or removing servers from the cluster. Currently, this is done through the ‘_cluster-update_’ operation by modifying the node-count attribute, for example:

```
# Examples
openstack coe cluster update myk8scluster replace node_count=3
openstack coe cluster update myk8scluster replace node_count=6
```

See below table for the possible change that can be made in a cluster:

|Attribute|add|replace|remove|
|---|---|---|---|
|_node_count_|no|add/remove nodes in default-worker nodegroup.|reset to default of 1|
|_master_count_|no|no|no|
|_name_|no|no|no|
|_discovery_url_|no|no|no|

You can only initiate a ‘_cluster-update_‘ operation when there is no other operation is in progress.

In this example we are setting the worker group node count to **3.** The cluster from previous output had only one node.

```
$ openstack coe cluster update k8s-cluster-02 replace node_count=3
Request to update cluster k8s-cluster-02 has been accepted.

$ openstack coe cluster list -f json
[
  {
    "uuid": "f99fc7df-b5f3-4621-92a4-cc7a695acb60",
    "name": "k8s-cluster-02",
    "keypair": "admin",
    "node_count": 3,
    "master_count": 1,
    "status": "UPDATE_IN_PROGRESS",
    "health_status": "HEALTHY"
  }
]
```

If you’re performing a remove operation, Magnum will attempt to find _nodes_ with no containers to remove. If some nodes with containers must be removed, Magnum will log a warning message.

Give it few minutes and new nodes will show:

```
$ openstack server list --column Name --column Status
+--------------------------------------+--------+
| Name                                 | Status |
+--------------------------------------+--------+
| k8s-cluster-02-soooe6tdv773-node-2   | ACTIVE |
| k8s-cluster-02-soooe6tdv773-node-1   | ACTIVE |
| k8s-cluster-02-soooe6tdv773-node-0   | ACTIVE |
| k8s-cluster-02-soooe6tdv773-master-0 | ACTIVE |
+--------------------------------------+--------+
```

If you check Cluster status it should be _UPDATE_COMPLETE_ if the update was successful.

```
$ openstack coe cluster list -f json
[
  {
    "uuid": "f99fc7df-b5f3-4621-92a4-cc7a695acb60",
    "name": "k8s-cluster-02",
    "keypair": "admin",
    "node_count": 3,
    "master_count": 1,
    "status": "UPDATE_COMPLETE",
    "health_status": "HEALTHY"
  }
]
```

From kubectl results node count should be updated as well.

```
$ kubectl  get nodes
NAME                                   STATUS   ROLES    AGE   VERSION
kk8s-cluster-02-soooe6tdv773-node-2    Ready    master   10d   v1.21.1
k8s-cluster-02-soooe6tdv773-node-0     Ready    <none>   10d   v1.21.1
k8s-cluster-02-soooe6tdv773-node-1     Ready    <none>   5m    v1.21.1
k8s-cluster-02-soooe6tdv773-node-2     Ready    <none>   5m    v1.21.1
```

You can describe node status to see if there are any taints resulting from some failures that could prevent the applications from running:

```
$ kubectl describe node <node-name>
```

**_Books For Learning Kubernetes Administration:_**

- [Best Kubernetes Study books](https://computingforgeeks.com/best-kubernetes-study-books/)

More guides on Kubernetes / OpenStack:

- [Create Kubernetes Cluster on OpenStack Magnum with Fedora CoreOS](https://computingforgeeks.com/create-kubernetes-cluster-on-openstack-magnum-with-fedora-coreos/)
- [Upgrade Kubernetes Cluster on OpenStack Magnum](https://computingforgeeks.com/upgrade-kubernetes-cluster-on-openstack-magnum/)
- [Install and Configure OpenStack Barbican Key Manager Service](https://computingforgeeks.com/install-and-configure-openstack-barbican-key-manager-service/)
- [Deploy VM instance on OpenStack using Terraform](https://computingforgeeks.com/deploy-vm-instance-on-openstack-with-terraform/)

https://computingforgeeks.com/scale-up-worker-nodes-in-openstack-magnum-kubernetes/