

**/etc/systemd/system/certbot.service**

```sh
[Unit]
Description=Let's Encrypt renewal via Certbot
Documentation=https://letsencrypt.readthedocs.io/en/latest/

[Service]
Type=oneshot
ExecStart=/usr/bin/certbot renew --quiet --agree-tos

```

**/etc/systemd/system/certbot.timer**

```sh
[Unit]
Description=Twice daily renewal of Let's Encrypt's certificates

[Timer]
OnCalendar=0/12:00:00
RandomizedDelaySec=1h
Persistent=true

[Install]
WantedBy=timers.target
```

`systemctl daemon-reload`

`systemctl enable --now certbot.timer`

```bash
#!/bin/bash
export FQDN="tuxsupport.de"
cp /etc/letsencrypt/live/$FQDN/fullch```bash
cp /etc/letsencrypt/live/$FQDN/privkey.pem /etc/cockpit/ws-certs.d/$FQDN.key
```


## Cockpit Einstellungen

Zertifikate bekannt geben

```bash
remotectl certificate
certificate: /etc/cockpit/ws-certs.d/$FQDN.crt
```

**/etc/letsencrypt/renewal-hooks/post/001-restart-cockpit.sh**

```sh
#!/usr/bin/env bash

echo "SSL certificates renewed"

cp /etc/letsencrypt/live/$FQDN/fullchain.pem /etc/cockpit/ws-certs.d/$FQDN.crt`
cp /etc/letsencrypt/live/$FQDN/privkey.pem /etc/cockpit/ws-certs.d/$FQDN.key
chown cockpit-ws:cockpit-ws /etc/cockpit/ws-certs.d/$FQDN.crt /etc/cockpit/ws-certs.d/$FQDN.key

echo "Restarting Cockpit"
systemctl restart cockpit

```

**Certbot mit einem anderem Port benutzen**


## Certbot Konfig

`certbot --config cli.ini`

```sh
# This is an example of the kind of things you can do in a configuration file.
# All flags used by the client can be configured here. Run Certbot with
# "--help" to learn more about the available options.
#
# Note that these options apply automatically to all use of Certbot for
# obtaining or renewing certificates, so options specific to a single
# certificate on a system with several certificates should not be placed
# here.

# Use ECC for the private key
key-type = ecdsa
elliptic-curve = secp384r1

# Use a 4096 bit RSA key instead of 2048
rsa-key-size = 4096

# Uncomment and update to register with the specified e-mail address
# email = foo@example.com

# Uncomment to use the standalone authenticator on port 443
# authenticator = standalone

# Uncomment to use the webroot authenticator. Replace webroot-path with the
# path to the public_html / webroot folder being served by your web server.
# authenticator = webroot
# webroot-path = /usr/share/nginx/html

# Uncomment to automatically agree to the terms of service of the ACME server
# agree-tos = true

# An example of using an alternate ACME server that uses EAB credentials
# server = https://acme.sectigo.com/v2/InCommonRSAOV
# eab-kid = somestringofstuffwithoutquotes
# eab-hmac-key = yaddayaddahexhexnotquoted
```

## certbot manual

certbot certonly --manual --preferred-challenges=http --manual-auth-hook /path/to/http/authenticator.sh --manual-cleanup-hook /path/to/http/cleanup.sh -d secure.example.com

### Certbot cleanup

```sh
#!/bin/bash

# Get your API key from https://www.cloudflare.com/a/account/my-account
API_KEY="your-api-key"
EMAIL="tobkern1980@gmail.com"

if [ -f /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID ]; then
        ZONE_ID=$(cat /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID)
        rm -f /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID
fi

if [ -f /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID ]; then
        RECORD_ID=$(cat /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID)
        rm -f /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID
fi

# Remove the challenge TXT record from the zone
if [ -n "${ZONE_ID}" ]; then
    if [ -n "${RECORD_ID}" ]; then
        curl -s -X DELETE "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID" \
                -H "X-Auth-Email: $EMAIL" \
                -H "X-Auth-Key: $API_KEY" \
                -H "Content-Type: application/json"
    fi
fi
```


## certbot Authenticator 


authenticator.sh

