## Verstehen von Rate Limit Directives in NGINX

Key NGINX Rate Limiting Directives

NGINX konfiguriert die Geschwindigkeitsbegrenzung mit bestimmten Direktiven, die jeweils eine einzigartige Funktion erfüllen:

- **limit-req-zone:** Die Direktive in NGINX richtet eine gemeinsame shared memory zone für die Speicherung von rate-limiting Informationen ein. Im http-Kontext positioniert, bestimmt es den zulässigen Satz von Anfragen und legt effektiv die die Rathe von requets für Geschwindigkeitsbegrenzung fest.
- **limit-req:** Die im Standortkontext verwendete Grenzwertregelung erzwingt die im Ortsumlage gesetzte Preislimit durch limit-req-zone. Es wendet diese Einschränkungen auf bestimmte Orte oder URLs an und steuert so die Zugriffshäufigkeit zu diesen Endpunkten.
- **limit-req-status:** Die im Standortkontext positionierte Richtlinie limit-req-status gibt den HTTP-Statuscode an, der zurückgegeben wird, wenn die Anfragerate das Limit überschreitet. Dieser Feedback-Mechanismus ist entscheidend für die Information von Benutzern oder automatisierten Systemen, wenn sie die zulässige Anfragefrequenz überschreiten.

```nginx
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=5r/s;

    server {
        listen 80;
        server_name example.com;
        
        location / {
            limit_req zone=mylimit burst=10;
            proxy_pass http://backend;
        }
    }
}
```

- **limit_req_zone**: Defines a memory zone (mylimit) for storing the rate limit states with a size of 10 megabytes. It sets the rate limit to 2 requests per second (2r/s).
- **server block**: Listens on port 80 for incoming traffic to example.com.
- **location block**: Applies the rate limit to the root URL (/). The burst parameter allows a burst of up to 5 requests, providing flexibility for short spikes in traffic. The proxy_pass directive forwards the request to the specified backend server.
- **Burst Parameter**: The burst=5 setting in the limit_req directive allows a short-term increase in requests, up to 5 additional requests above the set rate. This feature is crucial in handling sudden, legitimate increases in traffic without impacting user experience.
- **Proxy Pass**: The proxy_pass http://backend; directive forwards the traffic to a backend server. This is commonly used in load-balancing scenarios or when NGINX acts as a reverse proxy.

Erklärung der Konfiguration:

Ziele 1 ) Mit __http__ wird beschrieben  das die Konfiguration im abschnitt http definiert werden soll. In der Praxis macht das auch so sinn nicht nur weil es so von Hersteller vorgeschlagen ist sondern auch weil es intuitiv wieder zu finden ist.
Es waöre aber auch möglich eigene Variablen oder Templates zu definieren etc. die man dann wiederverwenden kann da es aber wenig übersichtlich ist, sind includes mit einer Verzeichnisstruktur einfacher zu nutzen und man 
Ausfürliches ist in der officelen Seite zu finden [](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html)

_limit_req_zone_ ist die Direktive dir wir zur defenierung eines Schlüssels verwenden .
_$binary_remote_addr_ ist die Single IP des Quellsystems im Binären Format .
_zone=mylimit:10m_ Zone defeniert die zone mit dem Namen mylimit mit dem wert 10m 
_rate=5r/s_ rate defeniert 5r/s


```nginx

http {
    limit_req_zone $binary_remote_addr zone=low:10m rate=1r/s;
    limit_req_zone $binary_remote_addr zone=high:10m rate=10r/s;

    server {
        listen 80;
        server_name example.com;

        location /api/low {
            limit_req zone=low burst=3;
            proxy_pass http://backend;
        }

        location /api/high {
            limit_req zone=high burst=20;
            proxy_pass http://backend;
        }
    }
}

```

**Whitelisting IPs from Rate Limiting**

```nginx
http {
    geo $rate_limit {
        default 1;
        192.168.0.0/24 0;
        10.0.0.0/8 0;
    }

    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=5r/s;

    server {
        listen 80;
        server_name example.com;

        location / {
            if ($rate_limit) {
                limit_req zone=mylimit burst=10;
            }

            proxy_pass http://backend;
        }
    }
}
```


**Using curl for Simple Request Testing**

```bash
for i in {1..10}; do curl -I http://example.com; done
```

**Benchmarking Server Responses with Apache Bench**

```bash
ab -n 100 -c 10 http://example.com/
```


## Multi-Domain Server

```nginx
server {
    listen 80;
    server_name example1.com example2.com;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";

    location / {
        # Configuration details
    }
}
```

## Server with SSL Termination

```nginx
server {
    listen 443 ssl;
    server_name example.com;
    ssl_certificate /path/to/certificate;
    ssl_certificate_key /path/to/private/key;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";

    location / {
        # Configuration details
    }
}
```


## NGINX Rewrite Rules

