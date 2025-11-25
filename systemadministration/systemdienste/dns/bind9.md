---
tags:
  - bind
  - named
  - dns
  - system-administration
---
# Bind9

**/etc/bind/named.conf.local**

```c
zone "root.tuxsupport.de" {
    type master;
    file "/etc/bind/zones/db.root.tuxsupport.de"; # zone file path
    allow-transfer { 10.128.20.12; };             # ns2 private IP address - secondary
};
```

**/etc/bind/db.local.tuxsupport.de**

```c
;; DB.domainname
;; Forwardlookupzone für domainname
;;
$TTL 604800
@  IN    SOA    local.tuxsupport.de. root.tuxsupport.de. (
             2      ; Serial
            8H      ; Refresh
            2H      ; Retry
            4W      ; Expire
            3H )    ; Negative Cache TTL
;
@                              IN      NS      ns.local.tuxsupport.de.
                               IN      NS      ns2.local.tuxsupport.de.
                               IN      MX      10 local.tuxsupport.de.
                               IN      A       192.168.4.14
                               IN      AAAA

r1                            IN    A       192.168.4.1
lp.tuxsupport.de.             IN    A       192.168.4.10

rp1                           IN    A       192.168.4.11
rp2                           IN    A       192.168.4.12
rp3                           IN    A       192.168.4.13
rp4                           IN    A       192.168.4.14

tobkern-desktop.tuxsupport.de.            IN      A      192.168.4.21
tobkern-desktop-win11.tuxsupport.de       IN      A       192.168.4.20

ns                              IN      CNAME   ns
ns2                             IN      CNAME   ns2
imap                            IN      CNAME   rp4
pop3                            IN      CNAME   rp4
ldapmaster                      IN      CNAME   rp4
dhcp                            IN      CNAME   rp4
dns                             IN      CNAME   rp4
ILOCZ1539004J                   IN      CNAME   ilo4
smb                             IN      CNAME   share1
nfs                             IN      CNAME   share1
www                             IN      CNAME   srv1
apache2                         IN      CNAME   rp4
home                            IN      CNAME   rp4
dokuwiki                        IN      CNAME   srv1
kvm                             IN      CNAME   srv1
administration                  IN      CNAME   rp4
nas                             IN      CNAME   share1
san                             IN      CNAME   share1
ntp                             IN      CNAME   r1
krb5                            IN      CNAME   rp4
bind                            IN      CNAME   rp4
wiki                            IN      CNAME   srv1
dev                             IN      CNAME   srv1
ldap                            IN      CNAME   rp4
puppet                          IN      CNAME   srv2
printer                         IN      CNAME   lp
printer1                        IN      CNAME   lp
lp1                             IN      CNAME   lp
kodi                            IN      CNAME   rp3
mysql                           IN      CNAME   srv1
pgsql                           IN      CNAME   srv1
elk                             IN      CNAME   srv2
git                             IN      CNAME   srv2
dashboard                       IN      CNAME   srv2
logstash                        IN      CNAME   srv1
ci                              IN      CNAME   srv1
test                            IN      CNAME   srv1
node1                           IN      CNAME   srv1
node2                           IN      CNAME   srv1
node3                           IN      CNAME   srv1
node4                           IN      CNAME   srv1
node5                           IN      CNAME   srv1
backup                          IN      CNAME   srv1
bacula                          IN      CNAME   srv1
es1                             IN      CNAME   srv1
es2                             IN      CNAME   srv1
ftp                             IN      CNAME   srv2
sftp                            IN      CNAME   srv1
http                            IN      CNAME   srv1
node                            IN      CNAME   srv1
homenet                         IN      CNAME   srv1
kibana                          IN      CNAME   srv1
mailserver                      IN      CNAME    rp4
mail                            IN      CNAME    rp4

_kerberos._tcp                          IN    SRV      0    0      88      ipa
_kerberos._udp                          IN    SRV     10    0      88      ipa
_kerberos-master._tcp                   IN    SRV      0    0      88      ipa
_kerberos-master._udp                   IN    SRV      0    0      88      ipa
_kerberos-adm._tcp                      IN    SRV      0    0     749      ipa
_kpasswd._udp                           IN    SRV      0    0     464      ipa

_domain._tcp.local.tuxsupport.de.               IN      SRV      0   0     53      rp4.local.tuxsupport.de.
_domain._udp.local.tuxsupport.de.               IN      SRV      0   0     53      rp4.local.tuxsupport.de.
_domain._tcp.local.tuxsupport.de.               IN      SRV     10   0     53     r1.local.tuxsupport.de.
_domain._udp.local.tuxsupport.de.               IN      SRV     10   0     53     r1.local.tuxsupport.de.
_tftp._tcp.local.tuxsupport.de.                 IN      SRV     10   0     69      share1.local.tuxsupport.de.
_tftp._udp.local.tuxsupport.de.                 IN      SRV     10   0       69      share1.local.tuxsupport.de.
_tftp._tcp.local.tuxsupport.de.                 IN      SRV     20   0     69     srv1.local.tuxsupport.de.
_tftp._udp.local.tuxsupport.de.                 IN      SRV     20   0     69     srv1.local.tuxsupport.de.
_tftp._tcp.local.tuxsupport.de.                 IN      SRV     30   0     69     rp2.local.tuxsupport.de.
_tftp._udp.local.tuxsupport.de.                 IN      SRV     30   0     69     rp2.local.tuxsupport.de.
_ldap._tcp.local.tuxsupport.de.                 IN      SRV     10   0    389     rp4.local.tuxsupport.de.
_ldaps._tcp.local.tuxsupport.de.                IN      SRV     10   0    636     rp4.local.tuxsupport.de.
_ldap._udp.local.tuxsupport.de.                 IN      SRV     20   0    389     rp4.local.tuxsupport.de.
_ldaps._udp.local.tuxsupport.de.                IN      SRV     20   0    636     rp4.local.tuxsupport.de.
_ipp._tcp.local.tuxsupport.de.                  IN      SRV      0   0    631     lp.local.tuxsupport.de.
_ipp._udp.local.tuxsupport.de.                  IN      SRV      0   0    631     lp.local.tuxsupport.de.
_iscsi._tcp.local.tuxsupport.de.                IN      SRV      0   0    860     share1.local.tuxsupport.de.
_iscsi._udp.local.tuxsupport.de.                IN      SRV      0   0    860     share1.local.tuxsupport.de.
_rsync._tcp.local.tuxsupport.de.                IN      SRV      0   0    873     share1.local.tuxsupport.de.
_rsync._udp.local.tuxsupport.de.                IN      SRV      0   0    873     share1.local.tuxsupport.de.
_tacacs._udp.local.tuxsupport.de                IN      SRV      0   0    49      rp4.local.tuxsupport.de.
_www._tcp.local.tuxsupport.de                   IN      SRV      1    0    49      rp4.local.tuxsupport.de.
_www._udp.local.tuxsupport.de                   IN      SRV      1    0    49      rp4.local.tuxsupport.de.
_www._tcp.local.tuxsupport.de                   IN      SRV      10   0    49      srv2.local.tuxsupport.de.
_www._udp.local.tuxsupport.de                   IN      SRV      10   0    49      srv2.local.tuxsupport.de.
_www._tcp.local.tuxsupport.de                   IN      SRV      30   0    49      tobkern-desktop.local.tuxsupport.de.
_www._udp.local.tuxsupport.de                   IN      SRV      30   0    49      tobkern-desktop.local.tuxsupport.de.
_nfs._tcp.local.tuxsupport.de.                  IN      SRV      0   0    2049     share1.local.tuxsupport.de.
_nfs._udp.local.tuxsupport.de.                  IN      SRV      0   0    2049     share1.local.tuxsupport.de.
_docker._tcp.local.tuxsupport.de.               IN      SRV       10    0   2375     tobkern-desktop.local.tuxsupport.de.
_nfs._tcp.local.tuxsupport.de.                  IN      SRV       10    0   2375     tobkern-desktop.local.tuxsupport.de.
_dockers._tcp.local.tuxsupport.de.              IN      SRV      0    0   2376     srv2.local.tuxsupport.de.
_dockers._tcp.local.tuxsupport.de.              IN      SRV      0    0   2376     tobkern-desktop.local.tuxsupport.de.

_mysql._tcp.local.tuxsupport.de.                  IN      SRV      30    0   3306     srv1.local.tuxsupport.de.
_mysql._udp.local.tuxsupport.de.                  IN      SRV      30    0   3306     srv1.local.tuxsupport.de.

srv1.local.tuxsupport.de.                         IN      A        192.168.4.92
srv2.local.tuxsupport.de.                         IN      A        192.168.4.93
share.local.tuxsupport.de.                        IN      A        192.168.4.94
share1.local.tuxsupport.de.                       IN      A        192.168.4.95
share2.local.tuxsupport.de.                       IN      A        192.168.4.96
share3.local.tuxsupport.de.                       IN      A        192.168.4.97
spacewalk.local.tuxsupport.de.                    IN      A        192.168.4.98
redmine.local.tuxsupport.de.                      IN      A        192.168.4.99
elk-stack.local.tuxsupport.de.                    IN      A        192.168.4.101
icinga2.local.tuxsupport.de.                      IN      A        192.168.4.130
icinga2-node1.local.tuxsupport.de.                IN      A        192.168.4.131
icinga2-node2.local.tuxsupport.de.                IN      A        192.168.4.132
fileserver.local.tuxsupport.de.                   IN      A        192.168.4.133
store1.local.tuxsupport.de.                       IN      A        192.168.4.137
store2.local.tuxsupport.de.                       IN      A        192.168.4.138
store3.local.tuxsupport.de.                       IN      A        192.168.4.139
store.local.tuxsupport.de.                        IN      A        192.168.4.140
foreman.local.tuxsupport.de.                      IN      A        192.168.4.141
ox-app.local.tuxsupport.de.                       IN      A        192.168.4.186
ipa.local.tuxsupport.de.                          IN      A        192.168.4.187
ilo4.local.tuxsupport.de.                         IN      A        192.168.4.197
jenkins.local.tuxsupport.de.                      IN      A        192.168.4.201
pdc.local.tuxsupport.de.                          IN      A        192.168.4.210
openldap.local.tuxsupport.de.                     IN      A        192.168.4.211
openldap2.local.tuxsupport.de.                    IN      A        192.168.4.212
openattic.local.tuxsupport.de.                    IN      A        192.168.4.213
openattic-2.local.tuxsupport.de.                  IN      A        192.168.4.214
packetfence.local.tuxsupport.de.                  IN      A        192.168.4.215
sw1.local.tuxsupport.de.                          IN      A        192.168.4.253
```

**forward zone Konfigiuration testen**

`named-checkzone nyc3.example.com /etc/bind/zones/db.nyc3.example.com`

**PTR zone Konfigiuration testen**

`named-checkzone 128.10.in-addr.arpa /etc/bind/zones/db.10.128`
