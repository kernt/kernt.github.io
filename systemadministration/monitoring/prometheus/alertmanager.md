---
tags:
  - monitoring
  - alertmanager
  - prometheus
  - amtool
---
# Konfiguriere Prometheus

**Anpassen der Prometheus Konfigurationsdatei**

```yml
alerting:
  alertmanagers:
  - static_configs:
    - targets: ["localhost:9093"]
```

**Default Port is 9093**

```sh

### Prometheus Alertmanager

* [prometheus alerts rules](https://awesome-prometheus-alerts.grep.to/rules)

#### Prometheus Alertmanager Email

Prometeus Werkzeuge
- cortex Horizontally scalable HA multi-tenant logterm storage für Prometeus
- Thanos HA Prometeus mit long term storage
- (Timescale)[https://github.com/timescale/timescaledb] An open-source time-series SQL database optimized for fast ingest and complex queries. Packaged as a PostgreSQL extension

```

**Systemctl service einrichten**

```ini
[Unit]
Description=Alertmanager for prometheus
Documentation=https://prometheus.io/docs/alerting/alertmanager/

[Service]
Restart=on-failure
User=prometheus
EnvironmentFile=/etc/default/prometheus-alertmanager
ExecStart=/usr/bin/prometheus-alertmanager $ARGS
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```


*/etc/default/prometheus-alertmanager*

```sh
# Set the command-line arguments to pass to the server.
# Due to shell escaping, to pass backslashes for regexes, you need to double
# them (\\d for \d). If running under systemd, you need to double them again
# (\\\\d to mean \d), and escape newlines too.
ARGS=""
```

# basic auth

**Password Hash**

```Python
import getpass
import bcrypt

password = getpass.getpass("password: ")
hashed_password = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt())
print(hashed_password.decode())
```

``

**basic-auth mit https**

```YAML
# Alertmanager configuration
alerting:
  alertmanagers:
  - scheme: https
    basic_auth:
      username: tobkern
      password: $2a$12$UW41TRlltgpzS1RE9LLxiulvRZhXtJVkQBGGLKPTYH2d27yqcFMOO
    tls_config:
      ca_file: ca.crt
      cert_file: ca.crt
      key_file: ca.key
    static_configs:
    - targets: ['localhost:9093']

```

**basic-auth mit http**

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
  - scheme: https
    basic_auth:
      username: tobkern
      password: $2a$12$UW41TRlltgpzS1RE9LLxiulvRZhXtJVkQBGGLKPTYH2d27yqcFMOO
    tls_config:
      ca_file: ca.crt
      cert_file: ca.crt
      key_file: ca.key
    static_configs:
    - targets: ['localhost:9093']

```


Unter Debian nach 

`/usr/share/prometheus/alertmanager/generate-ui.sh` can automatically build and deply the UI

`/usr/share/prometheus/alertmanager/generate-ui.sh`