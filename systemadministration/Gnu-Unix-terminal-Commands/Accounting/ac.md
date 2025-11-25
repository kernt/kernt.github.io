prints statistics about users' connect time. `ac` can tell you how long a particular user or group of users were connected to your system, printing totals by day or for all of the entries in the `wtmp` file.
### Display Statistics of Users Connect Time

**ac** command without specifying any argument will display total statistics of connect time in hours based on the user logins/logouts from the current **wtmp** file.

```sh
ac
```
### Display Statistics of Linux Users Day-wise

Using the command “**ac -d**” will print out the total login time in hours by day-wise.

```sh
ac -d
```
### Display Total Login Time of All Linux Users

Using the command “**ac -p**” will print the total login time of each Linux user in hours.

```sh
ac -p
```
### Display Linux User Login Time

To get the total login statistics time of user “**tecmint**” in hours, use the command as.

```sh
ac tecmint
```

