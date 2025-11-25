---
tags:
  - rabbitmq
  - cloud
---

# RabbitMQ Installation auf einen Kubernetes Cluster

https://www.rabbitmq.com/kubernetes/operator/install-operator

# RabbitMQ Installation auf 

* [install-debian](https://www.rabbitmq.com/docs/install-debian)

# [management-cli](https://www.rabbitmq.com/docs/management-cli)


## Enable RabbitMQ Management Dashboard (Optional)

```sh
sudo rabbitmq-plugins enable rabbitmq_management
```

The Web service should be listening on TCP port `15672`

`ss -tunelp | grep 15672==`

If you have an active Firewalld service, allow ports `5672` and `15672`

```sh
sudo firewall-cmd --add-port={5672,15672}/tcp --permanent
sudo firewall-cmd --reload
```

**Einen Administrator user anlegen**

```sh
$ sudo rabbitmqctl add_user admin StrongPassword
Adding user "admin" …

$ sudo rabbitmqctl set_user_tags admin administrator
Setting tags for user "admin" to [administrator] …
```

**User Löschen**

```sh
sudo rabbitmqctl delete_user user
```

**User Password ändern**

```sh
sudo rabbitmqctl change_password user strongpassword
```

**Einen neuen Virtualhost anlegen**

```sh
sudo rabbitmqctl add_vhost /my_vhost
```

**Virtualhosts auflisten**

```sh
sudo rabbitmqctl list_vhosts
```

**Einen virtualhost löschen**

```sh
sudo rabbitmqctl delete_vhost /myvhost
```

**Grant user Berechtigungen für einen vhost**

```sh
sudo rabbitmqctl set_permissions -p /myvhost user ".*" ".*" ".*"
```

**Auflisten der berechtigungen an einem Vhost**

```sh
sudo rabbitmqctl list_permissions -p /myvhost
```

**Benuter permissions auflisten**

```sh
rabbitmqctl list_user_permissions user
```

**user permissions entfernen**

```sh
rabbitmqctl clear_permissions -p /myvhost user
```