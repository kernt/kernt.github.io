
**Server states**

```shell
nfsstat -s
```

**Client Traffic**

```shell
nfsstat -c
```

https://www.redhat.com/sysadmin/using-nfsstat-nfsiostat

Die erste Unit endet auf .mount, die zweite auf .automount. 

.mount

```
[Unit]   Description=Mount NFS Share
[Mount]   What=192.168.XXX.YY:/Music   Where=/media/ft/nas
Type=nfs
Options=soft,async
```

.automount

```sh
[Unit]
Description=Automount NFS-Share
Requires=NetworkManager.service
After=network-online.target
Wants=network-online.target

[Automount]
Where=/media/ft/nas
TimeoutIdleSec=10min

[Install]
WantedBy=multi-user.target
```

**Ein Skript nach einem NFS mound laufen lassen**

Beispiel Skript /root/bin/nfs-optimiation.sh

```sh
#!/bin/bash
device_number=$(stat -c '%d' /cbz_efs/)
((major = (device_number & 0xFFF00) >> 8))
((minor = (device_number & 0xFF) | ((device_number >> 12) & 0xFFF00)))
_dev="/sys/class/bdi/$major:$minor/read_ahead_kb"
echo "DRVICE: $_dev"
echo "CURRENT VALUE: $(cat $_dev)"
echo "$0 called after mount /cbz_efs/"
echo 15000 > "$_dev"
```

`sudo chmod +x -v /root/bin/nfs-optimiation.sh`

**Test your script using the shellcheck command for errors and caching errors**

`sudo shellcheck /root/bin/nfs-optimiation.sh`

**Mounds anzeigen lassen**

`sudo systemctl list-units --type=mount`

**Create a new systemd unit name after-cbz_efs-mount as follows**

`sudo systemctl edit --force --full after-cbz_efs-mount`

```sh
[Unit]
Description=Script to run after fstab mount for /cbz_efs/
Requires=cbz_efs.mount
After=cbz_efs.mount
RequiresMountsFor=/cbz_efs
 
[Service]
ExecStart=/root/bin/nfs-optimiation.sh
 
[Install]
WantedBy=cbz_efs.mount
```