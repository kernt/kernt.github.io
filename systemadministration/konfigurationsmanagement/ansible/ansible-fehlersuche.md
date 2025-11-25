---
tags:
  - ansible
  - konfigurationsmanagement
---

# Fehlersuche mit Ansible

Um einzelne Hosts bei denen ein Fehler aufgetreten ein weiteres mal testen zu können ohne alle anderen auch zu testen und damit dem Vorgang stark zu beschleunigen legt Ansible default eine  Datei:

`retry_hosts.txt` an die man dann bei dem nächstem versuch benutzen sollte hier ein

Beispiel:

`ansible-playbook site.yml --limit @retry_hosts.txt`

## Quelle

* [intro_patterns](http://docs.ansible.com/ansible/intro_patterns.html)
