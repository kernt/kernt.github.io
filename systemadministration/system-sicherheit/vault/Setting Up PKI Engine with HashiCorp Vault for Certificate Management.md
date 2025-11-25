In the previous post, we set up a highly available HashiCorp Vault cluster to ensure secure management of secrets and certificates. With the cluster in place, we are now ready to take it a step further by enabling the PKI (Public Key Infrastructure) secret engine. Then we’ll generate a Root CA and an Intermediate CA, create roles, issue certificates, and enable TLS in our Vault cluster. These steps will enhance the security of our Vault setup, allowing us to manage certificates with precision and flexibility.

In previous post, we had setup vault cluster in HA mode:

[manjit28.medium.com](https://manjit28.medium.com/setting-up-a-secure-and-highly-available-hashicorp-vault-cluster-for-secrets-and-certificates-0ce01a370582?source=post_page-----ca35f10c9600---------------------------------------)

In this post, we’ll go through the intricate process of implementing a robust certificate management system using Vault. We’ll cover:

1. Enabling and configuring the PKI secret engine
2. Generating a secure Root Certificate Authority (CA)
3. Creating and managing an Intermediate CA for enhanced security
4. Defining roles to streamline certificate issuance
5. Enhancing our Vault cluster’s security with TLS

By the end of this guide, we’ll have transformed our Vault cluster into a fully-fledged PKI store, capable of managing certificates with precision and flexibility. Whether we’re securing microservices, implementing mTLS, or managing SSL/TLS certificates for web applications, this setup will provide us the foundation for robust, scalable, and automated certificate management.

![Root => Intermediate => Identity](hWl-xoUwH88PgHCuUYsajxA.webp)
Certificate Chain

# Enable PKI (Public Key Infrastructure) Secret Engine and Setup

As we saw in previous post, we have to enable all secret engines that we plan to use with Vault. For Certificates, we would enable PKI at any path we like:

_vault secrets enable -path=pki-indubit pki_

Then we set max TTL to 10 years:  
_vault secrets tune -max-lease-ttl=87600h pki-indubit_

**We can generate Root CA:**  
_vault write -field=certificate pki-indubit/root/generate/internal common_name=”indubit.local” issuer_name=”indubit-root-2024" ttl=87600h > indubit_root_2024_ca.crt_

We can get key of issuer:  
_vault list pki-indubit/issuers_

Now let us create some Roles that we can use when generating certificates. For this example, we’d keep same permissions but just use different TTL like 1 month, 3 month and 1 year:

```sh
vault write pki-indubit/roles/indubit-local-90 \  
      issuer_ref="$(vault read -field=default pki-indubit/config/issuers)"  \  
      allowed_domains="indubit.local"      allow_subdomains=true   \  
       max_ttl="2160h"  
  
vault write pki-indubit/roles/indubit-local-30 \  
     issuer_ref="$(vault read -field=default pki-indubit/config/issuers)" \  
     allowed_domains="indubit.local"      allow_subdomains=true    \  
     max_ttl="720h"  
  
vault write pki-indubit/roles/indubit-local-365 \  
     issuer_ref="$(vault read -field=default pki-indubit/config/issuers)"  \  
     allowed_domains="indubit.local"      allow_subdomains=true \  
     max_ttl="8760h"  
  
vault write pki-indubit/roles/all-365  \  
    issuer_ref="$(vault read -field=default pki-indubit/config/issuers)" \  
     allowed_domains="indubit.local"      allow_subdomains=true   \  
   max_ttl="8760h"  
  
  
vault write pki-indubit/config/urls  \  
    issuing_certificates="https://lb.indubit.local/vlt/v1/pki-indubit/ca" \  
    crl_distribution_points="$VAULT_ADDR/v1/pki-indubit/crl"
```

What we can instead do is that create Intermediate CA with shorter TTL than Root CA and only use that to issue our identity certificates:

What we can instead do is that create Intermediate CA with shorter TTL than Root CA and only use that to issue our identity certificates:

```sh
vault secrets enable -path=pki-indubit-int pki  
# 5 Year Max TTL  
vault secrets tune -max-lease-ttl=43800h pki-indubit-int  
# Generate Intermediate CA  
vault write -format=json pki-indubit-int/intermediate/generate/internal \  
     common_name="indubit.local Intermediate Authority" \  
     issuer_name="indubit-dot-local-intermediate"   \  
   | jq -r '.data.csr' > pki-indubit-intermediate.csr  
  
#Extract our cert  
vault write -format=json pki-indubit/root/sign-intermediate  \  
    issuer_ref="indubit-root-2024"  \  
    csr=@pki-indubit-intermediate.csr      format=pem_bundle ttl="43800h"  \  
    | jq -r '.data.certificate' > intermediate.cert.pem  
  
#Sign  
vault write pki-indubit-int/intermediate/set-signed \   
certificate=@intermediate.cert.pem  
  
#Let us create Roles for Intermediate CA  
vault write pki-indubit-int/roles/indubit-local-90 \  
     issuer_ref="$(vault read -field=default pki-indubit-int/config/issuers)" \  
      allowed_domains="indubit.local"      allow_subdomains=true \  
      max_ttl="2160h"  
  
vault write pki-indubit-int/roles/indubit-local-30 \  
     issuer_ref="$(vault read -field=default pki-indubit-int/config/issuers)" \  
     allowed_domains="indubit.local"      allow_subdomains=true  \  
    max_ttl="720h"  
  
vault write pki-indubit-int/roles/indubit-local-365   \  
   issuer_ref="$(vault read -field=default pki-indubit-int/config/issuers)" \  
     allowed_domains="indubit.local"      allow_subdomains=true   \  
   max_ttl="8760h"  
  
vault write pki-indubit-int/roles/all-365  \  
    issuer_ref="$(vault read -field=default pki-indubit-int/config/issuers)" \  
     allowed_domains="indubit.local"      allow_subdomains=true  \  
    max_ttl="8760h"  
  
#Let us issue test certificate for 24 hour validity using our intermediate CA  
vault write pki-indubit-int/issue/indubit-local-30 \   
common_name="test1.indubit.local" ttl="24h"
```

At this stage, we have setup a Intermediate CA that we’ll use to issue certificates for our environment.

# Issue Certificates for Vault nodes and Enable TLS

As these are certificates for our servers, we would use 365 days validity. On first node, we’d run following commands:

```
# Issue a certificate in json format and store it in file  
vault write -format=json pki-indubit-int/issue/indubit-local-365 \  
 common_name="vault-1.indubit.local" alt_names="vault-1.indubit.local" \  
 ip_sans="192.168.4.155" ttl="8760h"  > vault-1-cert.json  
  
#Extract Certificate and Key  
jq -r '.data.certificate' < vault-1-cert.json > vault-1.crt  
  
jq -r '.data.private_key' < vault-1-cert.json > vault-1.key  
  
jq -r '.data.issuing_ca' < vault-1-cert.json >> vault-1.crt  # Append CA cert  
  
#copy certificate files  
cp vault-1.crt /etc/vault/ssl/vault-1.crt  
  
cp vault-1.key /etc/vault/ssl/vault-1.key  
  
```

Now, let us specify this cert in our Vault configuration and restart Vault:

```sh
ui = true  
  
  
listener "tcp" {  
  address     = "0.0.0.0:8200"  
  tls_cert_file = "/etc/vault/ssl/vault-1.crt"  
  tls_key_file = "/etc/vault/ssl/vault-1.key"  
  #tls_disable = 1  # For homelab/testing purposes only. Enable TLS in production.  
}  
  
storage "raft" {  
  path    = "/opt/vault/data"  
  node_id = "node1"  
}  
  
api_addr     = "https://vault-1.indubit.local:8200"  
cluster_addr = "https://192.168.4.155:8201"
```

We have specified cert and key path and also changed addr to https. Change VAULT_ADDR environment variable to also reflect switch to https.

```sh
sudo systemctl restart vault  
  
vault status
```

Now Repeat these steps for other two nodes. When we restart vault and check vault status we would also see that Leader keeps on changing. Anytime we restart vault on a node that is Active, chances are that one of other nodes would become Leader/Active.

As we are issuing certificates for our local authority, they would not be trusted by clients and browsers. We’d keep on getting security warnings. We can get Intermediate Root CA certificate and install on our machines. This may also be good time to explore UI. Go to created PKI Secret Engine and click on Details for Intermediate CA. From here we can download required certificate and copy it as per client operating system. In Ubuntu we can copy to /etc/ssl/certs and update.

![](ZoIFjHozeLkxcUxSP25P7A.webp)

![](QsGn3r15H3fD_rzuYMbyaQ.webp)

# Download and Revoke Certificates

We can Download or Revoke certificates from this UI also:

![](guFMFWnhyCCbnwulsPqBiw.webp)

![](i03r9ce44qxg4kpag0JIFA.webp)

**Revoke using CLI:**

```sh
#generate a certificate
vault write pki-indubit-int/issue/indubit-local-30 common_name="test3.indubit.local" ttl="24h"

#revoke certificate by serial number
vault write pki-indubit-int/revoke serial_number=2d:99:96:6b:12:c1:9e:7d:92:fc:7b:83:ce
Key                        Value
---                        -----
revocation_time            1727825652
revocation_time_rfc3339    2024-10-01T23:34:12.416202748Z
state                      revoked
```

Can Verify it in UI also (we see Revoke Time):

![](https://miro.medium.com/v2/resize:fit:590/1*m55SzcJI35zk-fuM8xqsmw.png)

Check Revoked Certificate

# Common Certificate

For Vault I had generated individual certificate for each node as they are mostly individually accessed (even if through redirection) but for my load balancer NGINX, I wanted a common certificate. So, I used common_name to specify load balancer DNS name and in aletrnate_names, I specified individual node names:

```
vault write -format=json pki-indubit-int/issue/indubit-local-365 common_name=”lb.indubit.local” alt_names=”nginx-1.indubit.local,nginx-2.indubit.local” ip_sans=”192.168.4.153,192.168.4.154" ttl=”8760” > vault-ngx-cert.json

common_name="lb.indubit.local":
```

The `common_name` is the primary domain name associated with the certificate. In this case, it is `"lb.indubit.local"`, which is the load balancer for the environment.

`**alt_names="nginx-1.indubit.local,nginx-2.indubit.local"**`:

`alt_names` refers to Subject Alternative Names (SANs). These are additional domain names that are also associated with the certificate. In this case, the certificate will be valid for both `nginx-1.indubit.local` and `nginx-2.indubit.local` in addition to the primary `common_name`.

`**ip_sans="192.168.4.153,192.168.4.154"**`:

`ip_sans` refers to IP-based Subject Alternative Names. This allows the certificate to also be valid for the specified IP addresses. In this case, the certificate will be valid for both `192.168.4.153` and `192.168.4.154`

Extracted certificate and key like earlier and copied to both NGINX nodes and specified paths in configuration file.

# Summary:

In this post, we have successfully configured HashiCorp Vault to enhance the security of our environment by setting up the PKI secret engine, generating root and intermediate CAs, and issuing certificates for our Vault nodes. By enabling TLS, we secured all communications within our Vault cluster, eliminating the need to disable TLS during testing or internal use. Beyond Vault, this setup allows us to securely manage certificates and enable encrypted communication for other critical services in our environment, including NGINX, Git, Kafka, and Elasticsearch.

With Vault as a centralized secrets manager, my homelab is now prepared for secure communication across a wide range of services, ensuring that sensitive data is protected throughout.

Thanks!