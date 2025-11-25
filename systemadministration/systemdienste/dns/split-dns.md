---
tags:
  - bind
  - named
  - dns
---
# Bind Split DNS

## Two-in-one DNS server mit BIND9

*/etc/bind/named.conf.local*

```sh
zone "tuxsupport.de" {
    type master;
    file "/etc/bind/db.tuxsupport.de";
};
```

 */etc/bind/db.tuxsupport.de*

```sh
; tuxsupport.de
$TTL    604800
@       IN      SOA     ns1.tuxsupport.de. root.tuxsupport.de. (
                     2006020201 ; Serial
                         604800 ; Refresh
                          86400 ; Retry
                        2419200 ; Expire
                         604800); Negative Cache TTL
;
@       IN      NS      ns1
        IN      MX      10 mail
        IN      A       192.0.2.1
ns1     IN      A       192.0.2.1
mail    IN      A       192.0.2.128 ; We have our mail server somewhere else.
www     IN      A       192.0.2.1
client1 IN      A       192.0.2.201 ; We connect to client1 very often.
```

*/etc/bind/named.conf.local*

```sh
acl slaves {
    195.234.42.0/24;    // XName
    193.218.105.144/28; // XName
    193.24.212.232/29;  // XName
};
```

und wir ändern die zone declaration

```sh
zone "tuxsupport.de" {
    type master;
    file "/etc/bind/db.tuxsupport.de";
    allow-transfer { slaves; };
};
```
## Internals und externals

Auf */etc/bind/named.conf.local* wir add fügen folgende definition hinzufügen ( ganz open oder unter die definition der slaves)

```sh
acl internals {
    127.0.0.0/8;
    10.0.0.0/24;
};
```

We will use a new feature of BIND9 called views. A view let's put a piece of configuration inside a conditional that can depend on a number of things, in this case we'll just depend on internals. We replace the zone declaration at */etc/bind/named.conf.local* with:

```sh
view "internal" {
    match-clients { internals; };
    zone "tuxsupport.de" {
        type master;
        file "/etc/bind/internals/db.tuxsupport.de";
    };
};
view "external" {
    match-clients { any; };
    zone "tuxsupport.de" {
        type master;
        file "/etc/bind/externals/db.tuxsupport.de";
        allow-transfer { slaves; };
    };
};
```

The match clients configuration directive allow us to conditionally show that view based on a set of IPs, "any" stands for any IP. Internal IPs will be cached by the internal view and the rest will be dropped on the external view. The outside world can't see the internal view, and that includes XName, our secondary DNS provider, but we removed the allow-transfer from the internal view since we don't want anyone to be able to transfer under any circumstances the contents of the internal view.

We also changed the path, we will have to create the directory */etc/bind/externals* and */etc/bind/internals* and move */etc/bind/db.tuxsupport.de* to */etc/bind/externals/*.

On */etc/bind/internals/db.tuxsupport.de* we put a zone file similar to the counterpart on external but holding the internal IPs:

```sh
; tuxsupport.de
$TTL    604800
@       IN      SOA     ns1.tuxsupport.de. root.tuxsupport.de. (
                     2006020201 ; Serial
                         604800 ; Refresh
                          86400 ; Retry
                        2419200 ; Expire
                         604800); Negative Cache TTL
;
@       IN      A       10.0.0.1
boss    IN      A       10.0.0.100
printer IN      A       10.0.0.101
scrtry  IN      A       10.0.0.102
sip01   IN      A       10.0.0.201
lab     IN      A       10.0.0.103
```

Great, we can now ping our boss' computer with

`ping boss.tuxsupport.de`

but trying to reach mail.tuxsupport.de will disappoint us, what happened ? There's no reference to mail.tuxsupport.de on the internal zone file and since we are in the internal network we can resolve *mail.tuxsupport.de*. Fine, let's just copy the contents of the external zone file to the internal zone file. That'll work.

But we are a small, smart start up, we can do better than copy-paste each modification to the zone file, furthermore, that is very error prone (will you always remember to modify the internal zone file when you modify the external one, or will you forget and spend some days debugging network problems ?).

