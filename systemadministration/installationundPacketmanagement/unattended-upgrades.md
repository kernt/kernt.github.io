# Install Unattended-Upgrades Package

To install the Unattended Upgrades Package, open the terminal and enter the following command:

```bash
sudo apt install unattended-upgrades
```

This command will install the package and all its dependencies on your Ubuntu system. The package should be installed by default. However, if you have removed it, use the command above to reinstall it.

You also need to install the apt-config-auto-update package if you want your Ubuntu system to restart after applying upgrades requiring the system to restart automatically. Use the following command to install it:

```bash
sudo apt install apt-config-auto-update
```

It is also recommended that laptop users install the powermgmt-base package to use unattended battery options. To install this package, open the terminal and enter the following command:

```bash
sudo apt install powermgmt-base
```

Once the package is installed, you must configure it to meet your needs. For example, you may want to customize which updates to install automatically, which to ignore, and when to install them. Note that the Unattended Upgrades Package requires root privileges to run. Therefore, you must run all the commands mentioned in this guide using the sudo command.

Verifying that the Unattended Upgrades Package is working correctly is also recommended. To do this, open the terminal and enter the following command:

```bash
sudo unattended-upgrades --dry-run --debug
```

It is important to be familiar with the systemctl commands for Unattended Upgrades, as you may need to check the status after making changes or restarting. To check the status of Unattended Upgrades, use the following command:

```bash
systemctl status unattended-upgrades
```

The following systemctl commands will allow you to start, stop, enable on boot, disable on boot, or restart the Unattended Upgrades service:

Start the unattended services:

```bash
sudo systemctl start unattended-upgrades
```

Stop the unattended services:

```bash
sudo systemctl stop unattended-upgrades
```

Enable on boot the unattended services:

```bash
sudo systemctl enable unattended-upgrades
```

Disable on boot the unattended services:

```bash
sudo systemctl disable unattended-upgrades
```

Restart the unattended services:

```bash
sudo systemctl restart unattended-upgrades
```

# Configure Unattended Upgrades

This section covers configuring the Unattended Upgrades package settings in the configuration file using terminal commands. It is important to note that these settings are optional and can be personalized according to your requirements. Each setting will be explained in detail to help you understand its purpose and functionality. This will enable you to make informed decisions when configuring the Unattended Upgrades package on your Ubuntu system.

Before going into the configuration file, first, here is a summary of all CLI options for the Unattended Upgrades package with explanations:

|Option|Description|
|---|---|
|-h, –help|Displays the help message and exits|
|-d, –debug|Enables debug messages|
|–apt-debug|Makes apt/libapt print verbose debug messages|
|-v, –verbose|Enables info messages|
|–dry-run|Simulates upgrade process and downloads but does not install|
|–download-only|Only downloads upgrades; do not attempt to install them|
|–minimal-upgrade-steps|Upgrades packages in minimal steps (and allows interruption with SIGTERM) – this is the default behavior|
|–no-minimal-upgrade-steps|Upgrades all packages together instead of in smaller sets|

These options can be used when running the unattended-upgrade command in the terminal to control the behavior of the automatic upgrade process. For example, the –dry-run option can help test the upgrade process without actually making any changes to the system, while the –download-only option can be used to download the upgrades without installing them.

The –minimal-upgrade-steps option is the default behavior for Unattended Upgrades, while the –no-minimal-upgrade-steps option upgrades all packages together instead of in smaller sets. Understanding these options when configuring Unattended Upgrades is important to ensure that the behavior matches your preferences and requirements.

## Editing the Configuration File

To edit the configuration file, it is recommended to use a text editor with root privileges, such as nano. To open the file with nano, run the following command:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

