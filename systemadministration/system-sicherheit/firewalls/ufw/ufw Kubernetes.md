---
tags:
  - sicherheit
  - ufw
  - microk8s
  - k8s
  - kubernetes
---

# Beispiel für microk8s


```sh
sudo ufw allow 179/tcp
sudo ufw allow 4789/tcp
sudo ufw allow 5473/tcp
sudo ufw allow 443/tcp
sudo ufw allow 6443/tcp
sudo ufw allow 2379/tcp
sudo ufw allow 4149/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10255/tcp
sudo ufw allow 10256/tcp
sudo ufw allow 9099/tcp
```


# Beispiel für K8s

**Auf den Nodes**

```
firewall-cmd --permanent --add-port=22/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
sudo ufw allow =2376/tcp
sudo ufw allow 2379/tcp
sudo ufw allow 2379/tcp 
sudo ufw allow 2380/tcp
sudo ufw allow 6443/tcp
sudo ufw allow 8472/udp
sudo ufw allow 9099/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10254/tcp
sudo ufw allow 30000-32767/tcp
sudo ufw allow 30000-32767/udp
```

**Auf den etcd Nodes**

```sh
sudo ufw allow 2376/tcp
sudo ufw allow 2379/tcp
sudo ufw allow 2380/tcp
sudo ufw allow 8472/udp
sudo ufw allow 9099/tcp
sudo ufw allow 10250/tcp
```

Für die Control PLanes nodes.

```sh
sudo ufw allow 2376/tcp
sudo ufw allow 6443/tcp
sudo ufw allow 8472/udp
sudo ufw allow 9099/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10259/tcp
sudo ufw allow 10257/tcp
```

**Auf den Worker Nodes**

```
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
```
