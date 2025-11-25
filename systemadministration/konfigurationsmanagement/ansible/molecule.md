**Umgebung einrichten**

```sh
git clone https://github.com/shudarshon/ansible_role.git
cd ansible_role/roles/tomcat
virtualenv --python=/usr/bin/python .venv
source .venv/bin/activate
pip install ansible
pip install molecule
pip install docker
```

Datei molecule/default/molecule.yml anlegen für Docker tests

```yaml
driver:
name: docker
lint:
name: yamllint
options:
config-file: molecule/default/yamllint.yml
```

```yaml
driver:
name: docker
lint:
name: yamllint
options:
config-file: molecule/default/yamllint.yml
```

