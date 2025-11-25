In der Praxis ist der default von KVM/Qemu mit der folgenden Konfiguration 


```
- name: Define a new network
  community.libvirt.virt_net:
    command: define
    name: br_nat
    xml: '{{ lookup("template", "network/bridge.xml.j2") }}'
```

```yaml
- name: Ensure that a network is active (needs to be defined and built first)
  community.libvirt.virt_net:
    state: active
    name: br_nat

- name: Ensure that a given network will be started at boot
  community.libvirt.virt_net:
    autostart: true
    name: br_nat

- name: Add Bridge for kvm
  nmcli:
    type: bridge
    conn_name: '{{ item.conn_name }}'
    ip4: '{{ item.ip4 }}'
    gw4: '{{ item.gw4 }}'
    mode: '{{ item.mode }}'
    state: present
  with_items:
      - '{{ nmcli_bridge }}'

```