partclone.btrfs
partclone.ext2, partclone.ext3, partclone.ext4
partclone.fat32, partclone.fat12, partclone.fat16
partclone.ntfs
partclone.exfat
partclone.hfsp
partclone.jfs
partclone.reiserfs
partclone.reiser4
partclone.ufs (with SU+J)
partclone.vmfs (v3 and v5)
partclone.xfs
partclone.f2fs
partclone.nilfs2

```
 clone /dev/hda1 to hda1.dd.img and display debug information.
   partclone.dd -d -s /dev/hda1 -o hda1.dd.img

 restore /dev/hda1 from hda1.dd.img and display debug information.
   partclone.dd -d -s hda1.dd.img -o /dev/hda1
```

* [partclone](https://partclone.org/)