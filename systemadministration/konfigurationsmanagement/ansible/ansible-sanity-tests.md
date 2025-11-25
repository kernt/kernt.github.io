---
tags:
  - konfigurationsmanagement
  - ansible
  - sanitys-tests
---

# Ansible Sanity Tests

**Run all sanity tests**

`ansible-test sanity`

**Run all sanity tests including disabled ones**

`ansible-test sanity --allow-disabled`

**Run all sanity tests against certain file(s)**

`ansible-test sanity lib/ansible/modules/files/template.py`

**Run all sanity tests against certain folder(s)**

`ansible-test sanity lib/ansible/modules/files/`

**Run all tests inside docker (good if you don't have dependencies installed)**

`ansible-test sanity --docker default`

**Run validate-modules against a specific file**

`ansible-test sanity --test validate-modules lib/ansible/modules/files/template.py`


## [Available Tests](https://docs.ansible.com/ansible/latest/dev_guide/testing_sanity.html#id3)

Tests can be listed with `ansible-test sanity --list-tests`

https://docs.ansible.com/ansible/latest/dev_guide/testing/sanity/index.html