---
tags:
  - cluster
  - dns
  - bind
  - named
---
# ha-dns-server

Configure Master DNS Server

**Installation and Firewall**

```sh
yum install bind bind-utils
systemctl enable named
```

**Configure firewall to allow inbount DNS traffic (we use iptables)**

```sh
iptables -A INPUT -s 10.11.1.0/24 -p tcp -m state --state NEW --dport 53 -j ACCEPT
iptables -A INPUT -s 10.11.1.0/24 -p udp -m state --state NEW --dport 53 -j ACCEPT
```
## Logs Directory

**Configure a custom logs directory**

```
mkdir /var/log/named
chown named:named /var/log/named
chmod 0700 /var/log/name
```
## RNDC Key Configuration

Do automatic rndc configuration, and use an authentication key of 512 bits. Note that the default key name is rndc-key

```sh
rndc-confgen -a -b 512 -r /dev/urandom
wrote key file "/etc/rndc.key"
```

**Harden file ownership and permissions**

```sh
chown root:named /etc/rndc.key
chmod 0640 /etc/rndc.key
```

Master named.conf Configuration and Internal Zones

The content of the master configuration file */etc/named.conf* can be seen below.

Note how the internal zone updates are only allowed for the servers that know the key

```c
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
include "/etc/rndc.key";

# Allow rndc management
controls {
	inet 127.0.0.1 port 953 allow { 127.0.0.1; } keys { "rndc-key"; };
};

# Limit access to local network and homelab LAN
acl "clients" {
	127.0.0.0/8;
	10.11.1.0/24;
};

options {
	listen-on port 53 { 127.0.0.1; 10.11.1.2; }; ## MASTER
	listen-on-v6 { none; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";

	tcp-clients 50;

	# Disable built-in server information zones
	version none;
	hostname none;
	server-id none;

	recursion yes;
	recursive-clients 50;
	allow-recursion { clients; };
	allow-query { clients; };
	allow-transfer { localhost; 10.11.1.3; }; ## SLAVE

	auth-nxdomain no;
	notify no;
	dnssec-enable yes;
	dnssec-validation auto;
	dnssec-lookaside auto;

	bindkeys-file "/etc/named.iscdlv.key";
	managed-keys-directory "/var/named/dynamic";
	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

# Specifications of what to log, and where the log messages are sent
logging {
	channel "common_log" {
		file "/var/log/named/named.log" versions 10 size 5m;
		severity dynamic;
		print-category yes;
		print-severity yes;
		print-time yes;
	};
	category default { "common_log"; };
	category general { "common_log"; };
	category queries { "common_log"; };
	category client { "common_log"; };
	category security { "common_log"; };
	category query-errors { "common_log"; };
	category lame-servers { null; };
};

zone "." IN {
	type hint;
	file "named.ca";
};

# Internal zone definitions
zone "hl.local" {
	type master;
	file "data/db.hl.local";
	allow-update { key rndc-key; };
	notify yes;
};

zone "1.11.10.in-addr.arpa" {
	type master;
	file "data/db.1.11.10";
	allow-update { key rndc-key; };
	notify yes;
};
```

The content of the internal zone file /var/named/data/db.hl.local

```c
$TTL 86400	; 1 day
@			IN SOA	dns1.hl.local. root.hl.local. (
				2018010700 ; Serial
				3600       ; Refresh (1 hour)
				3600       ; Retry (1 hour)
				604800     ; Expire (1 week)
				3600       ; Minimum (1 hour)
)
@		NS	dns1.hl.local.
@		NS	dns2.hl.local.
@		A	10.11.1.2
@		A	10.11.1.3
dns1		A	10.11.1.2
dns2		A	10.11.1.3
admin1		A	10.11.1.2
admin2		A	10.11.1.3
katello		A	10.11.1.4
mikrotik	A	10.11.1.1
pve		A	10.11.1.5
```

