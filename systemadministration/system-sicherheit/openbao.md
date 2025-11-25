# What is OpenBao?

OpenBao is an identity-based secrets and encryption management system. A _secret_ is anything that you want to tightly control access to, such as API encryption keys, passwords, and certificates. OpenBao provides encryption services that are gated by authentication and authorization methods. Using OpenBao’s UI, CLI, or HTTP API, access to secrets and other sensitive data can be securely stored and managed, tightly controlled (restricted), and auditable.

A modern system requires access to a multitude of secrets, including database credentials, API keys for external services, credentials for service-oriented architecture communication, etc. It can be difficult to understand who is accessing which secrets, especially since this can be platform-specific. Adding on key rolling, secure storage, and detailed audit logs is almost impossible without a custom solution. This is where OpenBao steps in.

OpenBao validates and authorizes clients (users, machines, apps) before providing them access to secrets or stored sensitive data.

# Installation

Wegen den vielen abhängigkeiten bietet sich die installation via Docker oder Kubernetes an



`docker pull docker.io/openbao/openbao:2.2.0`

## Docker Compose



```yaml
services:
  openbao:
    ports:
      - ":"
    image: "openbao/openbao"

```

## Kubernetes Installation

**Supported kubernetes versions**

The following [Kubernetes minor releases](https://kubernetes.io/releases/) are currently supported. The latest version is tested against each Kubernetes version. It may work with other versions of Kubernetes, but those are not supported.

- 1.30
- 1.31
- 1.32

## Kubernetes using the helm chart

Helm must be installed and configured on your machine. Please refer to the [Helm documentation](https://helm.sh/) for more information.

To use the Helm chart, add the OpenBao helm repository and check that you have access to the chart:

```sh
helm repo add openbao https://openbao.github.io/openbao-helm"openbao" has been added to your repositories$ helm search repo openbao/openbaoNAME            CHART VERSION   APP VERSION             DESCRIPTIONopenbao/openbao 0.4.0           v2.0.0-alpha20240329    Official OpenBao Chart
```

info

**Important:** The Helm chart is new and under significant development. Please always run Helm with `--dry-run` before any install or upgrade to verify changes.

Example chart usage:

Installing the latest release of the OpenBao Helm chart with pods prefixed with the name `openbao`.

```
$ helm install openbao openbao/openbao
```

Installing a specific version of the chart.

```
# List the available releases$ helm search repo openbao/openbao -lNAME            CHART VERSION   APP VERSION             DESCRIPTIONopenbao/openbao 0.4.0           v2.0.0-alpha20240329    Official OpenBao Chartopenbao/openbao 0.3.0           v2.0.0-alpha20240329    Official OpenBao Chartopenbao/openbao 0.2.0           v2.0.0-alpha20240329    Official OpenBao Chart...# Install version 0.4.0$ helm install openbao openbao/openbao --version 0.4.0
```

warning

**Security Warning:** By default, the chart runs in standalone mode. This mode uses a single OpenBao server with a file storage backend. This is a less secure and less resilient installation that is **NOT** appropriate for a production setup. It is highly recommended to use a [properly secured Kubernetes cluster](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/), [learn the available configuration options](https://openbao.org/docs/platform/k8s/helm/configuration/), and read the [production deployment checklist](https://openbao.org/docs/platform/k8s/helm/run/#architecture).

# Datien und Verzeichnisse

- /opt/openbao/tls TLS verzeichnis
- /usr/bin/bao Binäres file



# Server Konfiguration


**Server mit eigener Konfig starten**

`bao server -config`