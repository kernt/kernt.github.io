

Danke euch für die Hinweise. Nun bin ich ein klein bisschen weiter. mal geht es so:

`parted -s /dev/sda "resizepart 2 100% "`

Mal geht es so:

`parted -s /dev/sda "resizepart 2 Yes 100%"`

Das Zweite muss man verwenden, wenn noch der Hinweis kommt, dass die Partition in Verwendung ist.
Leider habe ich noch nicht so ganz begriffen, wann diese Nachfrage kommt, jedenfalls nicht immer.
Das passiert nicht unbedingt bei eingehängten Partitionen.

Einrichtung für LVM

```sh
parted --script /dev/sdb "mklabel gpt"
parted --script /dev/sdb "mkpart 'Linux LVM' 0% 100%"
parted --script /dev/sdb "set 1 lvm on"
```