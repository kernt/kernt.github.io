---
tags:
  - networking
  - mdns
---


get challange to browse/advertise mDNS information from/to rootless container in IoT linux device.

Rootless containers refers to the ability for an unprivileged user to create, run and otherwise manage containers. This term also includes the variety of tooling around containers that can also be run as an unprivileged user. When we say Rootless Containers, it means running the entire container runtime as well as the containers without the root privileges.

![](wAbnjcpCzK7PXjIhVNh_-w.webp)

in my case IoT host machine using Avahi to browse and advertise mDNS information from switching network. If you installed already rootless container you are aware about rootless container network limitation. It uses **slirp4netns** to manage user-mode networking (“slirp”) for unprivileged network namespaces. So, no use of `ipables`to easyly manage traffic between host and container. The communication utilizes the `tap0` or `tun0` interface, restricting access to host resources. When running a zeroconf application within a rootless container, it faces limitations in obtaining accurate ARP messages from the host, which gather information from the network

While it’s not overly challenging to create a custom application to address specific issues, this custom development requires ongoing support and maintenance. Typically reliant on a single or small team within the company, it needs updating in sync with core OS software updates to ensure its functionality. We need to seek solutions that rely on a large community to ensure the stability of the solution.

Most secure solution I found to use system `d-bus/system bus`to rootless container.

![](Xyhw5Xz3tIFx_CD3MVPthQ.webp)


But since docker rootless UID/GID, Linux does not allow to access socket file although permission for _others_ is **rw. As well as rootless container cannot access to any host resource which run via root unless it adjusted.**

I decided to use [socat](https://linux.die.net/man/1/socat) tool which is production ready to re-map socket file with required minimum permission and mount it to docker container.

```sh
socat UNIX-LISTEN:/tmp/dbus_socket/dbus.sock,fork,user=1000,group=1000,mode=777 UNIX-CONNECT:/var/run/dbus/system_bus_socket  
socat UNIX-LISTEN:/tmp/avahi_socket/avahi.sock,fork,user=1000,group=1000,mode=777 UNIX-CONNECT:/var/run/avahi-daemon/socket
```

> Note: The best solution is to leverage the HOST Avahi system to advertise device information and utilize a rootless container for reading/BROWSING network information. This allows for changing permissions from `mode=777` to `mode=444`, significantly enhancing security.

Now `/tmp/dbus_socket/dbus.sock` and `/tmp/avahi_socket/avahi.sock` _f_ile can be mounted to container.

```sh
docker run -it --rm  \  
-v /tmp/avahi_socket/avahi.sock:/var/run/avahi-daemon/socket \  
-v /tmp/dbus_socket/dbus.sock:/var/run/dbus/system_bus_socket \  
--name alpine_sock \  
--entrypoint ash \  
--privileged=true \  
--security-opt seccomp=unconfined \  
alpine
```

Now let’s install necessary libraries in `python`and try to obtain and broadcast **mDNS** information

```sh
/ # apk add python3 py3-dbus py3-avahi  py3-gobject3 avahi-tools   
fetch https://dl-cdn.alpinelinux.org/alpine/v3.19/main/armv7/APKINDEX.tar.gz  
fetch https://dl-cdn.alpinelinux.org/alpine/v3.19/community/armv7/APKINDEX.tar.gz  
(1/29) Installing dbus-libs (1.14.10-r0)  
(2/29) Installing libintl (0.22.3-r0)  
(3/29) Installing avahi-libs (0.8-r13)  
...  
...
```

Now we can use python programming to interact d-bus with dbus library.

## Example

Browsing mDNS data from rootless container

```Python
# http://avahi.org/wiki/PythonBrowseExample  
import dbus, avahi  
from dbus import DBusException  
from dbus.mainloop.glib import DBusGMainLoop  
from gi.repository import GObject  
# Looks for iTunes shares  
  
TYPE = "_http._tcp"  
  
def service_resolved(*args):  
    print ('service resolved')  
    print ('name:', args[2])  
    print ('address:', args[7])  
    print ('port:', args[8])  
  
def print_error(*args):  
    print ('error_handler')  
    print (args[0])  
  
def myhandler(interface, protocol, name, stype, domain, flags):  
    print ("Found service '%s' type '%s' domain '%s' " % (name, stype, domain))  
    if flags & avahi.LOOKUP_RESULT_LOCAL:  
            # local service, skip  
            pass  
    server.ResolveService(interface, protocol, name, stype,   
        domain, avahi.PROTO_UNSPEC, dbus.UInt32(0),   
        reply_handler=service_resolved, error_handler=print_error)  
  
loop = DBusGMainLoop()  
  
bus = dbus.SystemBus(mainloop=loop)  
  
server = dbus.Interface( bus.get_object(avahi.DBUS_NAME, '/'),  
        'org.freedesktop.Avahi.Server')  
  
sbrowser = dbus.Interface(bus.get_object(avahi.DBUS_NAME,  
        server.ServiceBrowserNew(avahi.IF_UNSPEC,  
            avahi.PROTO_UNSPEC, TYPE, 'local', dbus.UInt32(0))),  
        avahi.DBUS_INTERFACE_SERVICE_BROWSER)  
  
sbrowser.connect_to_signal("ItemNew", myhandler)  
  
GObject.MainLoop().run()
```

Advertising service from rootless container to outside of host.

```Python
import avahi
import dbus
from time import sleep


class ServiceAnnouncer:
    def __init__(self, name, service, port, txt):
        bus = dbus.SystemBus()
        server = dbus.Interface(bus.get_object(avahi.DBUS_NAME, avahi.DBUS_PATH_SERVER), avahi.DBUS_INTERFACE_SERVER)
        group = dbus.Interface(bus.get_object(avahi.DBUS_NAME, server.EntryGroupNew()),
                               avahi.DBUS_INTERFACE_ENTRY_GROUP)
        self._service_name = name
        index = 1
        while True:
            try:
                group.AddService(avahi.IF_UNSPEC, avahi.PROTO_INET, 0, self._service_name, service, '', '', port, avahi.string_array_to_txt_array(txt))
            except dbus.DBusException: # name collision -> rename
                index += 1
                self._service_name = '%s #%s' % (name, str(index))
            else:
                break
        group.Commit()
    def get_service_name(self):
        return self._service_name


if __name__ == '__main__':
    announcer = ServiceAnnouncer('Test Service', '_test._tcp', 12345, ['foo=bar', '42=true'])
    print (announcer.get_service_name())

    sleep(60)
```

Here we go.

# **Info**

> An alternative approach involves developing an application within a host that scans and broadcasts mDNS data, pushing it to MQTT over `127.0.0.1:1883` (if available). Rootless containers can then connect and manage data read/write functions. As previously mentioned, it’s essential to maintain consistent message lengths for MQTT and ensure stable performance across defined scenarios.

**Referances**:

Link to [rootless containers](https://rootlesscontaine.rs/)

Link to [slirp4netns](https://github.com/rootless-containers/slirp4netns)

Link to [socat](https://linux.die.net/man/1/socat)

Link to [mDNS](https://en.wikipedia.org/wiki/Multicast_DNS#:~:text=In%20computer%20networking%2C%20the%20multicast,Domain%20Name%20System%20\(DNS\).) and [mDNS-RFC](https://www.rfc-editor.org/rfc/rfc6762.html)

Link to [Linux D-BUS](https://www.freedesktop.org/wiki/Software/dbus/)

Link to [Avahi](https://avahi.org/)