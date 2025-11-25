Sysdig is a powerful open-source system monitoring and troubleshooting tool that provides deep visibility into the behavior of your system and applications. It allows you to capture and analyze system calls, network activity, and process information in real-time, making it invaluable for performance monitoring, security analysis, and debugging. Sysdig also includes a user-friendly graphical interface called csysdig, which provides a comprehensive overview of system activity and performance metrics.

To install Sysdig on Debian 12, 11, or 10, you can use the Sysdig official mirror APT repository. This guide will walk you through the installation process and cover basic Sysdig commands, along with tips for using the csysdig graphical interface.
## Update the Debian System Before Sysdig Installation

Before installing Sysdig, ensuring that your Debian system is current is essential. This ensures that all existing packages are updated to their latest versions, improving your system’s stability and security.

To update your system, execute the following command in your terminal:

```bash
sudo apt update && sudo apt upgrade
```
## Install Required Packages

You must install some prerequisite software packages to successfully install Sysdig on your Debian system. These packages enable proper functionality and integration of Sysdig with your system.

Run the following command in your terminal to install the required packages:

```bash
sudo apt install software-properties-common apt-transport-https ca-certificates ncurses-term dkms -y
```
## Import Sysdig APT Repository on Debian

By default, Sysdig is not available in Debian’s official repository. However, its developers maintain a dedicated repository.

To add this repository to your system, follow these steps:
### Import the Sysdig GPG key

The GPG key ensures the authenticity and integrity of the packages downloaded from the Sysdig repository. Run the following command to import the GPG key:

```bash
curl -s https://s3.amazonaws.com/download.draios.com/DRAIOS-GPG-KEY.public | gpg --dearmor | sudo tee /usr/share/keyrings/sysdig.gpg
```
### Add the Sysdig APT repository

After importing the GPG key, you can add the Sysdig repository to your system’s sources list by running the following command:

```bash
echo 'deb [signed-by=/usr/share/keyrings/sysdig.gpg] https://download.sysdig.com/stable/deb stable-$(ARCH)/' | sudo tee /etc/apt/sources.list.d/sysdig.list
```

Note that the $(ARCH) variable will be automatically replaced with the appropriate architecture Sysdig currently supports.
## Refresh APT Cache Index

After adding the Sysdig repository to your system, update your package list to include the newly added repository. This ensures that the Sysdig package will be available for installation. To update the package list, run the following command:

```bash
sudo apt update
```
## Finalize Sysdig Installation via APT Command

Now that you have added the Sysdig repository to your system and updated the package list, you can install Sysdig. To do so, run the following command in your terminal:

```bash
sudo apt install linux-headers-$(uname -r) sysdig -y
```

This command installs the appropriate Linux headers for your kernel version and the Sysdig package. The installation process should be relatively quick, taking at least a few minutes.

## Verify Sysdig Installation

After completing the installation, verifying that Sysdig has been installed correctly on your Debian system is essential. To check the version and build of Sysdig, run the following command in your terminal:

```bash
sysdig --version
```
## Getting Started with Sysdig Commands

Sysdig offers a wide range of commands that allow you to monitor and troubleshoot your containerized environments effectively. This section will explore some of the most useful Sysdig commands, divided into several categories for easy understanding. The examples provided will help you start using Sysdig commands for various purposes.

### Basic Sysdig Commands

Before diving into more advanced features, let’s start with some basic Sysdig commands that help you understand your system’s overall status.
#### List Running Processes

To display a list of currently running processes on your system, use the following command:

```bash
sysdig -l
```

This command will output a running process list, giving you an overview of your system’s current state.
#### Monitor System Activity

If you want to monitor your system’s real-time activity, you can use the following command:

```bash
sysdig -c topprocs_cpu
```

This command will display the top processes consuming the most CPU resources, helping you quickly identify resource-intensive applications and potential performance bottlenecks.
### Filtering Sysdig Output

Sysdig allows you to apply filters to the output, enabling you to focus on specific processes, containers, or events of interest. Here are some examples of using filters with Sysdig commands:

```bash
sysdig proc.name=nginx
```
#### Filter by Container Name

Similarly, you can filter the output to display events related to a specific container. To do so, use the container.name filter as shown in the following example:

```bash
sysdig container.name=my_container
```

Replace my_container with the actual name of the container you want to monitor.
### Advanced Sysdig Commands

Sysdig also offers advanced commands that provide deeper insights into your containerized environments. Let’s explore some of these commands.
#### Monitor File I/O Activity

