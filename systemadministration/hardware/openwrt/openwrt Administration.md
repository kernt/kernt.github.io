
Manage services using web interface.

1. Navigate to **LuCI → System → Startup**.
2. See the list of all the available services and use buttons to execute actions.
# [Command-line instructions](https://openwrt.org/docs/guide-user/base-system/managing_services#command-line_instructions)

Manage services using command-line interface. Use the “equivalent” column inside [scripts](https://openwrt.org/docs/guide-developer/write-shell-script "docs:guide-developer:write-shell-script"), [hotplug](https://openwrt.org/docs/guide-user/base-system/hotplug "docs:guide-user:base-system:hotplug") or [cron](https://openwrt.org/docs/guide-user/base-system/cron "docs:guide-user:base-system:cron"). Check [syslog](https://openwrt.org/docs/guide-user/base-system/log.essentials "docs:guide-user:base-system:log.essentials") for troubleshooting.

| Command                                 | Equivalent                                  | Description                                                                  |
| --------------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------- |
| `service`                               | `ls /etc/init.d`                            | Print a list of available services.                                          |
| `service <service>`                     | `/etc/init.d/<service>`                     | Print a list of available actions for a service.                             |
| `service <service> <action>`            | `/etc/init.d/<service> <action>`            | Execute that action on a specific service.                                   |
| `service <service> <action> <instance>` | `/etc/init.d/<service> <action> <instance>` | Execute that action on a specific service instance, e.g. OpenVPN connection. |
**Common actions supported by most services.**

| Action    | Description                                          |
| --------- | ---------------------------------------------------- |
| `start`   | Start the service.                                   |
| `stop`    | Stop the service.                                    |
| `restart` | Restart the service.                                 |
| `reload`  | Reload configuration files or restart if that fails. |
| `enable`  | Enable service autostart.                            |
| `disable` | Disable service autostart.                           |
| `enabled` | Check if the service is enabled.                     |
| `running` | Check if the service is running.                     |
| `status`  | Service status.                                      |
| `trace`   | Start with syscall trace.                            |
| `info`    | Dump procd service info.                             |
# PXE tftpd dnsmasq

**Mount USB-Stick reboot persistent**

```sh
uci set fstab.@mount[0].label=tftp
uci set fstab.@mount[0].target=/mnt/usb
uci set fstab.@mount[0].options=noatime,nodiratime
uci commit

block mount
```



```
uci set dhcp.@dnsmasq[0].enable_tftp=1
uci set dhcp.@dnsmasq[0].tftp_root=/mnt/usb/
uci set dhcp.@dnsmasq[0].dhcp_boot=netboot.xyz.kpxe
uci commit
/etc/init.d/dnsmasq restart
```


## Wichtige Dienste

- rclone
- nnn
- redis-server
- dnsmasq
- freeadius
- restic
- dovecot
- postfix
- fail2ban
- 

# 

/usr/lib/acme/acme.sh

https://forum.openwrt.org/t/letsencrypt-acme-sh-and-luci-app-acme-support-topic/196821/2