A debian package is a Unix [ar archive](https://en.wikipedia.org/wiki/Ar_\(Unix\)) that includes two tar archives: one containing the control information and another with the program data to be installed.

## View contents of a Debian package using `dpkg`

The debian package manager `dpkg` comes with a utility to view the contents of a package. Assuming you have the actual debian package, the following command will list its contents:

```
$ dpkg -c ./path/to/test.deb
```

For example:

```
$ dpkg -c ./test_2.0.0_amd64.deb

drwxr-xr-x root/root         0 2015-06-27 19:00 ./
drwxr-xr-x root/root         0 2015-06-27 19:00 ./usr/
drwxr-xr-x root/root         0 2015-06-27 19:00 ./usr/bin/
-rwxr-xr-x root/root  44790352 2015-06-27 19:00 ./usr/bin/test
drwxr-xr-x root/root         0 2015-06-27 19:00 ./usr/share/
drwxr-xr-x root/root         0 2015-06-27 19:00 ./usr/share/doc/
drwxr-xr-x root/root         0 2015-06-27 19:00 ./usr/share/doc/test/
-rw-r--r-- root/root       148 2015-06-27 18:45 ./usr/share/doc/test/changelog.gz
-rw-r--r-- root/root        33 2015-06-27 18:44 ./usr/share/doc/test/copyright
```

As you can see in the example above, the package will install an executable binary called `test` into `/usr/bin/` and supporting documentation will be dropped into `/usr/share/`.
## Extract files from a Debian package

### Using the `ar` command

A debian package is just an `ar` archive. To extract data from a deb package, use the command `ar` with the `-x` flag:

```
$ ar -x ./test_2.0.0_amd64.deb
$ ls
control.tar.gz data.tar.gz debian-binary test_2.0.0_amd64.deb
```

The files extracted from the deb package are `control.tar.gz` `data.tar.gz` and `debian-binary`. These are the control files and package data along with the `debian-binary` file which contains the version string of the package.
#### Extract files from `control.tar.gz` and `data.tar.gz` using `tar`

Extracting files from `tar` archives is straightforward, using the `-xzf` flags to extract to the current working directory:

```
$ tar -xzf control.tar.gz
```

Extracts the following files:

```
control md5sums
```

The program files are located in the `data.tar.gz` archive. Extracting this archive will effectively pull all the program files into the current working directory, in this case the `usr/` directory:

```
$ tar -xzf data.tar.gz
$ ls
control control.tar.gz data.tar.gz debian-binary md5sums test_2.0.0_amd64.deb usr

$ ls usr/bin
test
```

### Using `dpkg-deb`

To extract files from a debian package, use the following command:

```
$ dpkg-deb -x ./path/to/test.deb ./path/to/destination
```

For example:

```
$ dpkg-deb -x ./test_2.0.0_amd64.deb .

$ file ./usr/bin/test

usr/bin/test: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0x32b2d15656286b7b0e39ba1768be7767a0e7e9e8, stripped
```

This command extracts the contents of the package (without installing it) into the `./path/to/destination` directory. The `./path/to/destination` directory will be created if necessary, and the proper permissions given to match the contents of the package. The command can also be written as:

```
$ dpkg -x ./test_2.0.0_amd64.deb .
```

**NOTE** simply extracting the packages to the root directory will NOT ensure a correct installation. Please use `dpkg` or `apt-get` to install packages.
#### Extract `control` information from a Debian package using `dpkg-deb`

To extract the control section from a debian package, use the `dpkg` command with the `-e` option. This will extract the control files for a package into the specified directory:

```sh
$ dpkg -e ./test_2.0.0_amd64.deb
$ ls
control md5sums postinst postrm preinst prerm
$ cat ./DEBIAN/md5sums

aff2ef681a6f055bb1b3c524520d9542  usr/bin/test
c95b234e1d551b6198b5e375a61e2441  usr/share/doc/test/changelog.gz
1699fdbd753f1bc26e6fcb312b26b4b7  usr/share/doc/test/copyright
```
## What are `preinst`, `postinst`, `prerm` and `postrm` files?

The `preinst`, `postinst`, `prerm`, and `postrm` files are scripts that will automatically execute before or after a package is installed or removed. These scripts are part of the control section of a Debian package.

```sh
$ dpkg -e ./test_2.0.0_amd64.deb
$ ls
control md5sums postinst postrm preinst prerm
$ cat ./DEBIAN/postinst

#!/bin/sh
# This is an example script that does nothing...

exit 0
```

## Using `apt-file` to view the contents of debian packages on remote repositories

It can be helpful to view the contents of packages that aren’t downloaded or installed on your the system. If you’ve configured an apt repository (for example a [packagecloud repo](https://packagecloud.io/docs#what_is_a_pkgcloud_repo)) you can use `apt-file` to list the contents of a package in that repository without fetching or installing the package.

Make sure `apt-file` is installed on your system:

```sh
apt-get install apt-file
```

Before using `apt-file` you have to make sure that you’ve updated it with the repositories configured on the system. To update `apt-file` run the following command:

```sh
apt-file update
```

Example output (using a packagecloud repo):

```sh
apt-file update

Downloading complete file https://packagecloud.io/armando/test/ubuntu/dists/precise/Contents-amd64.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    90  100    90    0     0    251      0 --:--:-- --:--:-- --:--:--   251
```

After the update you can list package contents using the following command:

```sh
apt-file list <packagename>
```

For example:

```sh
apt-file list test
test=2.0.0: /usr/bin/test
test=2.0.0: /usr/share/doc/test/changelog.gz
test=2.0.0: /usr/share/doc/test/copyright
```

Note that the `apt-file` command takes the name of a package that exists in the repository and not the file path to a debian package. It will search for packages by name from the apt contents metadata.

