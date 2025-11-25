---
tags:
  - openstack
---
# Openstack Übersicht

https://www.openstack.org/software/project-navigator/openstack-components#openstack-services

| Modulname | Beschreibung      |
| --------- | ----------------- |
| Keystone  | Identität         |
| Ironic    | BareMettal        |
| Cinder    | BlockStorage      |
| Horizon   | Dashboard         |
| Octavia   | Loadbalencer      |
| Swift     | ObjektStorage     |
| Heat      | Orchestration     |
| Manila    | Shared filesystem |
| Nora      | Computing         |
| Glance    | Image Service     |
| Neutron   | Networking        |
## Storage

[[openstack cinder]]
[[openstsck swift]]

## opstack komandline tool

Upload image to glance

```sh
openstack image create \
    --container-format bare \
    --disk-format qcow2 \
    --file debian-10-openstack-amd64.qcow2 \
    Debian-10
```

Confirm image is uploaded and active

```sh
$ openstack image list
+--------------------------------------+--------------+--------+
| ID                                   | Name         | Status |
+--------------------------------------+--------------+--------+
| 211daeef-eee7-4b13-a778-72c06b8d2c27 | Cirros-0.5.1 | active |
| 5ceccfb1-2389-4ffd-a82f-348d6cd6b3b2 | Debian-10    | active |
+--------------------------------------+--------------+--------+
```

Generate new SSH key if you don’t have one already

Upload the public key to Openstack. This key will be used during instance creation for passwordless authentication.

```sh
openstack keypair create --public-key ~/.ssh/id_rsa.pub admin
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | 19:7b:5c:14:a2:21:7a:a3:dd:56:c6:e4:3a:22:e8:3f |
| name        | admin                                           |
| user_id     | 513f0abd6eba4b0fab2754166f38e0f2                |
+-------------+-------------------------------------------------+
```

Confirm keypair is available on OpenStack

```sh
openstack keypair list
+-------+-------------------------------------------------+
| Name  | Fingerprint                                     |
+-------+-------------------------------------------------+
| admin | 19:7b:5c:14:a2:21:7a:a3:dd:56:c6:e4:3a:22:e8:3f |
+-------+-------------------------------------------------+
```

**List all available images in Glance:**

```sh
openstack image list
+--------------------------------------+--------------+--------+
| ID                                   | Name         | Status |
+--------------------------------------+--------------+--------+
| 3d042db5-2a0e-4b1d-8a71-12bb43bc638c | CentOS-7     | active |
| cbed9a33-e854-4894-bf9b-1b4596c49782 | CentOS-8     | active |
| 211daeef-eee7-4b13-a778-72c06b8d2c27 | Cirros-0.5.1 | active |
| 5ceccfb1-2389-4ffd-a82f-348d6cd6b3b2 | Debian-10    | active |
| 636f3209-2c08-49f8-92c9-3a3b50d8850e | Ubuntu-20.04 | active |
| 4aee3927-da23-4f83-b887-a9b7eafd7192 | fcos         | active |
+--------------------------------------+--------------+--------+
```
### Create clouds.yml file

**Create new directory where _clouds.yml_ file will be stored**

```sh
mkdir ~/.config/openstack
```

**Create clouds.yml file**

```sh
vim ~/.config/openstack/clouds.yaml
```

