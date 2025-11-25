**Initializing a repository**

`sudo restic init -r /mnt/restic-repo`

**Creating our first backup**

```
sudo restic backup -r /mnt/restic-repo /etc
```

**Excluding files and directories from a snapshot**

`sudo restic backup -r /mnt/restic-repo /etc --exclude *.py`

**Listing the snapshots in a repository**

`sudo restic snapshots -r /mnt/restic-repo`

**Listing the files in a snapshot**

`sudo restic ls -r /mnt/restic-repo a91be944 /etc/xdg`

**Mounting a repository**

`sudo restic mount -r /mnt/restic-repo /media`

**Restoring data**

```sh
sudo restic restore -r /mnt/restic-repo latest --target=/
sudo restic restore -r /mnt/restic-repo latest:/etc/xdg --target=/
sudo restic restore -r /mnt/restic-repo latest --include=*.conf --target=/
```

