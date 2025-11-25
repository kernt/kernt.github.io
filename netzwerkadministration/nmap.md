---
tags:
  - networking
---
# Nmap

nc NetCat Benutzen

**nc -v  172.17.42.1  4243**

#### Operating System Scan:

To initiate an operating system scan, you can use the following command:

```bash
nmap -O --osscan-guess [IP address] or [website address]
```

#### Port Specification and Scan Order:

To initiate a custom port scan, you can use the -p flag followed by the ports you wish to scan:

```bash
nmap –p 80,443,8080,9090 [IP address] or [website address]
```

#### Services Scan:

To initiate a services scan, you can use the following command:

```bash
nmap -sV [IP address] or [website address]
```

#### TCP SYN Scan:

To initiate a TCP SYN scan, you can use the following command:

```bash
nmap -sS [IP address] or [website address]
```