```sh
#!/bin/bash

# Get your API key from https://www.cloudflare.com/a/account/my-account
API_KEY="your-api-key"
EMAIL="tobkern1980@gmail.com"

# Strip only the top domain to get the zone id
DOMAIN=$(expr match "$CERTBOT_DOMAIN" '.*\.\(.*\..*\)')

# Get the Cloudflare zone id
ZONE_EXTRA_PARAMS="status=active&page=1&per_page=20&order=status&direction=desc&match=all"
ZONE_ID=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones?name=$DOMAIN&$ZONE_EXTRA_PARAMS" \
     -H     "X-Auth-Email: $EMAIL" \
     -H     "X-Auth-Key: $API_KEY" \
     -H     "Content-Type: application/json" | python -c "import sys,json;print(json.load(sys.stdin)['result'][0]['id'])")

# Create TXT record
CREATE_DOMAIN="_acme-challenge.$CERTBOT_DOMAIN"
RECORD_ID=$(curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
     -H     "X-Auth-Email: $EMAIL" \
     -H     "X-Auth-Key: $API_KEY" \
     -H     "Content-Type: application/json" \
     --data '{"type":"TXT","name":"'"$CREATE_DOMAIN"'","content":"'"$CERTBOT_VALIDATION"'","ttl":120}' \
             | python -c "import sys,json;print(json.load(sys.stdin)['result']['id'])")
# Save info for cleanup
if [ ! -d /tmp/CERTBOT_$CERTBOT_DOMAIN ];then
        mkdir -m 0700 /tmp/CERTBOT_$CERTBOT_DOMAIN
fi
echo $ZONE_ID > /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID
echo $RECORD_ID > /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID

# Sleep to make sure the change has time to propagate over to DNS
sleep 25
```

# certbot Plugins

## RFC 2136

```sh
# Target DNS server (IPv4 or IPv6 address, not a hostname)
dns_rfc2136_server = 192.0.2.1
# Target DNS port
dns_rfc2136_port = 53
# TSIG key name
dns_rfc2136_name = keyname.
# TSIG key secret
dns_rfc2136_secret = 4q4wM/2I180UXoMyN4INVhJNi8V9BCV+jMw2mXgZw/CSuxUT8C7NKKFs AmKd7ak51vWKgSl12ib86oQRPkpDjg==
# TSIG key algorithm
dns_rfc2136_algorithm = HMAC-SHA512
# TSIG sign SOA query (optional, default: false)
dns_rfc2136_sign_query = false
```

Kommandline Option --dns-rfc2136-credentials`

## Bind Konfiguration

**SHA512 TSIG key Genarieren**

`tsig-keygen -a HMAC-SHA512 keyname.`

**Bind Konfiguration**

```
key "keyname." {
  algorithm hmac-sha512;
  secret "4q4wM/2I180UXoMyN4INVhJNi8V9BCV+jMw2mXgZw/CSuxUT8C7NKKFs AmKd7ak51vWKgSl12ib86oQRPkpDjg==";
};

// adjust internal-addresses to suit your needs
acl internal-address { 127.0.0.0/8; 10.0.0.0/8; 192.168.0.0/16; 172.16.0.0/12; };

