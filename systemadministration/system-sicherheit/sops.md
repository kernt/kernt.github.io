https://github.com/getsops/sops
# soap Installation

```sh
SOPS_VERSION="v3.10.2"
# Download the binary
curl -LO https://github.com/getsops/sops/releases/download/$SOPS_VERSION/sops-$SOPS_VERSION.linux.amd64

# Move the binary in to your PATH
mv sops-$SOPS_VERSION.linux.amd64 /usr/local/bin/sops

# Make the binary executable
chmod +x /usr/local/bin/sops

# Download the checksums file, certificate and signature
curl -LO https://github.com/getsops/sops/releases/download/$SOPS_VERSION/sops-$SOPS_VERSION.checksums.txt
curl -LO https://github.com/getsops/sops/releases/download/$SOPS_VERSION/sops-$SOPS_VERSION.checksums.pem
curl -LO https://github.com/getsops/sops/releases/download/$SOPS_VERSION/sops-$SOPS_VERSION.checksums.sig

# Verify the binary using the checksums file
sha256sum -c sops-$SOPS_VERSION.checksums.txt --ignore-missing

```
# sops einrichten


# sops encrypt



# soap mit Vault

```sh
# Substitute this with the address Vault is running on
export VAULT_ADDR=http://127.0.0.1:8200

# this may not be necessary in case you previously used `vault login` for production use
export VAULT_TOKEN=${VAULTTOKEN}

# to check if Vault started and is configured correctly
$ vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.2.0
Cluster Name    vault-cluster-618cc902
Cluster ID      e532e461-e8f0-1352-8a41-fc7c11096908
HA Enabled      false

# It is required to enable a transit engine if not already done (It is suggested to create a transit engine specifically for SOPS, in which it is possible to have multiple keys with various permission levels)
vault secrets enable -path=sops transit
Success! Enabled the transit secrets engine at: sops/

# Then create one or more keys
vault write sops/keys/firstkey type=rsa-4096
Success! Data written to: sops/keys/firstkey

vault write sops/keys/secondkey type=rsa-2048
Success! Data written to: sops/keys/secondkey

vault write sops/keys/thirdkey type=chacha20-poly1305
Success! Data written to: sops/keys/thirdkey

sops encrypt --hc-vault-transit $VAULT_ADDR/v1/sops/keys/firstkey vault_example.yml

```sh
cat <<EOF > .sops.yaml
creation_rules:
    - path_regex: \.dev\.yaml$
      hc_vault_transit_uri: "$VAULT_ADDR/v1/sops/keys/secondkey"
    - path_regex: \.prod\.yaml$
      hc_vault_transit_uri: "$VAULT_ADDR/v1/sops/keys/thirdkey"
EOF
```

sops encrypt --verbose prod/raw.yaml > prod/encrypted.yaml
```