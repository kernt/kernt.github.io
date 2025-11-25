his document describes what and how Bifrost installs in the default setup (e.g. one created by `./bifrost-cli install`).

## Components[](https://docs.openstack.org/bifrost/latest/user/architecture.html#components "Link to this heading")

Bifrost installs the following main components:

[ironic](https://docs.openstack.org/ironic/latest/)

Ironic is the main service that provides bare metal capabilities. Its [bare metal API](https://docs.openstack.org/api-ref/baremetal/) is served on TCP port 6385.

[ironic-inspector](https://docs.openstack.org/ironic-inspector/latest/)

Inspector is an auxiliary service that provides [in-band inspection](https://docs.openstack.org/ironic/latest/admin/inspection/inspector.html). Its [introspection API](https://docs.openstack.org/api-ref/baremetal-introspection/) is served on TCP port 5050.

Inspector is deprecated and can be enabled by setting `enable_inspector=true`. Otherwise, Ironic’s [native in-band inspection](https://docs.openstack.org/ironic/latest/admin/inspection/index.html) is used.

[mariadb](https://mariadb.org/)

MariaDB is used as a database to persistently store information.

[nginx](https://nginx.org/)

Nginx is used as a web server for three main purposes:

- to serve iPXE configuration scripts when booting nodes over the network,
- to serve virtual media ISO images to BMC when booting nodes over virtual CD,
- to serve instance images to the ramdisk when deploying nodes.

Uses HTTP port 8080 by default (can be changed via the `file_url_port` parameter).

When TLS is enabled, Nginx serves as a TLS proxy for Ironic and Inspector. It listens on ports 6385 and 5050 and passes requests to the services via unix sockets.

[dnsmasq](https://dnsmasq.org/)

Dnsmasq is used as a DHCP and TFTP server (but not for DNS by default) when booting nodes over the network. It can also be used to provide DHCP to deployed nodes.

Dnsmasq can be disabled by setting `enable_dhcp=false` or passing `--disable-dhcp` to `bifrost-cli`.

The following components can be enabled if needed:

[keystone](https://docs.openstack.org/keystone/latest/)

Keystone is an OpenStack Identity service. It can be used to provide sophisticated authentication to Ironic and Inspector instead of HTTP basic authentication. Its [identity API](https://docs.openstack.org/api-ref/identity/v3/index.html) is served using uWSGI and Nginx on the port 5000, the systemd service is called `uwsgi@keystone-public`.

See [Using Keystone](https://docs.openstack.org/bifrost/latest/user/keystone.html) for details.

[ironic-prometheus-exporter](https://docs.openstack.org/ironic-prometheus-exporter/latest/)

An exporter that exposes hardware metrics from the node’s BMC (using IPMI or Redfish) for consumption by [Prometheus](https://prometheus.io/).

The exporter can be enabled by setting `enable_prometheus_exporter=true` or passing `--enable-prometheus-exporter` to `bifrost-cli`.

## Networking[](https://docs.openstack.org/bifrost/latest/user/architecture.html#networking "Link to this heading")

[Bifrost Installation](https://docs.openstack.org/bifrost/latest/install/index.html) requires to specify the network interface which will be used for communication with the nodes. On systems that use [firewalld](https://firewalld.org/), a new zone `bifrost` is created with this interface, and firewall settings are adjusted to allow boot traffic (DHCP, TFTP, etc) through this interface.

Virtual testing environments use network interface `virbr0`, IP subnet `192.168.122.1/24` and the `libvirt` firewalld zone by default.

### Parameters[](https://docs.openstack.org/bifrost/latest/user/architecture.html#parameters "Link to this heading")

`network_interface`

the main network interface for provisioning nodes. It is recommended to use an isolated network for this purpose.

`internal_ip`

IPv4 address of the Bifrost node on the provisioning interface. Normally derived automatically. You can use the following command to detect it:

```sh
sudo awk '/^tftp_server =/ { print $3 }' /etc/ironic/ironic.conf
192.168.122.1
```

This IP address is used for all provisioning traffic: TFTP, iPXE, call-backs to Ironic and Inspector. It is also used for the traffic between the services.

`public_ip`

IPv4 address of the Bifrost node that is accessible outside of the provisioning environment. Used e.g. to provide public URLs for the service catalog, when Keystone support is enabled.

Note

Bifrost does not currently support IPv6, contributions are welcome.

## Locations[](https://docs.openstack.org/bifrost/latest/user/architecture.html#locations "Link to this heading")

Note

Many locations are not writable or even readable by the regular user (even the one that started installation). You need to use `sudo` or add the user to `ironic` and `nginx` groups.

### Installation locations[](https://docs.openstack.org/bifrost/latest/user/architecture.html#installation-locations "Link to this heading")

`/opt/stack/bifrost`

is a Python virtual environment where all Python packages are installed.

`/opt/stack/<reponame>`

is a source directory for project `<reponame>` (e.g. `ironic`) installed from source. Services are installed this way, while libraries are mostly installed as packages.

Unlike most other locations, these paths will be owned by the user that installed Bifrost (i.e. your regular user).

### Log locations[](https://docs.openstack.org/bifrost/latest/user/architecture.html#log-locations "Link to this heading")

journald

is used for logging from most services. For example, to get Inspector logs:

sudo journalctl -u ironic-inspector

`/var/log/ironic/deploy`

contains tarballs with ramdisk logs from deployment or cleaning. The file name format is `<node UUID>-<node name>-[cleaning-]<datatime>.tar.gz`. You can access the main logs like this:

cd $(mktemp -d)

sudo tar -xzf \
    /var/log/ironic/deploy/493aacf2-90ec-5e3d-9ce5-ea496f12e2a5_testvm3_2021-11-08-17-34-18.tar.gz

less journal  # for ramdisks that use systemd, e.g. DIB-built

less var/log/ironic-python-agent.log # for tinyIPA and similar

`/var/log/ironic-inspector/ramdisk`

contains tarballs with ramdisk logs from inspection. They are very similar to ramdisk logs from deployment and cleaning.

`/var/log/nginx/`

contains logs for serving files (iPXE scripts, images, virtual media ISOs).

`/var/log/nginx/keystone`

contains HTTP logs for Keystone API, complementing the logs from the `uwsgi@keystone-public` systemd unit.

### Runtime locations[](https://docs.openstack.org/bifrost/latest/user/architecture.html#runtime-locations "Link to this heading")

`/var/lib/ironic/httpboot`

HTTP root directory. Contains iPXE scripts and images, including `deployment_image.qcow2` which is built or downloaded during Bifrost installation.

You can detect the root URL with the following command:

sudo awk '/^http_url =/ { print $3 }' /etc/ironic/ironic.conf
http://192.168.122.1:8080/

`/var/lib/tftpboot`

TFTP root directory. Normally contains only the binaries to run iPXE on systems that don’t have an iPXE firmware built in. Can contain images when the `pxe` boot interface is used.

`/var/lib/ironic/master_images`

cache for instance images.

`/var/lib/ironic/master_iso_images`

cache for virtual media ISO images.

`/var/lib/ironic/certificates`

TLS certificates that are used to communicate to the ramdisk on the nodes when cleaning or deploying.

`/run/ironic`

When TLS is enabled, this directory contains unix sockets of Ironic and Inspector, which Nginx uses to pass requests.