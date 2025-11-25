---
tags:
  - selinux
  - sicherheit
---
# SELinux Basis

**Se**cure **E**nhanced **Linux**

## SELinux Administration

cat /var/log/audit/audit.log | grep AVC | tail -1

**ist of the ports where SELinux allows sshd to listen on**

`semanage port -l | grep ssh`

**Modify the existing SELinux rule and assign that port to SSH instead**

`semanage port -m -t ssh_port_t -p tcp 9999`

**check if the port was correctly assigned**

`semanage port -lC`

**To change the label of /websrv/sites/gabriel/public_html recursively to `httpd_sys_content_t`**

`semanage fcontext -a -t httpd_sys_content_t "/websrv/sites/gabriel/public_html(/.*)?"`

**apply the policy (and make the label change effective immediately)**

`restorecon -R -v /websrv/sites/gabriel/public_html`

## Beispiel an fail2ban

`ls -Z fail2ban.log

`restorecon -v '/srv/fail2ban/'`
`restorecon -v '/srv/fail2ban/fail2ban.log'

## Beispiel mit Apache

**http Proxy**

```sh
setsebool -P httpd_can_network_connect 1
setsebool -P httpd_graceful_shutdown 1
setsebool -P httpd_can_network_relay 1
ausearch -c 'httpd' --raw | audit2allow -M proxy-httpd
ausearch -c 'httpd' --raw | audit2allow -M proxy-httpd
semodule -X 300 -i proxy-httpd.pp
cat /var/log/audit/audit.log | grep AVC | tail -1
```

**Ausgabe:**

```sh
type=AVC msg=audit(1639607216.276:4752751): avc:  denied  { getattr } for  pid=1192239 comm="check_file_mtim" path="/usr/local/bin/k3s" dev="dm-0" ino=33577517 scontext=system_u:system_r:icinga2_t:s0 tcontext=system_u:object_r:container_
runtime_exec_t:s0 tclass=file permissive=0
```

```sh
semodule -X 300 -i my-httpd.pp
semanage port -l | grep ssh
semanage port -m -t ssh_port_t -p tcp 9999
semanage port -lC
semanage port -l | grep ssh
cat /var/log/audit/audit.log | grep AVC | tail -1
restorecon -R -v /websrv/sites/gabriel/public_html
```

## Beispiel mit icinga2

```sh
dev_list_sysfs(icinga2_t)
miscfiles_map_generic_certs(icinga2_t)
```

```sh
audit2allow -M icinga2-files-mtime < /var/log/audit/audit.log
setsebool -P nis_enabled
ausearch -c 'httpd' --raw | audit2allow -M my-httpd
```

# icinga2 - SELinux - Ensure the policy directory exists
 

[how-can-i-make-selinux-allow-access-to-a-file](https://superuser.com/questions/988853/how-can-i-make-selinux-allow-access-to-a-file/988862)
[mandatory-access-control-with-selinux-or-apparmor-linux](https://www.tecmint.com/mandatory-access-control-with-selinux-or-apparmor-linux/)