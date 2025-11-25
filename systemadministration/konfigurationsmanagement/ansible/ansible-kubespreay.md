---
tags:
  - konfigurationsmanagement
  - ansible
  - kubespreay
  - kubernetes
  - terraform
  - opentofu
---

# Ansible Kubsprey benutzen

```
VENVDIR=venv
KUBESPRAYDIR=kubespray
python3 -m venv $VENVDIR
source $VENVDIR/bin/activate
cd $KUBESPRAYDIR
pip install -U -r requirements.txt
```