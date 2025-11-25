---
tags:
  - konfigurationsmanagement
  - ansible
  - statements
---
# ansible statements

```yaml
tasks:
  - name: Configure SELinux to start mysql on any port
    ansible.posix.seboolean:
      name: mysql_connect_any
      state: true
      persistent: true
    when: ansible_selinux.status == "enabled"
    # all variables can be used directly in conditionals without double curly braces
```

* [ansible statements](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)