# /etc/security/access.conf

**the etc/security/access.conf file can create access and deny rules based on users/address pairs**

```
 Allows user foo and members for admins group from the console and the specified IPv6 address and IPv4 subnet
+ : @admins foo : LOCAL 2001:db8:0:101::/64 10.1.1.0/24

# Disallow console logins to all but the shutdown, sync and all other accounts, which are a member of the wheel group.
- : ALL EXCEPT (wheel) shutdown sync : LOCAL

# Block all other users from all other sources
- : ALL : ALL
```