The content of the internal reverse zone file /var/named/data/db.1.11.10

```c
$TTL 86400	; 1 day
@			IN SOA	dns1.hl.local. root.hl.local. (
				2018010700 ; Serial
				3600       ; Refresh (1 hour)
				3600       ; Retry (1 hour)
				604800     ; Expire (1 week)
				3600       ; Minimum (1 hour)
)
@		NS	dns1.hl.local.
@		NS	dns2.hl.local.
@		PTR	hl.local.
dns1		A	10.11.1.2
dns2		A	10.11.1.3
2		PTR	dns1.hl.local.
3		PTR	dns2.hl.local.

1		PTR	mikrotik.hl.local.
2		PTR	admin1.hl.local.
3		PTR	admin2.hl.local.
4		PTR	katello.hl.local.
5		PTR	pve.hl.local.
```

Ensure that file ownership is sane and SELinux file context applied.

```
[admin1]# chown named:named /var/named/data/db.hl.local /var/named/data/db.1.11.10
[admin1]# semanage fcontext -a -t named_zone_t /var/named/data/db.hl.local
[admin1]# semanage fcontext -a -t named_zone_t /var/named/data/db.1.11.10
[admin1]# restorecon -Rv /var/named/
```

**Allow named to write master zones**

`setsebool -P named_write_master_zones=1`

**Checks the syntax of the master configuration file**

`named-checkconf /etc/named.conf`

Check zone files

```
named-checkzone hl.local /var/named/data/db.hl.local
zone hl.local/IN: loaded serial 2018010700
OK
```

```
named-checkzone hl.local /var/named/data/db.1.11.10
zone hl.local/IN: loaded serial 2018010700
OK
```

**The content of /etc/resolv.conf can be seen below**

`nameserver 10.11.1.2`

**Restart the service**

`systemctl restart named`

**Display status of the server**

```sh
rndc status
version: 9.9.4-RedHat-9.9.4-51.el7 (version.bind/txt/ch disabled)
CPUs found: 1
worker threads: 1
UDP listeners per interface: 1
number of zones: 103
debug level: 0
xfers running: 0
xfers deferred: 0
soa queries in progress: 0
query logging is ON
recursive clients: 0/0/50
tcp clients: 0/50
server is up and runnin
```

**Dig the zone**

```sh
dig @10.11.1.2 ns hl.local

; <<>> DiG 9.9.4-RedHat-9.9.4-51.el7 <<>> @10.11.1.2 ns hl.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22854
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;hl.local.			IN	NS

;; ANSWER SECTION:
hl.local.		86400	IN	NS	dns2.hl.local.
hl.local.		86400	IN	NS	dns1.hl.local.

;; ADDITIONAL SECTION:
dns1.hl.local.		86400	IN	A	10.11.1.2
dns2.hl.local.		86400	IN	A	10.11.1.3

;; Query time: 0 msec
;; SERVER: 10.11.1.2#53(10.11.1.2)
;; WHEN: Sun Jan 07 16:20:28 GMT 2018
;; MSG SIZE  rcvd: 107
```

**Configure Slave DNS Server**

Installation and Firewall
This part is the same as for the master server. Install packages

```sh
yum install bind bind-utils
systemctl enable named
```

**Configure firewall to allow inbount DNS traffic (we use iptables)**

```sh
iptables -A INPUT -s 10.11.1.0/24 -p tcp -m state --state NEW --dport 53 -j ACCEPT
iptables -A INPUT -s 10.11.1.0/24 -p udp -m state --state NEW --dport 53 -j ACCEPT
```

# Logs Directory

Configure a custom logs directory:

```sh
mkdir /var/log/named
chown named:named /var/log/named
chmod 0700 /var/log/name
```

Slave named.conf Configuration

The content of the slave configuration file /etc/named.conf can be seen below

