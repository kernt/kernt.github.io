# The “default” network

When **libvirt** is in use and the **libvirtd** daemon is running, a default network is created. We can verify that this network exists by using the `virsh` utility, which on the majority of Linux distribution usually comes with the `libvirt-client` package. To invoke the utility so that it displays all the available virtual networks, we should include the `net-list` subcommand:

`sudo virsh net-list --all`

In the example above we used the `--all` option to make sure also the **inactive** networks are included in the result, which should normally correspond to the one displayed below:

```
Name      State    Autostart   Persistent
--------------------------------------------
 default   active   yes         yes
```

To obtain detailed information about the network, and eventually modify it, we can invoke virsh with the `edit` subcommand instead, providing the network name as argument:

```sh
$ sudo virsh net-edit default
```

Ausgabe:

```xml
<network>
  <name>default</name>
  <uuid>168f6909-715c-4333-a34b-f74584d26328</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:48:3f:0c'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```



```xml
<network>
  <name>default</name>
  <uuid>168f6909-715c-4333-a34b-f74584d26328</uuid>
  <forward mode='bridge'/>
  <bridge name='enp2s0' />
</network>
```