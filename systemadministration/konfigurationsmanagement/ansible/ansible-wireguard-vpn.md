---
tags:
  - konfigurationsmanagement
  - ansible
  - wireguard
  - vpn
---
### Encrypting the Private Key

It's a good practice to AVOID having secrets in plaintext (like the VPN private key above). This is especially true if those secrets will be shared with anyone else, like via a git repo. Let's prevent this by using [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html). Vault is a tool for encrypting secret values and using them in playbooks. Encrypt the private key with:

```sh
ansible-vault encrypt_string --ask-vault-password --stdin-name server_privkey
```

You'll be prompted twice for a Vault encryption password, after which you'll paste your `privkey` value and hit `Ctrl+d` twice. If the command completed after a single `Ctrl+d`, try again and make sure you're not copy-pasting an invisible newline character at the end of the `privkey` value. Copy the output into your playbook, which will now look like:

```yaml
---
- name: setup vpn server
  hosts: vpn_server
  vars:
    server_privkey: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          646438636565343063343631326136386239623935393637336539653636386135363
          663386639393232346534643163656363316234306439306566306534610a31326664
          363763663139383034636632343230376365333130333230373866353033326563303
          5636138373830633534373033303536303566663166616539360a3936353033663263
          336662663034376661616631343661333164363134373061343739633637623739306
          465653532383838393662396333623966343165366635353132396332313762343534
          65313761623964653532623839356633343838
    server_pubkey: 7/6f7bUT+2hWMEP5BxeK51PGuMuTnQ9pRpkxg5jUSTo=
  tasks:
  ... add your tasks here!
```

Make sure to remember your encryption password (and save it in a password manager); you'll need to enter it every time you run the playbook.

## [exploring-ansible-via-setting-up-a-wireguard-vpn](https://dev.to/tangramvision/exploring-ansible-via-setting-up-a-wireguard-vpn-3389#installing-and-configuring-wireguard)Installing and Configuring WireGuard

Next, we'll remove our testing `ping` and `debug` tasks and write tasks for steps 1, 3, 4, and 5 from the above list. These steps translate neatly into Ansible tasks in our updated `playbook.yml`:

```


---
- name: setup vpn server
  hosts: vpn_server
  vars:
    server_privkey: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          646438636565343063343631326136386239623935393637336539653636386135363
          663386639393232346534643163656363316234306439306566306534610a31326664
          363763663139383034636632343230376365333130333230373866353033326563303
          5636138373830633534373033303536303566663166616539360a3936353033663263
          336662663034376661616631343661333164363134373061343739633637623739306
          465653532383838393662396333623966343165366635353132396332313762343534
          65313761623964653532623839356633343838
    server_pubkey: 7/6f7bUT+2hWMEP5BxeK51PGuMuTnQ9pRpkxg5jUSTo=
  tasks:
  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
  - name: install wireguard package
    apt:
      name: wireguard
      state: present
      update_cache: yes

  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html
  - name: create server wireguard config
    template:
      dest: /etc/wireguard/wg0.conf
      src: server_wg0.conf.j2
      owner: root
      group: root
      mode: '0600'

  # https://docs.ansible.com/ansible/latest/collections/ansible/posix/sysctl_module.html
  - name: enable and persist ip forwarding
    sysctl:
      name: net.ipv4.ip_forward
      value: "1"
      state: present
      sysctl_set: yes
      reload: yes

  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html
  - name: start wireguard and enable on boot
    systemd:
      name: wg-quick@wg0
      enabled: yes
      state: started


```