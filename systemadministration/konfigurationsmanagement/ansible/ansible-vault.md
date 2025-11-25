---
tags:
  - konfigurationsmanagement
  - ansible
  - sicherheit
  - vault
---
# Basis Umgang mit der ansible Vault

**Password mit 30 Zeichen für die  Vault generieren **

`mkpasswd -m sha-512 | tail -c 30`

**Berechtigungen für das Password file setzen**

`chmod 0600 /home/$USER/.vault_pass`

**Encrypted fiele anlegen**

`ansible-vault create ssh_priv_keys.yml --vault-password-file /home/$HOME/.vault_pass`

**Datein encrypten**

```ansible-vault encrypt ssh_priv_keys.yml --vault-password-file $HOME/.vault_pass```

**encrypt der ansible.cfg**

```sh
cd ansible_tasks3  
sudo ansible-vault encrypt ansible.cfg --vault-password-file /home/vagrant/.vault_pass
```

**encrypted output**

`sudo cat ansible.cfg`

**decrypt ansible.cfg**

`sudo ansible-vault decrypt ansible.cfg --vault-password-file /home/vagrant/.vault_pass`

**View ansible.cfg**

`ansible-vault view ansible_tasks3/ansible.cfg --vault-password-file .vault_pass`

**Edit ansible.cfg**

`ansible-vault edit ansible_tasks3/ansible.cfg --vault-password-file .vault_pass`

**Key Ändern**

`ansible-vault rekey ansible_tasks3/ansible.cfg --vault-password-file ./.vault_pass`

**Verschlüsseln eines bestimmten Inhalts innerhalb einer Datei**

`ansible-vault encrypt_string 'variableToEncrypt' --vault-id label@password_file --name 'variable_name'`

`ansible-vault encrypt_string 'ansible3' --vault-id ansible-demo@/home/vagrant/.vault_key --name 'password'`

Hier ist *ansible3* das password und das Label ist *ansible-demo* 

**Beispeil mit einem Playbook**

```sh
ansible_host: 192.168.53.61  
password: !vault |  
          $ANSIBLE_VAULT;1.2;AES256;ansible-demo  
666562353266313139333461326437323436306230626230353433396561376230643537313862396663383030626334356661313339623161363066653062350a373963663337656261613139343238303931383065666534326137366533393834373766353836316666663737373465646465616465343464336332396662310a6663353332363964306233383439383433663166656539313733646261386536
apache: httpd  
php: php  
sudo_group: wheel
```

**Vault Password für die ansible.cfg defenieren**

```ini
[defaults]  
...  
vault_password_file=/home/vagrant/.vault_key
...
```

**Password der vault ändern**

```sh
ansible-vault rekey encrypt_me.txt
```

# Ansible

`echo -e | ansible-vault encrypt_string `

https://raw.githubusercontent.com/ansible-community/contrib-scripts/refs/heads/main/vault/vault-keyring-client.py