In the dynamic realm of DevOps, secrets play a pivotal role in accessing sensitive resources like servers and databases. However, storing secrets in plain text poses a significant security risk, potentially exposing your systems and data to malicious actors. Enter [SOPS](https://github.com/getsops/sops), a powerful tool designed to securely store secrets — be it either your GitHub/GitLab/BitBucket repository or on a local server. By encrypting secrets with an external secret managing tool, SOPS ensures that your sensitive information remains shielded. In this article, I’ll guide you through the process of leveraging SOPS to fortify your secret management practices.

# SOPS vs. Modern Secret Storage

Nowadays, secrets often find their home in secure locations such as [HashiCorp Vault](https://www.vaultproject.io/), [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html), [GCP KMS](https://cloud.google.com/security-key-management), [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/basic-concepts),, or popular credential storage solutions like [1Password](https://1password.com/). While this approach isolates secrets from the code, it introduces an extra step. Because before one can use, it needs to fetch it from the secret managing system and incorporate it into code. And now consider the code is containerized, meaning you’d potentially needed to rebuild the container image and who knows how many additional steps might be in your specific scenario.

SOPS offers a simplified version of secret storage. With SOPS, the secret values are directly stored in the code, and they are encrypted using a key that is stored in a separate file managed by an external key management system. This way, the secret values are never stored in plain text, even in the code. To decrypt the secret values, the code must use the key file. This eliminates the need to fetch the secret values from a vault or credential manager, which reduces the security risk.
# SOPS and Kubernetes Secrets

Let’s take a practical look at SOPS and Kubernetes Secrets. We’ll deploy a Kubernetes Secret object manifest to a cluster, encrypt it using GCP KMS, and share it securely with other developers via your VCS.

First, a quick rundown of the terms and tools:

- [**Kubernetes Secret Object**](https://kubernetes.io/docs/concepts/configuration/secret/): A manifest holding secrets for Kubernetes use that keeps your confidential data separate from your application code.
- [**GCP KMS**](https://cloud.google.com/security-key-management?hl=en): A key management service for creating and managing encryption keys. Perfect for encrypting secrets in your Kubernetes cluster deployment.

![](JilrdzQQEkTqQtC5dPVBiA.webp)

Now, imagine you’ve got this Kubernetes Secret you want to store on GitHub or GitLab:

```yaml
apiVersion: v1  
kind: Secret  
metadata:  
    name: example-secret  
stringData:  
    SECRET: <secret>
```

But hold up, you can’t toss this plain text file onto your VCS. Your repository might be secure and only a limited amount of developers have access to it, but what if the repo gets compromised or you’re just VCS-paranoid?

In this case, SOPS can help. First things first, install it — check out their documentation [**here**](https://github.com/getsops/sops). Next, on the cloud provider side, like GCP in our case, create a GCP KMS key. Ensure your authenticated GCP Service Account has the right access and permissions for this key. Once sorted, you’re ready to encrypt your secret. Just follow these steps:

- Create the SOPS configuration file and place at the root of your repository:

```yaml
creation_rules:  
    gcp_kms: <absolute-path-to-the-gcp-kms>  
    encrypted_regex: ^(stringData)$
```

- Encrypt your secrets:

`sops --encrypt -i ./your-secret-file.yaml > your-secret-file.enc.yaml`

Now you’ve got an encrypted file ready for commiting it to your VCS. Any dev or GCP Service Account with access to GCP KMS can decrypt and use it:

```yaml
apiVersion: v1
kind: Secret  
metadata:  
    name: example-secret  
stringData:  
    SECRET: <encrypted-secret>  
sops:  
    kms: []  
    gcp_kms:  
        - resource_id: <path-to-your-gcp-kms>  
          created_at: "2023-10-08T08:24:07Z"  
          enc: <...>  
    azure_kv: []  
    hc_vault: []  
    age: []  
    lastmodified: "2023-10-08T08:24:08Z"  
    mac: <...>  
    pgp: []  
    encrypted_regex: ^(stringData|SECRET)$  
    version: 3.7.3
```

> 📌 **Pro Tip**: If you’re up for encrypting and applying the Kubernetes Secret in one go, use this one-liner: `**sops --decrypt your-secret-file.enc.yaml | kubectl apply -f -**`. Furthermore you can automate this decryption and application easily with tools like Terraform or Ansible.

