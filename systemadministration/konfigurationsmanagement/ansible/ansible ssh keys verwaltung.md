
roles/ssh_key_management/defaults/main.yml

```yaml

---
# Keine Standardwerte erforderlich

# roles/ssh_key_management/vars/vault.yml
# Hinweis: Diese Datei muss mit `ansible-vault create` erstellt und verschlüsselt werden
```

```yaml
---
private_key: |
  -----BEGIN RSA PRIVATE KEY-----
  ... (Ihr Private-Key hier)
  -----END RSA PRIVATE KEY-----
public_key: ssh-rsa AAA... (Ihr Public-Key hier)
```

**roles/ssh_key_management/tasks/main.yml**

```yaml
---
- name: Ensure keys directory exists
  file:
    path: /var/www/keys
    state: directory
    mode: '0755'

- name: Write public key to file
  copy:
    content: "{{ public_key }}"
    dest: /var/www/keys/public_key.pub
    mode: '0644'

- name: Ensure Nginx is installed
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"

- name: Configure Nginx to serve public key
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
  notify: restart nginx
```

**roles/ssh_key_management/templates/nginx.conf.j2**

```
server {
    listen 80;
    server_name localhost;

    location /api/ssh/public_key {
        alias /var/www/keys/public_key.pub;
    }
}
```


roles/ssh_key_management/handlers/main.yml

```yaml
---
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

Beispiel-Playbook: playbook.yml

```yaml
---
- name: Manage SSH Keys
  hosts: localhost
  become: yes
  roles:
    - ssh_key_management
```