We went through the [configuration of OpenLDAP](https://raduzaharia.medium.com/building-an-identity-server-with-openldap-and-a-raspberry-pi-4-e19f829dd2eb) on a typical Ubuntu Server Raspberry PI system and last time we added [sudoers](https://raduzaharia.medium.com/adding-sudoers-to-openldap-e0a6b0c4c7ab) to OpenLDAP. All this to have a general [sense of identity](https://raduzaharia.medium.com/managing-user-identities-in-a-network-703094e4835a) in the home network and to help NFS figure out who is the [fair owner](https://raduzaharia.medium.com/thoughts-on-nfs-file-sharing-d1b6d3c22a9) of the network shares.

OpenLDAP is extensible and you can basically store anything you want there, provided you have a schema for the data type you want to add. We saw this last time when we added sudoers: we needed the sudoer schema. The same is true for automounts. And in my searches to achieve ultimate levels of LDAP configuration I even encountered a schema for Gnome dconf settings. That is to apply the stored Gnome settings at login for each user. So yes, things can get quite wild. But nevermind that, let’s see the autofs schema (available on [github](https://github.com/raduzaharia-medium/openldap/blob/main/autofs.schema.ldif)):

```
dn: cn=autofs,cn=schema,cn=config  
objectClass: olcSchemaConfig  
cn: autofsolcAttributeTypes: {0}( 1.3.6.1.1.1.1.25 NAME 'automountInformation' DESC 'Information used by the autofs automounter' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 SINGLE-VALUE )olcObjectClasses: {0}( 1.3.6.1.1.1.1.13 NAME 'automount' DESC 'An entry in an automounter map' SUP top STRUCTURAL MUST ( cn $ automountInformation $ objectclass ) MAY description )olcObjectClasses: {1}( 1.3.6.1.4.1.2312.4.2.2 NAME 'automountMap' DESC 'An group of related automount objects' SUP top STRUCTURAL MUST ou )
```

As for sudoers, you have to place this in `/etc/ldap/schema/autofs.ldif` and we need to register it with OpenLDAP:

```
#ldapadd -D cn=config -H ldapi:/// -w admin-password -f /etc/ldap/schema/autofs.ldif
```

Again, note the `admin-password` in there, the one configured in the `root.ldif` file [here](https://raduzaharia.medium.com/building-an-identity-server-with-openldap-and-a-raspberry-pi-4-e19f829dd2eb). Now we need to create the `shares.ldif` file. Before we do that, we need to take note of our configured NFS shares because that’s what we are trying to automount. You can automount whatever shares you want, SMB or any other kind but for now we will focus on NFS. So we will assume this NFS `/etc/exports` file described [here](https://raduzaharia.medium.com/adding-an-external-ssd-to-your-fedora-server-on-a-raspberry-pi-e60daaba438b) (available on [github](https://github.com/raduzaharia-medium/openldap/blob/main/exports)):

```
/mnt/storage/radu      192.168.68.0/24(rw,all_squash,no_subtree_check,anonuid=1000,anongid=1000)  
/mnt/storage/shared    192.168.68.0/24(rw,all_squash,no_subtree_check,anonuid=1000,anongid=1000)  
/mnt/storage/media     192.168.68.0/24(rw,all_squash,no_subtree_check,anonuid=1000,anongid=1000)
```

So now we need to reference each share in `shares.ldif`(also on [github](https://github.com/raduzaharia-medium/openldap/blob/main/shares.ldif)):

```
version: 1dn: ou=automount,dc=yourdomain,dc=com
objectClass: organizationalUnit
objectClass: top
ou: automountdn: ou=auto.direct,ou=automount,dc=yourdomain,dc=com
objectClass: automountMap
objectClass: top
ou: auto.directdn: ou=auto.master,ou=automount,dc=yourdomain,dc=com
objectClass: automountMap
objectClass: top
ou: auto.masterdn: cn=/-,ou=auto.master,ou=automount,dc=yourdomain,dc=com
objectClass: automount
objectClass: top
automountInformation: ldap:razah.io:ou=auto.direct,ou=automount,dc=yourdomain,dc=com
cn: /-dn: cn=/mnt/storage/radu,ou=auto.direct,ou=automount,dc=yourdomain,dc=com
objectClass: automount
objectClass: top
automountInformation: 192.168.68.11:/radu
cn: /mnt/network-storage/radudn: cn=/mnt/storage/shared,ou=auto.direct,ou=automount,dc=yourdomain,dc=com
objectClass: automount
objectClass: top
automountInformation: 192.168.68.11:/shared
cn: /mnt/network-storage/shareddn: cn=/mnt/storage/media,ou=auto.direct,ou=automount,dc=yourdomain,dc=com
objectClass: automount
objectClass: top
automountInformation: 192.168.68.11:/media
cn: /mnt/network-storage/media
```

Basically the LDAP hierarchy above is like this:

```sh
automount  
   auto.direct  
      /mnt/storage/radu  
      /mnt/storage/media  
      /mnt/storage/shared  
   auto.master  
      /- (this enables auto.direct)
```

Messing up the LDAP hirearchy can cause a lot of headaches. You can check if you get it right by running `automount --dumpmaps`. This will give you the LDAP hirearchy as read and understood by OpenLDAP. You can check here to see if what you have in your mind is the same thing OpenLDAP understands from the `shares.ldif` configuration.

Also, keep in mind to use your domain definition in `shares.ldif`. Where I wrote `dc=yourdomain,dc=com` you need to specify your actual domain. At last we need to restart `slapd` to enable the changes:

```sh
sudo systemctl restart slapd
```
## Client setup

For the client we need to enable OpenLDAP to read automount definitions and we need to install `autofs`, the Linux service which actually performs the automount:

`sudo apt install autofs autofs-ldap`

We also need to configure `sssd` by editing `/etc/sssd/sssd.conf`(also on [github](https://github.com/raduzaharia-medium/openldap/blob/main/sssd.conf)):

```
[domain/yourdomain.com]ldap_autofs_search_base = dc=yourdomain,dc=com  
ldap_autofs_map_object_class = automountMap  
ldap_autofs_entry_object_class = automount  
ldap_autofs_map_name = ou  
ldap_autofs_entry_key = cn  
ldap_autofs_entry_value = automountInformation[autofs]
```

Note in the listing above that we need to add the `[autofs]` defintion and we also need to add the schema description in the `[domain]` entry. Again, mind the `yourdomain.com` part. This needs to be updated to your domain. Now we just need to restart `sssd` and `autofs`to activate the changes:

```sh
sudo systemctl restart sssd  
sudo systemctl restart autofs
```

Log out and back in and you should be able to see in `/mnt/storage` your shares. If you don’t, try to execute `ls /mnt/storage/media` even though you don’t see `/media` in `/mnt/storage`. Automounts are different from permanent mounts in that the mount point gets created only on demand. And listing the content of the mount point initiates the mount mechanism.

What you accomplished here is not an easy task and figuring out the share hierarchy for `shares.ldif` is not easy. But when it works, basically you give all your LDAP users automatic mounts for your desired NAS shares. And they don’t have to do anything to get them. When they log in, they will see the NAS shares in `/mnt/storage` automatically. No need to mount, to ask around the addresses of the shares: nothing. It is a very useful feature and I was glad to find it in OpenLDAP. It simplified my setup a lot.

Having said that, we have arrived at the end of the OpenLDAP configuration articles. The [next one](https://raduzaharia.medium.com/building-an-identity-server-with-freeipa-and-a-raspberry-pi-4-735d108db1fe) will deal with another LDAP system, FreeIPA. FreeIPA is industry grade. It is a lot easier to install and a lot easier to configure. You don’t have to mess around with `ldif` files because it comes with a web admin application. There is a catch though. If OpenLDAP was no burden on your Raspberry PI, FreeIPA will be. But all that and more in the next article. See you soon!