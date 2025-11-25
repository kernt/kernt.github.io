If you do not have an existing Vault cluster available, do one of the following:

1. Follow [Hashicorp Vault install guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-deploy) to create a new cluster
2. Create a [single node dev instance](https://min.io/docs/kes/integrations/hashicorp-vault-keystore/#single-node-dev-vault-instance)

The docs below discuss setting up a single node dev instance for development purposes using the `AppRole` authentication method.
### Single Node Dev Vault Instance

The following command starts a Vault server in dev mode:

```sh
vault server -dev
```

Select the **Output** tab above to see example command output.

Output

This starts a single-node Vault server in dev mode listening on `127.0.0.1:8200`. A dev server is ephemeral and is **not** meant to be run in production.
### Connect Vault to the Vault CLI

1. Set `VAULT_ADDR` endpoint
The Vault CLI needs to know the Vault endpoint:

- ```sh
    export VAULT_ADDR='https://127.0.0.1:8200'
    ```
- Set `VAULT_TOKEN`
    
    The Vault CLI needs an authentication token to perform operations.
    
- ```sh
    export VAULT_TOKEN=hvs.O6QNQB33ksXtMxtlRKlRZL0R
    ```
    
    Replace the token value to your own Vault access token, such as the `Root token` provided in the output of the `vault server -dev` command.
    
- Enable `K/V` Backend
    
    KES stores the secret keys at the Vault K/V backend. Vault provides two [K/V engines](https://www.vaultproject.io/docs/secrets/kv), `v1` and `v2`.
    
    MinIO recommends the K/V `v1` engine.

The following command enables the K/V `v2` secret engine:

1. ```sh
    vault secrets enable -version=2 kv
    ```
    
    The Vault policy for KES depends on the chosen K/V engine version. A policy designed for K/V `v1` will not work with a K/V `v2` engine. Likewise, a policy designed for K/V `v2` will not work with a K/V `v1` engine.
    
    For more information about migrating from `v1` to `v2` refer to the [Hashicorp docs on upgrading from v1](https://www.vaultproject.io/docs/secrets/kv/kv-v2#upgrading-from-version-1).
### Setup KES Access to Vault

1. Create Vault Policy
    
    The Vault policy defines the API paths the KES server can access. Create a text file named `kes-policy.hcl`.
    
    The contents of the policy vary depending on the K/V engine used.
Use the following `kes-policy.hcl` policy for the K/V `v2` backend:

- ```hcl
    path "kv/data/*" {
       capabilities = [ "create", "read" ]
    }
    path "kv/metadata/*" {
       capabilities = [ "list", "delete" ]       
    }
    ```
- Write the policy to the Vault

    The following command creates the policy at Vault:

- ```sh
    vault policy write kes-policy kes-policy.hcl
    ```
    
- Enable Authentication
    This step allows the KES server to authenticate to Vault. For this tutorial, we use the `AppRole` authentication method.
- ```sh
    vault auth enable approle
    ```
    
- Create KES Role
    
    The following command adds a new role called `kes-server` at Vault:
    
- ```sh
    vault write auth/approle/role/kes-server token_num_uses=0  secret_id_num_uses=0  period=5m
    ```
    
- Bind Policy to Role
    
    The following command binds the `kes-server` role to the `kes-policy`:
    
- ```sh
    vault write auth/approle/role/kes-server policies=kes-policy
    ```
    
- Generate AppRole ID
    
    Request an AppRole ID for the KES server:
    
- ```sh
    vault read auth/approle/role/kes-server/role-id 
    ```
    
- Generate AppRole Secret
    
    Request an AppRole secret for the KES server:
    

1. ```sh
    vault write -f auth/approle/role/kes-server/secret-id 
    ```
    
    The AppRole secret prints as `secret_id`. You can ignore the `secret_id_accessor`.
    

## KES Server Setup

1. Generate KES Server Private Key & Certificate
    
    The following command generates a new TLS private key `server.key` and a self-signed X.509 certificate `server.cert` for the IP `127.0.0.1` and DNS name `localhost` (as SAN). If you want to refer to your KES server using another IP or DNS name, such as `10.1.2.3` or `https://kes.example.net`, adjust the `--ip` and/or `--dns` parameters accordingly.
    
```sh
kes identity new --key server.key --cert server.cert --ip "127.0.0.1" --dns localhost
```

The above command generates self-signed certificates. If you already have a way to issue certificates for your servers, you can use those.

Other tooling for X.509 certificate generation also works. For example, you could use `openssl`:

- ```sh
    openssl ecparam -genkey -name prime256v1 | openssl ec -out server.key
    
    openssl req -new -x509 -days 30 -key server.key -out server.cert \
        -subj "/C=/ST=/L=/O=/CN=localhost" -addext "subjectAltName = IP:127.0.0.1"
    ```
    
- Generate an API key
    
    The following command generates a new KES API key.
    
- ```sh
    kes identity new
    ```
    
    The output resembles the following:
    
    ```sh
    Your API key:
    
       kes:v1:ABfa1xsnIV0lltXQC8tHXic8lte7J6hT7EoGv6+s5QCW
    
    This is the only time it is shown. Keep it secret and secure!
    
    Your Identity:
    
       cf6c535e738c1dd47a1d746366fde7f0309d1e0a8471b9f6e909833906afbbfa
    
    The identity is not a secret. It can be shared. Any peer
    needs this identity in order to verify your API key.
    
    The identity can be computed again via:
    
        kes identity of kes:v1:ABfa1xsnIV0lltXQC8tHXic8lte7J6hT7EoGv6+s5QCW   
    ```
    
    The generated `identity` is **NOT** a secret and can be shared publicly. It will be used later on in the KES config file as admin identity or assigned to a policy.
    
    The `API key` itself **is** a secret and should not be shared. You can always recompute an API key’s identity.
    
- Configure KES Server
    
    Create the KES server configuration file: `config.yml`.
    
    Make sure that the identity in the policy section matches the `client.crt` identity. Add the approle `role_id` and `secret_id` obtained earlier.
    
- ```yaml
    admin:
      # Use the identity generated above by 'kes identity new'.
      identity: "" # For example: cf6c535e738c1dd47a1d746366fde7f0309d1e0a8471b9f6e909833906afbbfa
    
    tls:
      key: private.key    # The KES server TLS private key
      cert: public.crt    # The KES server TLS certificate
    
    keystore:
       vault:
         endpoint: https://127.0.0.1:8200
         version:  v1 # The K/V engine version - either "v1" or "v2".
         engine:   kv # The engine path of the K/V engine. The default is "kv".
         approle:
           id:     "" # Your AppRole ID
           secret: "" # Your AppRole Secret
    ```
    
- Start KES Server


```sh
kes server --config config.yml
```

In Linux environments, KES can use the [`mlock`](http://man7.org/linux/man-pages/man2/mlock.2.html) syscall to prevent the OS from writing in-memory data to disk (swapping). This prevents leaking sensitive data.

Use the following command to allow KES to use the `mlock` syscall without running with `root` privileges:

1. ```sh
    sudo setcap cap_ipc_lock=+ep $(readlink -f $(which kes))
    ```
    
    Containers
    

## KES CLI Access

1. Set `KES_SERVER` endpoint
    
    The following environment variable specifies the KES server the KES CLI should talk to:
    

- ```sh
    export KES_SERVER=https://127.0.0.1:7373
    ```
    
- Define the CLI access credentials
    
    The following environment variable sets the key the client uses to talk to a KES server:
    
- ```sh
    export KES_API_KEY=kes:v1:ABfa1xsnIV0lltXQC8tHXic8lte7J6hT7EoGv6+s5QCW
    ```
    
    Replace the value with your server’s API Key. The server’s API key displays in the output when you start the server.
    
- Test the Configuration
    
    For example, check the status of the server:
    

1. ```sh
    kes status -k
    ```
    
    Use the key to generate a new data encryption key:
    
    ```sh
    kes key dek my-key-1 -k
    ```
    
    The command output resembles the following:
    
    ```sh
    {
      plaintext : UGgcVBgyQYwxKzve7UJNV5x8aTiPJFoR+s828reNjh0=
      ciphertext: eyJhZWFkIjoiQUVTLTI1Ni1HQ00tSE1BQy1TSEEtMjU2IiwiaWQiOiIxMTc1ZjJjNDMyMjNjNjNmNjY1MDk5ZDExNmU3Yzc4NCIsIml2IjoiVHBtbHpWTDh5a2t4VVREV1RSTU5Tdz09Iiwibm9uY2UiOiJkeGl0R3A3bFB6S21rTE5HIiwiYnl0ZXMiOiJaaWdobEZrTUFuVVBWSG0wZDhSYUNBY3pnRWRsQzJqWFhCK1YxaWl2MXdnYjhBRytuTWx0Y3BGK0RtV1VoNkZaIn0=
    }
    ```
    

## Using KES with a MinIO Server

MinIO Server requires KES to enable server-side data encryption.

See the [KES for MinIO instruction guide](https://min.io/docs/kes/tutorials/kes-for-minio/) for additional steps needed to use your new KES Server with a MinIO Server.

## Configuration References

The following section describes the Key Encryption Service (KES) configuration settings to use Hashicorp Vault Keystore as the root KMS to store external keys, such as the keys used for Server-Side Encryption on a MinIO Server.

MinIO Server Requires Expanded Permissions:

Starting with [MinIO Server RELEASE.2023-02-17T17-52-43Z](https://github.com/minio/minio/releases/tag/RELEASE.2023-02-17T17-52-43Z), MinIO requires expanded KES permissions for functionality. The example configuration in this section contains all required permissions.

Fields with `${<STRING>}` use the environment variable matching the `<STRING>` value. You can use this functionality to set credentials without writing them to the configuration file.

The YAML assumes a minimal set of permissions for the MinIO deployment accessing KES. As an alternative, you can omit the `policy.minio-server` section and instead set the `${MINIO_IDENTITY}` hash as the `${ROOT_IDENTITY}`.

```yaml
address: 0.0.0.0:7373
root: ${ROOT_IDENTITY}

tls:
  key: kes-server.key
  cert: kes-server.cert

api:
  /v1/ready:
    skip_auth: false
    timeout:   15s

policy:
  minio-server:
    allow:
    - /v1/key/create/*
    - /v1/key/generate/*
    - /v1/key/decrypt/*
    - /v1/key/bulk/decrypt
    - /v1/key/list/*
    - /v1/status
    - /v1/metrics
    - /v1/log/audit
    - /v1/log/error
    deny:
    - /v1/key/generate/my-app-internal*
    - /v1/key/decrypt/my-app-internal*
    identities:
    - ${MINIO_IDENTITY}

    my-app:
    allow:
    - /v1/key/create/my-app*
    - /v1/key/generate/my-app*
    - /v1/key/decrypt/my-app*
    deny:
    - /v1/key/generate/my-app-internal*
    - /v1/key/decrypt/my-app-internal*
    identities:
    - df7281ca3fed4ef7d06297eb7cb9d590a4edc863b4425f4762bb2afaebfd3258
    - c0ecd5962eaf937422268b80a93dde4786dc9783fb2480ddea0f3e5fe471a731

keys:
  - name: "minio-encryption-key-alpha"
  - name: "minio-encryption-key-baker"
  - name: "minio-encryption-key-charlie"

cache:
  expiry:
    any: 5m0s
    unused: 20s
    offline: 0s

log:

  # Log error events to STDERR. Valid values are "on" or "off". 
  # Default is "on".
  error: on

  # Log audit events to STDOUT. Valid values are "on" and "off". 
  # Default is "off".
  audit: off

keystore:
  vault:
    endpoint: ""  
    engine: ""    
    version: ""   
    namespace: "" 
    prefix: ""    
    transit:      
      engine: ""  
      key: ""     
    approle:    
      namespace: "" 
      engine: ""    
      id: ""        
      secret: ""    
    kubernetes: 
      namespace: "" 
      engine: ""    
      role: ""      
      jwt:  ""      
    tls:        
      key: ""   
      cert: ""  
      ca: ""    
    status:     
      ping: 10s 
```

Reference

## Advanced Configuration

These additional configuration steps may solve specific problems.

### Multi-Tenancy with K/V prefixes

Vault can serve as backend for multiple, isolated KES tenants. Each KES tenant can consist of `N` replicas. There can be `M` KES tenants connected to the same Vault server/cluster.

This means `N × M` KES server instances can connect to a single Vault.

In these configurations, each KES tenant has a separate prefix at the K/V secret engine. For each KES tenant, there must be a corresponding Vault policy.

- For K/V `v1`:
    

- ```hcl
    path "kv/<tenant-name>/*" {
       capabilities = [ "create", "read", "delete" ]
    }
    ```
    
- For K/V `v2`:
    

- ```hcl
    path "kv/data/<tenant-name>/*" {
      capabilities = [ "create", "read" ]
    }
    path "kv/metadata/<tenant-name>/*" {
      capabilities = [ "list", "delete" ]       
    }
    ```
    

Create a different configuration file for each KES tenant. The file contains the Vault K/V prefix for the tenant to use.

```yaml
keystore:
   vault:
     endpoint: https://127.0.0.1:8200
     prefix: <tenant-name>
     approle:
       id:     "" # Your AppRole ID
       secret: "" # Your AppRole Secret
       retry:  15s
     status:
       ping: 10s
     tls:
       ca: vault.crt # Manually trust the vault certificate since we use self-signed certificates
```

### Multi-Tenancy with Vault Namespaces

Vault can serve as the backend for multiple, isolated KES tenants. Each KES tenant can consist of `N` replicas. There can be `M` KES tenants connected to the same Vault server/cluster.

This means `N × M` KES server instances can connect to a single Vault.

Therefore, each KES tenant has a separate prefix at the K/V secret engine. For each KES tenant there has to be a corresponding Vault policy.

- For K/V `v1`:
    

- ```hcl
    path "kv/<tenant-name>/*" {
       capabilities = [ "create", "read", "delete" ]
    }
    ```
    
- For K/V `v2`:
    

- ```hcl
    path "kv/data/<tenant-name>/*" {
       capabilities = [ "create", "read" ]
    }
    path "kv/metadata/<tenant-name>/*" {
       capabilities = [ "list", "delete" ]       
    }
    ```
    

Use a different configuration file for each KES tenant. The file contains the Vault namespace which the KES tenant should use.

```yaml
keystore:
   vault:
     endpoint: https://127.0.0.1:8200
     namespace: <vault-namespace>
     approle:
       id:     "" # Your AppRole ID
       secret: "" # Your AppRole Secret
       retry:  15s
     status:
       ping: 10s
     tls:
       ca: vault.crt # Manually trust the vault certificate since we use self-signed certificates
```

### Encrypt Vault-stored Keys

Hashicorp’s [Transit](https://developer.hashicorp.com/vault/docs/secrets/transit) functionality provides a means to encrypt and decrypt keys stored in the vault. This provides an additional layer of encryption that may be useful in specific cases.

When enabled, Hashicorp stores a key in the Vault to encrypt or decrypt the other keys stored in the vault. KES then uses the vault-managed key to store or retrieve keys from the Vault.

Potential data loss:

If the specified transit key is incorrect, disabled, removed, or otherwise unaccessible, KES cannot retrieve any vault keys nor perform any en/decryption operations relying on those keys.

To configure Transit, add the following section to the KES Configuration YAML’s `keystore.vault` section:

```sh
keystore:
  vault:
    transit:      # Optionally encrypt keys stored on the K/V engine with a Vault-managed key.
      engine: ""  # The path of the transit engine - e.g. "my-transit". If empty, defaults to: transit (Vault default)
      key: ""     # The key name that should be used to encrypt entries stored on the K/V engine.
```

## References

- [Server API Doc](https://min.io/docs/kes/concepts/server-api/)
- [Go SDK Doc](https://pkg.go.dev/github.com/minio/kes)