# fluentd Installation

1) **Ntp einrichten**

2) **Datei  Descriptoren einrichten**

_/etc/security/limits.conf_

```sh
root soft nofile 65536
root hard nofile 65536
* soft nofile 65536
* hard nofile 65536
```

3) **Network Kernel Parameter Optimieren**

_/etc/sysctl.conf_

```sh
net.core.somaxconn = 1024
net.core.netdev_max_backlog = 5000
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_wmem = 4096 12582912 16777216
net.ipv4.tcp_rmem = 4096 12582912 16777216
net.ipv4.tcp_max_syn_backlog = 8096
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 10240 65535
# If forward uses port 24224, reserve that port number for use as an ephemeral port.
# If another port, e.g., monitor_agent uses port 24220, add a comma-separated list of port numbers.
# net.ipv4.ip_local_reserved_ports = 24220,24224
net.ipv4.ip_local_reserved_ports = 24224
```

**Benutzen von sticky bit symlink/hardlink protection**

Fluentd sometimes uses predictable paths for dumping, writing files, and so on. This default settings for the protections are in `/etc/sysctl.d/10-link-restrictions.conf`, or `/usr/lib/sysctl.d/50-default.conf` or elsewhere.

For symlink attack protection, check the following parameters are set to `1`:

```
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
```

This settings are almost enough for time-of-check to time-of-use (TOCTOU, TOCTTOU or TOC/TOU) which are a class of software bugs.

If you turned off these protections, please turn them on.

Use `sysctl -p` command or reboot your node for the changes to take effect.

4) Installation des Packages

Install from Apt Repository

_/etc/apt/sources.list.d/fluent-lts.sources_

For Ubuntu Noble:

**fluent-package 5 (LTS)**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-ubuntu-noble-fluent-package5-lts.sh | sh
```

**fluent-package 5**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-ubuntu-noble-fluent-package5.sh | sh
```

For Ubuntu Jammy:

**fluent-package 5 (LTS)**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-ubuntu-jammy-fluent-package5-lts.sh | sh
```

**fluent-package 5**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-ubuntu-jammy-fluent-package5.sh | sh
```

For Ubuntu Focal:

**fluent-package 5 (LTS)**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-ubuntu-focal-fluent-package5-lts.sh | sh
```

**fluent-package 5**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-ubuntu-focal-fluent-package5.sh | sh
```

For Debian Bookworm:

**fluent-package 5 (LTS)**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-debian-bookworm-fluent-package5-lts.sh | sh
```

**fluent-package 5**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-debian-bookworm-fluent-package5.sh | sh
```

For Debian Bullseye:

**fluent-package 5 (LTS)**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-debian-bullseye-fluent-package5-lts.sh | sh
```

**fluent-package 5**

```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-debian-bullseye-fluent-package5.sh | 
```

