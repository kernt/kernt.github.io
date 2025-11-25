# virsh

**Reboot a virtual machine**

`virsh reboot <vm_name>`

**Suspend a virtual machine**

`virsh suspend <vm_name>`

**Resume a suspended virtual machine**

`virsh resume <vm_name>`

**Attach a virtual network interface to a virtual machine**

`virsh attach-interface --domain <vm_name> --type network --source <network_name> --model <model_type>`

**Detach a virtual network interface from a virtual machine**

`virsh detach-interface --domain <vm_name> --mac <mac_address>`

**Create a new virtual machine**

`virt-install --name <vm_name> --ram <memory_size> --vcpus <vcpu_count> --disk path=<disk_path>,size=<disk_size> --cdrom <iso_path> --network network=<network_name> --graphics <graphics_type>`


