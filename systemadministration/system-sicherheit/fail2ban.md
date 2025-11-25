---
tags:
  - sicherheit
  - fail2ban
---
# Fail2ban

```sh
sudo dnf install epel-release
sudo dnf install fail2ban
```


```shell
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

```bash
sudo nano /etc/fail2ban/jail.local
```

Within this file, locate the section with IPTABLES settings. These should be replaced with Firewalld-specific configurations, reflecting the change in the firewall backend:

```bash
; Replace this
banaction = iptables-multiport
banaction_allports = iptables-allports

; With this
banaction = firewallcmd-rich-rules[actiontype=]
banaction_allports = firewallcmd-rich-rules[actiontype=]
```

**/etc/fail2van/jail.local**

```
ignoreip = 127.0.0.1/8 192.168.1.10 192.168.1.20

```

**## Unbanning a system**

```shell
sudo fail2ban-client set sshd unbanip 192.168.1.69
```

## Customize Fail2Ban Settings

It is crucial to configure Fail2Ban to meet your security requirements. This involves setting parameters like ban time, whitelisting IPs, and email notifications.

### Ban Time Increment:

To escalate the ban duration for repeat offenders, adjust the `bantime.multipliers` setting. By uncommenting and modifying this line, you can define how the ban time increases with each subsequent offense:

```bash
bantime.multipliers = 1 5 30 60 300 720 1440 2880
```

This configuration means that the initial ban time is 1 minute, increasing to 5 minutes, then 30 minutes, and so on, up to a maximum of 48 hours.
### Whitelist IPs

Excluding specific IPs from bans is sometimes necessary, especially for trusted networks or your IP. This is where the `ignoreip` setting comes into play. Modify it to add any IP addresses you wish to whitelist:

```sh
ignoreip = 127.0.0.1/8 ::1 180.53.31.33
```
### Default Ban Time Settings

You can set general ban time rules for various services. For instance, adjusting the ban time and retry limits for the Apache service:

```
[apache-noscript]
enabled = true
port     = http,https
logpath  = %(apache_error_log)s
bantime = 1d
maxretry = 3
```

In this example, the ban time is set to 1 day with a maximum retry limit of 3 attempts.

### Email Notifications

Configure Fail2Ban to send email notifications in case of a ban action. This is done by setting the `destemail` and `sender` fields:

```
destemail = admin@example.com
sender = fail2ban@example.com
```

This configuration ensures you are promptly informed of any security actions taken by Fail2Ban.
## Additional Fail2ban Jails Configuration

### Fail2ban Nginx Jail

A custom jail can be created for Nginx servers to block requests with invalid hostnames. This is useful for mitigating attacks that target non-existent domains on your server:

```
[nginx-badhost]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 3
bantime = 2h
```

#### Postfix Jail

### Fail2ban Postfix Jail

To protect your Postfix mail server from spam-related breaches, create a jail specifically targeting spam activities:

```ini
[postfix-spam]
enabled = true
port = smtp,ssmtp
logpath = /var/log/mail.log
maxretry = 5
bantime = 1h
```
#### Dovecot Jail

### Fail2ban Dovecot Jail

For Dovecot IMAP services, it’s essential to have a jail that targets unauthorized access attempts, enhancing the security of your email server:

```ini
[dovecot-auth]
enabled = true
port = imap,imaps
logpath = /var/log/dovecot.log
maxretry = 5
bantime = 1h
```
### Restart Fail2Ban

After all configurations are set, restart Fail2Ban to activate the changes. This ensures that your newly set parameters are applied, and the system is protected according to the new rules:

```bash
sudo systemctl restart fail2ban
```
## Custom Filters and Actions with Fail2ban

Creating custom filters and actions in Fail2Ban empowers you to tailor your server’s security to meet specific needs on Fedora Linux.
### Custom Filters

Fail2Ban utilizes filters, defined by regular expressions, to identify unauthorized access attempts in log files. These filters are located in the `/etc/fail2ban/filter.d` directory.

#### Creating a Custom Filter

To create a custom filter, you open a new file in the filter directory.

Execute the command:

```bash
sudo nano /etc/fail2ban/filter.d/mycustomfilter.conf
```

This step involves using `nano`, a text editor, to create and open a new filter configuration file.

Next, define your custom filter within this file. The filter definition usually comprises a `failregex`, a regular expression that matches log entries indicating unauthorized access attempts. For instance:

```
[Definition]
failregex = ^.*unauthorized access attempt from <HOST>.*$
ignoreregex =
```

In this example, the `failregex` pattern is set to match any log entry that includes an unauthorized access attempt from a specific IP address, denoted by `<HOST>`.
### Custom Actions

In Fail2Ban, actions define how the system responds to detected offenses and are stored in the `/etc/fail2ban/action.d` directory.
#### Creating a Custom Action

To create a custom action, start by creating a new file in the action directory:

```bash
sudo nano /etc/fail2ban/action.d/mycustomaction.conf
```

This opens a new file in `nano` for editing your custom action’s configuration.

Now, in this file, add your action definition:

```
[Definition]
actionstart = /usr/local/bin/mycustomaction start <name>
actionstop = /usr/local/bin/mycustomaction stop <name>
actioncheck = /usr/local/bin/mycustomaction status <name>
actionban = /usr/local/bin/mycustomaction ban <ip>
actionunban = /usr/local/bin/mycustomaction unban <ip>
```

Replace `/usr/local/bin/mycustomaction` with the path to your custom action script. Parameters like `<name>` and `<ip>` will be dynamically replaced by Fail2Ban when the action is executed.
#### Integrating Custom Filters and Actions

After creating your custom filters and actions, integrate them into your jail configuration. For example:

```
[mycustomjail]
enabled = true
port = 8080
logpath = /var/log/mycustomservice.log
filter = mycustomfilter
action = mycustomaction
maxretry = 3
bantime = 1h
```

This jail configuration employs the custom filter and action you created. It monitors the specified log file and enforces bans based on your settings.

#### Restarting Fail2Ban

To activate your new configurations, restart Fail2Ban:

```bash
sudo systemctl restart fail2ban
```

This ensures all your custom settings are loaded and operational.
## Fail2ban-Client for Ban and Unban Operations

The `fail2ban-client` command is integral for managing Fail2Ban configurations and jails, particularly for manual ban and unban operations on IP addresses.
### Ban an IP Address

To ban an IP address in a specific jail, execute the command:

```bash
sudo fail2ban-client set <jail-name> banip <ip-addresss>
```

In this command, replace `<jail-name>` with the relevant jail name and `<ip-address>` with the IP you wish to ban. For example:

```bash
sudo fail2ban-client set apache-badbots banip 192.168.1.1
```

This particular command will ban the IP address 192.168.1.1 in the `apache-badbots` jail, effectively preventing access from that IP to the services protected by this jail.

### Unban an IP Address

To reverse a ban, unbanning an IP address in a specific jail, use:

```bash
sudo fail2ban-client set <jail-name> unbanip <ip-address>
```

Here, substitute `<jail-name>` and `<ip-address>` with the appropriate jail name and IP address. For instance:

```bash
sudo fail2ban-client set apache-badbots unbanip 192.168.1.1
```

This command will lift the ban on the IP address 192.168.1.1 from the `apache-badbots` jail.

### Display the List of Banned IP Addresses

To view a list of currently banned IP addresses in a specific jail, the command is:

```bash
sudo fail2ban-client status <jail-name>
```

Replace `<jail-name>` with the name of the jail you wish to inspect.

For example:

```bash
sudo fail2ban-client status apache-badbots
```

Executing this command will show the status of the `apache-badbots` jail, including a list of all IP addresses currently banned by that jail.
### Help and Documentation

For further assistance or details about the `fail2ban-client` command, utilize the `-h` flag:

```bash
sudo fail2ban-client -h
```

This command will display a help menu, offering a comprehensive list of available options and commands for the `fail2ban-client`.

Utilizing the `fail2ban-client` command streamlines the process of managing jails, and enables the manual banning and unbanning of IP addresses, along with the ability to monitor the status of your Fail2Ban configurations.

## Verifying Firewalld and Fail2ban Integration

Ensuring that Firewalld and Fail2ban function correctly is crucial for optimal server security. A straightforward test can confirm their integration.
### Enabling the SSHD Jail for Testing

To test the integration between Fail2ban and Firewalld, temporarily enable the SSHD jail in Fail2ban, even if it’s not in use. This action allows for a practical assessment of the system’s functionality.

Open the Fail2ban jail configuration file using this command:

```bash
sudo nano /etc/fail2ban/jail.local
```

This command opens the Fail2ban configuration file in the nano editor, allowing you to modify the settings.

In the file, locate the `[sshd]` section and change the `enabled` setting to `true`:

```
[sshd]
enabled = true
```

Setting this to `true` activates the SSHD jail, a crucial step for testing.

After modifying the file, save the changes and exit the editor. Then, restart Fail2ban to apply these changes:

```bash
sudo systemctl restart fail2ban
```

Restarting Fail2ban ensures that the new configuration is active.

### Testing the Ban Functionality

To manually test the ban functionality:

Ban an IP address in the SSHD jail using the `fail2ban-client` command:

```bash
sudo fail2ban-client set sshd banip 192.155.1.7
```

This command instructs Fail2ban to ban the specified IP address in the SSHD jail.

Next, verify if Firewalld has applied the ban. Use the `firewall-cmd` command to list the rich rules:

```bash
firewall-cmd --list-rich-rules
```

This lists all rich rules in Firewalld, including those added by Fail2ban.

An example output indicating successful integration:

```bash
rule family="ipv4" source address="192.155.1.7" port port="ssh" protocol="tcp" reject type="icmp-port-unreachable"
```

Fail2ban and Firewalld have correctly banned this IP address, as the output indicates.

### Reverting the Changes

After completing the test, revert the changes if you no longer require the SSHD jail:

Open the Fail2ban jail configuration file again:

```bash
sudo nano /etc/fail2ban/jail.local
```

Find the `[sshd]` section and reset the `enabled` setting back to `false`:

```
[sshd]
enabled = false
```

Save your changes and exit the editor. Finally, restart Fail2ban to apply the reversal:

```bash
sudo systemctl restart fail2ban
```

Restarting Fail2ban deactivates the SSHD jail, returning the configuration to its original state.

## Monitor Fail2Ban Logs

Effectively monitoring Fail2Ban logs is essential to ensure the smooth operation of your jails on Fedora Linux. The default location for Fail2Ban logs is `/var/log/fail2ban.log`.
### Watching the Log File in Real-Time

To observe the log file as events occur, utilize the `tail -f` command. This allows you to view log entries in real time, which is particularly useful for immediate troubleshooting or monitoring ongoing activities:

```bash
tail -f /var/log/fail2ban.log
```

Executing this command in a terminal window maintains the log file open and updates it in real-time with each new entry added.

### Displaying a Specific Number of Lines from the Log File

If you need to review a specific number of recent log entries, the `tail` command can be modified with the `-n` flag. This flag specifies the number of lines from the end of the file to display:

```bash
tail -f /var/log/fail2ban.log -n 30
```

This example will display the last 30 lines of the log file. You can replace `30` with any number of lines that fit your requirements.
### Searching the Log File for Specific Keywords or IP Addresses

For more targeted log analysis, such as finding entries related to a specific IP address or event, use the `grep` command. This tool searches through the log file for specified patterns:

```bash
grep '192.168.1.1' /var/log/fail2ban.log
```

In this example, `grep` is used to locate all occurrences of the IP address `192.168.1.1` in the Fail2Ban log. This method is particularly effective for quickly identifying actions taken against a specific IP or identifying patterns related to certain types of security incidents.
## Managing Fail2ban

### Remove (Uninstall) Fail2ban

When you no longer need Fail2Ban on your Fedora system, uninstall it to remove the software and its related unused dependencies. This action ensures your system remains tidy and efficient.

Execute the following command to uninstall Fail2Ban and its Firewalld integration:

```bash
sudo dnf remove fail2ban fail2ban-firewalld
```

Note that this will also remove all the unused dependencies installed with Fail2Ban.
### Quellen

* [install-fail2ban-centos-8](https://linuxtips.us/install-fail2ban-centos-8/)