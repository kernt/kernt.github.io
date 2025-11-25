---
tags:
  - cluster
  - pacemaker
  - ha
---
# ha pacemaker logging

# High Availability Pacemaker Cluster Logging

We’re going to configure cluster logging on RHEL 7.

**Corosync Logging Options**

The cluster should be stopped before making changes.

If we take a look at the man page for corosync.conf, we’ll see that it’s possible to configure Corosync to log to the following:

 - to_stderr
 - to_logfile
 - to_syslog

These specify the destination of logging output. Any combination of these options may be specified with valid options being “yes” and “no”. The default is syslog and stderr.

Logging verbosity can be specified by using syslog_priority and logfile_priority.

If the to_logfile directive is set to yes, then the logfile parameter specifies the pathname of the log file.

Example configuration of /etc/corosync/corosync.conf is shown below

```sh
logging {
  to_syslog: yes
  to_logfile: yes
  logfile: /var/log/corosync.log
  syslog_priority: info
  logfile_priority: info
  debug: off
}
```

Note that if we are using to_logfile and want to rotate the log file, we need use logrotate with the option copytruncate, e.g. add the following to /etc/logrotate.d/corosync

```sh
/var/log/corosync.log {
  missingok
  compress
  notifempty
  daily
  rotate 7
  copytruncate
}
```

This will rotate Corosync logs on a daily basis.

It’s necessary to sync the cluster configuration after making changes to corosync.conf

`pcs cluster sync`

If we don’t log to a logfile, we can use the systemd journal to retrieve Corosync logs

`journalctl -lf -u corosync`

Example Corosync log entry is provided below

```sh
grep ERROR /var/log/corosync.log
apache(webserver)[13085]:	2019/01/12_19:39:20 ERROR: (13)Permission denied: AH00072: make_sock: could not bind to address [::]:88 (13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:88 no listening sockets available, shutting down AH00015: Unable to open logs
```

**Pacemaker Logging Options**

By default Pacemaker will inherit the logfile specified in corosync.conf.

We can specify a different log file by editing /etc/sysconfig/pacemaker and changing the following line

`PCMK_logfile=/var/log/pacemaker.log`

Logging verbosity can be changed as well

`PCMK_logpriority=warning`

Please note that according to the man page, a value of “info” will be far too verbose for most installations, and “debug” is almost certain to send you blind.

It’s important to copy the file /etc/sysconfig/pacemaker to all cluster nodes after making changes.

The systemd journal can be used to retrieve Pacemaker logs

`journalctl -lf -u pacemaker`

Example Pacemaker error log entry is provided below

```sh
grep ERROR /var/log/pacemaker.log
apache(webserver)[17659]:	2019/01/12_18:57:12 ERROR: AH00526: Syntax error on line 119 of /etc/httpd/conf/httpd.conf: DocumentRoot must be a directory
```

This entry was posted in High Availability, Linux and tagged CentOS, Corosync, EX436, Pacemaker, RHCA, RHEL. Bookmark the permalink. If you notice any errors, please contact us.

* [high-availability-pacemaker-cluster-logging](https://www.lisenet.com/2019/high-availability-pacemaker-cluster-logging/)