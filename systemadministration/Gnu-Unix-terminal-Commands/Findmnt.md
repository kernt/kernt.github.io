## Findmnt basic usage

The simplest way to use findmnt consists into invoking it without any options or arguments; in such case, the utility displays information about all mounted filesystems, organized in four columns:

- TARGET
- SOURCE
- FSTYPE
- OPTIONS

The first column, “TARGET”, holds information about the filesystem **mountpoint**; the second, “SOURCE”, contains the path of the **source device** which hosts the filesystem. The third column, “FSTYPE”, reports information about the filesystem type (e.g. ext4 or XFS); finally, the fourth column displays the mount options used for the filesystem.

Those mentioned above are the default columns used in the findmnt output. To retrieve the list of all those available, together with their brief description, we can invoke the utility with the `-H` option:

```sh
findmnt -H  
     ACTION <string>        action detected by --poll  
      AVAIL <string|number> filesystem size available, use <number> if --bytes is given  
       FREQ <integer>       dump(8) period in days [fstab only]  
     FSROOT <string>        filesystem root  
     FSTYPE <string>        filesystem type  
 FS-OPTIONS <string>        FS specific mount options  
         ID <integer>       mount ID  
  INO.AVAIL <string>        number of available inodes  
  INO.TOTAL <string>        total number of inodes  
   INO.USED <string>        number of used inodes  
   INO.USE% <string>        percentage of INO.USED divided by INO.TOTAL  
      LABEL <string>        filesystem label  
    MAJ:MIN <string>        major:minor device number  
OLD-OPTIONS <string>        old mount options saved by --poll  
 OLD-TARGET <string>        old mountpoint saved by --poll  
    OPTIONS <string>        all mount options  
 OPT-FIELDS <string>        optional mount fields  
     PARENT <integer>       mount parent ID  
  PARTLABEL <string>        partition label  
   PARTUUID <string>        partition UUID  
     PASSNO <integer>       pass number on parallel fsck(8) [fstab only]  
PROPAGATION <string>        VFS propagation flags  
       SIZE <string|number> filesystem size, use <number> if --bytes is given  
     SOURCE <string>        source device  
    SOURCES <string>        all possible source devices  
     TARGET <string>        mountpoint  
        TID <integer>       task ID  
       USED <string|number> filesystem size used, use <number> if --bytes is given  
       USE% <string>        filesystem use percentage  
       UUID <string>        filesystem UUID  
VFS-OPTIONS <string>        VFS specific mount options
```

## Retrieving information about a specific filesystem

As we have already discussed, when invoked without any options or arguments, `findmnt` displays information about all mounted filesystems. This is seldom useful: most of the time we are interested in gathering information about a specific filesystem, which we can reference by _device name_, _label_, _UUID_, or by _major:minor_ number.

The most common way to look for a filesystem, is probably by its source device:

`findmnt /dev/mapper/vg_system-lv_root`

Another option consists into passing the directory on which we know the filesytem is mounted (its mountpoint):

`findmnt /`

Findmnt is able to match the argument against both the SOURCE and TARGET columns, unless we explicitly use the `--target`, `--mountpoint` or `--source` options. In the example below, the utility returns no output, since it interprets the argument as a mountpoint:

`findmnt --mountpoint /dev/mapper/vg_system-lv_root`

To reference a specific filesystem, we can also use its UUID or LABEL if we know them beforehand. Suppose we want to retrieve information about the filesystem with UUID 331392ba-8966-4877-828c-99cc4bc418ec; we would run:

`findmnt UUID=331392ba-8966-4877-828c-99cc4bc418ec`

Similarly, to reference a filesystem by its LABEL, we would run:

`findmnt LABEL=backups`

## Searching in a specific file

By default, findmnt searches for filesystems in the `/etc/fstab`, `/etc/mtab`, and `/proc/self/mountinfo` files. Some options allow us to change this behavior. The `-s` (or `--fstab`) option, for example, makes findmnt consider only filesystems in the `/etc/fstab` file; similarly, the `-m` ( `--mtab`) option, restricts the search to the content of the `/etc/mtab` file, and `-k`, (short for `--kernel`), narrows the search to the `/proc/self/mountinfo` file.

We can also provide the path of a custom, alternative file, by using the `-F` option (short for `--tab-file`). To search for filesystems in the `/etc/fstab-backup` file, for example, we would run:

`findmnt -F /etc/fstab-backup`

## Filtering results by filesystem type

In certain cases, we don’t want to target a specific filesystem, and neither to obtain the list of all mounted ones. Instead, we may want to apply a filter based on some parameter. To make findmnt return only filesystems of a certain type, for example, we can use the `-t` option (`--type`), and pass the list of types we want to use as a filter. Supposing we want to gather information about all mounted **vfat** or **XFS** filesystems, we would run:

`findmnt -t vfat,xfs`

## Filtering results by mount options

