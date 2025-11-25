So you decided to [manage the users](https://raduzaharia.medium.com/managing-user-identities-in-a-network-703094e4835a) in your network from a central server and you [successfully configured](https://raduzaharia.medium.com/building-an-identity-server-with-openldap-and-a-raspberry-pi-4-e19f829dd2eb) OpenLDAP both on the server and the clients. You are able to login on your laptop using the newly defined users on your [LDAP server](https://raduzaharia.medium.com/building-a-raspberry-pi-home-server-9cab1ba7ab57). This is great progress but you should not stop just now. There is one principal attribute a Linux user has, especially a home user, and you want to control that using OpenLDAP: the ability to execute `sudo` commands.

Setting up users in OpenLDAP was easy enough and setting up sudoers is the same. You need a `ldif` file containing the list of sudoers, each referencing a network user that you previously configured. Let’s go.

First, we need to install the sudoers LDAP schema. Frankly I was surprised to see it’s not installed by default and I also couldn’t figure out a way to install it. I could not find a package or something that would make it available so I simply searched for it online and came up with this (available on [github](https://github.com/raduzaharia-medium/openldap/blob/main/sudoers.schema.ldif)):

```ldif
dn: cn=sudo,cn=schema,cn=config  
objectClass: olcSchemaConfig  
cn: sudoolcAttributeTypes: {0}( 1.3.6.1.4.1.15953.9.1.1 NAME 'sudoUser' DESC 'User(s) who may  run sudo' EQUALITY caseExactIA5Match SUBSTR caseExactIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )olcAttributeTypes: {1}( 1.3.6.1.4.1.15953.9.1.2 NAME 'sudoHost' DESC 'Host(s) who may run sudo' EQUALITY caseExactIA5Match SUBSTR caseExactIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )olcAttributeTypes: {2}( 1.3.6.1.4.1.15953.9.1.3 NAME 'sudoCommand' DESC 'Command(s) to be executed by sudo' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )olcAttributeTypes: {3}( 1.3.6.1.4.1.15953.9.1.4 NAME 'sudoRunAs' DESC 'User(s) impersonated by sudo (deprecated)' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )olcAttributeTypes: {4}( 1.3.6.1.4.1.15953.9.1.5 NAME 'sudoOption' DESC 'Options(s) followed by sudo' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )olcAttributeTypes: {5}( 1.3.6.1.4.1.15953.9.1.6 NAME 'sudoRunAsUser' DESC 'User(s) impersonated by sudo' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )olcAttributeTypes: {6}( 1.3.6.1.4.1.15953.9.1.7 NAME 'sudoRunAsGroup' DESC 'Group(s) impersonated by sudo' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )olcAttributeTypes: {7}( 1.3.6.1.4.1.15953.9.1.8 NAME 'sudoNotBefore' DESC 'Start of time interval for which the entry is valid' EQUALITY generalizedTimeMatch ORDERING generalizedTimeOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.24 )olcAttributeTypes: {8}( 1.3.6.1.4.1.15953.9.1.9 NAME 'sudoNotAfter' DESC 'End of time interval for which the entry is valid' EQUALITY generalizedTimeMatch ORDERING generalizedTimeOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.24 )olcAttributeTypes: {9}( 1.3.6.1.4.1.15953.9.1.10 NAME 'sudoOrder' DESC 'an integer to order the sudoRole entries' EQUALITY integerMatch ORDERING integerOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 )olcObjectClasses: {0}( 1.3.6.1.4.1.15953.9.2.1 NAME 'sudoRole' DESC 'Sudoer Entries' SUP top STRUCTURAL MUST cn MAY ( sudoUser $ sudoHost $ sudoCommand $ sudoRunAs $ sudoRunAsUser $ sudoRunAsGroup $ sudoOption $ sudoOrder $ sudoNotBefore $ sudoNotAfter $ description ) )
```

The above text should be placed on the OpenLDAP server (the [Raspberry PI](https://raduzaharia.medium.com/building-an-identity-server-with-openldap-and-a-raspberry-pi-4-e19f829dd2eb) server we built earlier), in `/etc/ldap/schema/sudoers.ldif`. There: you have the sudoers schema. Isn’t that convenient? Now we just need to register it in OpenLDAP:

```sh
ldapadd -D cn=config -H ldapi:/// -w admin-password -f /etc/ldap/schema/sudoers.ldif
```

Note that the `admin-password` above is the one you configured [previously](https://raduzaharia.medium.com/building-an-identity-server-with-openldap-and-a-raspberry-pi-4-e19f829dd2eb) using the `root.ldif` file.

Now we can create the `sudoers.ldif` file (available on [github](https://github.com/raduzaharia-medium/openldap/blob/main/sudoers.ldif)):

```sh
version: 1dn: ou=sudoers,dc=yourdomain,dc=com  
objectClass: organizationalUnit  
objectClass: top  
ou: sudoersdn: cn=user1,ou=sudoers,dc=yourdomain,dc=com  
objectClass: sudoRole  
objectClass: top  
cn: user1  
sudoCommand: ALL  
sudoHost: ALL  
sudoRunAsUser: ALL  
sudoUser: user1  
sudoOrder: 2dn: cn=defaults,ou=sudoers,dc=yourdomain,dc=com  
objectClass: sudoRole  
objectClass: top  
cn: defaults  
sudoOption: env_reset  
sudoOption: mail_badpass  
sudoOption: secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin  
sudoOrder: 1
```

So in the above listing there are several things happening. First of all you need to replace `dc=yourdomain,dc=com` with your actual configured domain. Look out, it appears several times. Next `cn=user1` needs to be an actual user you want to grant sudo rights defined in OpenLDAP, not a local Linux user. Note that `cn=user1` also shows up twice in the listing. Change it in both places.

If you want to have more users having sudo rights, you simply need to duplicate the middle section of the listing starting from `dn` and ending at `sudoOrder` and increase the `sudoOrder` by one for each new user. And finally, mind the last part of the listing: the `cn=defaults`. It should be the last and have `sudoOrder: 1`. Users should begin at `sudoOrder: 2`.

Now we need to add the LDIF to OpenLDAP:

```sh
ldapadd -x -D cn=admin,dc=yourdomain,dc=com -w ldap-password -f sudoers.ldif
```

Again, mind the `dc=yourdomain,dc=com` and change it to be your actual domain, and also here we have the `ldap-password`, not the `admin-password`, so it’s the general LDAP management password configured previously when you installed OpenLDAP. Ok, the server is done so you should restart the LDAP service:

```sh
sudo systemctl restart slapd
```

## Client setup

In order to read the sudo definitions, the client has to have the `libsss-sudo` package installed:

```sh
sudo apt install libsss-sudo
```

And next we need to add right at the end of `/etc/sssd/sssd.conf`:

`[sudo]`

Done. Restart `sssd` to apply the changes:

`sudo systemctl restart sssd`

Log out and back in and your network user should have sudo rights.

This is a simple procedure which for me was needlessly complicated because I couldn’t find the sudoers schema and I didn’t even know I needed one. And then there was the issue with installing the schema to OpenLDAP which again I did not know how to do. But here we are with the process fully demystified. I hope it will be of help to you if you are trying to configure your network users.

Next time we will talk about setting up [automounts](https://raduzaharia.medium.com/adding-automount-to-openldap-a9dd1356864a) in OpenLDAP, another useful addition to your OpenLDAP server. See you next time and for any questions feel free to use the comments area!