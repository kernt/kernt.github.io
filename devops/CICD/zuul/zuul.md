---
tags:
  - zuul
  - cicd
  - ansible
  - git
  - gerrit
---

# Zuul Installation

## Red Hat / CentOS:

```SHELL
sudo yum install podman git python3
sudo python3 -m pip install git-review podman-compose
```

## Fedora:

```SHELL
sudo dnf install podman git python3
sudo python3 -m pip install git-review podman-compose
```

## OpenSuse:

```SHELL
sudo zypper install podman git python3
sudo python3 -m pip install git-review podman-compose
```

## Ubuntu / Debian:

```SHELL
sudo apt-get update
sudo apt-get install podman git python3-pip
sudo python3 -m pip install git-review podman-compose
```

**Zuul repository Clonen**

`git clone https://opendev.org/zuul/zuul`

**Ins Zielverzeichnis wechseln und podman ausführen**

```SHELL
cd zuul/doc/source/examples
podman-compose -p zuul-tutorial up
```