Another option is to apply a filter based on mount options. To use this feature, all we have to do, is to use the `-O` option (short for `--options`) and provide the comma-separated list of options we want to use as a filter. To retrieve the list of all filesytems mounted with the “relatime” and “ro” options, for example, we would run:

`findmount -O relatime,ro`

When multiple options are specified, only filesystems mounted with all the specified options will match the query (this works like a logical AND).

Another thing to notice is that, by default, when using an option with the “no” prefix (e.g. nosuid), the expression will not match literally; instead, it will be interepted as an “exclusion” filter, and it will match all filesystems mounted without the option. Tale a look at the content of this `/etc/fstab` file:

```
/         UUID=331392ba-8966-4877-828c-99cc4bc418ec xfs    defaults,x-systemd.device-timeout=0  
/boot     UUID=7e28e28a-fc1b-4652-8f86-4e2bdefdad85 xfs    defaults,nosuid  
/boot/efi UUID=ABBE-D50B                            vfat   defaults,uid=0,gid=0,umask=077,shortname=winnt  
/home     UUID=244ed50a-4223-41a5-9edb-658488d02279 xfs    defaults,x-systemd.device-timeout=0
```

As you can see, the second filesystem in the list, the one with the `7e28e28a-fc1b-4652-8f86-4e2bdefdad85` UUID, is the only one which has “nosuid” has a mount option. Let’s run findmnt with the `-O nosuid` option, and let’s see what it returns:

`findmnt --fstab -O nosuid` 

The output of the command above is:

```
TARGET    SOURCE                                    FSTYPE OPTIONS  
/         UUID=331392ba-8966-4877-828c-99cc4bc418ec xfs    defaults,x-systemd.device-timeout=0  
/boot     UUID=7e28e28a-fc1b-4652-8f86-4e2bdefdad85 xfs    defaults,nosuid  
/boot/efi UUID=ABBE-D50B                            vfat   defaults,uid=0,gid=0,umask=077,shortname=winnt  
/home     UUID=244ed50a-4223-41a5-9edb-658488d02279 xfs    defaults,x-systemd.device-timeout=0
```

The “query” we used, somehow unexpectedly, instead of matching only the filesystem mounted with the “nosuid” option, returned all those **without** the “suid” option, so all of them. If we want the expression to match literally the filesystems with the “nosuid” option, we must prefix the option with a “+”:

`findmnt --fstab -O +nosuid`

With this syntax, the expression matches the “nosuid” option literally, and yields the expected result:

```sh
TARGET SOURCE                                    FSTYPE OPTIONS  
/boot  UUID=7e28e28a-fc1b-4652-8f86-4e2bdefdad85 xfs    defaults,nosuid
```
## Formatting findmnt output

A series of options allow us to modify and format the output of the findmnt command. When using findmnt in shell scripts, for example, we probably want to suppress the header row, and change the behavior of the utility so it only returns a single piece of information.

The header row can be suppressed by using the `-n` option (short for `--noheadings`), while to make the utility just return specific information, we can use the `-o` option (`--output`) and pass the comma-separated list of column(s) we are interested in, as argument. In the example below, we store the fstype of the filesystem mounted on `/` in the “$root_fstype” variable:

```sh
root_fstype="$(findmnt --noheadings --output FSTYPE /)"  
echo "${root_fstype}"  
xfs
```

Findmnt, by default, organizes its output in a tree-like format, which shows also the relationships between filesystems. We can change this and format the output as JSON, using the `-J` (`--json`) option. Supposing we want to reorganize the filesystem contained in the `/etc/fstab` file in JSON, for example, we would run:

`findmnt -J --fstab`

We would obtain a result similar to the following:

```yaml
{  
   "filesystems": [  
      {  
         "target": "/",  
         "source": "UUID=331392ba-8966-4877-828c-99cc4bc418ec",  
         "fstype": "xfs",  
         "options": "defaults,x-systemd.device-timeout=0"  
      },{  
         "target": "/boot",  
         "source": "UUID=7e28e28a-fc1b-4652-8f86-4e2bdefdad85",  
         "fstype": "xfs",  
         "options": "defaults,nosuid"  
      },{  
         "target": "/boot/efi",  
         "source": "UUID=ABBE-D50B",  
         "fstype": "vfat",  
         "options": "defaults,uid=0,gid=0,umask=077,shortname=winnt"  
      },{  
         "target": "/home",  
         "source": "UUID=244ed50a-4223-41a5-9edb-658488d02279",  
         "fstype": "xfs",  
         "options": "defaults,x-systemd.device-timeout=0"  
      }  
   ]  
}
```

Alternatively, we can use the “list” format by passing the `-l` (`--list`) option. This format is automatically used when we apply filters to restrict a query.

## Conclusions

In this tutorial, we learned to use the findmnt utility to retrieve information about mounted filesystems on Linux. The utility is installed by default in practically all the major distributions. We saw how to list all mounted filesystems, how to reference a filesystem by UUID, LABEL, name and mountpoint, and how to filter findmnt results by mount options and filesystem types.