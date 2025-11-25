---
tags:
  - prometheus
  - monitoring
---

# Docker exporter

Docker kann nativ compatible metriken ausgeben.
Dazu muss nur beim Start folgender Prameter gesetzt werden.

`--metrics-addr 127.0.0.1:9323`

* [Docker daemon-metrics](https://docs.docker.com/reference/cli/dockerd/#daemon-metrics)

# [influxdb_exporter](https://github.com/prometheus/influxdb_exporter)

# [graphite_exporter](https://github.com/prometheus/graphite_exporter)

# [bind_exporter](https://github.com/prometheus-community/bind_exporter)

## Troubleshooting

[](https://github.com/prometheus-community/bind_exporter#troubleshooting)

Make sure BIND was built with libxml2 support. You can check with the following command: `named -V | grep libxml2`.

Configure BIND to open a statistics channel. It's recommended to run the bind_exporter next to BIND, so it's only necessary to open a port locally.

```
statistics-channels {
  inet 127.0.0.1 port 8053 allow { 127.0.0.1; };
};
```

# nomad-exporter

https://gitlab.com/yakshaving.art/nomad-exporter

# [consul_exporter](https://github.com/prometheus/consul_exporter)

_exporter_defaults_

```
    consul.agent-only: Only export metrics about services registered on local agent.
    consul.allow_stale: Allows any Consul server (non-leader) to service a read.
    consul.ca-file: File path to a PEM-encoded certificate authority used to validate the authenticity of a server certificate.
    consul.cert-file: File path to a PEM-encoded certificate used with the private key to verify the exporter's authenticity.
    consul.health-summary: Collects information about each registered service and exports consul_catalog_service_node_healthy. This requires n+1 Consul API queries to gather all information about each service. Health check information are available via consul_health_service_status as well, but only for services which have a health check configured. Defaults to true. Disable using --no-consul.heatlh-summary.
    consul.key-file: File path to a PEM-encoded private key used with the certificate to verify the exporter's authenticity.
    consul.insecure: Disable TLS host verification.
    consul.require_consistent: Forces the read to be fully consistent.
    consul.server: Address (host and port) of the Consul instance we should connect to. This could be a local agent (localhost:8500, for instance), or the address of a Consul server.
    consul.server-name: When provided, this overrides the hostname for the TLS certificate. It can be used to ensure that the certificate name matches the hostname we declare.
    consul.timeout: Timeout on HTTP requests to consul.
    consul.request-limit: Limit the maximum number of concurrent requests to consul, 0 means no limit.
    log.format: Set the log target and format. Example: logger:syslog?appname=bob&local=7 or logger:stdout?json=true
    log.level: Logging level. info by default.
    web.listen-address: Address to listen on for web interface and telemetry.
    web.telemetry-path: Path under which to expose metrics.
    version: Show application version.

```

**Are my services healthy?**

```
min(consul_catalog_service_node_healthy) by (service_name)
```

Values of 1 mean that all nodes for the service are passing. Values of 0 mean at least one node for the service is not passing.

**What service nodes are failing?**

```
sum by (node, service_name)(consul_catalog_service_node_healthy == 0)
```

**What service checks are critical?**

```
consul_health_service_status{status="critical"} == 1
```

You can query for the following health check states: "maintenance", "critical", "warning" or "passing"

# [mysqld_exporter](https://github.com/prometheus/mysqld_exporter)

**Berechtigungen setzen**

```sql
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'XXXXXXXX' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
```

# json_exporter

[json_exporter releases](https://github.com/prometheus-community/json_exporter/releases)

https://github.com/prometheus-community/json_exporter/releases/download/v0.6.0/json_exporter-0.6.0.linux-amd64.tar.gz

# rsyslog_exporter

https://github.com/prometheus-community/rsyslog_exporter/releases/download/v1.1.0/rsyslog-exporter_1.1.0-1_amd64.deb

https://github.com/prometheus-community/rsyslog_exporter/releases/download/v1.1.0/rsyslog_exporter_1.1.0_Linux_x86_64.tar.gz

# fluentd Expose Metrics

https://docs.fluentd.org/monitoring-fluentd/monitoring-prometheus

# cloudflare_exporter

https://gitlab.com/gitlab-org/cloudflare_exporter

# GitLab Prometheus metrics

https://docs.gitlab.com/ee/administration/monitoring/prometheus/gitlab_metrics.html

# consul_exporter

https://github.com/prometheus/consul_exporter

# MINIO

https://min.io/docs/minio/linux/operations/monitoring/collect-minio-metrics-using-prometheus.html?ref=docs-redirect

```
   - job_name: minio-job
     bearer_token: TOKEN
     metrics_path: /minio/v2/metrics/cluster
     scheme: https
     static_configs:
     - targets: [minio.example.net]
```

> Geht nicht auf Anhieb

## Alert Rule

```
groups:
- name: minio-alerts
  rules:
  - alert: NodesOffline
    expr: avg_over_time(minio_cluster_nodes_offline_total{job="minio-job"}[5m]) > 0
    for: 10m
    labels:
      severity: warn
    annotations:
      summary: "Node down in MinIO deployment"
      description: "Node(s) in cluster {{ $labels.instance }} offline for more than 5 minutes"

  - alert: DisksOffline
    expr: avg_over_time(minio_cluster_drive_offline_total{job="minio-job"}[5m]) > 0
    for: 10m
    labels:
      severity: warn
    annotations:
      summary: "Disks down in MinIO deployment"
      description: "Disks(s) in cluster {{ $labels.instance }} offline for more than 5 minutes"
```

# PowerDNS 

# Prometheus Data Endpoint[](https://doc.powerdns.com/recursor/http-api/prometheus.html#prometheus-data-endpoint "Permalink to this headline")

New in version 4.3.0.

`GET` `/metrics`[](https://doc.powerdns.com/recursor/http-api/prometheus.html#get--metrics "Permalink to this definition")

> Get statistics from Recursor in [Prometheus](https://prometheus.io) format. Uses [webserver-password](https://doc.powerdns.com/recursor/settings.html#setting-webserver-password) and returned list can be controlled with [stats-api-blacklist](https://doc.powerdns.com/recursor/settings.html#setting-stats-api-blacklist)

**Example request**:

curl -i -u=#:webpassword http://127.0.0.1:8081/metrics

https://doc.powerdns.com/recursor/http-api/prometheus.html

http://www.ypbind.de/maus/projects/prometheus-powerdns-exporter/index.html

https://github.com/wrouesnel/pdns_exporter

https://github.com/janeczku/powerdns_exporter

# ssh_exporter

https://github.com/treydock/ssh_exporter

# nftables_exporter

https://github.com/Intrinsec/nftables_exporter

# filestat_exporter

https://github.com/michael-doubez/filestat_exporter

# wireguard_exporter

https://github.com/MindFlavor/prometheus_wireguard_exporter



# Ansible

https://docs.ansible.com/automation-controller/latest/html/administration/metrics.html

* [Ansible exporter](https://prometheus-community.github.io/ansible/branch/main/process_exporter_role.html#ansible-collections-prometheus-prometheus-process-exporter-role)

* [ansible-prometheus_exporter_exporter](https://github.com/umanit/ansible-prometheus_exporter_exporter)

[how-to-install-prometheus-and-node-exporter-using-an-ansible-role](https://blog.devops.dev/how-to-install-prometheus-and-node-exporter-using-an-ansible-role-b9ccd3d4bf3c)