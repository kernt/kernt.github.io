Als Basis dient das normale Binäre file

Vault Proxy is a client daemon that provides the following features:

- [Auto-auth](https://developer.hashicorp.com/vault/docs/agent-and-proxy/autoauth) - Automatically authenticate to Vault and manage the token renewal process for locally-retrieved dynamic secrets.
- [API proxy](https://developer.hashicorp.com/vault/docs/agent-and-proxy/proxy/apiproxy) - Acts as a proxy for Vault's API, optionally using (or forcing the use of) the auto-auth token.
- [Caching](https://developer.hashicorp.com/vault/docs/agent-and-proxy/proxy/caching) - Allows client-side caching of responses containing newly created tokens and responses containing leased secrets generated off of these newly created tokens. The agent also manages the renewals of the cached tokens and leases.

Beispiel Konfiguration

```
pid_file = "./pidfile"

log_file = "/var/log/vault-agent.log"

vault {
  address = "https://vault-fqdn:8200"
  retry {
    num_retries = 5
  }
}

auto_auth {
  method "aws" {
    mount_path = "auth/aws-subaccount"
    config = {
      type = "iam"
      role = "foobar"
    }
  }

  sink "file" {
    config = {
      path = "/tmp/file-foo"
    }
  }

  sink "file" {
    wrap_ttl = "5m"
    aad_env_var = "TEST_AAD_ENV"
    dh_type = "curve25519"
    dh_path = "/tmp/file-foo-dhpath2"
    config = {
      path = "/tmp/file-bar"
    }
  }
}

cache {
  // An empty cache stanza still enables caching
}

template_config {
  static_secret_render_interval = "10m"
  exit_on_retry_failure = true
  max_connections_per_host = 20
}

template {
  source = "/etc/vault/server.key.ctmpl"
  destination = "/etc/vault/server.key"
}

template {
  source = "/etc/vault/server.crt.ctmpl"
  destination = "/etc/vault/server.crt"
}

```

**Vault agent starten**

```sh
vault agent -config=/etc/vault/agent-config.hcl
```

https://developer.hashicorp.com/vault/docs/agent-and-proxy/agent

https://developer.hashicorp.com/vault/install#linux

