Managing secrets in a Kubernetes cluster can be a challenging task. Secrets are sensitive pieces of information such as passwords, API keys, and certificates that need to be stored securely and accessed only by authorized applications. In this blog post, we will explore how to use FluxCD, an open-source GitOps tool, to manage secrets in a Kubernetes cluster using external providers.

FluxCD is a tool that automates the deployment of applications to a Kubernetes cluster. It uses a Git repository as the source of truth for the desired state of the cluster and continuously monitors the repository for changes. When a change is detected, FluxCD automatically updates the cluster to match the desired state.

To manage secrets with FluxCD, we will use an external provider such as HashiCorp Vault or AWS Secrets Manager. These providers offer secure storage and retrieval of secrets and can be integrated with FluxCD using a plugin.

# HashiCorp Vault Plugin for FluxCD

HashiCorp Vault is a popular secrets management tool that provides secure storage and retrieval of secrets. The Vault plugin for FluxCD allows you to manage secrets stored in Vault as Kubernetes secrets.

To use the Vault plugin, you need to install it in your cluster and configure it to connect to your Vault instance. Here is an example configuration:

```yaml
apiVersion: flux.weave.works/v1beta1  
kind: HelmRelease  
metadata:  
  name: vault-plugin  
  namespace: flux-system  
spec:  
  releaseName: vault-plugin  
  chart: fluxcd/vault-plugin  
  values:  
    vault:  
      address: https://vault.example.com  
      auth:  
        path: "auth/approle"  
        roleId: "my-role-id"  
        secretId: "my-secret-id"
```

This configuration creates a HelmRelease resource that installs the Vault plugin chart. The `values` section contains the configuration for the plugin, including the address of the Vault instance, the authentication path, and the role and secret IDs.

Once the plugin is installed and configured, you can create a Kubernetes secret that references a secret stored in Vault. Here is an example:

```yaml
apiVersion: v1  
kind: Secret  
metadata:  
  name: my-secret  
  namespace: my-namespace  
type: Opaque  
data:  
  password: $(vault:secret/my-secret:password)
```

This secret references a secret stored in Vault at the path `secret/my-secret`. The `password` key is populated with the value of the `password` field in the Vault secret.

When FluxCD detects this secret, it will use the Vault plugin to retrieve the secret from Vault and create a Kubernetes secret with the same name and namespace.

# AWS Secrets Manager Plugin for FluxCD

AWS Secrets Manager is a secrets management service provided by Amazon Web Services. The Secrets Manager plugin for FluxCD allows you to manage secrets stored in Secrets Manager as Kubernetes secrets.

To use the Secrets Manager plugin, you need to install it in your cluster and configure it to connect to your AWS account. Here is an example configuration:

```yaml
apiVersion: flux.weave.works/v1beta1  
kind: HelmRelease  
metadata:  
  name: aws-plugin  
  namespace: flux-system  
spec:  
  releaseName: aws-plugin  
  chart: fluxcd/aws-secretsmanager-plugin  
  values:  
    aws:  
      region: us-west-2  
      accessKeyID: my-access-key  
      secretAccessKey: my-secret-key
```

This configuration creates a HelmRelease resource that installs the Secrets Manager plugin chart. The `values` section contains the configuration for the plugin, including the AWS region, access key ID, and secret access key.

Once the plugin is installed and configured, you can create a Kubernetes secret that references a secret stored in Secrets Manager. Here is an example:

```yaml
apiVersion: v1  
kind: Secret  
metadata:  
  name: my-secret  
  namespace: my-namespace  
type: Opaque  
data:  
  password: $(aws:secret/my-secret:password)
```

This secret references a secret stored in Secrets Manager with the name `my-secret`. The `password` key is populated with the value of the `password` field in the Secrets Manager secret.

When FluxCD detects this secret, it will use the Secrets Manager plugin to retrieve the secret from Secrets Manager and create a Kubernetes secret with the same name and namespace.