---
tags:
  - acct
  - net-acct
  - nfacct
  - pmacct
---
# acct - process accounting file ac

Packete

- [[acct]]
- [[net-acct]]
- [[nfacct]]
- [[pmacct]]

**Starting psacct or acct service**

```sh
sudo systemctl status psacct
sudo systemctl start psacct
sudo systemctl enable psacct
sudo systemctl status psacct
```

**Display Statistics of Linux Users Day-wise**

```sh
ac -d
```

**Display Total Login Time of All Linux Users**

`ac -p`

**Display Linux User Login Time**

`ac username`

# sa

[ -a | --list-all-names ] 
[ -b | --sort-sys-user-div-calls ]          
[ -c | --percentages ] [ -d | --sort-avio ]    
[ -D | --sort-tio ] [ -f | --not-interactive ]
[ -i | --dont-read-summary-files ]        
[ -j | --print-seconds ] [ -k | --sort-cpu-avmem ]
[ -K | --sort-ksec ] [ -l | --separate-times ]
[ -m | --user-summary ] [ -n | --sort-num-calls ]  
[ -p | --show-paging ] [ -P | --show-paging-avg ]  
[ -r | --reverse-sort ] [ -s | --merge ]
[ -t | --print-ratio ] [ -u | --print-users ]

**Print All Linux Commands Executed by Users**

`sa`

**Print Linux User Information**

`sa -u`

**Print Number of Linux Processes**

`sa -m`

**Print and Sort Usage by Percentage**

`sa -c`


# savacct

`savacct` Eine Zusammenfassung der Systemprozessabrechnung, sortiert nach Befehl.
# usracct

`usracct`   A summary of system process accounting sorted by user ID.

