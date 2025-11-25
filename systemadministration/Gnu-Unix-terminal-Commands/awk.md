**OS Release name Search**

`awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release`

**Prints field #3 of file $filename to stdout**

`awk '{print $3}' $filename`

**Prints fields #1, #5, and #6 of file $filename**

`awk '{print $1 $5 $6}' $filename`

**cat $filename . . . or . . . sed '' $filename**

`awk '{print $0}' $filename`

****