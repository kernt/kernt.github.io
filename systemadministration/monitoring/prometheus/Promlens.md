---
tags:
  - prometheus
  - monitoring
  - promlens
---
# promlens Installation

**Deploy mit docker**

`docker run -p 8080:8080 prom/promlens`

https://github.com/prometheus/promlens/releases/download/v0.3.0/promlens-0.3.0.linux-amd64.tar.gz

>Die Binäe datei ist veraltet 

*Systemd Unit service Datei*

```sh
# -*- mode: conf -*-

[Unit]
Description=Pomtail
Documentation=https://github.com/prometheus/node_exporter
After=network.target

[Service]
EnvironmentFile=-/etc/default/promlens
User=prometheus
ExecStart=/usr/bin/promlens $PROMLENS_OPTS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5s


[Install]
WantedBy=multi-user.target

```

*/etc/default/promlens*

```sh
# Options for PROMLENS_OPTS
#--shared-links.gcs.bucket=""
#                           Name of the GCS bucket for storing shared links. Set the GOOGLE_APPLICATION_CREDENTIALS environment variable to point to the JSON
#                           file defining your service account credentials (needs to have permission to create, delete, and view objects in the provided
#                           bucket).
#--shared-links.sql.driver=""
#                           The SQL driver to use for storing shared links in a SQL database. Supported values: [mysql, sqlite].
#--shared-links.sql.dsn=""  SQL Data Source Name when using a SQL database to shared links (see https://github.com/go-sql-driver/mysql#dsn-data-source-name)
#                           for MySQL, https://github.com/glebarez/go-sqlite#example for SQLite). Alternatively, use the environment variable
#                           PROMLENS_SHARED_LINKS_DSN to indicate this value.
#--shared-links.sql.create-tables
#                           Whether to automatically create the required tables when using a SQL database for shared links.
#--shared-links.sql.retention=0
#                           The maximum retention time for shared links when using a SQL database (e.g. '10m', '12h'). Set to 0 for infinite retention.
#--grafana.url="https://vmd36612.tail0c330.ts.net:3000"           The URL of your Grafana installation, to enable the Grafana datasource selector.
#--grafana.api-token="${GRAFANAAPITOKEN}"     The auth token to pass to the Grafana API.
#--grafana.api-token-file=""
#                           A file containing the auth token to pass to the Grafana API.
#--grafana.default-datasource-id=0
#                           The default Grafana datasource ID to use (overrides Grafana's own default).
#--web.external-url=""      The URL under which PromLens is externally reachable (for example, if PromLens is served via a reverse proxy).
#Used for generating
#                           relative and absolute links back to PromLens itself.
#                           If the URL has a path portion, it will be used to prefix all HTTP endpoints
#                           served by PromLens. If omitted, relevant URL components will be derived automatically.
#--web.route-prefix=""      Prefix for the internal routes of web endpoints. Defaults to path of --web.external-url.
#--web.default-prometheus-url=""
#                           The default Prometheus URL to load PromLens with.
#--log.level=info           Only log messages with the given severity or above. One of: [debug, info, warn, error]
#--log.format=logfmt        Output format of log messages. One of: [logfmt, json]
#--web.systemd-socket       Use systemd socket activation listeners instead of port listeners (Linux only).
#--web.listen-address=:8080 ...
#                           Addresses on which to expose metrics and web interface. Repeatable for multiple addresses.
#--web.config.file=""       [EXPERIMENTAL] Path to configuration file that can enable TLS or authentication.

PROMLENS_OPTS="--grafana.api-token='GRAFANA_API-TOKEN' --grafana.url='https://vmd36612.tail0c330.ts.net:3000' "

```

> Die Konfiguration wird von Grafana gelesen

> Für das Link Scharing muss eine Datenbank eingerichtet werden