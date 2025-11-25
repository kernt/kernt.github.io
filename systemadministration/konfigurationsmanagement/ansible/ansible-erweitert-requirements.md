---
tags:
  - konfigurationsmanagement
  - ansible
---

# Ansible Galaxy requirements

In der Datei `requirements.yml` kann man seine Abhängigkeiten definieren `ansible-galaxy` kümmert sich dann darum alle rollen die benötigt werden auch  zu laden.

Die Datei `requirements.yml` liegt im quell Verzeichnis

Die Abhängigkeiten können wie Folder aufgelöst werden.

```sh
ansible-galaxy install -r requirements.yml
```

Hier ist eine Beispiel wie eine solche datei aufgebaut ist

```yml
---
- src: https://github.com/myprojekt/ansible-apache2.git
- src: https://github.com/myprojekt/ansible-mariadb-mysql.git
- src: https://github.com/myprojekt/ansible-mariadb-galera-cluster.git
- src: https://github.com/myprojekt/ansible-powerdns.git
```

In diesem Beispiel muss `- src:` eine erreichbare http|https Quelle sein.
Alternativ sind aber alternative quellen möglich.


## Initiales Projekt verwenden

`ansible-galaxy init--role-skeleton biodec.template/ -p . biodec.role_name`
