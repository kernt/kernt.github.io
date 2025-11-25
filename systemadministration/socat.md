---
tags:
  - socat
  - system-administration
---

```plaintext
socat [options] <address> <address>
```

 **Verbindung via TCP port 80  local remote remote system**

```shell
socat - TCP4:www.example.com:80
```

**Use `socat` as a TCP port forwarder single connection**

```shell
socat TCP4-LISTEN:81 TCP4:192.168.1.10:80
```

**Für merere Verbindungen ,  benutze die `fork` option wie in dem Beispiel unten:**

```shell
socat TCP4-LISTEN:81,fork,reuseaddr TCP4:TCP4:192.168.1.10:80
```

**This command starts `socat` and configures it to listen by using port 3307.**

```shell
socat TCP-LISTEN:3307,reuseaddr,fork UNIX-CONNECT:/var/lib/mysql/mysql.sock &
```
