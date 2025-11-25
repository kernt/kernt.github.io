---
tags:
  - sicherheit
  - sssd
---
# Introduction to SSSD and Realmd

Starting from Red Hat 7 and CentOS 7, SSSD or ‘System Security Services Daemon’  and realmd have been introduced. SSSD’s main function is to access a remote identity and authentication resource through a common framework that provides caching and offline support to the system. SSSD provides PAM and NSS integration and a database to store local users, as well as core and extended user data retrieved from a central server.

The main reason to transition from Winbind to SSSD is that SSSD can be used for both direct and indirect integration and allows to switch from one integration approach to another without significant migration costs. The most convenient way to configure SSSD or Winbind in order to directly integrate a Linux system with AD is to use the realmd service. Because it allows callers to configure network authentication and domain membership in a standard way. The realmd service automatically discovers information about accessible domains and realms and does not require advanced configuration to join a domain or realm.

The realmd system provides a clear and simple way to discover and join identity domains. It does not connect to the domain itself but configures underlying Linux system services, such as SSSD or Winbind, to connect to the domain.

## Realmd Pam SSSD

Please read through this Windows integration guide from Red Hat if you want more information. This extensive guide contains a lot of useful information about more complex situations.

Realmd / SSSD Use Cases
How to join an Active Directory domain?

```sh
    yum install realmd sssd oddjob oddjob-mkhomedir adcli samba-common samba-common-tools ntpdate ntp
    Configure ntp to prevent time sync issues:
    systemctl enable ntpd.service
    ntpdate ntp.domain
    sysemctl start ntpd.service

    systemctl enable ntpd.service
    ntpdate ntp.domain
    sysemctl start ntpd.service
    Join the server to the domain:
    realm join --user=domainadminuser@domain domain

    Also add the default domain suffix to the sssd configuration file:
    vim /etc/sssd/sssd.conf

    Add the following beneath [sssd]

    default_domain_suffix = domain

    Finally move the computer object to an organizational unit in Active Directory.
```

## How to leave an Active Directory domain?

I saw multiple times that although the computer object was created in Active Directory it was still not possible to login with an ad account. The solution was each time to remove the server from the domain and then just add it back.

`realm leave --user=domainadminuser@domain domain`

How to permit only one Active Directory group to logon

As it can be very useful to only allow one Active Directory group. For example a group with Linux system administrators.

`realm permit -g activedirectorygroup@domain`

 How to give sudo permissions to an Active Directory group

`visudo`
Add
`%adgroup@domain ALL=(ALL) ALL`
Or
`%domain\\adgroup ALL=(ALL) ALL`

Example sssd.conf Configuration

The following is an example sshd.conf configuration file. I’ve seen it happen once that somehow access_provider was set to ad. I haven’t got the chance to play with that setting, as simple worked almost every time for now.

```sh
[sssd]
domains = addomain
config_file_version = 2
services = nss, pam
default_domain_suffix = addomain

[domain/addomain]
ad_domain = addomain
krb5_realm = ADDOMAIN
realmd_tags = manages-system joined-with-samba
cache_credentials = True
id_provider = ad
krb5_store_password_if_offline = True
default_shell = /bin/bash
ldap_id_mapping = True
use_fully_qualified_names = True
fallback_homedir = /home/%u@%d
access_provider = simple
simple_allow_groups = adgroup@addomain

[sssd]
domains = addomain
config_file_version = 2
services = nss, pam
default_domain_suffix = addomain

[domain/addomain]
ad_domain = addomain
krb5_realm = ADDOMAIN
realmd_tags = manages-system joined-with-samba
cache_credentials = True
id_provider = ad
krb5_store_password_if_offline = True
default_shell = /bin/bash
ldap_id_mapping = True
use_fully_qualified_names = True
fallback_homedir = /home/%u@%d
access_provider = simple
simple_allow_groups = adgroup@addomain

```

## Required security permissions in AD

A few months ago, we had a problem where some users were no longer able to authenticate. After an extended search we discovered the reason was a hardening change in permissions on some ou’s in our AD. My colleague Jenne and I discovered that the Linux server computer objects need minimal permissions on the ou which contains the users that want to authenticate on your Linux servers. After testing almost all obvious permissions, we came to the conclusions that the computer objects need “Read remote access information”!

## How to debug SSSD and realmd?

The logfile which contains information about successful or failed login attempts is /var/log/secure. It contains information related to authentication and authorization privileges. For example, sshd logs all the messages there, including unsuccessful login. Be sure to check that logfile if you experience problems logging in with an Active Directory user.
How to clear the SSSD cache?

```sh
sudo systemctl stop sssd
</code>sudo rm -f /var/lib/sss/db/*
sudo systemctl start sssd
```
## Just wanted to add this command which also helped me in one case

`authconfig --updateall`

Sources :

* https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Windows_Integration_Guide/realmd-ad-unenroll.html
* https://outsideit.net/realmd-sssd-ad-authentication/
