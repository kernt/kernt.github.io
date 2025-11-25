CLI

```
vault kv get $CLI path
```

API read secret version

```
curl \
  --header "X-Vault-Token: ..." \
  --request GET \
  https://127.0.0.1:8200/v1/kv/data/ansible/vault/myservers
```
## vault secret cubbyhole

## vault secret internal

## vault secret kv

**./ansible/vault/myservers**

vault_key hat das password

Metadata

`type:ansible_vault_key`

paths

API path for metadata: /v1/kv/metadata/ansible/vault/myservers
CLI path -mount="kv" "ansible/vault/myservers"
API path /v1/kv/data/ansible/vault/myservers
## vault secret pki

## vault secret ssh

## vault secret sops

## vault secret transit

## vault secret consul

## vault secret nomad

## vault secret totp

