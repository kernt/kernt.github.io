# Ansible Konfiguration um ara nutzen zu können

```sh
$ python3 -m ara.setup.path
/usr/lib/python3.7/site-packages/ara

$ python3 -m ara.setup.plugins
/usr/lib/python3.7/site-packages/ara/plugins

$ python3 -m ara.setup.action_plugins
/usr/lib/python3.7/site-packages/ara/plugins/action
$ export ANSIBLE_ACTION_PLUGINS=$(python3 -m ara.setup.action_plugins)

$ python3 -m ara.setup.callback_plugins
/usr/lib/python3.7/site-packages/ara/plugins/callback
$ export ANSIBLE_CALLBACK_PLUGINS=$(python3 -m ara.setup.callback_plugins)

$ python3 -m ara.setup.lookup_plugins
/usr/lib/python3.7/site-packages/ara/plugins/lookup
$ export ANSIBLE_LOOKUP_PLUGINS=$(python3 -m ara.setup.lookup_plugins)

# Note: This doesn't export anything, it only prints the commands.
# To export directly from the command, use:
#     source <(python3 -m ara.setup.env)
$ python3 -m ara.setup.env
export ANSIBLE_CALLBACK_PLUGINS=/usr/lib/python3.7/site-packages/ara/plugins/callback
export ANSIBLE_ACTION_PLUGINS=/usr/lib/python3.7/site-packages/ara/plugins/action
export ANSIBLE_LOOKUP_PLUGINS=/usr/lib/python3.7/site-packages/ara/plugins/lookup

$ python3 -m ara.setup.ansible
[defaults]
callback_plugins=/usr/lib/python3.7/site-packages/ara/plugins/callback
action_plugins=/usr/lib/python3.7/site-packages/ara/plugins/action
lookup_plugins=/usr/lib/python3.7/site-packages/ara/plugins/lookup
```

`ansible.cfg`

```
[ara]
api_client = http
api_server = https://localhost:8080
api_username = user
api_password = password
api_timeout = 15
callback_threads = 4
argument_labels = check,tags,subset
default_labels = prod,deploy
ignored_facts = ansible_env,ansible_all_ipv4_addresses
ignored_files = .ansible/tmp,vault.yaml,vault.yml
ignored_arguments = extra_vars,vault_password_files
localhost_as_hostname = true
localhost_as_hostname_format = fqdn
```

**ARA environment variables**

```sh
export ARA_API_CLIENT=http
export ARA_API_SERVER="https://demo.recordsansible.org"
export ARA_API_USERNAME=user
export ARA_API_PASSWORD=password
export ARA_API_TIMEOUT=15
export ARA_CALLBACK_THREADS=4
export ARA_ARGUMENT_LABELS=check,tags,subset
export ARA_DEFAULT_LABELS=prod,deploy
export ARA_IGNORED_FACTS=ansible_env,ansible_all_ipv4_addresses
export ARA_IGNORED_FILES=.ansible/tmp,vault.yaml,vault.yml
export ARA_IGNORED_ARGUMENTS=extra_vars,vault_password_files
export ARA_LOCALHOST_AS_HOSTNAME=true
export ARA_LOCALHOST_AS_HOSTNAME_FORMAT=fqdn
```

## ### ad-hoc commands aufnehmen

Voraussetzungen:
- ara 1.4.1
- ansible 2.9.7

Konfiguration:

environment variable : `ANSIBLE_LOAD_CALLBACK_PLUGINS=True`
ansible.cfg: `bin_ansible_callbacks`


