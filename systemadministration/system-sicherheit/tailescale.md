**Port durchreichen**

tobkern@srv3:~$ sudo tailscale funnel --bg --https=8443 localhost:9090
Available on the internet:

```sh
https://srv3.tail0c330.ts.net:8443/
|-- proxy http://127.0.0.1:9090
```

Funnel started and running in the background.
To disable the proxy, run: tailscale funnel --https=8443 off

**DNS von tailscale verhindern**

`tailscale set --accept-dns=false`

**Expose an HTTP server running at 127.0.0.1:3000 in the foreground**

`tailscale funnel 3000`

**Expose an HTTP server running at 127.0.0.1:3000 in the background**

`tailscale funnel --bg 3000`

**Expose an HTTPS server with invalid or self-signed certificates at https://localhost:8443**

`tailscale funnel https+insecure://localhost:8443`

**Funnel started and running in the background. To disable the proxy, run**

`tailscale funnel --https=443 off`

For more examples and use cases visit our docs site https://tailscale.com/kb/1247/funnel-serve-use-cases

# Configuring Linux DNS

There are [an incredible number of ways](https://tailscale.com/blog/sisyphean-dns-client-linux) to configure DNS on Linux.

If you're using both NetworkManager and systemd-resolved (as in common in many distros), you'll want to make sure that `/etc/resolv.conf` is a symlink to `/run/systemd/resolve/stub-resolv.conf`. That should be the default. If not,

```sh
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

When NetworkManager sees that symlink is present, its default behavior is to use systemd-resolved and not take over the resolv.conf file.

After fixing, restart everything:

```sh
sudo systemctl restart systemd-resolved
sudo systemctl restart NetworkManager
sudo systemctl restart tailscaled
```

### [DHCP dhclient overwriting `/etc/resolv.conf`](https://tailscale.com/kb/1188/linux-dns#dhcp-dhclient-overwriting-etcresolvconf)

Without any DNS management system installed, DHCP clients like `dhclient` and programs like `tailscaled` have no other options than rewriting the `/etc/resolv.conf` file themselves, which results in them sometimes fighting with each other. (For instance, a DHCP renewal rewriting the `resolv.conf` resulting in loss of MagicDNS functionality.)

Possible workarounds are to use `resolvconf` or `systemd-resolved`. [Issue 2334](https://github.com/tailscale/tailscale/issues/2334) tracks making Tailscale react to other programs updating `resolv.conf` so Tailscale can add itself back.

# tailscale create acme cert

```
tailscale cert $(hostname -f)
```