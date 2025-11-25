---
tags:
  - system-administration
  - user
  - posix
---
## POSIX IDs

### The problem(s)

Different systems are created differently, even within the GNU/Linux world. Debian based systems regard all IDs below 1000 to be system IDs, while Red Hat uses all IDs below 500 to be system IDs.

Even more complex is the situation around _nobody_. The user _nobody_ and group _nogroup_ came from the NFS software and was defined as being having the highest ID, since the function was oposite to the _root_. However a 16-bit system has another highest number then a 32-bit system:

- 16-bit systems: 65.536 unique IDs
- 32-bit systems: 4.294.967.296 unique IDs

This resulted in some confusion. To this confusion was added the use of using -2 for the nobody ID, as was done by the software itself if _nobody_ and _nogroup_ where not defined. GNU/Linux distribution creators defined the account as 65534, however Red Hat supplied under that ID _nfsnobody_ with another _nobody_ having ID 99. And there is _nogroup_ usage, but also groups that are called _nobody_.

All in all a rough overview of what is used where can be created like this:

| IDs         | Usage                                                             |
| ----------- | ----------------------------------------------------------------- |
| -2          | nobody on AIX and Mac OS X                                        |
| 0-99        | Unix local users and groups, statically asigned                   |
| 99          | Red Hat based system nobody user and group ID                     |
| 100-499     | Unix local users and groups, dynamic                              |
| 529         | Used as ID for nobody on some systems (and not used by Microsoft) |
| 32767       | Historic reservation for nobody (have not find any use)           |
| 60001       | Nobody on IRIX and SunOS                                          |
| 65530-65535 | Unix nobody user and (no)group (Debian and nfsnobody RHEL)        |
| 4294967292  | Group-owner on Isilon BSD                                         |
| 4294967293  | Null user on Isilon BSD                                           |
| 4294967294  | Everyone on Isilon BSD                                            |
|             |                                                                   |
|             |                                                                   |

### Our configuration suggestion

Different systems are created differently, even within the GNU/Linux world. Debian based systems regard all IDs below 1000 to be system IDs, while Red Hat uses all IDs below 500 to be system IDs.

Even more complex is the situation around _nobody_. The user _nobody_ and group _nogroup_ came from the NFS software and was defined as being having the highest ID, since the function was oposite to the _root_. However a 16-bit system has another highest number then a 32-bit system:

- 16-bit systems: 65.536 unique IDs
- 32-bit systems: 4.294.967.296 unique IDs

This resulted in some confusion. To this confusion was added the use of using -2 for the nobody ID, as was done by the software itself if _nobody_ and _nogroup_ where not defined. GNU/Linux distribution creators defined the account as 65534, however Red Hat supplied under that ID _nfsnobody_ with another _nobody_ having ID 99. And there is _nogroup_ usage, but also groups that are called _nobody_.

All in all a rough overview of what is used where can be created like this:

|IDs|Usage|
|---|---|
|-2|nobody on AIX and Mac OS X|
|0-99|Unix local users and groups, statically asigned|
|99|Red Hat based system nobody user and group ID|
|100-499|Unix local users and groups, dynamic|
|500-999|Microsoft RIDs for LOCAL, DOMAIN and BUILTIN, static|
|529|Used as ID for nobody on some systems (and not used by Microsoft)|
|32767|Historic reservation for nobody (have not find any use)|
|60001|Nobody on IRIX and SunOS|
|65530-65535|Unix nobody user and (no)group (Debian and nfsnobody RHEL)|
|4294967292|Group-owner on Isilon BSD|
|4294967293|Null user on Isilon BSD|
|4294967294|Everyone on Isilon BSD|
|4294967295|Nobody (32-bit)|

On Mac OS X Apple put everything into the DirectoryService which can be accessed by using dscl. By using:

dscl /Local/Default -list /Users UniqueID

The design with the biggest amount of UIDs and GIDs on 32-bit systems and better can be designed like this:

|IDs|Function|
|---|---|
|0-99|Static system UIDs and GIDs|
|100-499|Dynamic system UIDs and GIDs|
|500-999|Reserved for Windows RIDs for LOCAL, DOMAIN and BUILTIN, static|
|1000-65535|Reserved and used for legacy systems|
|7000-4000000000|Dynamic allocation of UIDs and GIDs from LDAP|
|4294967290-4294967295|Reserved|

We will design with the following assumptions: 0-999 is reserved for use by the different systems in the network, and we will not use 60000-65535 to make our Debian and RHEL based machines happy and make extra room to support other OSes if needed. IDs 1000-59999 are reserved for local system users, meaning these are only available to the local system, they are not part of LDAP.

To make your local machines use the right UIDs and GIDs when they create local accounts set in /etc/login.defs:

```sh
SYS_UID_MIN    100
SYS_UID_MAX    499

SYS_GID_MIN    100
SYS_GID_MAX    499

UID_MIN      1000
UID_MAX      59999

GID_MIN      1000
GID_MAX      59999

```
On Debian based systems also adjust /etc/adduser.conf:

```sh
FIRST_SYSTEM_UID=100
LAST_SYSTEM_UID=499

FIRST_SYSTEM_GID=100
LAST_SYSTEM_GID=499

FIRST_UID=1000
LAST_UID=59999

FIRST_GID=1000
LAST_GID=59999
```

### User Personal Groups (UPG)

