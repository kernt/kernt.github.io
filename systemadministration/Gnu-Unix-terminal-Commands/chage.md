https://linuxhandbook.com/chage-command/

To disable password aging / expiration for user foo, type command as follows and set:
Minimum Password Age to 0
Maximum Password Age to 99999
Password Inactive to -1
Account Expiration Date to -1
Interactive mode command:

`chage username`

oder

`chage -I -1 -m 0 -M 99999 -E -1 username`

**Maximales Password alter**

`chage -M -1 mybackup`

**User aging Details**

`chage -l USER` 

`sudo chage -d 2018-12-01 root`

**passwd updates the password itself or defines it if it’s not already created**

`passwd -x -1 baeldung`

