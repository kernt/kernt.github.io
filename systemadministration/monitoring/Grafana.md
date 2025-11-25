---
tags:
  - monitoring
  - cloud
  - grafana
  - ansible
---
# Grafana Installation rpm basierend

**Install Grafana**

```sh
dnf install grafana
```

**Verifikation**

```sh
ss -ntlp | grep 3000
```

Ensure that the `grafana-pcp` plugin is installed

```sh
grafana-cli plugins ls | grep performancecopilot-pcp-app
```

[Grafana Install](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/monitoring_and_managing_system_status_and_performance/setting-up-graphical-representation-of-pcp-metrics_monitoring-and-managing-system-status-and-performance#setting-up-a-grafana-server_setting-up-graphical-representation-of-pcp-metrics)

# Grafana Administration

Migrate data and encrypt passwords migrates passwords from unsecured fields to secure_json_data field

```sh
grafana cli admin data-migration encrypt-datasource-passwords
```

**Admin password reset**

```sh
grafana cli admin
```

**Update all installed plugins**

```sh
grafana cli plugins update-all
```

## #Grafana mit #ansible

`ansible-galaxy collection install community.grafana`

```yml
---
- name: Create or update a Grafana user
  community.grafana.grafana_user:
    url: "https://grafana.example.com"
    url_username: admin
    url_password: changeme
    name: "Bruce Wayne"
    email: batman@gotham.city
    login: batman
    password: robin
    is_admin: true
    state: present

- name: Delete a Grafana user
  community.grafana.grafana_user:
    url: "https://grafana.example.com"
    url_username: admin
    url_password: changeme
    login: batman
    state: absent
```

# Grafana Dashboards

[gitlab](https://gitlab.com/gitlab-org/grafana-dashboards/-/tree/master)