```c
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

acl "clients" {
	127.0.0.0/8;
	10.11.1.0/24;
};

options {
	listen-on port 53 { 127.0.0.1; 10.11.1.3; }; ## SLAVE
	listen-on-v6 { none; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";

	tcp-clients 50;

	# Disable built-in server information zones
	version none;
	hostname none;
	server-id none;

	recursion yes;
	recursive-clients 50;
	allow-recursion { clients; };
	allow-query { clients; };
	allow-transfer { none };

	auth-nxdomain no;
	notify no;
	dnssec-enable yes;
	dnssec-validation auto;
	dnssec-lookaside auto;

	bindkeys-file "/etc/named.iscdlv.key";
	managed-keys-directory "/var/named/dynamic";
	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

# Specifications of what to log, and where the log messages are sent
logging {
	channel "common_log" {
		file "/var/log/named/named.log" versions 10 size 5m;
		severity dynamic;
		print-category yes;
		print-severity yes;
		print-time yes;
	};
	category default { "common_log"; };
	category general { "common_log"; };
	category queries { "common_log"; };
	category client { "common_log"; };
	category security { "common_log"; };
	category query-errors { "common_log"; };
	category lame-servers { null; };
};

zone "." IN {
	type hint;
	file "named.ca";
};

# Internal zone definitions
zone "hl.local" {
	type slave;
	file "data/db.hl.local";
	masters { 10.11.1.2; };
	allow-notify { 10.11.1.2; };
};

zone "1.11.10.in-addr.arpa" {
	type slave;
	file "data/db.1.11.10";
	masters { 10.11.1.2; };
	allow-notify { 10.11.1.2; };
};
```

Checks the syntax of the slave configuration file.

`named-checkconf /etc/named.conf`

The content of /etc/resolv.conf can be seen below

```sh
nameserver 10.11.1.3
nameserver 10.11.1.2
```

Allow named to write master zones

`setsebool -P named_write_master_zones=1`

Restart the service

`systemctl restart named`

Dig the zone

```sh
[admin2]# dig @10.11.1.3 ns hl.local

; <<>> DiG 9.9.4-RedHat-9.9.4-51.el7 <<>> @10.11.1.3 ns hl.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37134
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;hl.local.			IN	NS

;; ANSWER SECTION:
hl.local.		86400	IN	NS	dns2.hl.local.
hl.local.		86400	IN	NS	dns1.hl.local.

;; ADDITIONAL SECTION:
dns1.hl.local.		86400	IN	A	10.11.1.2
dns2.hl.local.		86400	IN	A	10.11.1.3

;; Query time: 0 msec
;; SERVER: 10.11.1.3#53(10.11.1.3)
;; WHEN: Sun Jan 07 16:20:07 GMT 2018
;; MSG SIZE  rcvd: 107
```


# How to Edit Dynamic DNS

Dynamic DNS editor, nsupdate, is used to make edits on a dynamic DNS without the need to edit zone files and restart the DNS server. Because we have declared a zone dynamic, this is the way that we should be making edits.

For example, to delete all records of any type attached to a domain name, we can do:

```
nsupdate -k /etc/rndc.key
> update delete example.hl.local
> send
> quit
```

Note that rndc won’t allow us to reload a dynamic zone:

```
# rndc reload hl.local
rndc: 'reload' failed: dynamic zone
```

To do that, we need to temporarily stop allowing dynamic updates

`rndc freeze hl.local`

Now we can edit the zone file if required. When done, we can allow dynamic updates again

```
rndc reload hl.local
rndc thaw hl.local
```

This entry was posted in DNS, High Availability, Linux and tagged BIND, CentOS, failover, homelab, nsupdate, rndc. Bookmark the permalink. If you notice any errors, please contact us.

**Source**

* [bind-dns-servers-with-failover-and-dynamic-updates](https://www.lisenet.com/2018/configure-bind-dns-servers-with-failover-and-dynamic-updates-on-centos-7/)