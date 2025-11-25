
[[parted]] resizepart is a part from parted

You need too know the size in blocks

https://www.baeldung.com/linux/blockdev-storage-block-size

https://dev.to/pgradot/resizing-partitions-on-linux-408o

```
---
- name: Automate Disk Resizing on RHEL
  hosts: all
  become: yes
  tasks:
    - name: Rescan SCSI bus
      ansible.builtin.shell: rescan-scsi-bus.sh -u

    - name: Resize partition 2 on /dev/sda
      ansible.builtin.expect:
        command: parted ---pretend-input-tty /dev/sda resizepart
        responses:
          "Fix/Ignore?": "Fix"
          "Partition number?": "2"
          "End?": "100%"

    - name: Inform the OS of partition table changes
      ansible.builtin.shell: partprobe /dev/sda

    - name: Resize physical volume /dev/sda2
      ansible.builtin.shell: pvresize /dev/sda2

    - name: Extend logical volume /dev/mapper/rhel-root
      ansible.builtin.shell: lvextend -r -l +100%FREE /dev/mapper/rhel-root

    - name: Rescan SCSI bus
      ansible.builtin.shell: rescan-scsi-bus.sh -u
```