```nginx
server {
    listen 80;
    server_name example.com;
    location /products {
        rewrite ^/products/([0-9]+)/([0-9]+)$ /product?id=$1&page=$2 last;
    }
}
```

**Streamlining URLs: Duplicate Slash Removal in NGINX Rewrite Rules**

```nginx
server {
    server_name example.com;
    if ($request_uri ~* "\/\/") {
        rewrite ^/(.*)/$ /$1 permanent;
    }
}
```

**Directory Redirection: NGINX Rewrite Rules in Action**

```nginx
location ^~ /old-directory/ {
    rewrite ^/old-directory/(.*)$ /new-directory/$1 permanent;
}
```

**Query String Manipulation Using Rewrite Rules in NGINX**

```nginx
if ($args ~ "^id=(.*)&lang=(.*)") {
    set $id $1;
    set $lang $2;
    rewrite ^/page.php$ /page/$lang/$id? permanent;
}
```

**Ensuring Uniform URLs: Trailing Slash in NGINX Rewrite Rules**

```nginx
rewrite ^([^.]*[^/])$ $1/ permanent;
```

**Method-Based Redirection: Employing Rewrite Rules in NGINX**

```nginx
if ($request_method = POST ) {
    return 301 https://example.com$request_uri;
}
```

**Protecting Images with NGINX Rewrite Rules**

```nginx
location ~ .(gif|png|jpe?g)$ {
    valid_referers none blocked ~.google. ~.bing. ~.yahoo. example.com *.example.com;
    if ($invalid_referer) {
        rewrite ^/images/(.*)$ /stop-hotlinking.$1 last;
    }
}
```

**Enforcing Lowercase URLs for Consistency**

```nginx
location ~ [A-Z] {
    rewrite ^(.*)$ $scheme://$host$1 lowercase;
}
```

**Handling Changes in URL Structure**

```nginx
server {
    listen 80;
    server_name example.com;
    
    location ~* ^/oldpath/(.*) {
        rewrite ^/oldpath/(.*)$ /newpath/$1 permanent;
    }
}
```


**Creating Clean URLs for CMS Platforms**

```nginx
location / {
    try_files $uri $uri/ @extensionless-php;
    index index.html index.htm index.php;
}

location ~ \.php$ {
    try_files $uri =404;
}

location @extensionless-php {
    rewrite ^(/[^.]*[^/])$ $1.php last;
}
```

## Nginx Reverse Proxy

**Implementing Proxy Header Settings in Nginx**

```nginx
server {
    listen 80;
    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://backendserver_address;
    }
}
```

- `X-Real-IP`: This header is used to pass the original client’s IP address to the backend server.
- `X-Forwarded-For`: Similar to `X-Real-IP`, this header carries a list of all the servers the request has traversed through, including the client’s IP.
- `X-Forwarded-Proto`: This header indicates the protocol (HTTP or HTTPS) used in the client’s original request.

## NGINX URL Redirects

- **301 Redirects**:
    - Purpose: Indicate a permanent URL change.
    - Use Case: Ideal when a web page has moved permanently to a new location.
    - SEO Impact: Transfers SEO rankings from the old URL to the new one, preserving search engine credibility.
- **302 Redirects**:
    - Purpose: Signify a temporary URL change.
    - Use Case: Useful during site maintenance or temporary content shifts, signaling a future return to the original URL.
    - SEO Impact: Tells search engines to keep the original URL indexed, as the change is not permanent.
- **303 Redirects**:
    - Purpose: Manage form submissions by preventing data re-submission on page refresh.
    - Use Case: Primarily employed in situations involving form submission confirmations.
    - User Experience: Enhances user experience by preventing duplicate form submissions and potential data errors.


**Redirect with Nginx Server Blocks**

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 $scheme://www.example.com$request_uri;
}

server {
    listen 80;
    server_name www.example.com;
    # Host your website content here
}
```

```nginx
server {
    listen 80;
    server_name www.example.com;
    rewrite ^(.*)$ $scheme://example.com$1 permanent;
}
```

**Conditional Redirection: IP Address-Specific Response**

```nginx
server {
    listen 80;
    server_name yourwebsite.com;

    location / {
        if ($remote_addr = "203.0.113.5") {
            rewrite ^ /special-landing-page.html last;
        }

        if ($remote_addr != "203.0.113.5") {
            rewrite ^ /default-landing-page.html last;
        }
    }
}
```

**Dynamic Content Delivery: User Agent-Based Customization**

```nginx
server {
    listen 80;
    server_name yourwebsite.com;

    location / {
        if ($http_user_agent ~* (msie|trident)) {
            root /var/www/html/ie;
        }

        if ($http_user_agent !~* (msie|trident)) {
            root /var/www/html/non-ie;
        }
    }
}
```

**Securing Specific Routes: Conditional Security Headers**

```nginx
server {
    listen 80;
    server_name yourwebsite.com;

    location /secure-area {
        if ($scheme = https) {
            add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        }
    }
}
```