What we will do is include the external zone file in the internal file this way

```sh
$include "/etc/bind/external/db.tuxsupport.de"
@       IN      A       10.0.0.1
boss    IN      A       10.0.0.100
printer IN      A       10.0.0.101
scrtry  IN      A       10.0.0.102
sip01   IN      A       10.0.0.201
lab     IN      A       10.0.0.103
```

and voila! Finding the $include directive in the current documentation was hard.. Just remember to change the serial of the external zone file whenever you change the internal one, but it is not a big deal, if you forget, nothing bad will happen since you are not likely to have caching servers inside your own small network.

### Security

It is not recommended to use the same DNS server as primary for some domain (in our case tuxsupport.de) and as caching DNS server, but in our case we are forced to do it. From the outside world 192.0.2.1 is the primary DNS server for tuxsupport.de, from our own internal network, 192.0.2.1 (or its private address, 10.0.0.1) is our caching name server that should be configured as the nameserver to use on each workstation we have.

One of the problems is that someone might start using our caching nameserver from the outside, there's an attack called cache-poisoning and many other nasty things that can be done and are documented on [SINS] (including how to avoid them).

To improve our security a bit, we'll add a couple of directives to the configuration file

```sh
view "internal" {
    match-clients { internals; };
    recursion yes;
    zone "tuxsupport.de" {
        type master;
        file "/etc/bind/internals/db.tuxsupport.de";
    };
};
view "external" {
    match-clients { any; };
    recursion no;

    zone "tuxsupport.de" {
        type master;
        file "/etc/bind/externals/db.tuxsupport.de";
        allow-transfer { slaves; };
    };
};
```

That will prevent anyone on the dangerous Internet to use our server recursively while we, on our own network, can still do it.

### Configuration files

*/etc/bind/named.conf.local*

```sh
acl slaves {
    195.234.42.0/24;    // XName
    193.218.105.144/28; // XName
    193.24.212.232/29;  // XName
};

acl internals {
    127.0.0.0/8;
    10.0.0.0/24;
};

view "internal" {
    match-clients { internals; };
    recursion yes;
    zone "tuxsupport.de" {
        type master;
        file "/etc/bind/internals/db.tuxsupport.de";
    };
};
view "external" {
    match-clients { any; };
    recursion no;
    zone "tuxsupport.de" {
        type master;
        file "/etc/bind/externals/db.tuxsupport.de";
        allow-transfer { slaves; };
    };
};
```

*/etc/bind/externals/db.tuxsupport.de*

```sh
; tuxsupport.de
$TTL    604800
@       IN      SOA     ns1.tuxsupport.de. root.tuxsupport.de. (
                     2006020201 ; Serial
                         604800 ; Refresh
                          86400 ; Retry
                        2419200 ; Expire
                         604800); Negative Cache TTL
;
@       IN      NS      ns1
        IN      MX      10 mail
        IN      A       192.0.2.1
ns1     IN      A       192.0.2.1
mail    IN      A       192.0.2.128 ; We have our mail server somewhere else.
www     IN      A       192.0.2.1
client1 IN      A       192.0.2.201 ; We connect to client1 very often.
```

*/etc/bind/internals/db.tuxsupport.de*

```s
$include "/etc/bind/external/db.tuxsupport.de"
@       IN      A       10.0.0.1
boss    IN      A       10.0.0.100
printer IN      A       10.0.0.101
scrtry  IN      A       10.0.0.102
sip01   IN      A       10.0.0.201
lab     IN      A       10.0.0.103
```

## split-horizon-dns-masterslave-with-bind

*/etc/named.conf* on the master

