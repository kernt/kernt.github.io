---
tags:
  - sicherheit
  - sicherheits-tools
  - smallstep
  - traefik
  - acme
  - TLS
---
# Konfiguration smallstep Server

**CA starten**

```sh
$ step-ca $(step path)/config/ca.json

Please enter the password to decrypt /Users/bob/.step/secrets/intermediate_ca_key: abc123

2019/02/18 13:28:58 Serving HTTPS on 127.0.0.1:8443 ...

```

**step-ca.service**

```
[Unit]
Description=step-ca service
Documentation=https://smallstep.com/docs/step-ca
Documentation=https://smallstep.com/docs/step-ca/certificate-authority-server-production
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=30
StartLimitBurst=3
ConditionFileNotEmpty=/etc/step-ca/config/ca.json
ConditionFileNotEmpty=/etc/step-ca/password.txt

[Service]
Type=simple
User=step
Group=step
Environment=STEPPATH=/etc/step-ca
WorkingDirectory=/etc/step-ca
ExecStart=/usr/bin/step-ca config/ca.json --password-file password.txt
ExecReload=/bin/kill --signal HUP $MAINPID
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
StartLimitInterval=30
StartLimitBurst=3

; Process capabilities & privileges
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
SecureBits=keep-caps
NoNewPrivileges=yes

; Sandboxing
; This sandboxing works with YubiKey PIV (via pcscd HTTP API), but it is likely
; too restrictive for PKCS#11 HSMs.
;
; NOTE: Comment out the rest of this section for troubleshooting.
ProtectSystem=full
ProtectHome=true
RestrictNamespaces=true
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
PrivateTmp=true
ProtectClock=true
ProtectControlGroups=true
ProtectKernelTunables=true
ProtectKernelLogs=true
ProtectKernelModules=true
LockPersonality=true
RestrictSUIDSGID=true
RemoveIPC=true
RestrictRealtime=true
PrivateDevices=true
SystemCallFilter=@system-service
SystemCallArchitectures=native
MemoryDenyWriteExecute=true
ReadWriteDirectories=/etc/step-ca/db

[Install]
WantedBy=multi-user.target
```

[step-ca.service](https://github.com/smallstep/certificates/blob/master/systemd/step-ca.service)

**CA Zugriff. Konfiguration einer Neuen Maschiene **

```sh
$ step ca bootstrap --ca-url [CA URL] --fingerprint [CA fingerprint]
The root certificate has been saved in /home/alice/.step/certs/root_ca.crt.
Your configuration has been saved in /home/alice/.step/config/defaults.json.

```

## smallstep certs ansible

https://github.com/smallstep/certificates/blob/master/examples/ansible/smallstep-certs/tasks/main.yml

/home/tobkern/.step/

Default configuration: /home/tobkern/.step/config/defaults.json

[github smallstep certificates](https://github.com/smallstep/certificates)

# [step cli](https://github.com/smallstep/cli)

## Systemd unit files für `step`

```
wget https://raw.githubusercontent.com/smallstep/cli/refs/heads/master/systemd/cert-renewer.target
wget https://raw.githubusercontent.com/smallstep/cli/refs/heads/master/systemd/cert-renewer%40.service
wget https://raw.githubusercontent.com/smallstep/cli/refs/heads/master/systemd/cert-renewer%40.timer
wget https://raw.githubusercontent.com/smallstep/cli/refs/heads/master/systemd/ssh-cert-renewer.service
wget https://raw.githubusercontent.com/smallstep/cli/refs/heads/master/systemd/ssh-cert-renewer.timer
```

## autocomplete step cli

`step completion bash`

Unterstützt sind:
- bash
- zsh

### acme-protocol-acme-clients

**Traefik v2**

```
[certificatesResolvers]
  [certificatesResolvers.myresolver]
    [certificatesResolvers.myresolver.acme]
      caServer = "https://step-ca.internal/acme/acme/directory"
      email = "carl@smallstep.com"
      storage = "acme.json"
      certificatesDuration = 24
      tlsChallenge = true
```

[traefik](https://smallstep.com/docs/tutorials/acme-protocol-acme-clients/index.html#traefik)

### acme-protocol-acme-clients

```
sudo REQUESTS_CA_BUNDLE=$(step path)/certs/root_ca.crt \
    certbot certonly -n --standalone -d foo.internal \
    --server https://ca.internal/acme/acme/directory

```

[acme-protocol-acme-clients](https://smallstep.com/docs/tutorials/acme-protocol-acme-clients/index.html#certbot)

# [step-ca](https://github.com/smallstep/certificates)

Gegenstück zum step cli tool stellt also den Server dar