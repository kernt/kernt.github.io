---
tags:
  - anydesk
---
AnyDesk is a powerful remote desktop application that allows you to connect to and control other computers securely and efficiently. It offers low latency, high frame rates, and high-quality connections, making it an ideal solution for remote support, collaboration, and accessing files or applications from anywhere. AnyDesk ensures secure and reliable remote connections with its advanced encryption and security features.
## Update Rocky Linux Before AnyDesk Installation

Before proceeding with the tutorial, ensuring your system is up-to-date with all existing packages is good.

```bash
sudo dnf upgrade --refresh
```

## Import AnyDesk RPM

Before proceeding, you must copy and paste the following command in your terminal, creating the AnyDesk repository file under the /etc/yum.repos.d/ directory.

_AMD64 (64-bit) users:_

```bash
sudo tee /etc/yum.repos.d/anydesk.repo<<EOF
[anydesk]
name=AnyDesk Rocky Linux
baseurl=http://rpm.anydesk.com/centos/x86_64/
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://keys.anydesk.com/repos/RPM-GPG-KEY
EOF
```

Alternatively, you can verify the repository was added using the dnf repolist command.

```bash
dnf repolist | grep anydesk
```

_Example output:_

```
[joshua@rocky-linux ~]$ dnf repolist | grep anydesk
anydesk                 AnyDesk Rocky Linux
```

## Install AnyDesk via DNF Install Command

Finally, install AnyDesk with the following command.

```bash
sudo dnf install anydesk -y
```

Optionally, you can check the version installed using the following command.

```bash
anydesk --version
```

_Example output:_

```
anydesk --version
```

## Launch AnyDesk User Interface

Launching can be done in a few ways now that you have the software installed.

Using the command line terminal, you can open it quickly by using the following command.

```bash
anydesk
```

Also, while in the terminal, you can grab your AnyDesk ID using the following command.

```bash
anydesk --get-id
```

Then, you can connect to other computers or remote systems using the ID in a terminal command.

```bash
anydesk <AnyDesk ID>
```

The best way to use AnyDesk for desktop users who prefer not to use the command line terminal is to open the GUI of the application by following the path: 

Activities > Show Applications > AnyDesk.

## Additional AnyDesk Commands

### Update AnyDesk

The software should update itself with your system packages for desktop users using the DNF package manager. Use the following command in your terminal for users who want to check manually.

```bash
sudo dnf update --refresh
```

### Remove AnyDesk

When you no longer want the video conference software installed on your system, use the following command to remove it.

```bash
sudo dnf autoremove anydesk
```

Remove the repository if you plan not to re-install AnyDesk again.

```bash
sudo rm /etc/yum.repos.d/anydesk.repo
```