```c
options {
        listen-on port 53 { 127.0.0.1; 192.168.202.101;};
        [[listen-on-v6]] port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { any;};
        recursion no;
        dnssec-enable no;
        dnssec-validation no;
        dnssec-lookaside auto;
        bindkeys-file "/etc/named.iscdlv.key";
        managed-keys-directory "/var/named/dynamic";
        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
        notify yes;
        also-notify { 192.168.202.102; };
        allow-transfer { 127.0.0.1; 192.168.202.102; };
};
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
acl internal-acl {
   192.168.202.0/24;
};
// INTERNAL
view "internal-view" {
        match-clients {internal-acl; };
        include "/etc/named.internal.zones";
        include "/etc/named.common.zones";
};
// EXTERNAL
view "external-view" {
        match-clients { any; };
        include "/etc/named.external.zones";
        include "/etc/named.common.zones";
};
```

The basic options and logging remain as they were. The rest of the configuration is changed.

27-29: contains an ACL. Here you can list the host or subnets that are matched by that ACL named internal-acl
31-35: contains the view called internal-view and it matches the ACL internal-acl. So hosts that are in the subnet 192.168.202.0/24 will end up in this view
37-41: contains the view called external-view and it matches all hosts that weren’t matched before. So hosts that are not valid for the internal-acl will end up here.

### Zone configuration

As you can see, the zone configuration is excluded from named.conf so we can re-use the common zone definitions (in /etc/named.common.zones) for both views. A restriction of using views is that all zones must be part of one or more views.

As mentioned earlier, we want the zone blaat.test to be different for the external and internal view so we need to define this zone twice.

The internal zone definitions are made in /etc/named.internal.zones

```c
zone "blaat.test" {
        type master;
        file "/var/named/data/db_internal.blaat.test";
        check-names fail;
        allow-update { none; };
        allow-query { any; };
};
```

The external zone definitions are made in /etc/named.external.zones

```c
zone "blaat.test" {
        type master;
        file "/var/named/data/db_external.blaat.test";
        check-names fail;
        allow-update { none; };
        allow-query { any; };
};
```

Finally, the common zone definitions are made in /etc/named.common.zones

