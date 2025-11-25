
sudo useradd -s /bin/bash -d /opt/stack -m stack
sudo chmod +x /opt/stack
git clone https://opendev.org/openstack/devstack
cd devstack

```
# *Datei local.conf anpassen*

```sh
[[local|localrc]]

ADMIN_PASSWORD=secret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
```

*Install stack*

`./stack.sh`

Default Password ist secret

[openstack-setup-all-in-one](https://www.openstackfaq.com/openstack-setup-all-in-one/)