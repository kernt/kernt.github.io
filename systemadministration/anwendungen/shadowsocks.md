---
tags:
  - dns
  - socks5
  - shadowsocks
---
# cli Implementationen

||[ss](https://github.com/shadowsocks/shadowsocks)|[ss-libev](https://github.com/shadowsocks/shadowsocks-libev)|[go-ss2](https://github.com/shadowsocks/go-shadowsocks2)|[ss-rust](https://github.com/shadowsocks/shadowsocks-rust)|
|---|---|---|---|---|
|[TCP Fast Open](https://en.wikipedia.org/wiki/TCP_Fast_Open)|✓|✓|✗|✓|
|[Multiuser](https://github.com/shadowsocks/shadowsocks/wiki/Configure-Multiple-Users)|✓|✓|✗|✓|
|[Management API](https://github.com/shadowsocks/shadowsocks/wiki/Manage-Multiple-Users)|✓|✓|✗|✓|
|Redirect mode|✗|✓|✓|✓|
|Tunnel mode|✓|✓|✓|✓|
|UDP Relay|✓|✓|✓|✓|
|[MPTCP](https://en.wikipedia.org/wiki/Multipath_TCP)|✗|✓|✗|✓|
|[AEAD ciphers](https://shadowsocks.org/doc/aead.html)|✓|✓|✓|✓|
|[Plugin](https://shadowsocks.org/doc/sip003.html)|✗|✓|✗|✓|
|[Plugin UDP (Experimental)](https://github.com/shadowsocks/shadowsocks-org/issues/180)|✗|✗|✗|✓|

## [shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust) ss-rust

Zu Installation sollte man das Native Projekt  [crates](https://crates.io/)verwenden alternativ bietet sich noch 
[[docker]] oder [[kubernetes reference]] an.
## docker und [shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust)

```sh
docker run --name sslocal-rust \
  --restart always \
  -p 1080:1080/tcp \
  -v /path/to/config.json:/etc/shadowsocks-rust/config.json \
  -dit ghcr.io/shadowsocks/sslocal-rust:latest

docker run --name ssserver-rust \
  --restart always \
  -p 8388:8388/tcp \
  -p 8388:8388/udp \
  -v /path/to/config.json:/etc/shadowsocks-rust/config.json \
  -dit ghcr.io/shadowsocks/ssserver-rust:latest
```

## Kubernetes und [shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust)

**Using kubectl**

`kubectl apply -f https://github.com/shadowsocks/shadowsocks-rust/raw/master/k8s/shadowsocks-rust.yaml`

**Using helm**

`helm install my-release k8s/chart -f my-values.yaml`

Unten stehet die Konfiguration deren Werte angepasst werden müssen

```yml
# This is the shadowsocks config which will be mount to /etc/shadowocks-rust.
# You can put arbitrary yaml here, and it will be translated to json before mounting.
servers:
- server: "::"
  server_port: 8388
  service_port: 80 # the k8s service port, default to server_port
  password: mypassword
  method: aes-256-gcm
  fast_open: true
  mode: tcp_and_udp
  # plugin: v2ray-plugin
  # plugin_opts: server;tls;host=github.com

# Whether to download v2ray and xray plugin.
downloadPlugins: false

# Name of the ConfigMap with config.json configuration for shadowsocks-rust.
configMapName: ""

service:
  # Change to LoadBalancer if you are behind a cloud provider like aws, gce, or tke.
  type: ClusterIP

# Bind shadowsocks port port to host, i.e., we can use host:port to access shawdowsocks server.
hostPort: false

replicaCount: 1

image:
  repository: ghcr.io/shadowsocks/ssserver-rust
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "latest"
```

## [shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust) build

 Ram >= 2GiB
```sh
cargo build --release
```
# ## [shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust) benutzen

**aes-128-gcm genarieren**

`ssservice genkey -m "aes-128-gcm"`

Erstellen einer ShadowSocks Konfiguration

```json
{
    "server": "my_server_ip",
    "server_port": 8388,
    "password": "rwQc8qPXVsRpGx3uW+Y3Lj4Y42yF9Bs0xg1pmx8/+bo=",
    "method": "aes-256-gcm",
    // ONLY FOR `sslocal`
    // Delete these lines if you are running `ssserver` or `ssmanager`
    "local_address": "127.0.0.1",
    "local_port": 1080
}
```

[shadowsocks](https://shadowsocks.org/)