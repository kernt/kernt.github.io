
```sh
git clone https://github.com/hashicorp-education/learn-vault-secrets-operator
cd learn-vault-secrets-operator
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install vault hashicorp/vault -n vault --create-namespace --values vault/vault-values.yaml
kubectl get pods -n vault
```

Vault Konfiguieren

```sh
kubectl exec --stdin=true --tty=true vault-0 -n vault -- /bin/sh
cd tmp
vault auth enable -path demo-auth-mount kubernetes
vault write auth/demo-auth-mount/config \
   kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
vault secrets enable -path=kvv2 kv-v2
tee webapp.json <<EOF
path "kvv2/data/webapp/config" {
   capabilities = ["read", "list"]
}
EOF
vault policy write webapp webapp.json
vault write auth/demo-auth-mount/role/role1 \
   bound_service_account_names=demo-static-app \
   bound_service_account_namespaces=app \
   policies=webapp \
   audience=vault \
   ttl=24h
vault kv put kvv2/webapp/config username="static-user" password="static-password"
exit
```
## Install the Vault Secrets Operator

```
helm install vault-secrets-operator hashicorp/vault-secrets-operator -n vault-secrets-operator-system --create-namespace --values vault/vault-operator-values.yaml
cat vault/vault-operator-values.yaml
```
## Deploy and sync a secret

```
kubectl create ns app
kubectl apply -f vault/vault-auth-static.yaml
kubectl apply -f vault/static-secret.yaml
```

1. If you examine the `static-secret.yaml` just used, look near the bottom.
    
    Either version you will find a field called `refreshAfter`. That fields determines how often to check the secret for updates. In the Vault community edition example, it set up to refresh after 30 seconds.
    
    In the Vault enterprise example the `refreshAfter` field is a fallback option, set to 1 hour. Since the `instantUpdates` is enabled, so the `refreshAfter` field is ignored.
    

```
   ...
   syncConfig:
    instantUpdates: true
   ..
```
## Rotate the static secre

https://developer.hashicorp.com/vault/tutorials/kubernetes/vault-secrets-operator