```c
zone "miauw.test" {
        type master;
        file "/var/named/data/db.miauw.test";
        check-names fail;
        allow-update { none; };
        allow-query { any; };
};
zone "." IN {
        type hint;
        file "named.ca";
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

When looking at the zones defined in named.internal.zones and named.external.zones, you can see that both files contain the same zone configuration except for the files that contains the zone data

/var/named/data/db_internal.blaat.test

```c
@       IN SOA  ns.blaat.test admin.blaat.test. (
                                2014082202      ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
@                       NS      ns.blaat.test.
ns                      IN      A               192.168.202.101
blaat.test.             IN      A               192.168.202.1
host1.blaat.test.       IN      A               192.168.202.10
host2.blaat.test.       IN      A               192.168.202.20
```

/var/named/data/db_external.blaat.test

```c
@       IN SOA  ns.blaat.test admin.blaat.test. (
                                2014082202      ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
@                       NS      ns.blaat.test.
ns                      IN      A               192.168.202.101
blaat.test.             IN      A               10.10.10.1
host1.blaat.test.       IN      A               10.10.10.10
host2.blaat.test.       IN      A               10.10.10.20
```

The common zone miauw.test remains unchanged.

After changing all of the above files, reload the changes

`sudo systemctl reload named`

Test to see if the server replies different when the request originates from a source within the specified subnet or from outside the subnet

```sh
nslookup host1.blaat.test localhost
Server: localhost
Address: 127.0.0.1#53
Name: host1.blaat.test
Address: 10.10.10.10
```

```sh
nslookup host1.blaat.test 192.168.202.101
Server: 192.168.202.101
Address: 192.168.202.101#53
Name: host1.blaat.test
Address: 192.168.202.10
```

As you can see in the example, the server gives a different answer for a query that originated to localhost (so using 127.0.0.1 as source) or it’s real IP (so using 192.168.202.101 as source) which matches the internal-acl.

As a last test, we can check if the common zone is known from within both views and that the answer is equal

```sh
nslookup host1.miauw.test localhost
Server: localhost
Address: 127.0.0.1#53
Name: host1.miauw.test
Address: 192.168.202.10
```

```sh
nslookup host1.miauw.test 192.168.202.101
Server: 192.168.202.101
Address: 192.168.202.101#53
Name: host1.miauw.test
Address: 192.168.202.10
```

The next step: add a slave for the split horizon master

Until now, the changes involved in comparison with a regular setup are not very complicated. It’s only at the moment when a slave comes into the picture that it’s getting (a little) more complicated.

We need to make sure that, in the slave’s configuration, the same zone get’s transferred twice. Once for each view. Since the zone’s name is equal for both views, it can easily be confused at the slave level because zone transfers are not aware of views. When we wouldn’t take any measures, the last updated zone, regardless of which view it was updated for, would overwrite the zone data of both views on the slave.

To resolve this problem, we will need to create the same views on the slave and restrict the zone transfer to the slave for each of those views. There are multiple ways to do this but for this example, I will use TSIG (Transaction SIGnatures). The key used for the zone-transfer will be different for each view ensuring that the correct zone+view get’s transferred to the same one on the slave.

The first step is to generate two keys for TSIG, one for the internal-view transfers and the other for the external-view transfers. For that, you can use the dnssec-keygen

```sh
dnssec-keygen -a HMAC-MD5 -n HOST -b 128 internal
Kinternal.+157+64609
dnssec-keygen -a HMAC-MD5 -n HOST -b 128 external
Kexternal.+157+25576
```

The keygen generates two files, a.key and a .private. We only need they key which is generated in the .key file

`ls -l K*`

Now that we have our keys, we can start adjusting our /etc/named.conf on the master to restrict zone-transfers to the slave, depending on the key.

```c
options {
        listen-on port 53 { 127.0.0.1; 192.168.202.101;};
        [[listen-on-v6]] port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { any;};
        recursion no;
        dnssec-enable no;
        dnssec-validation no;
        dnssec-lookaside auto;
        bindkeys-file "/etc/named.iscdlv.key";
        managed-keys-directory "/var/named/dynamic";
        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
        notify yes;
        also-notify { 192.168.202.102; };
        allow-transfer { 127.0.0.1; 192.168.202.102; };
};
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
acl internal-acl {
   192.168.202.0/24;
};

key "external-key" {
        algorithm hmac-md5;
        secret "DLYrQqPB6ZJMCO/yYQP7/w==";
};

key "internal-key" {
        algorithm hmac-md5;
        secret "j10uJPBhPhhmmDhUwZmqQg==";
};

// INTERNAL
view "internal-view" {
        match-clients { key internal-key; !key external-key; internal-acl; };
        server 192.168.202.102 { keys internal-key; };
        include "/etc/named.internal.zones";
        include "/etc/named.common.zones";
};
// EXTERNAL
view "external-view" {
        match-clients { key external-key; !key internal-key;  any; };
        server 192.168.202.102 { keys external-key; };
        include "/etc/named.external.zones";
        include "/etc/named.common.zones";
};
```

Explanation of the changes:

31-34: key definition of key named external-key for the external-view transfers
36-39: key definition of key named internal-key for the internal-view transfers
43: the internal-key matches the internal-view (en the external-key doesn’t match)
44: matches the slave-server to the internal-key
50: the external-key matches the external-view (en the internal-key doesn’t match)
51: matches the slave-server to the external-key

On the slave, we need to make similar changes as we first did to our master to make it view-aware plus the changes involved for doing the correct zone-transfer. The changes of the slave are made on the configuration which was explained in a previous post about master/slave DNS.

First, we’ll change the /etc/named.conf of the slave

```c
options {
        listen-on port 53 { 127.0.0.1; 192.168.202.102;};
        [[listen-on-v6]] port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { any;};
        recursion no;
        dnssec-enable no;
        dnssec-validation no;
        dnssec-lookaside auto;
        bindkeys-file "/etc/named.iscdlv.key";
        managed-keys-directory "/var/named/dynamic";
        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
acl internal-acl {
   192.168.202.0/24;
};

key "external-key" {
        algorithm hmac-md5;
        secret "DLYrQqPB6ZJMCO/yYQP7/w==";
};

key "internal-key" {
        algorithm hmac-md5;
        secret "j10uJPBhPhhmmDhUwZmqQg==";
};

// INTERNAL
view "internal-view" {
        match-clients { key internal-key; !key external-key; internal-acl; };
        server 192.168.202.101 { keys internal-key; };
        include "/etc/named.internal.zones";
        include "/etc/named.common.zones";
};
// EXTERNAL
view "external-view" {
        match-clients { key external-key; !key internal-key;  any; };
        server 192.168.202.101 { keys external-key; };
        include "/etc/named.external.zones";
        include "/etc/named.common.zones";
};
```

The only difference between the slave and master’s configuration, besides the standard options, is the IP-address of the server-statement in both views. The real zone defintions are made in the included files (named.external.zones, named.internal.zones & named.common.zones). Those files need to be created on the slave

**/etc/named.internal.zones**

```c
zone "blaat.test" {
        type slave;
        file "/var/named/data/db_internal.blaat.test";
        masters { 192.168.202.101; };
        check-names fail;
        allow-update { none; };
        allow-query { any; };
};
```

**/etc/named.external.zones**

```c
zone "blaat.test" {
        type slave;
        file "/var/named/data/db_external.blaat.test";
        masters { 192.168.202.101; };
        check-names fail;
        allow-update { none; };
        allow-query { any; };
};
```

**/etc/named.common.zones**

```c
zone "miauw.test" {
        type slave;
        file "/var/named/data/db.miauw.test";
        masters { 192.168.202.101; };
        check-names fail;
        allow-update { none; };
        allow-query { any; };
};
zone "." IN {
        type hint;
        file "named.ca";
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

After changing the configuration on both the slave and master, we can reload the configuration to make the changes active. To prevent incorrect zone transfers, it’s better to first stop the slave.

`sudo systemctl stop named`
`sudo systemctl reload named`
`sudo systemctl start named`

After reload the configuration of the master and restarting the slave, the /var/named/data/-directory, where we chose to store our zone-data on the slave should contain some data, transferred from the master

```c
 sudo ls -l /var/named/data
total 20
-rw-r--r--. 1 named named 338 Aug 25 11:32 db_external.blaat.test
-rw-r--r--. 1 named named 338 Aug 25 11:32 db_internal.blaat.test
-rw-r--r--. 1 named named 338 Aug 25 11:32 db.miauw.test
-rw-r--r--. 1 named named 7315 Aug 25 11:32 named.run
```

Since we have data here, the transfer between the master and slave is working fine. This should also be visible in /var/named/data/named.run. To test of the split horizon configuration works on the slave too, we can test it.

```sh
nslookup host1.blaat.test localhost
Server: localhost
Address: 127.0.0.1#53
Name: host1.blaat.test
Address: 10.10.10.10
```

```sh
nslookup host1.blaat.test 192.168.202.102
Server: 192.168.202.102
Address: 192.168.202.102#53
Name: host1.blaat.test
Address: 192.168.202.10
```

In case the transfer wouldn’t initiate correctly or the data isn’t correct while you are sure that your configuration is, you can force a retransfer of the zones with the follow commands

```sh
sudo rndc retransfer blaat.test IN internal-view
sudo rndc retransfer blaat.test IN external-view
```

Be sure to check the ip-addresses of the master and slave in the /etc/named.conf on both the master and slave (they are different) if the zone transfer doesn’t work as expected.

After following this (rather long) example, creating a split horizon DNS with master and slave should be a piece of cake :)

## Quellen

* [two_in_one_dns_bind9_views](https://www.howtoforge.com/two_in_one_dns_bind9_views)
* [split-horizon-dns-masterslave-with-bind](http://jensd.be/160/linux/split-horizon-dns-masterslave-with-bind)
* [is-it-possible-to-split-a-domain-using-dns-bind9](https://serverfault.com/questions/730919/is-it-possible-to-split-a-domain-using-dns-bind9)
* [linux-unix-bind9-named-configure-views](https://www.cyberciti.biz/faq/linux-unix-bind9-named-configure-views/)