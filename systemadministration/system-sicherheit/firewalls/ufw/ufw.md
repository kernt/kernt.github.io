---
tags:
  - sicherheit
  - ufw
---

UFW (Uncomplicated Firewall) is a straightforward tool for managing firewall rules on Linux systems. It simplifies the process of configuring a firewall, making it especially useful for newer users who might find the traditional iptables challenging. UFW provides an easy way to create and manage firewall rules through simple commands, making it both accessible and effective.

Technical features of UFW include:

- **Ease of Use**: Commands are simplified for common firewall tasks.
- **Pre-configured Profiles**: Comes with profiles for common services.
- **Logging**: Allows easy configuration of logs to monitor activity.
- **IPv6 Support**: Fully supports IPv6.
- **Extensible**: Can be extended with additional rules for more advanced setups.

For new users, UFW is often better than iptables because it hides the complexity while still offering robust security features. It lets users set up firewall rules quickly without needing to understand the detailed syntax and operations of iptables, reducing the chance of errors.
# Install UFW (Uncomplicated Firewall)

```bash
sudo apt update && sudo apt upgrade
```

```bash
sudo apt install ufw
```

**UFW Firewall activsetzen**

```bash
sudo systemctl enable ufw --now
```

**Status der Firewall Prüfen**

```bash
systemctl status ufw
```

**Enable UFW Firewall**

```bash
sudo ufw enable
```

## Check UFW Status

After enabling the UFW firewall, verifying that the rules are active and correctly configured is crucial. You can check the status of your firewall using the following command:

```bash
sudo ufw status verbose
```

To get a more concise view of your firewall rules, you can use the “numbered” option instead. This option shows your firewall rules in a numbered sequence, making identifying and managing them easier. Use the following command to list your firewall rules in numbered sequence:

```bash
sudo ufw status numbered
```

The numbered output displays the rules in a more organized manner, making it easier to identify and manage them. You can use the rule numbers to modify or delete specific rules using the “delete” command.

In conclusion, verifying the status of your firewall is essential to ensure your system is protected from unauthorized access. Using the commands outlined in this step, you can quickly check the status of your UFW firewall and identify any misconfigurations.

## IPv6 mit NFW

_/etc/default/ufw_

```
...
IPV6=yes
...
```

## Verbindugen erlauben und verbieten

```
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 80

```

**Portbereiche**

```sh
sudo ufw allow 6000:6007/tcp
sudo ufw allow 6000:6007/udp
```

**Speziefische IP Addressen freigeben**

```sh
sudo ufw allow from 203.0.113.4
sudo ufw allow from 203.0.113.4 to any port 22
```

**Subneteze**

```sh
sudo ufw allow from 203.0.113.0/24
sudo ufw allow from 203.0.113.0/24 to any port 22
```

**Netzwerk Interfaces**

```sh
sudo ufw allow in on eth0 to any port 80
sudo ufw allow in on eth1 to any port 3306
```

**Verbindugen verbieten**

```sh
sudo ufw deny http
```

**Speziefische IP Addressen verbieten**

```sh
sudo ufw deny from 203.0.113.4
```

**Rules löschen**

```sh
sudo ufw status numbered

OutputStatus: active

     To                         Action      From
     --                         ------      ----
[ 1] 22                         ALLOW IN    15.15.15.0/24
[ 2] 80                         ALLOW IN    Anywhere

sudo ufw delete 2

```

**Aktuelle Rule löschen**

```
sudo ufw delete allow http
sudo ufw delete allow 80

```

**Status Prüfen**

`sudo ufw status verbose`

**Firewall zurücksetzen**

```sh
sudo ufw disable
sudo ufw reset
```

## Set UFW Default Policies

To modify the UFW rules, you can enter the following command in the terminal:

==To deny all incoming connections:==

```bash
sudo ufw default deny incoming
```

==To allow all outgoing connections:==

```bash
sudo ufw default allow outgoing
```

By default, UFW is already enabled with the default rules. However, you can modify these rules to suit your specific needs. For instance, if you want to block all outgoing connections and only allow specific outbound connections, you can use the following command to adjust the rules accordingly:

==To block all outgoing connections:==

```bash
sudo ufw default deny outgoing
```

It is worth noting that if you need to locate the default UFW firewall policies, you can find them in the /etc/default/ufw file. This file contains the configuration settings for UFW, including the default policies for incoming and outgoing traffic. By modifying the settings in this file, you can customize the firewall rules to meet your specific security needs.

### Erstellen von UFW Profilen

**Profiel _app profile title_ anlegen**

```
ufw app PROFILE {app profile title}
```

Beispiel Inhalt des Profiles

```
[appname]
title=1-liner here
description=a longer line here
ports=1,2,3,4,5,6,7,8,9,10,30/tcp|50/udp|53

```

**Infos zur App ausgeben**

```
ufw app info <app_name>
```

**App Update**

```
ufw app update <app_name> 
```

**Alle App Updates durchführen**

```
ufw app update --add-new <app_name> 
```

### Enable IPv6 on UFW

