---
tags:
  - apache
---
# apache heimdall

```sh
<VirtualHost *:80>
    RewriteEngine On
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>

<VirtualHost *:443>
	ServerAdmin webmaster@example.com
    ServerName admin.example.com
    ServerAlias www.admin.example.com

	ErrorLog ${APACHE_LOG_DIR}/error-admin-example-com.log
	CustomLog ${APACHE_LOG_DIR}/access-admin-example-com.log combined

	SSLEngine on
	SSLCertificateFile	/opt/ssl/example.com/cert.pem
	SSLCertificateKeyFile /opt/ssl/example.com/privkey.pem

    SSLProxyEngine On
    SSLProxyVerify none
    SSLProxyCheckPeerCN off
    SSLProxyCheckPeerName off
    SSLProxyCheckPeerExpire off

    ProxyPreserveHost On
    RequestHeader set X-Forwarded-Proto "https"
    ProxyPass / http://172.20.3.21:81/
    ProxyPassReverse / http://172.20.3.21:81/

    RewriteEngine on
    RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC]
    RewriteCond %{HTTP:CONNECTION} ^Upgrade$ [NC]
    RewriteRule .* ws://localhost:81%{REQUEST_URI} [P]
# Protocols h2 http/1.1
</VirtualHost>
```