Within the rapidly changing world of decentralized finance, a lot is at stake in terms of asset management: securely and effectively. **OpenBAO** takes it to the next level and builds on the strong foundations of HashiCorp Vault-or in short, HCP Vault-by providing a robust, decentralized solution for asset storage, tokenization, and financial agreements.

# 🚀 What is OenBao Vault?

OenBao Vault is a secure, cloud-native secret management solution, forked from HashiCorp Vault’s HCP (HashiCorp Cloud Platform). It retains the core functionalities of HashiCorp Vault (dynamic secrets, encryption-as-a-service, etc.), but with enhancements like:

- **🌐 Extended Plugin Ecosystem**: Supports integrations with more cloud platforms and enterprise apps.
- **🔐 Advanced User Management**: Features custom roles, better MFA support, and tighter IdP integration.
- **📊 Enhanced Auditing**: Comes with advanced auditing and compliance reporting capabilities.
- **📈 High Availability Optimized**: Ideal for mission-critical applications that need uptime guarantees.

# Key Differences Between OenBao Vault and HashiCorp Vault HCP🔍

1. **Cloud-First Design**: Unlike HashiCorp Vault HCP, which is designed for HashiCorp Cloud, OenBao Vault is built to be cloud-agnostic and integrates seamlessly with a broader range of cloud providers, including hybrid environments.
2. **Community-Driven**: The OenBao Vault community is highly active and regularly contributes new features and plugins, resulting in faster iteration cycles.
3. **Flexible Licensing**: OenBao Vault offers an open-source, community-driven alternative with fewer licensing restrictions than HashiCorp’s Enterprise offerings.
4. **Simplified UI**: OenBao Vault comes with a more user-friendly interface, reducing the complexity of secret management for teams without dedicated DevOps engineers.

# 🛠️ Tutorial: Installing OenBao Vault on Kubernetes (K8s)

In this section, we’ll walk through installing OenBao Vault on a Kubernetes cluster. We’ll also set up High Availability (HA) to ensure minimal downtime in production environments.

## Prerequisites

- A Kubernetes cluster up and running (minikube, k3s, or a managed service like GKE, EKS, etc.)
- Helm installed on your local machine
- kubectl configured for your cluster
- [OpenTofu](https://opentofu.org) installed (optional for later steps)

## Step 1: Add the OenBao Vault Helm Repository

Helm simplifies the installation of OenBao Vault. First, we need to add the official Helm repository:

```sh
helm repo add oenbao-vault https://charts.oenbao.com/vault  
helm repo update
```

## Step 2: Install OenBao Vault Using Helm

Next, we’ll install OenBao Vault using Helm. You can modify the values file to suit your requirements (e.g., storage backend, replication, etc.).

```sh
helm install oenbao-vault oenbao-vault/vault --namespace vault --create-namespace
```

The basic installation will spin up a single-instance deployment of OenBao Vault. However, for production use, you should configure it for High Availability (HA).

# Configuring High Availability (HA) for OenBao Vault

High Availability ensures that OenBao Vault remains up even if individual pods fail. We’ll set up OenBao Vault with an HA configuration, using Consul as the storage backend and leader election mechanism.

## Step 1: Set Up Consul (Storage Backend)

First, install Consul, which OenBao Vault will use to store its data and manage leader election. Use the Consul Helm chart for deployment:

```sh
helm repo add hashicorp https://helm.releases.hashicorp.com  
helm install consul hashicorp/consul --namespace vault
```

Once Consul is up and running, ensure it’s exposed internally so OenBao Vault can communicate with it.

## Step 2: Update OenBao Vault Values for HA

Now, configure OenBao Vault to run in HA mode and connect to the Consul backend. You can edit the `values.yaml` or pass parameters with `helm install`:

```yaml
ha:
  enabled: true
  replicas: 3
  storage:
    consul:
      address: "consul.vault:8500"
      path: "vault/"
```

Apply this configuration and update your deployment:

```sh
helm upgrade oenbao-vault oenbao-vault/vault --namespace vault --values values.yaml
```

With this setup, OenBao Vault will run 3 replicas, and Consul will manage storage and leader election.

# 🧑‍💻 Managing OenBao Vault with OpenTofu

If you’re looking for an open-source alternative to Terraform, **OpenTofu** is a great choice for managing your infrastructure. Let’s automate the deployment of OenBao Vault using OpenTofu.

## Step 1: Install OpenTofu

You can install OpenTofu by following their official guide here. Once installed, initialize a new Tofu project:

`opentofu init`

## Step 2: Define Your Kubernetes Provider

Create a file called `main.tf` and configure the Kubernetes provider:

```c
provider "kubernetes" {  
  config_path = "~/.kube/config"  
}  
  
resource "helm_release" "oenbao_vault" {  
  name       = "oenbao-vault"  
  repository = "https://charts.oenbao.com/vault"  
  chart      = "vault"  
  namespace  = "vault"  
    
  set {  
    name  = "ha.enabled"  
    value = "true"  
  }  
  set {  
    name  = "ha.replicas"  
    value = "3"  
  }  
  set {  
    name  = "storage.consul.address"  
    value = "consul.vault:8500"  
  }  
}

```
## Step 3: Apply Your Configuration

Finally, apply the configuration using OpenTofu:

`opentofu apply`