If your Debian system is configured with IPv6, you must ensure that UFW is configured to support IPv6 and IPv4 traffic. By default, UFW should automatically enable support for both versions of IP; however, it’s a good idea to confirm this.

To do so, open the default UFW firewall file using the following command:

```bash
sudo nano /etc/default/ufw
```

Once the file is open, locate the following line:

```bash
IPV6=yes
```

If the value is set to “no,” change it to “yes” and save the file by pressing `CTRL+O` and then `CTRL+X` to exit.

After making changes to the file, you’ll need to restart the UFW firewall service for the changes to take effect. To do so, run the following command:

```bash
sudo systemctl restart ufw
```

This will restart the UFW firewall service with the new configuration, enabling support for IPv6 traffic.

### Allow UFW SSH Connections

SSH (Secure Shell) is crucial to remotely accessing Linux servers. However, by default, UFW does not allow SSH connections. This can be problematic, especially if you have enabled the firewall remotely, as you may find yourself locked out. To allow SSH connections, you need to follow these steps.

First, enable the SSH application profile by typing the following command:

```bash
sudo ufw allow ssh
```

If you have set up a custom listening port for SSH connections other than the default port 22, for example, port 3541, you must open the port on the UFW firewall by typing the following.

```bash
sudo ufw allow 3541/tcp
```

The following commands can be used to block all SSH connections or change the port and block the old ones.

Use the following command to block all SSH connections (make sure local access is possible):

```bash
sudo ufw deny ssh/tcp
```

If changing the custom SSH port, open a new port and close the existing one, for example:

```bash
sudo ufw deny 3541/tcp 
```

### Enable UFW Ports

UFW can allow access to specific ports for applications or services. This section will cover how to open HTTP (port 80) and HTTPS (port 443) ports for a web server and how to allow port ranges.

To allow HTTP port 80, you can use any of the following commands:

**Allow by application profile:**

```bash
sudo ufw allow 'Nginx HTTP
```

**Allow by service name:**

```bash
sudo ufw allow http
```

**Allow by port number:**

```bash
sudo ufw allow 80/tcp
```

To allow HTTPS port 443, you can use any of the following commands:

**Allow by application profile:**

```bash
sudo ufw allow 'Nginx HTTPS'
```

**Allow by service name:**

```bash
sudo ufw allow https
```

**Allow by port number:**

```bash
sudo ufw allow 443/tcp
```

If you want to allow both HTTP and HTTPS ports, you can use the following command:

```bash
sudo ufw allow 'Nginx Full'
```

#### UFW Allow Port Ranges

You can allow individual ports and port ranges. When opening a port range, you must identify the port protocol.

To allow a port range with TCP and UDP protocols, use the following commands:

```bash
sudo ufw allow 6500:6800/tcp
sudo ufw allow 6500:6800/udp
```

Alternatively, you can allow multiple ports in one hit using the following commands:

```bash
sudo ufw allow 6500, 6501, 6505, 6509/tcp
sudo ufw allow 6500, 6501, 6505, 6509/udp
```

### Allow Remote Connections on UFW

Enabling remote connections on UFW can be crucial for networking purposes, and it can be done quickly with a few simple commands. This section explains how to allow remote connections to your system through UFW.

#### UFW Allow Specific IP Addresses

You can use the following command to allow specific IP addresses to connect to your system. This is useful when you need to allow only specific systems to connect to your server, and you can specify their IP addresses.

```bash
sudo ufw allow from 192.168.55.131
```

#### UFW Allow Specific IP Addresses on Specific Port

You can use the following command to allow an IP to connect to a specific port on your system. For instance, if you need to allow an IP to connect to your system’s port 3900, you can use this command:

```bash
sudo ufw allow from 192.168.55.131 to any port 3900
```

#### Allow Subnet Connections to a Specified Port

You can use the following command to allow connections from a range of IPs in a subnet to a specific port. This is useful when you need to allow connections from a specific range of IPs, and you can specify the subnet to allow connections.

```bash
sudo ufw allow from 192.168.1.0/24 to any port 3900
```

This command will connect all IP addresses from 192.168.1.1 to 192.168.1.254 to port 3900.

#### Allow Specific Network Interface

If you need to allow connections to a specific network interface, you can use the following command. This is useful when you have multiple network interfaces and need to allow connections to a specific interface.

```bash
sudo ufw allow in on eth2 to any port 3900
```

By using these commands, you can easily allow remote connections to your system through UFW while maintaining its security.

### Deny Remote Connections on UFW

If you’ve noticed suspicious or unwanted traffic coming from a particular IP address, you can deny connections from that address using UFW. UFW denies all incoming connections by default, but you can create rules to block connections from specific IPs or IP ranges.

For example, to block connections from a single IP address, you can use the following command:

```bash
sudo ufw deny from 203.13.56.121
```

If a hacker is using multiple IP addresses within the same subnet to attack your system, you can block the entire subnet by specifying the IP range in CIDR notation:

```bash
sudo ufw deny from 203.13.56.121/24
```

You can also create rules to deny access to specific ports for the blocked IP or IP range. For example, to block connections from the same subnet to ports 80 and 443, you can use the following commands:

