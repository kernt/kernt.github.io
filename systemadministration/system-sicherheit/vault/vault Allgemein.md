---
tags:
  - vault
  - hashicorp
---

# Hashicorp Vault

**Namespace erstellen und Import des Wildcard Zertifikats in Kubernetes**

```sh
k create ns vault
k create secret tls tanzu-mgmt-svc.os-fra.local-tls -n vault --cert=tanzu-mgmt-svc.os-fra.local.crt --key=tanzu-mgmt-svc.os-fra.local.key
```

**Initalisieren Vault**

```sh
#init
kubectl exec vault-0 -- vault operator init -key-shares=8 -key-threshold=3 -format=json > cluster-keys.json
#unseal
kubectl exec vault-0 -- vault operator unseal <key1>
kubectl exec vault-0 -- vault operator unseal <key2>
kubectl exec vault-0 -- vault operator unseal <key3>
#set root token
CLUSTER_ROOT_TOKEN=$(cat cluster-keys.json | jq -r ".root_token")
kubectl exec vault-0 -- vault login $CLUSTER_ROOT_TOKEN
#peers anzeigen
kubectl exec vault-0 -- vault operator raft list-peers
#vault-1 zum cluster joinen
kubectl exec vault-1 -- vault operator raft join http://vault-0.vault-internal:8200
#vault unseal für vault-1
kubectl exec vault-1 -- vault operator unseal <key1>
kubectl exec vault-1 -- vault operator unseal <key2>
kubectl exec vault-1 -- vault operator unseal <key3>
#vault-2 in den cluster joinen
kubectl exec vault-2 -- vault operator raft join http://vault-0.vault-internal:8200
#vault-2 unseal
kubectl exec vault-2 -- vault operator unseal <key1>
kubectl exec vault-2 -- vault operator unseal <key2>
kubectl exec vault-2 -- vault operator unseal <key3>
#in den container springen
kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh
#kv-v2 secrets aktivieren
vault secrets enable -path=secret kv-v2
```

# Introduction

