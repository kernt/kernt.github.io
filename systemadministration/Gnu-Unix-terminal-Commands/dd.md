`dd if=/dev/sda conv=sync,noerror bs=64K | gzip -c  > _/path/to/backup.img.gz_`

If there is not enough disk space locally, you may send the image through _ssh_

`dd if=/dev/sda conv=sync,noerror bs=64K | gzip -c | ssh user@local dd of=backup.img.gz`

Finally, save extra information about the drive geometry necessary in order to interpret the partition table stored within the image. The most important of which is the sector size

`fdisk -l /dev/sda > _/path/to/list_fdisk.info_`

## retore system

Backup and restore MBR

To save the MBR as `mbr_file.img`

`dd if=/dev/sd_X_ of=_/path/to/mbr_file.img_ bs=512 count=1`

