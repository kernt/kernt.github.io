---
tags:
  - bash
  - konsole
  - ss
  - system-administration
  - gnu-tools
---
# ss network Exempels

**listen and exemled connections**

`ss -u -a`

oder

`ss -tulpe`

**Connaction statics**

`ss -s`

**established connect on ipv6**

`ss -t6 state established`

**filter for destination net**

`ss dst 70.134.160.252/16`

**filter for destination port**

`ss dst 70.134.160.252:1335`

**dport only**

`ss -nt dport = :80`

**listen der ports**
```sh
ss -tlpn | awk '{print $1" "$2" "$3" "$4}'
```

List all connections
```sh
ss | less
```

Filter out tcp,udp or unix connections

```sh
ss -t
```

List all udp connections
```sh
ss -ua
```

Show only listening sockets

```sh
ss -ltn
# ss -lun
```

Print process name and pid
```sh
ss -ltp
```

Print summary statistics

```sh
ss -s
```

Filter connections by port number

```sh
ss -at '( dport = :22 or sport = :22 )'
```

Extended output of socket connections

```sh
ss -elt
```

View memory usage of socket connections

```sh
ss -mt
```

Kill IPv4 / IPv6 Socket Connection

```sh
ss -4
```
### By port number

On a web server it makes sense to see the open connections on HTTPS (port 443).

`ss -nt sport = :443`

To query multiple ports

`ss -nt '( sport = :443 or sport = :80 )'`

A slightly shorter version is by defining the side ‘src’ (source) or ‘dst’ (destination)

`ss -nt '( src :443 or src :80 )'`

### By destination

To see active connections with a specific destination, define an expression including the IP address or address. For example to see connections on the 192.168.x.x network:

`ss dst 192.168/16`

### See connection and transmission speed

The --info option reveals a lot of specifics, including the send and receive speed. Interesting fields

- send
- pacing_rate
- delivery_rate

```sh
ss --info dst 192.168.1.11 dport 2049
```

### TLS/SSL version and Ciphers

Some protocol specifics can be displayed as well. In this example we see TLSv1.3 with the cipher AES-GCM-256 being used.

```sh
ss -piment
```

## Monitoring connections

To see if there is traffic on a system, use the --events option. It will display the sockets that are destroyed. Or in other words, the connections that are closed. A great way to see the amount of traffic and great for monitoring or when to do system maintenance.

`ss -n --events`

## Manually close a connection

The [ss](https://linux-audit.com/cheat-sheets/ss/) command can also be used to close active connections. It works for IPv4 and IPv6 and can be used with the --kill option. Typically you want to combine this with a specific IP address and optionally a port.

`ss --kill dst 192.168.1.123 dport = 80`

## See timer information

Some services like SSH want to stay connected. They send a _keepalive_ signal now and then to keep the connection active. For TCP connections, we can request timer information and see when a timer expires.

`ss --options --tcp`

The value displayed after ‘keepalive’ refers to the expiry time. So when renewing it, the values typically go down.