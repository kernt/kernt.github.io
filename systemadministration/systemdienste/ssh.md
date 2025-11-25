
**Verbinden nur mit alias**

In der ~/.ssh/config eintrag nach  folgendem schema angeben

```sh
Host <alias_name>  
 HostName <remote_server_public_ip>  
 User <remote_server_username>
```

Danach kann man sich mit `ssh <alias_name>` anmelden.
