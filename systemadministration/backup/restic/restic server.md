Restic is an efficient and modern deduplicating backup system which supports encryption; it is able to store backups locally and remotely, via an SFTP connection or on one of the many supported storage platforms, such as Amazon S3 buckets and Google Cloud storage. By using the restic REST backend API, it is also possible to push backups using the HTTP or HTTPS protocols to a remote server which implements the restic REST API.

In this tutorial, we learn how to use Docker to deploy and configure a restic REST server instance on Linux.

**In this tutorial you will learn:**

- How to deploy a restic REST server using Docker on Linux
- How to configure authentication and use SSL/TLS encryption

## Creating the repository directory and generating credentials

Before we can deploy our restic REST server using docker, we need to choose the directory which will host our repository and backup data. For the sake of this tutorial, I will create a “data” directory inside my HOME. This directory will be bind-mounted inside the container:

`mkdir ~/data`

The restic REST server running inside the docker container, comes with authentication enabled by default. This is important in order to avoid unauthorized access to our data. The restic REST server authentication is implemented via an [.htpasswd](https://linuxconfig.org/how-to-restrict-access-to-a-resource-using-apache-on-linux) file; the REST server looks for this file in the same directory where snapshots are stored.

To generate our authentication credentials, we need to use the _htpasswd_ utility. On Fedora and Fedora-based distributions, the utility is included in the _httpd-tools_ package. We can install it with dnf:

`sudo dnf install httpd-tools`

On Debian and Debian-based distributions, instead, the package is called _apache2-utils_:

`sudo apt-get update && sudo apt-get install apache2-utils`

To populate the _.htpasswd_ file, we run:

```sh
htpasswd -B -c ~/data/.htpasswd resticuser
```

With the command above, we create the configuration required to access the server with the “resticuser” username. If we don’t provide a password directly as argument to the command, we are prompted to enter it interactively:

New password: 
Re-type new password: 
Adding password for user restserver

Since we used the `-B` option, the password will be hashed using the bcrypt algorithm, which is considered very secure.

## Running the restic REST server container

The restic REST server is free and open source software; like the restic application, it is written in Go, and developed on [GitHub](https://github.com/restic/rest-server). The quickest and easiest way to deploy a restic REST server is to use Docker (or Podman), to run a container based on the server image provided by the project, which is available on [DockerHub.](https://hub.docker.com/r/restic/rest-server) To create and spawn a restic REST server container, all we have to do, is to use the following command:

`docker run -p 8000:8000 -v ~/data:/data:z --name restic_rest_server docker.io/restic/rest-server`

Let’s take a look at the options we used with the command. By default, the REST server inside the container listens on port 8000, therefore we used the `-p` option to map port 8000 on the host to the same port inside the container (we could have used any host port, such as port 80, but using an unprivileged one, we can easily spawn the container as a non-root user. Take a look at [this tutorial](https://linuxconfig.org/how-to-bind-a-rootless-container-to-a-privileged-port-on-linux) If you want to use a higher port when running Docker or Podman without root privileges).

In order to make backups persistent, as previously said, we bind-mounted the `~/data` directory on the host, to the same directory inside the container, which is where snapshots are stored. In the example we used the _:z_ option for the bind mount: this is required only if SELinux is active, and causes a change of context for the mounted directory, so that it is accessible from the container.

## Initializing a repository

At this point, we should already be able to initialize a restic repository with the restic client, using the _rest:http://resticuser:resticpassword@localhost:8000_ URL:

`restic -r rest:http://resticuser:resticpassword@localhost:8000 init`

If everything goes as expected, the repository will be generated and initialized correctly:

`created restic repository 0216975ff5 at rest:http://resticuser:***@localhost:8000/`

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.

Although the repository was generated correctly, passing credentials over a plaintext HTTP connection is something we should really avoid. To use the restic REST server over HTTPS, we need to use an SSL/TLS certificate. Let’s see how to do it.

## Using a secure HTTPS connection

There are many ways we can obtain an SSL/TLS certificate. If we intend to expose our restic REST server to the internet, we need to use a valid third-party certificate for our domain. We can easily obtain one from [Let’s Encrypt, using certbot](https://linuxconfig.org/how-to-get-free-ssl-tls-certificates-with-lets-encrypt-and-certbot). If we just want to perform some tests, or we plan to use our server exclusively in our own network, however, it is enough to [generate a self-signed certificate](https://linuxconfig.org/how-to-generate-a-self-signed-ssl-certificate-on-linux).

Here I will assume we already have both the certificate (which contains the public key), and the private key, stored as _public_key_, and _private_key_, respectively. The easiest way we can use them with the REST server running inside a docker container, is to place both in the data directory we bind-mounted inside the container (the same directory where the _.htpasswd_ file is located), and run the server using the `--tls` option. Since, however, we are not running the server directly, we need to pass this option via the `OPTIONS` environment variable, when running the container:

`docker run -p 8000:8000 -v ~/data:/data:z -e OPTIONS="--tls" --name restic_rest_server docker.io/restic/rest-server`

We can now communicate with the server over HTTPS. Since, in this case, we used an invalid self-signed certificate, we need to run the restic client with the `--insecure-tls` option, which skips TLS certificate verification when connecting to a repository:

`restic -r rest:https://resticuser:resticpassword@localhost:8000 --insecure-tls init`

## Conclusions

In this tutorial, we learned how to deploy a restic REST server using Docker on Linux. With restic we can create backups locally or remotely, using SFTP connections, one of the supported third party storage services, or to a restic REST server. The latter provides better performance if compared to SFTP, and gives us the ability to run append-only connections. To know more about restic and how to perform efficient backups, you can take a look at our [step by step tutorial](https://linuxconfig.org/how-to-create-secure-and-efficient-backups-with-restic).
# # security of systemd services

## A test case: writing a backup service

In a previous tutorial we talked about [Restic](https://linuxconfig.org/how-to-create-secure-and-efficient-backups-with-restic), an efficient deduplicating backup program written in Go. For the sake of this article, let’s imagine we want to write a “restic” service to schedule a backup via a [systemd-timer](https://linuxconfig.org/how-to-schedule-tasks-with-systemd-timers-in-linux). We begin by writing the “Unit” section:

```
[Unit] 
Description=restic backup
Wants=network-online.target
After=network-online.target
```
As a first thing, we provided a service description, using the `Description` option. Since we want to be able to use remote repositories for our backups, by using the `Wants` and `After` options, we respectively declared a (weak) dependency of the service on the `network-online.target`, and established it must be started only after said target is reached and network interfaces have been configured.

Now, let’s populate the “Service” section of the unit. This is where we define our service behavior:

```
[Service]
Type=oneshot
User=restic
ExecStart=/usr/local/bin/restic_backup.sh
```

We used the “Type” option to define our service as “oneshot”. This influences how systemd treats the service: it will consider it “up” only after its the main process exits.

For obvious security reasons, we want to avoid running the service as root, therefore we created the “restic” unprivileged user, and with the `User` option, we specified the process should be launched with its privileges. Finally, with the `ExecStart` option, we defined the command/executable which should be invoked when the service is started; in this case, it is the /usr/local/bin/restic_backup.sh script, which contains the main backup logic.

In the “Service” section of the unit, we can use several other options to further tune the privileges of our service. Let’s see some of them.
## Running the process with capabilities

Our service will run with the privileges of the “restic” user. This is a good security measure, however, we must ensure restic is able to read the entire filesystem. In order to reach our goal, we can ensure the process runs with the appropriate [capability](https://linuxconfig.org/introduction-to-linux-capabilities):

```
AmbientCapabilities=CAP_DAC_READ_SEARCH
CapabilityBoundingSet=CAP_DAC_READ_SEARCH
```

The`AmbientCapabilities` option takes the comma-separated list of capabilities we want to include in the ambient capability set of the process as value. In this case we just used the “CAP_DAC_READ_SEARCH” capability, which allows bypassing files and directories read permission checks. As a security measure, we also used the `CapabilityBoundingSet` option to **limit the set of capabilities the process is allowed to obtain**.
## Hardening the service

The two capabilities-related options we used above, are just a small subset of the ones we can use to “isolate” the service. Most of them accept a boolean value. Let’s see some examples.
#### PrivateTmp

The “PrivateTmp” option protects temporary files created by the service, so that other processes cannot access them. When the option is active, systemd creates isolated /tmp and /var/tmp directories and mounts them in a private namespace.

#### ProtectKernelModules, ProtectKernelLogs and ProtectKernelTunables

These options protect the kernel state. The `ProtectKernelModules` one, when active, denies the service the ability to load and unload kernel modules, while `ProtectKernelLogs` denies access to the kernel log buffer. The behavior of certain kernel modules can be altered by writing appropriate values to files exposed in the /proc and /sys pseudo-filesystems; the `ProtectKernelTunables` option, when active, denies such actions.

#### NoNewPrivileges

This option can be used to make sure the service, and its child processes, cannot gain new privileges by executing other programs via the execve system call, which is part of the standard C library. When this option is active, it denies the execution of binaries with the [SETUID or SETGID bits](https://linuxconfig.org/how-to-use-special-permissions-the-setuid-setgid-and-sticky-bits) set.
#### RestrictSUIDSGID

This option, when true, prevents the process invoked by the service to set the SETUID or SETGID bits on files and directories.
#### RestrictAddressFamilies

This option accepts the space-separated list of address family names the process can access (e.g: AF_UNIX, AF_INET, AF_INET6);  “none” is also a valid value: it denies access to all of them.
#### PrivateDevices

When this option is active, it denies the process access to raw devices such as /dev/sda or /dev/mem.
#### ProtectClock

When set to true, it denies access to the system clock.
#### ProtectHostname

This option protects the system hostname, ensuring the process cannot change it.
#### RemoveIPC

When active, it causes the automatic removal of IPC (Inter Process Communication) resources allocated to the service.
#### PrivateMounts

When this option is active, the process runs in its own private and isolated filesystem, inacessible from the host.
## Obtaining an estimate security level of the service

Here is how our service looks like, in the end:

```
[Unit] 
Description=restic backup
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
User=restic
ExecStart=/usr/local/bin/restic_backup.sh
AmbientCapabilities=CAP_DAC_READ_SEARCH
CapabilityBoundingSet=CAP_DAC_READ_SEARCH
PrivateTmp=yes
ProtectKernelModules=yes
ProtectKernelLogs=yes
ProtectKernelTunables=yes
NoNewPrivileges=yes
RestrictSUIDSGID=yes
RestrictAddressFamilies=yes
PrivateDevices=yes
ProtectClock=yes
ProtectHostname=yes
RemoveIPC=yes
PrivateMounts=yes
```

Once we place the unit in one of the directories recognized by systemd (/etc/systemd/system, for example), to get its estimated security level, we just need to launch “systemd-analyze” with the “security” command, passing the unit name as argument. Supposing we saved the unit as “restic.service”, we would run:

`$ systemd-analyze security restic.service`

The command returns a list of the available security options, marking those present and those absent in the unit, and an overall exposure level: the lower this is, the better. Each unused option increases the exposure value by the amount reported in the “EXPOSURE” column. Here is the output we obtain by running the command against our service unit