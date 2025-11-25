#

https://developer.hashicorp.com/vault/tutorials/associate-cert-003/associate-study-003

https://brain-dumps.net/exams/hashicorp/hcva0-003

https://github.com/bmuschko/cva-crash-course/blob/master/exercises/01-installing-vault/solution/solution.md

export VAULT_ADDR=""
export VAULT_TOKEN=""

vault login -address=$VAULT_ADDR $VAULT_TOKEN

vault auth list

vault auth enable userpass
vault auth enable -path=global -max-lease-ttl=30m approle
vault auth list

curl -H "X-Vault-Token: $VAULT_TOKEN" -X GET $VAULT_ADDR/v1/sys/host-info | jq .

vault auth tune -description="Username and password" userpass/
vault auth tune -description="Role-based credentials" global/ # globaö muss angelegt sein !

vault auth list

vault list auth/userpass/users