In today’s digital landscape, the security of sensitive data is paramount. With cyber threats on the rise, organizations must prioritize robust secrets management to protect their critical information. [HashiCorp](https://www.hashicorp.com/?ref=8grams.tech) Vault emerges as a leading solution, offering a comprehensive platform to manage secrets and ensure data security. This article explores the various facets of HashiCorp Vault, from its inception to its practical applications, highlighting why it stands out as an essential tool for modern IT infrastructure.

## The Evolution of Secrets Management

Traditional methods of secrets management often involved storing sensitive information in configuration files or environment variables, which were easily accessible and vulnerable to breaches. Manual rotation and management of secrets added to the complexity, making it difficult to maintain security and compliance. These approaches were not scalable and could not keep up with the dynamic nature of modern IT environments.

HashiCorp Vault revolutionized secrets management by introducing automated processes and enhanced security protocols. With Vault, secrets are stored securely in an encrypted storage backend, significantly reducing the risk of unauthorized access. Vault’s ability to dynamically generate secrets on demand further enhances security, ensuring that credentials are short-lived and only available for as long as necessary. This shift towards automation and centralization has made secrets management more efficient and secure, aligning with modern DevOps practices and enabling seamless integration into CI/CD pipelines.

# Core Features

HashiCorp Vault offers a robust set of features designed to address the diverse needs of organizations. At its core, Vault provides secure storage mechanisms, ensuring that secrets are encrypted both at rest and in transit. This encryption is managed through various backends, each tailored to specific use cases, such as cloud-based storage or on-premises solutions.

![](https://miro.medium.com/v2/resize:fit:700/0*sWglO_jkTpIATPEr)

One of Vault’s standout features is its ability to generate dynamic secrets. Unlike static secrets, which remain unchanged until manually rotated, dynamic secrets are created on demand and have a limited lifespan. This reduces the window of opportunity for potential attackers and simplifies the process of managing and rotating credentials. For example, a dynamic secret could be a database credential that is valid only for a specific duration, after which it expires automatically.

Vault also offers encryption as a service (EaaS), enabling organizations to encrypt data without having to manage the underlying cryptographic infrastructure. This feature supports various use cases, such as database encryption and application-level encryption, providing flexibility and ease of use. Additionally, Vault’s role-based access control (RBAC) ensures that only authorized users and applications can access specific secrets, further enhancing security.

Comprehensive audit logging is another critical feature of Vault, providing detailed records of all interactions with the system. This includes who accessed what secrets and when, enabling organizations to maintain compliance with regulatory requirements and quickly identify any suspicious activity. Vault’s architecture is designed to be extensible, with support for various secret engines and plugins that allow users to customize and extend its functionality to meet specific needs.

# Use Cases

Adopting HashiCorp Vault brings numerous benefits to organizations, starting with enhanced security and compliance. By centralizing the management of secrets and automating their lifecycle, Vault reduces the risk of human error and ensures that sensitive information is always protected. This is particularly important for organizations operating in highly regulated industries, where maintaining compliance with standards such as GDPR, HIPAA, or PCI-DSS is crucial.

Vault’s centralized approach simplifies secrets management, allowing organizations to manage all their secrets from a single platform. This streamlines operations and makes it easier to enforce security policies consistently across the entire infrastructure. Additionally, Vault’s scalability ensures that it can grow with the organization, supporting a wide range of environments from small startups to large enterprises with complex, multi-cloud architectures.

Operational efficiency is another significant advantage of using Vault. Automating the management of secrets reduces the time and effort required to handle tasks such as credential rotation, access control configuration, and audit logging. This not only frees up valuable resources but also minimizes the risk of errors and security breaches. For example, an organization using Vault can automate the rotation of database passwords, ensuring that they are updated regularly without manual intervention.

## Practical Applications in Various Scenarios

HashiCorp Vault is versatile and can be applied to numerous real-world scenarios. One common use case is managing API keys and tokens. These sensitive credentials are often used to authenticate and authorize access to various services and applications. With Vault, API keys and tokens can be securely stored, rotated automatically, and accessed only by authorized users or systems, reducing the risk of unauthorized access and ensuring that credentials are always up to date.

Securing credentials across different environments is another critical application of Vault. Whether it’s a development, testing, or production environment, Vault provides a centralized solution for managing credentials, ensuring consistent security policies and reducing the risk of configuration errors. This is particularly useful in multi-cloud and hybrid environments, where maintaining consistent security practices can be challenging.

Vault’s encryption capabilities make it an ideal solution for protecting sensitive data. Organizations can use Vault to encrypt data stored in databases, filesystems, or cloud storage services, ensuring that even if the underlying storage is compromised, the data remains secure. Vault’s encryption as a service (EaaS) feature simplifies the process of integrating encryption into applications, providing developers with easy-to-use APIs to encrypt and decrypt data.

Dynamic secrets are particularly useful in scenarios where temporary access is required. For example, in a CI/CD pipeline, Vault can generate short-lived credentials for accessing various services during the build and deployment process. These credentials expire automatically after a set period, reducing the risk of them being misused if they are exposed. Similarly, dynamic secrets can be used to provide temporary access to sensitive systems for contractors or third-party vendors, ensuring that access is granted only for the necessary duration.

![](https://miro.medium.com/v2/resize:fit:700/0*MCSjWOr81IBWIRWd.png)

Implementing a zero trust security model is another area where Vault excels. Zero trust is based on the principle of “never trust, always verify,” meaning that every request for access must be authenticated and authorized, regardless of its origin. Vault’s robust authentication and authorization mechanisms, combined with its ability to generate dynamic secrets, make it an ideal tool for implementing zero trust security. Organizations can ensure that access to sensitive resources is tightly controlled and monitored, reducing the risk of unauthorized access and data breaches.

# Vault’s Architecture

Understanding the architecture of HashiCorp Vault is essential for grasping how it delivers its powerful features. At a high level, Vault consists of several core components: the storage backend, the API, and the user interface.

![](https://miro.medium.com/v2/resize:fit:700/0*9tBu33FHyg-K8cyM)

The storage backend is where Vault stores secrets and metadata. Vault supports a variety of storage backends, including cloud-based solutions like AWS S3 and Google Cloud Storage, as well as on-premises options like Consul and etcd. The choice of storage backend depends on the specific requirements of the organization, such as performance, scalability, and availability.

The API is the primary interface for interacting with Vault. It provides endpoints for storing, retrieving, and managing secrets, as well as for configuring and monitoring Vault itself. The API is designed to be flexible and extensible, allowing developers to integrate Vault into their applications and automate various tasks.

![](https://miro.medium.com/v2/resize:fit:700/0*6lwt_Vfcmc_S5wMo)

The user interface (UI) provides a visual interface for interacting with Vault. While the API is powerful and flexible, the UI makes it easier for users to perform common tasks, such as managing secrets, configuring policies, and viewing audit logs. The UI is particularly useful for administrators who need to manage Vault instances and monitor their activity.

![](https://miro.medium.com/v2/resize:fit:700/0*yopQvFDHoEflVCAI)

Authentication and authorization are critical aspects of Vault’s architecture. Vault supports a wide range of authentication methods, including token-based authentication, username and password, LDAP, AWS IAM, and more. This flexibility allows organizations to integrate Vault with their existing identity management systems. Once authenticated, access to secrets is controlled through policies, which define the actions that users or applications can perform on specific secrets.

Secret engines are pluggable components that extend Vault’s functionality. Each secret engine is responsible for managing a specific type of secret, such as database credentials, API keys, or certificates. Vault comes with several built-in secret engines, and the plugin architecture allows users to develop custom secret engines to meet their specific needs.

Extending Vault with plugins is another powerful feature. Plugins can be used to add new functionality to Vault, such as support for additional storage backends, authentication methods, or secret engines. This extensibility makes Vault highly adaptable to different use cases and environments.

Performance and scalability are also key considerations in Vault’s architecture. Vault is designed to handle high volumes of requests and large amounts of data, making it suitable for use in large, complex environments. Features such as performance replication and integrated storage help ensure that Vault can scale to meet the demands of growing organizations.

Security mechanisms within Vault’s architecture are robust and comprehensive. All data stored in Vault is encrypted, and strict access controls ensure that only authorized users and applications can access secrets. Detailed audit logging provides visibility into all interactions with Vault, helping organizations maintain compliance and quickly identify any suspicious activity.

# Competitive Landscape and Comparisons

HashiCorp Vault operates in a competitive landscape, with several other tools offering similar functionality. Some of the notable competitors include AWS Secrets Manager, Azure Key Vault, and CyberArk. Each of these tools has its strengths and weaknesses, and choosing the right one depends on the specific needs of the organization.

[AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) is a fully managed service that integrates seamlessly with other AWS services. It offers features such as automated secrets rotation and fine-grained access control. However, it is primarily designed for use within the AWS ecosystem, which may limit its appeal for organizations using multi-cloud or hybrid environments.

![](https://miro.medium.com/v2/resize:fit:700/0*etDiVlnDe74pkhPU.png)

[Azure Key Vault](https://azure.microsoft.com/en-us/products/key-vault) provides similar functionality within the Azure ecosystem. It supports the storage and management of secrets, certificates, and keys, and integrates with other Azure services. Like AWS Secrets Manager, its primary limitation is its focus on a specific cloud platform.

[CyberArk](https://www.cyberark.com/) is a more comprehensive solution that offers a wide range of security features, including privileged access management and threat detection. While it provides robust secrets management capabilities, it is generally more complex and expensive than Vault, making it less suitable for smaller organizations or those looking for a lightweight solution.

A table comparing the key features of these tools can help highlight Vault’s unique advantages:

![](https://miro.medium.com/v2/resize:fit:700/1*KwYx-kTyyaRoWTm01mB6mg.png)

Vault vs AWS Secret Manager vs Azure Key Vault vs CyberArk

Vault’s multi-cloud support, dynamic secrets generation, and extensibility with plugins make it a versatile and powerful tool for a wide range of use cases. Its open-source nature and active community also contribute to its ongoing development and improvement, ensuring that it continues to meet the evolving needs of its users.

# Install Vault on Kubernetes

# ntroduction

In today’s digital landscape, the security of sensitive data is paramount. With cyber threats on the rise, organizations must prioritize robust secrets management to protect their critical information. [HashiCorp](https://www.hashicorp.com/?ref=8grams.tech) Vault emerges as a leading solution, offering a comprehensive platform to manage secrets and ensure data security. This article explores the various facets of HashiCorp Vault, from its inception to its practical applications, highlighting why it stands out as an essential tool for modern IT infrastructure.

## The Story Behind Vault

The creation of [HashiCorp Vault](https://www.vaultproject.io/?ref=8grams.tech) was driven by a pressing need for more secure and efficient secrets management solutions. Prior to Vault, organizations often relied on outdated methods such as _hard-coded_ secrets into application code or manually managing credentials, which posed significant security risks. HashiCorp, a company renowned for its contributions to the DevOps and infrastructure automation space, identified these gaps and set out to develop a tool that would address these challenges comprehensively.

Vault’s development journey began with a clear vision: to provide a secure, automated, and centralized platform for managing secrets. The team at HashiCorp, led by industry experts, embarked on this mission, releasing the first version of Vault in 2015. Since then, Vault has undergone numerous updates, each enhancing its capabilities and solidifying its position as a leader in the field. The open-source community has played a crucial role in this evolution, contributing to Vault’s growth and ensuring it meets the ever-evolving needs of its users.

## The Evolution of Secrets Management

Traditional methods of secrets management often involved storing sensitive information in configuration files or environment variables, which were easily accessible and vulnerable to breaches. Manual rotation and management of secrets added to the complexity, making it difficult to maintain security and compliance. These approaches were not scalable and could not keep up with the dynamic nature of modern IT environments.

HashiCorp Vault revolutionized secrets management by introducing automated processes and enhanced security protocols. With Vault, secrets are stored securely in an encrypted storage backend, significantly reducing the risk of unauthorized access. Vault’s ability to dynamically generate secrets on demand further enhances security, ensuring that credentials are short-lived and only available for as long as necessary. This shift towards automation and centralization has made secrets management more efficient and secure, aligning with modern DevOps practices and enabling seamless integration into CI/CD pipelines.

# Core Features

HashiCorp Vault offers a robust set of features designed to address the diverse needs of organizations. At its core, Vault provides secure storage mechanisms, ensuring that secrets are encrypted both at rest and in transit. This encryption is managed through various backends, each tailored to specific use cases, such as cloud-based storage or on-premises solutions.

![](https://miro.medium.com/v2/resize:fit:700/0*sWglO_jkTpIATPEr)

One of Vault’s standout features is its ability to generate dynamic secrets. Unlike static secrets, which remain unchanged until manually rotated, dynamic secrets are created on demand and have a limited lifespan. This reduces the window of opportunity for potential attackers and simplifies the process of managing and rotating credentials. For example, a dynamic secret could be a database credential that is valid only for a specific duration, after which it expires automatically.

Vault also offers encryption as a service (EaaS), enabling organizations to encrypt data without having to manage the underlying cryptographic infrastructure. This feature supports various use cases, such as database encryption and application-level encryption, providing flexibility and ease of use. Additionally, Vault’s role-based access control (RBAC) ensures that only authorized users and applications can access specific secrets, further enhancing security.

Comprehensive audit logging is another critical feature of Vault, providing detailed records of all interactions with the system. This includes who accessed what secrets and when, enabling organizations to maintain compliance with regulatory requirements and quickly identify any suspicious activity. Vault’s architecture is designed to be extensible, with support for various secret engines and plugins that allow users to customize and extend its functionality to meet specific needs.

# Use Cases

Adopting HashiCorp Vault brings numerous benefits to organizations, starting with enhanced security and compliance. By centralizing the management of secrets and automating their lifecycle, Vault reduces the risk of human error and ensures that sensitive information is always protected. This is particularly important for organizations operating in highly regulated industries, where maintaining compliance with standards such as GDPR, HIPAA, or PCI-DSS is crucial.

Vault’s centralized approach simplifies secrets management, allowing organizations to manage all their secrets from a single platform. This streamlines operations and makes it easier to enforce security policies consistently across the entire infrastructure. Additionally, Vault’s scalability ensures that it can grow with the organization, supporting a wide range of environments from small startups to large enterprises with complex, multi-cloud architectures.

Operational efficiency is another significant advantage of using Vault. Automating the management of secrets reduces the time and effort required to handle tasks such as credential rotation, access control configuration, and audit logging. This not only frees up valuable resources but also minimizes the risk of errors and security breaches. For example, an organization using Vault can automate the rotation of database passwords, ensuring that they are updated regularly without manual intervention.

## Practical Applications in Various Scenarios

HashiCorp Vault is versatile and can be applied to numerous real-world scenarios. One common use case is managing API keys and tokens. These sensitive credentials are often used to authenticate and authorize access to various services and applications. With Vault, API keys and tokens can be securely stored, rotated automatically, and accessed only by authorized users or systems, reducing the risk of unauthorized access and ensuring that credentials are always up to date.

Securing credentials across different environments is another critical application of Vault. Whether it’s a development, testing, or production environment, Vault provides a centralized solution for managing credentials, ensuring consistent security policies and reducing the risk of configuration errors. This is particularly useful in multi-cloud and hybrid environments, where maintaining consistent security practices can be challenging.

Vault’s encryption capabilities make it an ideal solution for protecting sensitive data. Organizations can use Vault to encrypt data stored in databases, filesystems, or cloud storage services, ensuring that even if the underlying storage is compromised, the data remains secure. Vault’s encryption as a service (EaaS) feature simplifies the process of integrating encryption into applications, providing developers with easy-to-use APIs to encrypt and decrypt data.

Dynamic secrets are particularly useful in scenarios where temporary access is required. For example, in a CI/CD pipeline, Vault can generate short-lived credentials for accessing various services during the build and deployment process. These credentials expire automatically after a set period, reducing the risk of them being misused if they are exposed. Similarly, dynamic secrets can be used to provide temporary access to sensitive systems for contractors or third-party vendors, ensuring that access is granted only for the necessary duration.

![](https://miro.medium.com/v2/resize:fit:700/0*MCSjWOr81IBWIRWd.png)

Implementing a zero trust security model is another area where Vault excels. Zero trust is based on the principle of “never trust, always verify,” meaning that every request for access must be authenticated and authorized, regardless of its origin. Vault’s robust authentication and authorization mechanisms, combined with its ability to generate dynamic secrets, make it an ideal tool for implementing zero trust security. Organizations can ensure that access to sensitive resources is tightly controlled and monitored, reducing the risk of unauthorized access and data breaches.

# Vault’s Architecture

Understanding the architecture of HashiCorp Vault is essential for grasping how it delivers its powerful features. At a high level, Vault consists of several core components: the storage backend, the API, and the user interface.

![](https://miro.medium.com/v2/resize:fit:700/0*9tBu33FHyg-K8cyM)

The storage backend is where Vault stores secrets and metadata. Vault supports a variety of storage backends, including cloud-based solutions like AWS S3 and Google Cloud Storage, as well as on-premises options like Consul and etcd. The choice of storage backend depends on the specific requirements of the organization, such as performance, scalability, and availability.

The API is the primary interface for interacting with Vault. It provides endpoints for storing, retrieving, and managing secrets, as well as for configuring and monitoring Vault itself. The API is designed to be flexible and extensible, allowing developers to integrate Vault into their applications and automate various tasks.

![](https://miro.medium.com/v2/resize:fit:700/0*6lwt_Vfcmc_S5wMo)

The user interface (UI) provides a visual interface for interacting with Vault. While the API is powerful and flexible, the UI makes it easier for users to perform common tasks, such as managing secrets, configuring policies, and viewing audit logs. The UI is particularly useful for administrators who need to manage Vault instances and monitor their activity.

![](https://miro.medium.com/v2/resize:fit:700/0*yopQvFDHoEflVCAI)

Authentication and authorization are critical aspects of Vault’s architecture. Vault supports a wide range of authentication methods, including token-based authentication, username and password, LDAP, AWS IAM, and more. This flexibility allows organizations to integrate Vault with their existing identity management systems. Once authenticated, access to secrets is controlled through policies, which define the actions that users or applications can perform on specific secrets.

Secret engines are pluggable components that extend Vault’s functionality. Each secret engine is responsible for managing a specific type of secret, such as database credentials, API keys, or certificates. Vault comes with several built-in secret engines, and the plugin architecture allows users to develop custom secret engines to meet their specific needs.

Extending Vault with plugins is another powerful feature. Plugins can be used to add new functionality to Vault, such as support for additional storage backends, authentication methods, or secret engines. This extensibility makes Vault highly adaptable to different use cases and environments.

Performance and scalability are also key considerations in Vault’s architecture. Vault is designed to handle high volumes of requests and large amounts of data, making it suitable for use in large, complex environments. Features such as performance replication and integrated storage help ensure that Vault can scale to meet the demands of growing organizations.

Security mechanisms within Vault’s architecture are robust and comprehensive. All data stored in Vault is encrypted, and strict access controls ensure that only authorized users and applications can access secrets. Detailed audit logging provides visibility into all interactions with Vault, helping organizations maintain compliance and quickly identify any suspicious activity.

# Competitive Landscape and Comparisons

HashiCorp Vault operates in a competitive landscape, with several other tools offering similar functionality. Some of the notable competitors include AWS Secrets Manager, Azure Key Vault, and CyberArk. Each of these tools has its strengths and weaknesses, and choosing the right one depends on the specific needs of the organization.

[AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) is a fully managed service that integrates seamlessly with other AWS services. It offers features such as automated secrets rotation and fine-grained access control. However, it is primarily designed for use within the AWS ecosystem, which may limit its appeal for organizations using multi-cloud or hybrid environments.

![](https://miro.medium.com/v2/resize:fit:700/0*etDiVlnDe74pkhPU.png)

[Azure Key Vault](https://azure.microsoft.com/en-us/products/key-vault) provides similar functionality within the Azure ecosystem. It supports the storage and management of secrets, certificates, and keys, and integrates with other Azure services. Like AWS Secrets Manager, its primary limitation is its focus on a specific cloud platform.

[CyberArk](https://www.cyberark.com/) is a more comprehensive solution that offers a wide range of security features, including privileged access management and threat detection. While it provides robust secrets management capabilities, it is generally more complex and expensive than Vault, making it less suitable for smaller organizations or those looking for a lightweight solution.

A table comparing the key features of these tools can help highlight Vault’s unique advantages:

![](https://miro.medium.com/v2/resize:fit:700/1*KwYx-kTyyaRoWTm01mB6mg.png)

Vault vs AWS Secret Manager vs Azure Key Vault vs CyberArk

Vault’s multi-cloud support, dynamic secrets generation, and extensibility with plugins make it a versatile and powerful tool for a wide range of use cases. Its open-source nature and active community also contribute to its ongoing development and improvement, ensuring that it continues to meet the evolving needs of its users.

# Install Vault on Kubernetes

# https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket

```
resource "google_storage_bucket" "my-vault-bucket" {
  name = "my-vault-bucket"
  location = "US-CENTRAL1"
  storage_class = "STANDARD"

  uniform_bucket_level_access = false

  lifecycle_rule {
    condition {
      days_since_noncurrent_time = 7
    }
    action {
      type = "Delete"
    }
  }

  lifecycle_rule {
    condition {
      num_newer_versions = 3
      with_state = "ARCHIVED"
    }
    action {
      type = "Delete"
    }
  }
  
  versioning {
    enabled = true
  }
}

# https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/google_service_account
resource "google_service_account" "vault-bucket-sa" {
  account_id = "vault-bucket-sa"
}

# https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/google_project_iam
resource "google_project_iam_member" "vault-bucket-im" {
  project = "my-project"
  role    = "roles/storage.admin"
  member  = "serviceAccount:${google_service_account.vault-bucket-sa.email}"
}

# https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket_acl
resource "google_storage_bucket_acl" "my-vault-bucket-acl" {
  bucket = google_storage_bucket.my-vault-bucket.name

  role_entity = [
    "OWNER:user-${google_service_account.vault-bucket-sa.email}"
  ]
}
```

Download that Service Account’s key from . Save the key somewhere in your disk, for example `vault-sa.json`, and create a Kubernetes's Secret from that.

Download the Service Account key from the Google Cloud Console: navigate to [IAM > Service Account](https://console.cloud.google.com/iam-admin/serviceaccounts?ref=8grams.tech). Save the key to your disk, for example as `vault-sa.json`, and create a Kubernetes Secret from it.

~$ kubectl create namespace vault  
~$ kubectl create secret generic vault-config --from-file=/path/to/vault-sa.json/ -n vault

Create `values.yaml` to override Vault's Helm default configuration

```
server:
  extraEnvironmentVars: 
    GOOGLE_REGION: US-CENTRAL1
    GOOGLE_PROJECT: my-project
    GOOGLE_APPLICATION_CREDENTIALS: /vault/userconfig/vault-config/vault-sa.json

  extraVolumes:
    - type: secret
      name: vault-config
      path: null

  standalone:
    enabled: "-"
    config: |
      ui = true
      listener "tcp" {
        tls_disable = 1
        address = "[::]:8200"
        cluster_address = "[::]:8201"
      }
      storage "gcs" {
        bucket = "my-vault-bucket"
      }
```

# Install Vault via Helm

~$ helm repo add hashicorp https://helm.releases.hashicorp.com  
~$ helm install vault hashicorp/vault -n vault -f values.yml

Check the installation

~$ kubectl -n vault get pods  
NAME                               READY   STATUS    RESTARTS   AGE  
vault-0                            1/1     Running   0          1m  
vault-agent-injector-xxx           1/1     Running   0          1m

Nice! Vault is successfully installed in our Kubernetes cluster. However, the default state of Vault after installation is `sealed`. We must `unseal` it before we can use it.

Download Vault’s credentials and save them securely. Please be careful with this file as it is very important. It is our key to access Vault, where our credentials reside.

~$ kubectl -n vault exec vault-0 -- vault operator init \  
     -key-shares=1 \  
     -key-threshold=1 \  
     -format=json > cluster-keys.json

Get `unsealed` key from `cluster-keys.json` and unseal Vault

~$ cat cluster-keys.json | jq -r ".unseal_keys_b64[]"  
~$ kubectl -n vault exec vault-0 -- vault operator unseal <unseal key>

# Install Ingress

In order to our Vault can be accessed over the internet, we can install Ingress on it.

```
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vault
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-issuer
spec:
  rules:
    - host: vault.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service: 
                name: vault
                port: 
                  number: 8200
  tls:
    - hosts:
        - vault.example.com
      secretName: tls-secret-ops-vault
```

d then install

~$ kubectl -n vault apply ingress.yml

Open your browser and go to [https://vault.example.com](https://vault.example.com/?ref=8grams.tech) and you should see Vault login page like below:

![](https://miro.medium.com/v2/resize:fit:700/0*NAEcmiopflw1zNJi.png)

You can login with `root_token` from `cluster-keys.json`.