If you would like to work with Personal User Groups on your POSIX systems, I would suggest using:

- User account: name - uidnumber
- Group naming: name_group - uidnumber+1

This means you need two numbers for every entry. But it gives you the benefit that you can use domainSID-uidNumber or domainSID-gidNumber as the SID for the object. But in the end this is only cosmetics.

## Windows SIDs

Before we add users and groups to LDAP, we first need to explain the SID and RID used in the Microsoft environment. SID stands for Security IDentifier. Within an Microsoft networking environment the SID is globally unique. In comparision with Unix-like systems, you could create a group with gid 99 and a user with uid 99, meaning that on a system level both have an ID of 99. This is not possible in a Microsoft world. It should also be noted that you can not have a group with name "test" and a user called "test". Also the naming has to be unique within your domain.

RID is a Relative IDentifier. Relative to the SID that is. The RID is the last part and should be unique for a certain object within a domain.

The structure of the SID looks like this:

S-[Revision]-[IdentifierAuthority]-[SubAuthority0]-[SubAuthority1]-...-[SubAuthority[SubAuthorityCount]](-RID).

The components in this structure are:

Revision

The revision is always 1 for current NT versions.

Identifier Authorities and SubAuthorities

|SID|RID|Description|
|---|---|---|
|S-1|0|NULL SID authority: used to hold the "null" account SID|
|S-1-0|0|The null account|
|S-1|1|World SID authority: used for the "Everyone" group, which is the only account in this authority.|
|S-1-1|0|The Everyone group (\EVERYONE)|
|S-1|2|Local SID authority: used for the "Local" group, which is the only account in this group.|
|S-1-2|0|The Local group|
|S-1|3|Creator SID authority: responsible for the CREATOR_OWNER, CREATOR_GROUP, CREATOR_OWNER_SERVER and CREATOR_GROUP_SERVER well known SIDs. These SIDs are used as placeholders in an access control list (ACL) and are replaced by the user, group, and machine SIDs of the security principal.|
|S-1-3|0|Creator Owner account (\CREATOR OWNER)|
|S-1-3|1|Creator Group account (\CREATOR GROUP)|
|S-1-3|2|Creator Owner Server account (\CREATOR SERVER OWNER)|
|S-1-3|3|Creator Group Server account (\CREATOR SERVER GROUP)|
|S-1|4|Non-unique authority: Not used by NT|
|S-1|5|NT authority: accounts that are managed by the NT security subsystem.|
|S-1-5|2|NT authority: Network (AUTHORITY\NETWORK)|
|S-1-5|4|NT authority: Interactive (AUTHORITY\INTERACTIVE)|
|S-1-5|11|NT authority: Authenicated users (AUTHORITY\AUTHENTICATED USERS)|
|S-1-5|18|NT authority: System (AUTHORITY\SYSTEM)|
|S-1-5|19|NT authority: Local service (AUTHORITY\LOCAL SERVICE)|
|S-1-5|20|NT authority: Network service (AUTHORITY\NETWORK SERVICE)|
|S-1-5|21|Non-unique SIDs, used for domain SIDs: The SID S-1-5-21 is followed by 3 RIDs (96 bytes) that defines the domain. Which could look like this S-1-5-21-0123456789-0123456789-0123456789. The 3 RIDs are created during initial domain installation. Since it is a random number duplicates can exist, there is no such thing as a central domain number authority. The domain SID is followed by a RID identifying the account within the domain. This RID is just a simple counter assigning a new RID to an account. There are however a couple well known RIDs:<br><br>\|RID\|Name\|Type\|<br>\|---\|---\|---\|<br>\|500\|DOMAINNAME\Administrator\|User\|<br>\|501\|DOMAINNAME\Guest\|User\|<br>\|512\|DOMAINNAME\Domain Admins\|Group\|<br>\|513\|DOMAINNAME\Domain Users\|Group\|<br>\|514\|DOMAINNAME\Domain Guests\|Group\||
|S-1-5|32|Builtin resources:<br><br>\|RID\|Name\|Type\|<br>\|---\|---\|---\|<br>\|544\|BUILTIN\Administrators\|Group\|<br>\|545\|BUILTIN\Users\|Group\|<br>\|546\|BUILTIN\Groups\|Group\|<br>\|548\|BUILTIN\Account Operators\|Group\|<br>\|549\|BUILTIN\Server Operators\|Group\|<br>\|550\|BUILTIN\Print Operators\|Group\|<br>\|551\|BUILTIN\Backup Operators\|Group\|<br>\|552\|BUILTIN\Replicator\|Group\||
|S-1|9|Resource manager authority: is a catch-all that is used for 3rd party resource managers.|

## SAMBA UID/GID and SID Mapping

The idmap ranges for UID and GID are applicable only for local accounts. They should not be used at all when AD accounts are mapped into the UNIX UID/GID name space, when SAMBA is tied to LDAP it might depend on your setup and / or configuration if you need the idmap configuration parameters.

Mixing local accounts and AD accounts within the same name space, although this works, can confuse management of UNIX file system permissions as seen from a Windows client. **Strong advise:** do not use idmap entries when you tie SAMBA to AD.

### The Guest account

Make sure the Windows _Guest_ account is mapped to the Unix _nobody_ account by using the following in the **[global]** section of your smb.conf file:

	Guest account = nobody


[UisGidRid](http://pig.made-it.com/uidgid.html)

