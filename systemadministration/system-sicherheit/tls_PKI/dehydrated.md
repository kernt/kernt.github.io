---
tags:
  - sicherheit
  - acme
  - dehydrated
---
## dehydrated Konfiguration

registr with AMCE-Server (Let's Encrypt)

```sh
dehydrated --register --accept-terms
```

Configure nginx-ssl

```
server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  hostXX.example.com;
        root         /usr/share/nginx/html;

        #ssl_certificate "/etc/pki/nginx/server.crt";
        #ssl_certificate_key "/etc/pki/nginx/private/server.key";
        ssl_certificate "/etc/dehydrated/certs/hostXX.example.com/fullchain.pem";
        ssl_certificate_key "/etc/dehydrated/certs/hostXX.example.com/privkey.pem";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

**INFO: Using main config file /etc/dehydrated/config**

```sh
#############################################################
# This is the main config file for dehydrated               #
#                                                           #
# This is the default configuration for the Debian package. #
# To see a more comprehensive example, see                  #
# /usr/share/doc/dehydrated/examples/config                 #
#                                                           #
# For details please read:                                  #
# /usr/share/doc/dehydrated/README.Debian                   #
#############################################################

CONFIG_D=/etc/dehydrated/conf.d
BASEDIR=/var/lib/dehydrated
WELLKNOWN="${BASEDIR}/acme-challenges"
DOMAINS_TXT="/etc/dehydrated/domains.txt"
```

cp /usr/share/doc/dehydrated/examples/config /etc/dehydrated/conf.d

**INFO: Using additional config file /etc/dehydrated/conf.d/50_dehydrated-hook-ddns-tsig.sh**

`wget  https://raw.githubusercontent.com/dehydrated-io/dehydrated/refs/heads/master/docs/examples/domains.txt`

https://raw.githubusercontent.com/dehydrated-io/dehydrated/refs/heads/master/docs/examples/hook.sh


cronjob

`5 2 * * 6 /usr/bin/dehydrated -c -f /etc/dehydrated/config`

Quellen:

https://www.laub-home.de/wiki/Let%27s_Encrypt_mit_dehydrated_BASH_Script