To monitor file I/O activity on your system, you can use the spy_file Sysdig command. This command will display information about files being accessed, the processes accessing them, and the I/O operations performed. To use this command, run:

```bash
sysdig -c spy_file
```
#### Analyze Network Connections

Sysdig can help you analyze your system’s network connections and detect potential issues or security threats. To display information about network connections, use the netstat command as follows:

```bash
sysdig -c netstat
```

This command will output a list of active network connections, including the source and destination IP addresses, ports, and connection state.
### Creating Custom Sysdig Views

Sysdig allows you to create custom views, focusing on specific metrics and data points relevant to your needs. Here’s an example of creating a custom Sysdig view:
#### Custom View for CPU Usage

To create a custom view that displays the CPU usage of processes, use the following command:

```bash
sysdig -c topprocs_cpu "evt.type=execve and proc.name=my_process"
```

Replace my_process with the actual name of the process you want to monitor.

This custom view will display the top processes consuming the most CPU resources, filtered by the specified process name. You can customize this view by modifying the filter or adding additional metrics.
## Getting Started with cSysdig Commands

cSysdig is an interactive, terminal-based user interface for Sysdig that provides a more user-friendly way to navigate system metrics and events. cSysdig commands are similar to Sysdig commands but are executed within the cSysdig interface rather than directly on the terminal. This section will introduce you to cSysdig and some essential commands and features you can use within the interface.
### Launching cSysdig

To launch cSysdig, run the following command in your terminal:

```bash
csysdig
```

This will open the cSysdig interface, where you can explore various views and execute cSysdig commands.

Note: Depending on your user privileges, you may need to add `sudo` before the command to launch cSysdig with administrative permissions.

### Navigating the cSysdig Interface

cSysdig organizes information into several built-in views, each focusing on a specific aspect of your system. You can switch between these views using the F2 key or by typing : followed by the view name.

Here are some essential cSysdig views:

- **Processes:** Displays a list of running processes and their resource usage. (Shortcut: :processes)
- **Connections:** Shows active network connections, including source and destination IP addresses, ports, and connection state. (Shortcut: :connections)
- **Errors:** Highlights system errors and exceptions. (Shortcut: :errors)
- **Containers:** Lists running containers and their resource usage. (Shortcut: :containers)

### cSysdig Commands and Shortcuts

cSysdig provides several commands and shortcuts that help you navigate the interface and interact with the displayed data. Here are some useful cSysdig commands and shortcuts:

- **F1 or h:** Display the help menu, providing an overview of available commands and shortcuts.
- **F2 or v:** Switch between available views.
- **F4 or l:** Apply a filter to the current view. For example, you can filter processes by their name or containers by their ID.
- **F5 or s:** Sort the current view by a specific column.
- **F6 or a:** Add or remove columns from the current view.
- **Esc or q:** Quit cSysdig or close the current menu.

### Creating Custom Views in cSysdig

Like with Sysdig, you can create custom views in cSysdig to focus on specific metrics and data points relevant to your needs. To create a custom view, follow these steps:

1. Press F2 or type :addview to open the “Add View” menu.
2. Enter a name for your custom view.
3. Define the columns you want to include in your view by typing the respective column names.
4. Add a filter to your custom view by pressing F4 and entering the filter criteria.
5. Save your custom view by pressing Enter.

You can now switch to your custom view using the F2 key or by typing :your_view_name.

## ditional Commands for Sysdig

This section will cover additional helpful commands when working with Sysdig on Debian. These commands include updating, removing, and managing Sysdig’s installation on your system.
### Update Sysdig

Since you have imported the official APT repository for Sysdig, updating the software is quick and straightforward. To update Sysdig, run the following standard APT commands as you would when updating any other system package:

```bash
sudo apt update && sudo apt upgrade
```

This command will ensure that Sysdig and all other installed packages on your system are up to date.
### Remove Sysdig

If you no longer require Sysdig on your system, follow these steps to remove it:
#### Uninstall Sysdig

Use the following command to remove the Sysdig package from your system:

```bash
sudo apt remove sysdig
```
#### Remove the Sysdig GPG key

To remove the GPG key used to authenticate Sysdig packages, run the following command:

```bash
sudo rm /usr/share/keyrings/sysdig.gpg
```
#### Remove the Sysdig APT Repository

Finally, remove the Sysdig repository from your system by executing the following command:

```bash
sudo rm /etc/apt/sources.list.d/sysdig.list
```