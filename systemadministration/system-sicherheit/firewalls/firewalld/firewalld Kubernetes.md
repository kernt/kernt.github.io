---
tags:
  - sicherheit
  - firewalld
  - kubernetes
---
# Three-zone configuration:

- Public – open some ports and services to the whole world
- Trusted – open all ports and services for specific ip addresses (private network)
- Internal – open some ports and services for specific ip addresses

### Public zone

After installing the firewalld on ubuntu, this service starts automatically. Firewalld has _ssh_ in the public zone (which is the default zone) enabled, and it looks like this.

```bash
$ sudo firewall-cmd --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

# we do not need dhcpv6-client service
# remove dhcpv6-client from runtime and permanent configuration
$ sudo firewall-cmd --remove-service=dhcpv6-client
$ sudo firewall-cmd --permanent --remove-service=dhcpv6-client
```

You may notice that we did not specify a zone in the example. The default zone is public and we see that it has a _default_ target. According to [documentation](https://firewalld.org/documentation/man-pages/firewalld.zone.html) it’s something like _%%REJECT%%_ target. All packets that are not for _ssh_ service, are rejected.

No interfaces or source IP addresses are specified, so this zone is open to the whole world. This gives the default public zone an exception, because other zones must have a specified source ip address or interface to become active.  
You can also notice other possible zone settings, for example that icmp is enabled and therefore ping is not forbidden.

We use [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) to expose services running in a k8s cluster. Therefore, we must enable _http_ and _https_ on all workers in the public zone.

```bash
# enable http and https in runtime and permanent configuration
worker:~$ sudo firewall-cmd --add-service=http --add-service=https
worker:~$ sudo firewall-cmd --permanent --add-service=http --add-service=https
```
### Trusted zone

For Kubernetes to work properly, you need to enable multiple [ports](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports). Open ports for Kubernetes CNI (Container Network Interface) are also required. We use [Flannel](https://github.com/coreos/flannel/) plugin, which is simple and fully covers our requirements. For flannel, port _8472/udp_ is important if vxlan backend is used and _8285/udp_ for udp backend.

We had a bit easier work, because our cluster works on the private network _10.0.0.0/8_. Therefore, we configured flannel to use this network with the _–iface=[our private network interface]_ argument in _kube-flannel_ container. Subsequently, we added this network to the trusted zone of the master and all workers, making this zone active. We could also use a private interface instead of a private network.

```bash
# add private network to runtime and permanent configuration of trusted zone
$ sudo firewall-cmd --zone=trusted --add-source=10.0.0.0/8
$ sudo firewall-cmd --permanent --zone=trusted --add-source=10.0.0.0/8
$ sudo firewall-cmd --zone=trusted --list-all
trusted (active)
  target: ACCEPT
  icmp-block-inversion: no
  interfaces:
  sources: 10.0.0.0/8
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

Target is _ACCEPT_, which says that we accept all packets that come from private _10.0.0.0/8_ network. We have thus ensured the functioning of the kubernetes cluster. If we did not have a private network, we would have to add specific ip addresses to this zone or add specific ip addresses and ports to the internal zone.
### Internal zone

In order to be able to access this cluster from some public ip address as well, we must enable port _6443/tcp_ (Kubernetes API server) on the master.

```bash
# remove default services in internal zone
master:~$ sudo firewall-cmd --permanent --zone=internal --remove-service=dhcpv6-client --remove-service=mdns --remove-service=samba-client --remove-service=ssh
# add k8s api server port and some ip address to internal zone
master:~$ sudo firewall-cmd --permanent --zone=internal --add-port=6443/tcp
master:~$ sudo firewall-cmd --permanent --zone=internal --add-source=SOME_IP
# add permanent configuration to runtime
master:~$ sudo firewall-cmd --reload
master:~$ sudo firewall-cmd --zone=internal --list-all
internal (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: SOME_IP
  services:
  ports: 6443/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

The only problem that has occurred here is that _SOME_IP_ does not have access to the services and ports in public zone now. This is due to [firewalld settings](https://firewalld.org/documentation/man-pages/firewall-cmd) and the fact that both the internal and the public zone have _default_ target, and when a packet with _SOME_IP_ arrives, it will not find _ssh_ service in the internal zone and since the target is the _default_, this packet will look in the next zone. It finds the public zone but there is also a _default_ target and firewalld will reject this packet.

There are two ways to fix this:  
• enable [AllowZoneDrifting](https://firewalld.org/2020/01/allowzonedrifting) in the firewalld configuration, which is not recommended  
• add _ssh_ service to the internal zone as well, which we prefer

```bash
# add ssh service also to runtime and permanent configuration of internal zone
master:~$ sudo firewall-cmd --zone=internal --add-service=ssh
master:~$ sudo firewall-cmd --permanent --zone=internal --add-service=ssh
```
## Secure K8s CNI Flannel plugin

There are several ways to expose services in Kubernetes. One of them is [hostPort](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#support-hostport). This functionality is also available in flannel via the official plugin [portmap](https://www.cni.dev/plugins/meta/portmap/). This plugin uses iptables to map a host port to a container port. But it uses a prerouting chain. In this case, the rules are applied even earlier as firewalld rules, and a security hole may arise. If we want to prevent that, this plugin gave us the opportunity to do it. In the ConfigMap _kube-flannel-cfg_ it is necessary to add the parameter _“conditionsV4”_ to the portmap plugin and restart _kube-flannel-ds_ DaemonSet.

```bash
{
  “type”: “portmap”,
  “capabilities”:  {
    “portMappings”: true
  },
  “conditionsV4”: [“-s”, “10.0.0.0/8”]
}
```

If we add such a condition, the portmap plugin will add it to each iptables rule and in that case it will only expose it on the private network _10.0.0.0/8_. If we want, we can add other ip addresses separated by a comma, e.g. _“conditionsV4”: [“-s”, “10.0.0.0/8,SOME_IP”]_. Of course, it is possible to add other iptables rules.
## Secure K8s CRI Docker plugin

Here is the same problem as with hostPort. This problem arises if we want to run the container not in kubernetes but directly in the docker by command _docker run_ with _-p_ or _-P,_ or when k8s container interacts directly with the docker socket.

Docker CRI (Container Runtime Interface) adds iptables rules to the prerouting chain _DOCKER_, and thus applies its rules before any firewalld rules. This may caused a security hole. Docker knew about this problem and therefore created the [chain](https://docs.docker.com/network/iptables/#add-iptables-policies-before-dockers-rules) called _DOCKER-USER_, where we can add own rules. We can modify this iptables chain using the [firewalld direct interface](https://firewalld.org/documentation/direct/) and solve the security problem as is described [here](https://github.com/moby/moby/issues/35043#issuecomment-356036671) or [here](https://roosbertl.blogspot.com/2019/06/securing-docker-ports-with-firewalld.html).

```bash
# add the DOCKER-USER chain to firewalld
$ sudo firewall-cmd --permanent --direct --add-chain ipv4 filter DOCKER-USER
# allow connection for docker containers to the outside world
$ sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
# drop all other traffic to DOCKER-USER
$ sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 10 -j DROP

# Optional DOCKER-USER chain settings

## add your docker subnets to allow container communication
$ sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -s 172.17.0.0/16 -j RETURN

## add some private IP addresses
$ sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -s 10.0.0.0/8 -j ACCEPT

## add some public ports, use --ctorigdstport and --ctdir as is described on https://serverfault.com/a/933803
$ sudo firewall-cmd --permanent --direct --add-rule ipv4 filter DOCKER-USER 0 -p tcp -m conntrack --ctorigdstport 65535 --ctdir ORIGINAL -j ACCEPT

# restart services
$ sudo systemctl restart firewalld
$ sudo systemctl restart docker
```

When docker is restarted it adds _-A FORWARD -j DOCKER-USER_ rule to iptables which ensure that created _DOCKER-USER_ rules will be applied. This rule was not added in some cases and therefore we decided to add this rule to the docker service unit configuration file as follows:

```bash
$ cat /etc/systemd/system/docker.service.d/docker-user.conf
[Service]
# delete forward rule to prevent duplicates, ignore missing
ExecStartPost=-/bin/bash -c ‘iptables -D FORWARD -j DOCKER-USER’
# insert forward rule
ExecStartPost=/bin/bash -c ‘iptables -I FORWARD -j DOCKER-USER’
```