```bash
sudo ufw deny from 203.13.56.121/24 to any port 80
sudo ufw deny from 203.13.56.121/24 to any port 443
```

It’s important to note that blocking incoming connections can be an effective security measure, but it’s not foolproof. Hackers can still use techniques like IP spoofing to disguise their true IP address. Therefore, it’s important to implement multiple layers of security and not rely solely on IP blocking.

### Delete UFW Rules

Deleting unnecessary or unwanted UFW rules is essential for maintaining an organized and efficient firewall. You can delete UFW rules in two different ways. Firstly, to delete a UFW rule using its number, you need to list the rule numbers by typing the following command:

```bash
sudo ufw status numbered
```

The output will display a list of numbered UFW rules, allowing you to identify the rule you want to delete. Once you have determined the number of the rule you wish to remove, type the following command:

```bash
sudo ufw delete [rule number]
```

For instance, suppose you want to delete the third rule for IP Address 1.1.1.1. In that case, you need to find the rule number by running the “sudo ufw status numbered” command and type the following command in your terminal:

```bash
sudo ufw delete 3
```

Deleting rules that are no longer required helps to maintain the security and efficiency of your firewall.

### Access and View UFW Logs

The UFW firewall logs all events, and it is essential to review these logs periodically to identify potential security breaches or troubleshoot network issues. By default, UFW logging is set to low, which is adequate for most desktop systems. However, servers may require a higher level of logging to capture more details.

You can adjust the logging level of UFW to low, medium, or high or disable it entirely. To set the UFW logging level to low, use the following command:

```bash
sudo ufw logging low
```

To set UFW logging to medium:

```bash
sudo ufw logging medium
```

To set UFW logging to high:

```bash
sudo ufw logging high
```

The last option is to disable logging altogether; be sure you are happy with this, and it will not require log checking.

```bash
sudo ufw logging off
```

To view UFW logs, you can find them in the default location of /var/log/ufw.log. Using the tail command, you can view live logs or print out a specified number of recent log lines. For instance, to view the last 30 lines of the log, use the following command:

```bash
sudo ufw tail /var/log/ufw.log -n 30
```

Reviewing the logs can help you determine which IP addresses are attempting to connect to your system and identify any suspicious or unauthorized activities. Furthermore, reviewing the logs can help you understand network traffic patterns, optimize network performance, and identify any issues that may arise.

### Test UFW Rules

Testing your UFW firewall rules before applying them to ensure they work as intended is always a good idea. The “–dry-run” flag allows you to see the changes that would have been made without actually applying them. This is a helpful option for highly critical systems to prevent accidental changes.

To use the “–dry-run” flag, type the following command:

```bash
sudo ufw --dry-run enable
```

To disable the “–dry-run” flag, simply use the following command:

```bash
sudo ufw --dry-run disable
```

### Reset UFW Rules

Sometimes, you may need to reset your firewall back to its original state with all incoming connections blocked and outgoing connections allowed. This can be done using the “reset” command:

```bash
sudo ufw reset
```

Confirm reset, enter the following:

```bash
sudo ufw status
```

The output should be:

```bash
Status: inactive 
```

Once the reset is complete, the firewall will be inactive, and you will need to re-enable it and start adding new rules. It is important to note that the reset command should be used sparingly, as it removes all existing rules and can potentially leave your system vulnerable if not done correctly.

### How to find All Open Ports (Security Check)

Your system’s security should be a top priority, and one way to ensure it is by checking for open ports regularly. UFW blocks all incoming connections by default, but sometimes ports may be left open inadvertently or for legitimate reasons. In this case, knowing which ports are open and why is essential.

One way to check for open ports is to use Nmap, a well-known and trusted network exploration tool. To install Nmap, first type the following command to install it.

```bash
sudo apt install nmap
```

Next, find your system’s internal IP address by typing:

```bash
hostname -I
```

_Example output:_

```bash
192.168.50.45
```

With the IP address, run the following command:

```bash
nmap 192.168.50.45
```

Nmap will scan your system and list all open ports. If you find any open ports you are unsure about, investigate them before closing or blocking them, as it may break services or lock you out of your system.

Based on the information in this tutorial, you can create custom UFW rules to close or restrict open ports. UFW blocks all incoming connections by default, so ensure you do not block legitimate traffic before implementing changes.

## UFW Management APT Commands

### Remove the UFW Firewall

UFW is a valuable tool for managing firewall rules and securing your Debian system. However, there may be situations where you need to disable or remove it.

If you want to disable UFW temporarily, you can do so by typing the following command:

```bash
sudo ufw disable
```

This will turn off the firewall, allowing all traffic to pass through the system.

On the other hand, if you need to remove UFW from your system altogether, you can do so with the following command:

```bash
sudo apt remove ufw
```

However, it’s essential to note that removing UFW altogether from your system may leave it vulnerable to external attacks. Unless you have a solid alternative to manage your system’s firewall or understand how to use IPTables, it’s not advisable to remove UFW.

Therefore, before removing UFW, ensure you have an alternative solution to maintain your system’s security and prevent unauthorized access.

```sh
ufw allow ssh
ufw allow http
ufw allow https
ufw default deny incoming
ufw --force enable
```

