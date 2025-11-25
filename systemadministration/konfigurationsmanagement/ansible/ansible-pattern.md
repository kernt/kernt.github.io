---
tags:
  - konfigurationsmanagement
  - ansible
  - pattern
---
# Ansible Pattern

## Alias für ALL

```sh
all
*
```

## Oder Gruppe dbservers

`webservers:dbservers`

## Host muss ein zwei Gruppen sein. Hier webservers und staging

`webservers:&staging`

Exclude phoenix

`webservers:!phoenix`

## Kombination aller Abhängigkeiten

`webservers:dbservers:&staging:!phoenix`

## Auf Hosts in einer Gruppe via Index verweisen

```sh
webservers[0]       # == cobweb
webservers[-1]      # == weber
webservers[0:2]     # == webservers[0],webservers[1]
                    # == cobweb,webbing
webservers[1:]      # == webbing,weber
webservers[:3]      # == cobweb,webbing,weber
```

**Reguläre Ausdrücke benutzen. Muss mit _~_ anfangen**

`~(web|db).*\.example\.com`

**Liste von hosts aus einer Datei lesen. Hier ist @ wichtig**

`ansible-playbook site.yml --limit @retry_hosts.txt`

**Bestimmte gruppe ausschißen**

`ansible-playbook site.yml --limit datacenter2`

**Quellen:**

* [Ansible Pattern](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html)
