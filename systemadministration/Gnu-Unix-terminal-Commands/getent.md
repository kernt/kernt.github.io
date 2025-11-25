```sh
getent -s hosts:files ahosts localhost  
::1             STREAM localhost  
::1             DGRAM  
::1             RAW  
127.0.0.1       STREAM  
127.0.0.1       DGRAM  
127.0.0.1       RAW  
```
  
```sh
getent -s hosts:dns ahosts localhost  
::1             STREAM localhost  
::1             DGRAM  
::1             RAW  
127.0.0.1       STREAM  
127.0.0.1       DGRAM  
127.0.0.1       RAW  
```
  
```sh
getent -s hosts:myhostname ahosts localhost  
::1             STREAM localhost  
::1             DGRAM  
::1             RAW  
127.0.0.1       STREAM  
127.0.0.1       DGRAM  
127.0.0.1       RAW
```

**User ids suchen**

`getent passwd | grep -P ":x:(1005|1006)`

```sh
getent -s hosts:files ahosts localhost  
127.0.0.1       STREAM localhost  
127.0.0.1       DGRAM  
127.0.0.1       RAW  
127.0.0.1       STREAM  
127.0.0.1       DGRAM  
127.0.0.1       RAW  
```
  
```sh
getent -s hosts:dns ahosts localhost  
127.0.0.1       STREAM localhost  
127.0.0.1       DGRAM  
127.0.0.1       RAW  
```
  
```sh
getent -s hosts:myhostname ahosts localhost  
127.0.0.1       STREAM localhost  
127.0.0.1       DGRAM  
127.0.0.1       RAW
```

```sh
getent group sudo | awk -F: '{print $4}'
peter,james
```