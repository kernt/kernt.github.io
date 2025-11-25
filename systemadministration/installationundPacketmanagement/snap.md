---
tags:
  - packet-management
  - system-administration
  - snap
  - snapctl
---
## Installation unter CentOS Stream 9

**Install snapd**

`sudo dnf install snapd`

**create a symbolic link**

```sh
sudo ln -s /var/lib/snapd/snap/ /snap
```

**enable and start the Snap service**

```sh
sudo systemctl enable --now snapd.socket
Created symlink /etc/systemd/system/sockets.target.wants/snapd.socket → /usr/lib/systemd/system/snapd.socket.
```

**check the status of the service**

```sh
sudo systemctl status snapd.socket
```

**Basic Snap usage**

```sh
sudo snap install core
```

**search for packages**

```sh
sudo snap find [package]
```

**refresh all packages**

```sh
sudo snap refresh
```

**remove some packages**

```sh
sudo snap remove [package]
```

## Install Snap Core

Snap Core is an essential component for running Snap packages. You can install it using the following command:

```bash
sudo snap install core
```
## Enable Classic Confinement for Snap Packages

Specific Snap applications operate under ‘classic’ confinement, which provides the applications with broader permissions within your system. To accommodate these applications, it’s necessary to establish a symbolic link in your file system.

This is done by invoking the following command:

```bash
sudo ln -s /var/lib/snapd/snap /snap
```

The ln -s command in Linux creates a symbolic or soft link. Here, it links the /var/lib/snapd/snap directory to /snap, enabling classic confinement for Snap packages requiring it. This ensures full compatibility and proper functioning of all Snap packages on your Debian system.
## Basic Snap CLI Commands

This section will explore some fundamental Snap command-line interface (CLI) commands. Understanding these commands will empower you to manage your Snap applications efficiently.
### nstall a Snap Package

To install a Snap package, use the snap install command followed by the package name. For instance, to install the VLC media player, you would use:

```bash
sudo snap install vlc
```
### Remove a Snap Package

The `snap remove` command lets you uninstall a Snap package. For example, to remove the VLC media player, run:

```bash
sudo snap remove vlc
```
### Update a Snap Package

Snapd automatically updates your Snap packages in the background. However, if you wish to update a specific package, use snap refresh manually. For example:

```bash
sudo snap refresh vlc
```
### List Installed Snap Packages

To display a list of all installed Snap packages, use the snap list command:

```bash
snap list
```
### Check Snap Version

To view the installed version of Snapd, run the following command:

```bash
snap version
```
### Find Available Snap Packages

If you’re searching for a specific Snap package in the Snap Store, use the snap find command followed by your search term. For example, to find media players, use:

```bash
snap find "media player"
```
### Check Information About a Snap Package

To display detailed information about a specific Snap package, use the snap info command. For instance, to get information about the VLC media player, run:

```bash
snap info vlc
```
### Checking Snap Changes

The snap changes command lets you view the history of Snap tasks, including installations, updates, and removals:

```bash
snap changes
```
### Revert a Snap to a Previous Version

If a new version of a Snap package isn’t working as expected, you can use the snap revert command to roll back to the previous version. For example:

```bash
sudo snap revert vlc
```
#### Checking Snap Interfaces on Debian

The snap interfaces command provides an overview of your Snap packages and the system resources to which they have access:

```bash
snap interfaces
```

These basic commands form the foundation of Snap package management. The following section now looks at how to install Snap-Store for Debian desktop users.
## Install Snap Store on Debian

### Install Snap Store via Snap Command

Once you’ve successfully configured Snapd on your Debian system, you can add a layer of functionality and ease of use – the Snap Store. The Snap Store features a graphical user interface that offers an attractive and intuitive way to browse and handle Snap packages.

This step is not mandatory. However, the Snap Store is a user-friendly option for users who prefer a visual approach instead of the command line. To initiate the installation of the Snap Store, input the following command:

```bash
sudo snap install snap-store
```

This command instructs Snapd to download and install the snap-store package, thus introducing a graphical dimension to your Snap package management.
### Launching the Snap Store

With the installation process completed, the Snap Store can be launched in several ways.

A direct method while operating in the terminal would be to execute the following command:

```bash
snap run snap-store
```

This command triggers Snapd to run the Snap Store application. However, using the terminal each time to open the Snap Store might not be the most practical method.

For more intuitive access to the Snap Store, you can navigate through your desktop environment: Activities > Show Applications > Snap Store.

This pathway guides you to the Snap Store through your graphical user interface, offering a more traditional and user-friendly means of accessing and managing your Snap packages.
## Management Commands of Snap

### Handling Missing Snap Icons

While managing Snap packages on Debian, for the most part, Snap functions seamlessly with most packages. However, occasional anomalies can occur, such as missing application icons in the system’s app launcher. This can be resolved with the following steps:

Initiate the solution by creating a symbolic link using the `ln -s` command as follows:

```bash
sudo ln -s /etc/profile.d/apps-bin-path.sh /etc/X11/Xsession.d/99snap
```

This command creates a symbolic link between the `apps-bin-path.sh` and `99snap` files, enabling your system to locate Snap application icons.

Proceed by opening the `login.defs` file with a text editor, `nano` in this case:

```bash
sudo nano /etc/login.defs
```

Upon accessing the file, append the following line of code at its end:

```bash
ENV_PATH PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
```

This code augments the system’s environment path, providing an additional location to search for application icons.

Save the modifications made to the file by pressing CTRL+O and exit `nano` by pressing CTRL+X.

To enact these adjustments, a system logout and login cycle is required. However, for a comprehensive application of these changes, a system restart is recommended:

```bash
sudo reboot now
```

Upon logging back into the system post-restart, the previously missing Snap application icons should now be in the app launcher.
### Remove Snap and Snap Store

Snap showcases its efficiency and user-friendliness not just in installing packages but also in their removal. If you wish to remove all Snap installations alongside the Snap package manager, you don’t need to uninstall each Snap package.

The sole action required is the removal of the `snapd` service, which concurrently uninstalls all installed Snap packages:

```bash
sudo apt remove snapd
```

Remember, you do not need to remove all Snap installations; when removing snapd, it will remove all associated installed packages, making it an easy, quick, and clean removal of Snap and its associated installations.