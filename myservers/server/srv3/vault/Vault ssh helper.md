`config.hcl`

```hcl
vault_addr = "https://vault.example.com:8200"
ssh_mount_point = "ssh"
namespace = "my_namespace"
ca_cert = "/etc/vault-ssh-helper.d/vault.crt"
tls_skip_verify = false
allowed_roles = "*"
```

PAM Konfiguration

```
#@include common-auth
auth requisite pam_exec.so quiet expose_authtok log=/tmp/vaultssh.log /usr/local/bin/vault-ssh-helper -config=/etc/vault-ssh-helper.d/config.hcl
auth optional pam_unix.so not_set_pass use_first_pass nodelay
```

## SSHD Configuration

*/etc/ssh/sshd_config*

```
ChallengeResponseAuthentication yes
UsePAM yes
PasswordAuthentication no
```

https://github.com/hashicorp/vault-ssh-helper