view "external" {
  match-clients { !internal-addresses; any; };

zone "example.com." IN {
  type master;
  file "named.example.com";
  update-policy {
    grant keyname. name _acme-challenge.example.com. txt;
  };
};

view "internal" {
  zone "example.com." IN {
    in-view external;
  };
};
```

[special-considerations-for-multiple-views-in-bind)](https://certbot-dns-rfc2136.readthedocs.io/en/stable/#special-considerations-for-multiple-views-in-bind)

## Plugin haproxy

```sh
useradd -s /bin/bash -m -d /opt/certbot certbot
usermod -a -G certbot haproxy  # Allow HAProxy access to the certbot certs
mkdir -p /opt/certbot/logs
mkdir -p /opt/certbot/config
mkdir -p /opt/certbot/.config/letsencrypt
usermod -a -G certbot [ADD YOUR USER HERE]
```


Besides the working directory.

```shell
mkdir -p /opt/certbot/.config/letsencrypt
cat <<EOF > /opt/certbot/.config/letsencrypt/cli.ini
work-dir=/opt/certbot/
logs-dir=/opt/certbot/logs/
config-dir=/opt/certbot/config
EOF
```

cat <<EOF >> /etc/sudoers
%certbot ALL=NOPASSWD: /bin/systemctl restart haproxy
EOF

haproxy conf

```nginx
cat <<EOF > /etc/haproxy/haproxy.cfg
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

    # Default ciphers to use on SSL-enabled listening sockets.
    # Cipher suites chosen by following logic:
    #  - Bits of security 128>256 (weighing performance vs added security)
    #  - Key exchange: EECDH>DHE (faster first)
    #  - Mode: GCM>CBC (streaming cipher over block cipher)
    #  - Ephemeral: All use ephemeral key exchanges
    #  - Explicitly disable weak ciphers and SSLv3
    ssl-default-bind-ciphers AES128+AESGCM+EECDH+SHA256:AES128+EECDH:AES128+AESGCM+DHE:AES128+EDH:AES256+AESGCM+EECDH:AES256+EECDH:AES256+AESGCM+EDH:AES256+EDH:-SHA:AES128+AESGCM+EECDH+SHA256:AES128+EECDH:AES128+AESGCM+DHE:AES128+EDH:AES256+AESGCM+EECDH:AES256+EECDH:AES256+AESGCM+EDH:AES256+EDH:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!3DES:!DSS
    #ssl-default-bind-options no-sslv3 no-tls-tickets force-tlsv12
    ssl-default-bind-options no-sslv3 no-tls-tickets
    ssl-dh-param-file /opt/certbot/dhparams.pem

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

frontend http-in
    # Listen on port 80
    bind \*:80
    # Listen on port 443
    # Uncomment after running certbot for the first time, a certificate
    # needs to be installed *before* HAProxy will be able to start when this
    # directive is not commented.
    #
    bind \*:443 ssl crt /opt/certbot/haproxy_fullchains/__fallback.pem crt /opt/certbot/haproxy_fullchains

    # Forward Certbot verification requests to the certbot-haproxy plugin
    acl is_certbot path_beg -i /.well-known/acme-challenge
    rspadd Strict-Transport-Security:\ max-age=31536000;\ includeSubDomains;\ preload
    rspadd X-Frame-Options:\ DENY
    use_backend certbot if is_certbot
    # The default backend is a cluster of 4 Apache servers that you need to
    # host.
    default_backend nodes

backend certbot
    log global
    mode http
    server certbot 127.0.0.1:8000

    # You can also configure separate domains to force a redirect from port 80
    # to 443 like this:
    # redirect scheme https if !{ ssl_fc } and [PUT YOUR DOMAIN NAME HERE]

backend nodes
    log global
    balance roundrobin
    option forwardfor
    option http-server-close
    option httpclose
    http-request set-header X-Forwarded-Port %[dst_port]
    http-request add-header X-Forwarded-Proto https if { ssl_fc }
    option httpchk HEAD / HTTP/1.1\r\nHost:localhost
    server node1 127.0.0.1:8080 check
    server node2 127.0.0.1:8080 check
    server node3 127.0.0.1:8080 check
    server node4 127.0.0.1:8080 check
    # If redirection from port 80 to 443 is to be forced, uncomment the next
    # line. Keep in mind that the bind \*:443 line should be uncommented and a
    # certificate should be present for all domains
    redirect scheme https if !{ ssl_fc }

EOF

systemctl restart haproxy
```

[certbot HAproxy](https://github.com/greenhost/certbot-haproxy)

### edirect HTTP to HTTPS

First, we’ll set up a rule to redirect all HTTP traffic to HTTPS, ensuring that all connections to your server are secure. This rule excludes requests to the `.well-known/acme-challenge/` directory, which is used by Certbot for domain validation during the certificate issuance process. Add the following configuration within the `<VirtualHost *:80>` block:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/\.well\-known/acme\-challenge/
RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
```

### Enable SSL and Specify Certificates

Next, within the `<VirtualHost *:443>` block, we’ll enable SSL and specify the paths to your SSL certificate and private key:

```apacheconf
SSLEngine on
SSLCertificateFile      /path/to/signed_cert_and_intermediate_certs
SSLCertificateKeyFile   /path/to/private_key
```

Replace `/path/to/signed_cert_and_intermediate_certs` with the path to your SSL certificate file, and `/path/to/private_key` with the path to your private key file.

### Enable HTTP/2

To improve performance, we’ll enable HTTP/2 if it’s available:

```apacheconf
Protocols h2 http/1.1
```

### Implement HSTS

We’ll also add a Strict-Transport-Security header to enforce secure connections:

```apacheconf
Header always set Strict-Transport-Security "max-age=63072000"
```

### Configure SSL Protocols and Ciphers

Next, we’ll specify which SSL protocols and ciphers should be used to ensure high security and compatibility:

```apacheconf
SSLProtocol                        all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite                  ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305
SSLHonorCipherOrder    off
SSLSessionTickets            off
```

### Enable OCSP Stapling

Finally, we’ll enable OCSP stapling, a feature that improves the performance of SSL negotiation while maintaining visitor privacy:

```bash
SSLUseStapling On
SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"
```

### Validate and Apply the Changes

Once you’re done, save and exit the file. It’s vital to validate your Apache configuration to ensure no syntax errors. Run this command to check:

```bash
sudo apachectl configtest
```

If there are no issues, apply the changes by reloading Apache:

```bash
sudo systemctl restart apache2
```