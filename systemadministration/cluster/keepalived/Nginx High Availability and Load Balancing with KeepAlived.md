# Objective

Install and configure 2 Nginx web services, that do high availability, load balancing and proxy (forward) in-bound request to internal application (running inside of Kubernetes cluster)
# Environment

![](1_AG6df69ccZ2AKPsAlvivHw.webp)
Legend: **external IP** = public internet IP
# Architecture Diagram

The active-passive architecture. The Active-active is described later in this doc)

![](1_WgaSZ7BIppfavqd1QFRbjg.webp)

# Installation on Ubuntu

`sudo apt install nginx keepalived`
# Configure Nginx

The following is part of `/etc/nginx/nginx.conf`

```c
include /etc/nginx/conf.d/*.conf;  
 include /etc/nginx/sites-enabled/*; merge_slashes off; # set client body size to 2M #  
 client_max_body_size 2M; upstream app.domain.com {  
         server 10.30.10.9:31399;  
         server 10.30.10.5:31399;  
         server 10.30.10.6:31399;  
         server 10.30.10.24:31399;  
         server 10.30.10.22:31399;  
         server 10.30.10.17:31399;  
         server 10.30.10.25:31399;  
         server 10.30.10.32:31399;  
         server 10.30.10.8:31399;  
         server 10.30.10.23:31399;  
         server 10.30.10.15:31399;  
         server 10.30.10.42:31399;  
 } server {  
     ssl_certificate /home/ubuntu/certificates/cert.txt;  
     ssl_certificate_key /home/ubuntu/certificates/key.txt;  
     # include /etc/letsencrypt/options-ssl-nginx.conf;  
     # ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;     listen       443 ssl;  
     server_name  app.domain.com;     location / {  
         # proxy_pass [https://app.hostname.net:31712/;](https://iot.hostname.net:31712/;)  
         proxy_pass [https://app.domain.com/;](https://app.domain.com/;)  
     }     location /api/v1/wsock/websocket {  
         proxy_pass [https://app.domain.com;](https://app.domain.com;)  
         proxy_http_version 1.1;  
         proxy_set_header Upgrade $http_upgrade;  
         proxy_set_header Connection "upgrade";  
         proxy_read_timeout 86400;  
    }  
 }
```

> Port `31399` : `nodePort` of app exposed on Kubernetes cluster  
> `_client_max_body_size 2M;_`is to set client body size up to 2M

# Configure KeepAlived

- `/etc/keepalived/keepalived.conf`

On web01

```c
! Configuration File for keepalivedglobal_defs {  
...  
}vrrp_script check_nginx {  
    script "/etc/keepalived/check_nginx.sh"  
    interval 2  
    weight 50  
}vrrp_instance VI_1 {  
    state MASTER  
    interface ens3  
    virtual_router_id 50  
    priority 110  
    advert_int 1  
    virtual_ipaddress {  
	10.30.10.100  
    }  
    track_script {  
        check_nginx  
    }  
}
```

On web02

```c
! Configuration File for keepalivedglobal_defs {  
...  
}vrrp_script check_nginx {  
    script "/etc/keepalived/check_nginx.sh"  
    interval 2  
    weight 50  
}vrrp_instance VI_1 {  
    state BACKUP  
    interface ens3  
    virtual_router_id 50  
    priority 100  
    advert_int 1  
    virtual_ipaddress {  
	10.30.10.100  
    }  
    track_script {  
	check_nginx  
    }  
}
```

`/etc/keepalived/check_nginx.sh`

```sh
#!/bin/sh
if [ -z "`/bin/pidof nginx`" ]; then  
	systemctl stop keepalived.service  
	exit 1  
fi# Change mode of the script  
sudo chmod +x /etc/keepalived/check_nginx.sh
```

> _Nginx_ **_should_** _start before keepalived starts!_

Check which node the floating IP is being attached to

```sh
$ curl [http://10.30.10.100](http://10.30.10.100) # or curl [http://1.2.3.6](http://1.2.3.6)<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>  
<style>  
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }  
</style>
</head>
<body>
<h1>Welcome to nginx web01!</h1>
<p>If you see this page, the nginx web server is successfully installed and  
working. Further configuration is required.</p><p>For online documentation and support please refer to  
<a href="http://nginx.org/">nginx.org</a>.<br/>  
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p><p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

You can see `web01`is serving
# Full HA of Nginx (active-active)

If there are 2 virtualIPs, bind them to both Nginx web server to maximize the utilization of 2 Nginx web servers simultaneously on production

![](1_IiMwimMaoQ9adCqvD3RoEg.webp)

# PlantUML source code

```plantuml  
cloud freeWorldrectangle “dnsQuery: app.domain.com” as dns  
rectangle “publicIP: 1.2.3.6” as pubip  
rectangle “floatingIP: 10.30.10.100” as fip  
rectangle “web01: 10.30.10.19” as m01  
rectangle “web02: 10.30.10.11” as m02freeWorld → dns  
dns → pubip  
pubip → fip:nat  
fip → m01  
fip → m02  
```