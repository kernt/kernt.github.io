---
tags:
  - sicherheit
  - selinux
---

SELinux (Security-Enhanced Linux) is a security architecture integrated into the Linux kernel that provides a mechanism for supporting access control security policies. It helps protect systems from malicious activities by enforcing mandatory access controls (MAC), reducing the risk of security breaches. SELinux uses a set of security policies that define how processes and users can interact with different files, devices, and network ports.

To install and configure SELinux on Debian 12, 11, or 10, follow these steps. This guide will cover the installation process, basic configuration, and some common commands to manage SELinux policies and status.

## Deactivate AppArmor on Debian

Before installing SELinux, ensure you don’t have AppArmor, another security module, actively running on your Debian system. Running both security modules at the same time might cause conflicts.

To check if AppArmor is installed and active, run the following command:

```bash
sudo systemctl status apparmor
```

If AppArmor is running, you must deactivate it before installing SELinux. Use the following command to disable AppArmor:

```bash
sudo systemctl disable apparmor --now
```

Ensure you do this, or you might encounter issues when working with SELinux.

## Proceed with SELinux Installation

The ensuing step involves installing the necessary SELinux packages and subsequent activation of SELinux on your Debian system.

Start with these steps:

- Install policycoreutils: This package contains essential utilities for managing SELinux policies.
- Install selinux-utils: This package offers an extended set of SELinux utilities.
- Install selinux-basics: This provides the basic SELinux infrastructure.

To initiate the process, install the necessary packages for SELinux by running the following command:

```bash
sudo apt install policycoreutils selinux-utils selinux-basics selinux-policy-default
```

This command fetches and installs the essential packages to enable SELinux on your Debian system.

After installing the packages, the next move is to activate SELinux. To accomplish this, execute the ensuing command with root permissions:

```bash
sudo selinux-activate
```

This command prepares your system to load SELinux during the boot process, thus facilitating using SELinux security features.

Once you complete these steps, restart your system to implement the changes and initiate SELinux in the desired mode.

```bash
sudo reboot
```

After the system reboots, install, activate, and configure SELinux in enforcing mode. Then, configure SELinux according to your needs.

## Understanding the Modes of SELinux

SELinux, or Security-Enhanced Linux, offers a robust mechanism to manage access controls and enhance system security. Central to its functionality are three distinct modes:

- **Enforcing Mode**: This is the default setting. In this mode, SELinux actively enforces its security policies, denying access based on the predetermined rules.
- **Permissive Mode**: While this mode logs any policy violations, it stops short of enforcing them. It’s beneficial for testing and debugging your policies without causing disruptions.
- **Disabled Mode**: As the name suggests, this mode deactivates SELinux, ensuring no policies are enforced or logged.

### Modifying the SELinux Configuration File

The /etc/selinux/config file is the epicenter of the SELinux configuration. To gain access to this file using a text editor like Nano, the following command is used:

```bash
sudo nano /etc/selinux/config
```

Within this configuration file, you’ll need to alter the SELINUX line to suit your preferred mode, for instance:

Enforcing mode:

```bash
SELINUX=enforcing
```

Permissive mode:

```bash
SELINUX=permissive
```

Disabled mode:

```bash
SELINUX=disabled
```

### Applying the Configuration

For the modifications to the SELinux configuration to take effect, a system reboot is necessary:

```bash
sudo reboot
```

### Additional SELinux Configuration Options

SELinux offers various configuration options that can be tailored to your requirements. For example:

- SETLOCALDEFS: This setting specifies the use of locally defined file contexts. By setting this value to 0, you instruct the system to use the default file contexts provided by the SELinux policy. To disable the use of locally defined file contexts, edit the /etc/selinux/config file and update the SETLOCALDEFS line:

```bash
SETLOCALDEFS=0
```

- SELINUXTYPE: This setting defines the type of policy to be used. The most common policy type is “targeted”, a set of policies intended to protect specific system services without affecting the entire system. To set the policy type to “targeted”, edit the /etc/selinux/config file and update the SELINUXTYPE line:

```bash
SELINUXTYPE=targeted
```

### Configuring SELinux for a Web Server

For instance, let’s assume you are running a web server on your Debian system and aim to configure SELinux to allow HTTP and HTTPS traffic. The `semanage` command is used for this purpose to update the SELinux policy.

Start by installing the `semanage` utility:

```bash
sudo apt install policycoreutils-python-utils
```

Then, run the following commands to allow HTTP and HTTPS traffic:

```bash
sudo semanage port -a -t http_port_t -p tcp 80
sudo semanage port -a -t http_port_t -p tcp 443
```

These commands update the SELinux policy, allowing your web server to accept incoming connections on ports 80 (HTTP) and 443 (HTTPS).

These commands modify the SELinux policy, enabling your web server to receive incoming connections on ports 80 (HTTP) and 443 (HTTPS). The semanage command can also configure various aspects of your system’s security. For instance, to allow a specific user to access a directory, you might use a command like:

```bash
sudo semanage fcontext -a -t user_home_t "/home/myuser(/.*)?"
```

This command changes the file context for the “/home/myuser” directory, allowing the ‘myuser’ to access it. Please refer to the semanage man page for more details on its usage and syntax.

Lastly, you can always get an update on your SELinux status by running the following command:

```bash
sestatus
```

Now you should see a similar output as:

SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             default
Current mode:                   permissive
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33

Remember, this is an example; your output should differ depending on your setup.

## Addressing Common SELinux Issues

In this section, we will tackle some usual SELinux issues and give guidance on how to troubleshoot them.

### Restoring Default File Contexts

One prevalent issue with SELinux is incorrect file contexts. SELinux will deny access if the file context is wrong, leading to several application issues. To restore the default file contexts, use the restorecon command.

For instance, if you’re facing issues with the /var/www/html directory, execute:

```bash
sudo restorecon -Rv /var/www/html
```

### Temporarily Switching to Permissive Mode

If you’re uncertain whether SELinux causes an issue, you can temporarily switch to permissive mode for testing. To switch to permissive mode, execute:

```bash
sudo setenforce 0
```

Test your application to see if the problem is resolved. If it is, it’s likely related to SELinux policies. Remember to switch back to enforcing mode once you’ve completed testing:

```bash
sudo setenforce 1
```

### Reviewing SELinux Logs

To understand the root cause of a SELinux issue, it’s crucial to review the SELinux logs. On Debian, the primary log file for SELinux is /var/log/audit/audit.log. Use the tail command to view the most recent log entries:

```bash
sudo tail /var/log/audit/audit.log
```

Search for log entries containing “denied” or “AVC” to identify potential SELinux policy breaches.

### Using Audit2allow to Create Custom Policy Modules

If you encounter issues related to SELinux policies, the audit2allow tool can analyze the audit logs and generate a custom policy module to address the issue.

For example, to create a custom policy for a specific issue, execute:

```bash
sudo grep 'denied' /var/log/audit/audit.log | audit2allow -M mycustommodule
sudo semodule -i mycustommodule.pp
```

### Troubleshooting SELinux Booleans

SELinux Booleans lets you turn specific functionalities on or off. If you encounter an issue that might be related to a Boolean, use the getsebool -a command to list all available Booleans and their current values:

```bash
sudo getsebool -a
```

After identifying the relevant Boolean, you can toggle its value using the setsebool command. For instance, to enable the httpd_can_network_connect Boolean, execute:

```bash
sudo setsebool -P httpd_can_network_connect 1
```

Remember that the -P flag makes the change persist through reboots.