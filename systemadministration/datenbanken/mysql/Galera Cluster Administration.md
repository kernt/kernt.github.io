---
tags:
  - mariadb
  - datenbanken
  - galera
  - cluster
---
Für jeden Node führe folgende schritte aus:

1. Modify the repository configuration, so the system's package manager installs [MariaDB 10.11](https://mariadb.com/kb/en/what-is-mariadb-1011/). For example:
    - On Debian, Ubuntu, and other similar Linux distributions, see [Updating the MariaDB APT repository to a New Major Release](https://mariadb.com/kb/en/installing-mariadb-deb-files/#updating-the-mariadb-apt-repository-to-a-new-major-release) for more information.
    - On RHEL, CentOS, Fedora, and other similar Linux distributions, see [Updating the MariaDB YUM repository to a New Major Release](https://mariadb.com/kb/en/yum/#updating-the-mariadb-yum-repository-to-a-new-major-release) for more information.
    - On SLES, OpenSUSE, and other similar Linux distributions, see [Updating the MariaDB ZYpp repository to a New Major Release](https://mariadb.com/kb/en/installing-mariadb-with-zypper/#updating-the-mariadb-zypp-repository-to-a-new-major-release) for more information.

2. If you use a load balancing proxy such as MaxScale or HAProxy, make sure to drain the server from the pool so it does not receive any new connections.

3. [Stop MariaDB](https://mariadb.com/kb/en/starting-and-stopping-mariadb-starting-and-stopping-mariadb/).

4. Uninstall the old version of MariaDB and the Galera wsrep provider.
    - On Debian, Ubuntu, and other similar Linux distributions, execute the following:  
        `sudo apt-get remove mariadb-server galera-4`
    - On RHEL, CentOS, Fedora, and other similar Linux distributions, execute the following:  
        `sudo yum remove MariaDB-server galera-4`
    - On SLES, OpenSUSE, and other similar Linux distributions, execute the following:  
        `sudo zypper remove MariaDB-server galera-4`

5. Install the new version of MariaDB and the Galera wsrep provider.
    - On Debian, Ubuntu, and other similar Linux distributions, see [Installing MariaDB Packages with APT](https://mariadb.com/kb/en/installing-mariadb-deb-files/#installing-mariadb-packages-with-apt) for more information.
    - On RHEL, CentOS, Fedora, and other similar Linux distributions, see [Installing MariaDB Packages with YUM](https://mariadb.com/kb/en/yum/#installing-mariadb-packages-with-yum) for more information.
    - On SLES, OpenSUSE, and other similar Linux distributions, see [Installing MariaDB Packages with ZYpp](https://mariadb.com/kb/en/installing-mariadb-with-zypper/#installing-mariadb-packages-with-zypp) for more information.

6. Make any desired changes to configuration options in [option files](https://mariadb.com/kb/en/configuring-mariadb-with-option-files/), such as `my.cnf`. This includes removing any system variables or options that are no longer supported.

7. On Linux distributions that use `systemd` you may need to increase the service startup timeout as the default timeout of 90 seconds may not be sufficient. See [Systemd: Configuring the Systemd Service Timeout](https://mariadb.com/kb/en/systemd/#configuring-the-systemd-service-timeout) for more information.

8. [Start MariaDB](https://mariadb.com/kb/en/starting-and-stopping-mariadb-starting-and-stopping-mariadb/).

9. With wsrep off, run `[mariadb-upgrade](https://mariadb.com/kb/en/mariadb-upgrade/)` with the `--skip-write-binlog` option.
    - `mariadb-upgrade` does two things:
        1. Ensures that the system tables in the `[mysql](https://mariadb.com/kb/en/the-mysql-database-tables/)` database are fully compatible with the new version.
        2. Does a very quick check of all tables and marks them as compatible with the new version of MariaDB .

6. Restart the server with wsrep on

## Performing a Rolling Upgrade

The following steps can be used to perform a rolling upgrade from [MariaDB 10.5](https://mariadb.com/kb/en/what-is-mariadb-105/) to [MariaDB 10.6](https://mariadb.com/kb/en/what-is-mariadb-106/) when using Galera Cluster. In a rolling upgrade, each node is upgraded individually, so the cluster is always operational. There is no downtime from the application's perspective.

First, before you get started:

1. First, take a look at [Upgrading from MariaDB 10.5 to MariaDB 10.6](https://mariadb.com/kb/en/upgrading-from-mariadb-10-5-to-mariadb-10-6/) to see what has changed between the major versions.
    1. Check whether any system variables or options have been changed or removed. Make sure that your server's configuration is compatible with the new MariaDB version before upgrading.
    2. Check whether replication has changed in the new MariaDB version in any way that could cause issues while the cluster contains upgraded and non-upgraded nodes.
    3. Check whether any new features have been added to the new MariaDB version. If a new feature in the new MariaDB version cannot be replicated to the old MariaDB version, then do not use that feature until all cluster nodes have been upgrades to the new MariaDB version.
    
2. Next, make sure that the Galera version numbers are compatible.
    1. If you are upgrading from the most recent [MariaDB 10.5](https://mariadb.com/kb/en/what-is-mariadb-105/) release to [MariaDB 10.6](https://mariadb.com/kb/en/what-is-mariadb-106/), then the versions will be compatible.
    2. See [What is MariaDB Galera Cluster?: Galera wsrep provider Versions](https://mariadb.com/kb/en/what-is-mariadb-galera-cluster/#galera-wsrep-provider-versions) for information on which MariaDB releases uses which Galera wsrep provider versions.

3. You want to have a large enough gcache to avoid a [State Snapshot Transfer (SST)](https://mariadb.com/kb/en/introduction-to-state-snapshot-transfers-ssts/) during the rolling upgrade. The gcache size can be configured by setting `[gcache.size](https://mariadb.com/kb/en/wsrep_provider_options/#gcachesize)` For example:  
    `wsrep_provider_options="gcache.size=2G"`

Before you upgrade, it would be best to take a backup of your database. This is always a good idea to do before an upgrade. We would recommend [Mariabackup](https://mariadb.com/kb/en/mariabackup/).

Then, for each node, perform the following steps:

1. Modify the repository configuration, so the system's package manager installs [MariaDB 10.6](https://mariadb.com/kb/en/what-is-mariadb-106/). For example,
    - On Debian, Ubuntu, and other similar Linux distributions, see [Updating the MariaDB APT repository to a New Major Release](https://mariadb.com/kb/en/installing-mariadb-deb-files/#updating-the-mariadb-apt-repository-to-a-new-major-release) for more information.
    - On RHEL, CentOS, Fedora, and other similar Linux distributions, see [Updating the MariaDB YUM repository to a New Major Release](https://mariadb.com/kb/en/yum/#updating-the-mariadb-yum-repository-to-a-new-major-release) for more information.
    - On SLES, OpenSUSE, and other similar Linux distributions, see [Updating the MariaDB ZYpp repository to a New Major Release](https://mariadb.com/kb/en/installing-mariadb-with-zypper/#updating-the-mariadb-zypp-repository-to-a-new-major-release) for more information.
2. If you use a load balancing proxy such as MaxScale or HAProxy, make sure to drain the server from the pool so it does not receive any new connections.
3. [Stop MariaDB](https://mariadb.com/kb/en/starting-and-stopping-mariadb-starting-and-stopping-mariadb/).
4. Uninstall the old version of MariaDB and the Galera wsrep provider.
    - On Debian, Ubuntu, and other similar Linux distributions, execute the following:  
        `sudo apt-get remove mariadb-server galera-4`
    - On RHEL, CentOS, Fedora, and other similar Linux distributions, execute the following:  
        `sudo yum remove MariaDB-server galera-4`
    - On SLES, OpenSUSE, and other similar Linux distributions, execute the following:  
        `sudo zypper remove MariaDB-server galera-4`
5. Install the new version of MariaDB and the Galera wsrep provider.
    - On Debian, Ubuntu, and other similar Linux distributions, see [Installing MariaDB Packages with APT](https://mariadb.com/kb/en/installing-mariadb-deb-files/#installing-mariadb-packages-with-apt) for more information.
    - On RHEL, CentOS, Fedora, and other similar Linux distributions, see [Installing MariaDB Packages with YUM](https://mariadb.com/kb/en/yum/#installing-mariadb-packages-with-yum) for more information.
    - On SLES, OpenSUSE, and other similar Linux distributions, see [Installing MariaDB Packages with ZYpp](https://mariadb.com/kb/en/installing-mariadb-with-zypper/#installing-mariadb-packages-with-zypp) for more information.
6. Make any desired changes to configuration options in [option files](https://mariadb.com/kb/en/configuring-mariadb-with-option-files/), such as `my.cnf`. This includes removing any system variables or options that are no longer supported.
7. On Linux distributions that use `systemd` you may need to increase the service startup timeout as the default timeout of 90 seconds may not be sufficient. See [Systemd: Configuring the Systemd Service Timeout](https://mariadb.com/kb/en/systemd/#configuring-the-systemd-service-timeout) for more information.
8. [Start MariaDB](https://mariadb.com/kb/en/starting-and-stopping-mariadb-starting-and-stopping-mariadb/).
9. Run `[mariadb-upgrade](https://mariadb.com/kb/en/mariadb-upgrade/)` with the `--skip-write-binlog` option.
    - `mariadb-upgrade` does two things:
        1. Ensures that the system tables in the `[mysql](https://mariadb.com/kb/en/the-mysql-database-tables/)` database are fully compatible with the new version.
        2. Does a very quick check of all tables and marks them as compatible with the new version of MariaDB .

When this process is done for one node, move onto the next node.

Note that when upgrading the Galera wsrep provider, sometimes the Galera protocol version can change. The Galera wsrep provider should not start using the new protocol version until all cluster nodes have been upgraded to the new version, so this is not generally an issue during a rolling upgrade. However, this can cause issues if you restart a non-upgraded node in a cluster where the rest of the nodes have been upgraded.