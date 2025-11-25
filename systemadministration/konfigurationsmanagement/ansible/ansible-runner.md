---
tags:
  - konfigurationsmanagement
  - ansible-runner
---

# ansible-runner

```sh
.
├── env
│   ├── envvars
│   ├── extravars
│   ├── passwords
│   ├── cmdline
│   ├── settings
│   └── ssh_key
├── inventory
│   └── hosts
└── project
    ├── test.yml
    └── roles
        └── testrole
            ├── defaults
            ├── handlers
            ├── meta
            ├── README.md
            ├── tasks
            ├── tests
            └── vars
```

# Ansible Runner execute examples

**Runner run on $PWD with clutser.yaml**

`ansible-runner run . -p ./cluster.yml`

**Runner run on $PWD run adhoc command in hosts inventory**

`ansible-runner run . -m command -a "systemctl stop etcd && rm -rf /var/lib/etcd/*" --hosts ./inventory/mycluster/hosts.yaml`