This will open the configuration file in the nano editor. From here, you can modify the various options listed above. Remember to remove any comments (lines beginning with “//” or “#”) in the file to activate the options.

Once you have finished editing the file, save and exit by pressing “Ctrl+X”, then “Y”, and then “Enter”.

## Allowed Origins

This option specifies the sources from which updates should be installed. By default, only the security and update repositories are allowed. You can add additional repositories to the list by uncommenting this option and adding the desired repositories, as shown in the example below:

```
Unattended-Upgrade::Allowed-Origins {
      "${distro_id}:${distro_codename}";
      "${distro_id}:${distro_codename}-security";
      "${distro_id}:${distro_codename}-updates";
      "${distro_id}:${distro_codename}-proposed";
      "${distro_id}:${distro_codename}-backports";
};
```

## Package Blacklist

This option lets you specify packages that should be excluded from automatic updates. To exclude packages, uncomment this option and add the desired packages as shown in the example below:

```
Unattended-Upgrade::Package-Blacklist {
      "my-package";
      "my-other-package";
};
```

## AutoFixInterruptedDpkg

This option determines whether the system should automatically fix interrupted “dpkg” installations. To enable automatic fixing, set the value to “true” as shown in the example below:

```
Unattended-Upgrade::AutoFixInterruptedDpkg "true";
```

## DevRelease “auto”

This option specifies whether to upgrade to the development release automatically. By default, this option is disabled. To enable automatic upgrades to the development release, set the value to “auto” as shown in the example below:

```
Unattended-Upgrade::DevRelease "auto";
```

## MinimalSteps “true”

This option specifies whether to perform minimal steps during upgrades. By default, this option is enabled. To disable minimal steps, set the value to “false” as shown in the example below:

```
Unattended-Upgrade::MinimalSteps "false";
```

## InstallOnShutdown “false”

This option specifies whether to install upgrades on shutdown. By default, this option is disabled. To enable installation on shutdown, set the value to “true” as shown in the example below:

```
Unattended-Upgrade::InstallOnShutdown "true";
```
## Mail

This option specifies the email address to which notifications should be sent. By default, notifications are not sent. To specify an email address, add it as a string value as shown in the example below:

```bash
Unattended-Upgrade::Mail "example@mail.com";
```

## MailReport “on-change”

This option specifies when to send email notifications. By default, notifications are sent only when there is a change. To send notifications every time, set the value to “on-start” as shown in the example below:

```
Unattended-Upgrade::MailReport "on-start";
```
## Remove-Unused-Kernel-Packages

This option specifies whether to remove unused kernel packages after an upgrade. By default, this option is enabled. To disable the removal of unused kernel packages, set the value to “false” as shown in the example below:

```
Unattended-Upgrade::Remove-Unused-Kernel-Packages "false";
```

## Remove-New-Unused-Dependencies

This setting controls removing new dependencies the system no longer requires after package upgrades. This setting is enabled by default, and any new dependencies no longer needed will be automatically removed from your system. To enable this setting, use the following command:

```
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";
```

If you prefer to keep these new dependencies, you can disable this setting by changing the value to “false”. However, be aware that this may result in unused dependencies accumulating on your system, taking up valuable disk space.
## Remove-Unused-Dependencies

This option specifies whether to remove unused dependencies after an upgrade. By default, this option is disabled. To enable the removal of unused dependencies, set the value to “true” as shown in the example below:

```
Unattended-Upgrade::Remove-Unused-Dependencies "true";
```
## Automatic-Reboot

This option specifies whether to automatically reboot the system after an upgrade. By default, this option is disabled. To enable automatic reboot, set the value to “true” as shown in the example below:

```
Unattended-Upgrade::Automatic-Reboot "true";
```
## Automatic-Reboot-WithUsers

This option specifies whether to reboot the system when users are logged in automatically. By default, this option is enabled. To disable automatic reboot when users are logged in, set the value to “false” as shown in the example below:

```
Unattended-Upgrade::Automatic-Reboot-WithUsers "false";
```

## Automatic-Reboot-Time

This option specifies the time the system should automatically reboot after an upgrade. By default, this option is set to “02:00”. To change the reboot time, modify the value as shown in the example below:

```
Unattended-Upgrade::Automatic-Reboot-Time "03:00";
```
## Acquire::http::Dl-Limit

This option specifies the maximum download speed for package downloads in kilobytes per second. By default, this option is disabled. To enable download speed limiting, set the value to the desired speed as shown in the example below:

```
Acquire::http::Dl-Limit "100";
```
## SyslogEnable

This option specifies whether to log upgrade events to the system log. By default, this option is enabled. To disable logging, set the value to “false” as shown in the example below:

```
Unattended-Upgrade::SyslogEnable "false";
```
## SyslogFacility

This option specifies the facility to which upgrade events should be logged. By default, events are logged to the “daemon” facility. To log events to a different facility, modify the value as shown in the example below:

```
Unattended-Upgrade::SyslogFacility "local7";
```
## OnlyOnACPower

This option specifies whether to perform upgrades only when the system is connected to AC power. By default, this option is disabled. To enable upgrades only on AC power, set the value to “true” as shown in the example below:

```
Unattended-Upgrade::SyslogFacility "local7";
```
## Skip-Updates-On-Metered-Connections

This option specifies whether to skip updates when the system is connected to a metered connection. By default, this option is enabled. To download updates on metered connections, set the value to “false” as shown in the example below:

```
Unattended-Upgrade::Skip-Updates-On-Metered-Connections "false";
```
## Verbose

This option specifies whether to output verbose upgrade information. By default, this option is disabled. To enable verbose output, set the value to “true” as shown in the example below:

```
Unattended-Upgrade::Verbose "true";
```
## Debug

This option specifies whether to output debug information during upgrades. By default, this option is disabled. To enable debug output, set the value to “true” as shown in the example below:

```
Unattended-Upgrade::Debug "true";
```
## Allow-downgrade

This option specifies whether to allow downgrades of packages during upgrades. By default, this option is disabled. To allow downgrades, set the value to “true” as shown in the example below:

```
Unattended-Upgrade::Allow-downgrade "true";
```

Remember that allowing downgrades can be dangerous and lead to system instability or security vulnerabilities. It is recommended that you enable this option only when necessary and consider the risks involved carefully.
## Testing and Applying Changes

After making any changes to the configuration file, it is important to run the following command to ensure that the changes are applied:

```
sudo unattended-upgrades --dry-run --debug
```

This command will simulate an upgrade run and display any errors or warnings that may occur due to the changes you made to the configuration file. If everything looks good, you can then run the command below to perform the actual upgrade:

```
sudo unattended-upgrades
```

This command will automatically check and install available upgrades based on the configuration settings.

With the above steps, you should better understand how to configure and modify the Unattended-Upgrades package on your Ubuntu system.

# Scheduling Automatic Upgrades with a Cron Job

To schedule automatic unattended upgrades on your Ubuntu system, you can use a cron job. A cron job is a time-based task scheduler in Linux that allows you to run commands or scripts automatically at specified times or intervals.

To create a cron job for unattended-upgrades, follow the steps below:

Open the crontab configuration file by running the following command:

```bash
sudo crontab -e
```

This will open the crontab file in the nano editor. Add the following line to the bottom of the file:

```bash
0 0 * * * /usr/bin/unattended-upgrade -d
```

This line specifies that the unattended-upgrade command should run daily at midnight (0 0 * * *). Save and exit the file by pressing “Ctrl+X”, then “Y”, and then “Enter”.

The above cron job will run the unattended-upgrade command daily at midnight, check for available upgrades, and install them automatically based on the configuration settings.

You should only schedule automatic upgrades when your system is not in use, as upgrades may require a reboot or cause applications to restart.
# Checking Unattended Upgrade Logs

The Unattended-Upgrades package, by default, logs all upgrade activity to the syslog facility. The logs are in the /var/log/syslog file and other system logs.

To check the Unattended-Upgrades logs, you can use the following command:

```bash
sudo grep unattended-upgrades /var/log/syslog
```

This command will search the syslog file for entries containing the “unattended-upgrades” keyword and display them on the screen. You can also use the following command to display the last 50 entries in the syslog file:

```bash
sudo tail -n 50 /var/log/syslog | grep unattended-upgrades
```

This command will display the last 50 entries in the syslog file that contain the “unattended-upgrades” keyword.

In addition to the above commands, you can use various grep options to filter the logs based on specific criteria. For example, to filter the logs by date and time, you can use the following command:

```bash
sudo grep "unattended-upgrades.*YYYY-MM-DD" /var/log/syslog
```

Replace “YYYY-MM-DD” with the desired date in the year-month-day format. This command will display all log entries that contain the “unattended-upgrades” keyword and match the specified date.

You can also use the following command to filter the logs by package name:

```bash
sudo grep "unattended-upgrades.*<package_name>" /var/log/syslog
```

Replace “<package_name>” with the package name you want to search for. This command will display all log entries that contain the “unattended-upgrades” keyword and match the specified package name.

With the above commands, you can easily check and filter the Unattended-Upgrades logs on your Ubuntu system and troubleshoot any issues that may arise during the upgrade process.