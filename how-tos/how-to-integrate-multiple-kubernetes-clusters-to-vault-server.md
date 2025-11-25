---
tags:
  - vault
  - sicherheit
  - kubernetes
---
In case you have many clusters as part of your infrastructure and you would wish to introduce _HashiCorp Vault_ to take care of your secrets, then worry no more. In this guide, we are going to configure Develop, Staging and Production Kubernetes Authentication methods in Vault server then proceed to add secrets, policies, roles and link them all together to our separate Kubernetes clusters (Develop, Staging and Production). If you have more clusters, you can use similar pattern that we will be using to add them.

Definition of terms

- **Policies**: Policies provide a declarative way to grant or forbid access to certain paths and operations in Vault.
- **Roles:** Policies are attached to tokens and roles to enforce client permissions on Vault.
- **Secrets**: A secret is anything that you want to tightly control access to, such as API encryption keys, passwords, or certificates

Since we have three clusters, we are going to begin with Develop, then Staging and then complete with Production.
## 1. Develop cluster

### Configuring Dev cluster Kubernetes Auth Method

Before we can configure Kubernetes Auth, we need to have a few things in our basket. We need JWT reviewer token, Kubernetes ca certificate, Kubernetes service account and the Kubernetes host control plane. So we shall begin by gathering these things in each of the clusters.
### Get Dev Cluster control plane

**Change your context to Dev Kubernetes cluster**

```sh
kubectl config use-context dev-cluster
```

**Then get the Dev control plane URL.**

```sh
kubectl cluster-info
Kubernetes control plane is running at https://192.168.20.22
```
### Get/Create Dev service account

This service account is going to be used to generate the JWT reviewer token as we will see very soon. So let us create a ClusterRole and ClusterRoleBinding for the service account. Note that if there is a service account with the requisite permissions in the cluster already, you can use it here. The service account can be in any namespace.

```sh
vim vault-sa.yaml

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
   name: role-tokenreview-binding ##This Role!
   namespace: default
roleRef:
   apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: vault-auth
  namespace: default
```

**Create the Service Account**

```sh
kubectl create sa vault-auth
kubectl apply -f vault-sa.yaml
```

NOTE: The service account must have _**role-tokenreview-binding**_ ClusterRoleBinding permissions.
### Get Dev Reviewer JWT Token

**Once we have our service account, we can now generate the reviewer JWT Token as follows**

```sh
export DEV_VAULT_SA_NAME=$(kubectl -n <namespace> get sa vault-auth --output jsonpath="{.secrets[*]['name']}")
export DEV_SA_JWT_TOKEN=$(kubectl -n <namespace> get secret $DEV_VAULT_SA_NAME --output 'go-template={{ .data.token }}' | base64 --decode)
echo $DEV_SA_JWT_TOKEN >> ~/dev-cluster-jwt-token
```

The reviewer JWT Token will be stored in “_~/dev-cluster-jwt-token_” file.

### Get Dev CA Certificate

**Lastly, let us generate the ca certificate of the Dev Cluster. Over to the terminal, run the commands below:**

```sh
DEV_KUBE_CA_CERT=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.certificate-authority-data}' | base64 --decode)
echo $DEV_KUBE_CA_CERT >> ~/dev-ca.crt
```

The CA certificate will be stored in “_~/dev-ca.crt_” file.

Once the two files have been generated, I presume in your local machine. You will need to send them to the Vault server. Our server is in GCP in this example, and the files were transferred as follows:

```sh
scp ~/dev-cluster-jwt-token user@vault-server:~
scp ~/dev-ca.crt user@vault-server:~
```

At this point, we have everything we need to configure Dev cluster’s Kubernetes Auth Method
### Configure Dev cluster’s Kubernetes Auth Method

To configure this, login to your Vault instance and run the commands below.

**Create Kubernetes Dev Auth method path**

```sh
vault auth enable --path=dev-cluster kubernetes
```

**Then configure the new path with the details we created as follows**

```sh
vault write auth/dev-cluster/config \
    token_reviewer_jwt="$(cat ~/dev-cluster-jwt-token)" \
    kubernetes_host=https://192.168.20.22 \
    kubernetes_ca_cert="$(cat ~/dev-ca.crt)" \
    issuer="https://kubernetes.default.svc.cluster.local"
```

Once we are at this point, our Dev auth configuration is successfully done and the only thing remaining is the configuration of secrets, policies and roles.
#### Create Secrets for respective applications in each cluster

First, you can create a path that will hold secrets in Dev cluster then you can add secrets therein. An example is given below

```sh
vault secrets enable -path=dev-secrets kv-v2
vault kv put dev-secrets/dev-payments-service/config username=geeks-dev password=12345
```
#### Create a policy with requisite permissions for the secrets accessibility

Once the secret is created, you need the best policy/permissions for it. Here, we will give “_read_” permissions only for this policy. You can have many policies for different use-cases.

```sh
vault policy write dev-app - <<EOF
  path "dev-secrets/data/dev-payments-service/config" {
    capabilities = ["read"]
 }
EOF
```

You can also permit the policy to read everything under “dev-secrets” path as follows:

```sh
vault policy write dev-app - <<EOF
  path “dev-secrets/*” {
    capabilities = ["read"]
 }
EOF
```
#### Create a role and attach the policy we created above

The policy is a set of permissions and a role needs access to secrets with given permissions declared in policies. As an example, we will create a role that will have “_dev-app’s_” policy permissions. This role will authenticate to the cluster and will only have read permissions in the given secrets. You can bind the authentication to specific Kubernetes service accounts or you can grant all namespaces and service accounts permission to authenticate.

**In this case below, applications in default namespace only will be able to successfully authenticate.**

```sh
vault write auth/dev-cluster/role/devweb-app \
    bound_service_account_names=default \
    bound_service_account_namespaces=default \
    policies=dev-app \
    ttl=24h
```

**Or you can grant all namespaces and service accounts permission to authenticate as follows:**

```sh
vault write auth/dev-cluster/role/devweb-app \
    bound_service_account_names=* \
    bound_service_account_namespaces=* \
    policies=dev-app \
    ttl=24h
```

## 2. Staging cluster

### Configuring Staging cluster Kubernetes Auth method

Just like in our Dev cluster, before we can configure Kubernetes Auth, we need to have a few things in our plate. We need JWT reviewer token, Kubernetes ca certificate, Kubernetes service account and the Kubernetes host control plane. So we shall begin by gathering these things in each of the clusters.
### Get Staging control plane

**Change your context to Staging Kubernetes cluster**

```sh
kubectl config use-context staging-cluster
```

**Then get the Staging control plane URL.**

```sh
kubectl cluster-info
Kubernetes control plane is running at https://192.168.20.23
```
### Get/Create Staging service account

This service account is going to be used to generate the JWT reviewer token as we will see very soon. So let us create a ClusterRole and ClusterRoleBinding for the service account. Note that if there is a service account with the requisite permissions in the cluster already, you can use it here. The service account can be in any namespace.

```sh
vim vault-sa.yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
   name: role-tokenreview-binding  ## This!!
   namespace: default
roleRef:
   apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: vault-auth
  namespace: default
```

Create the Service Account

```sh
kubectl create sa vault-auth
```

REMEMBER: The service account must have role-tokenreview-binding ClusterRoleBinding permissions.
### Get Staging Reviewer JWT Token

**Once we have our service account, we can now generate the reviewer JWT Token as follows**

```sh
export STAGING_VAULT_SA_NAME=$(kubectl -n <namespace> get sa vault-auth --output jsonpath="{.secrets[*]['name']}")
export STAGING_SA_JWT_TOKEN=$(kubectl -n <namespace> get secret $STAGING_VAULT_SA_NAME --output 'go-template={{ .data.token }}' | base64 --decode)
echo $STAGING_SA_JWT_TOKEN >> ~/staging-cluster-jwt-token
```

Replace with the namespace where the service account resides.

The reviewer JWT Token will be stored in “_~/staging-cluster-jwt-token_” file.
### Get Staging CA Certificate

Lastly, let us generate the ca certificate of the Staging Cluster. Over to the terminal, run the commands below:

```sh
STAGING_KUBE_CA_CERT=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.certificate-authority-data}' | base64 --decode)
echo $STAGING_KUBE_CA_CERT >> ~/staging-ca.crt
```

The CA certificate will be stored in “_~/staging-ca.crt_” file.

Once the two files have been generated, I presume in your local machine, you will need to send them to the Vault server. Our server is in GCP in this example, and the files were transferred as follows:

```sh
scp ~/staging-cluster-jwt-token  user@vault-server:~
scp ~/staging-ca.crt  user@vault-server:~
```

At this point, we have everything we need to configure Staging cluster’s Kubernetes Auth Method
### Configure Staging cluster’s Kubernetes Auth Method

To configure this, login to your same Vault instance and run the commands below.

**Create Kubernetes Staging Auth method path**

```sh
vault auth enable --path=staging-cluster kubernetes
```

**Then configure the new path with the details we created as follows**

```sh
vault write auth/staging-cluster/config \
    token_reviewer_jwt="$(cat ~/staging-cluster-jwt-token)" \
    kubernetes_host=https://192.168.20.23 \
    kubernetes_ca_cert="$(cat ~/staging-ca.crt)" \
    issuer="https://kubernetes.default.svc.cluster.local"
```

Once we are at this point, our Staging Auth configuration is successfully done and the only thing remaining is the configuration of secrets, policies and roles.
#### Create Secrets for respective applications in each cluster

First, you can create a path that will hold secrets in Staging cluster then you can add secrets therein. An example is given below

```sh
vault secrets enable -path=staging-secrets kv-v2
vault kv put staging-secrets/staging-payments-service/config username=geeks-staging password=12345
```
#### Create a policy with requisite permissions for the secrets accessibility

Once the secret is created, you need the best policy/permissions for it. Here, we will give “_read_” permissions only for this policy. You can have many policies for different use-cases.

```sh
vault policy write staging-app - <<EOF
  path "staging-secrets/data/staging-payments-service/config" {
    capabilities = ["read"]
 }
EOF
```

**You can also permit the policy to read everything under “staging-secrets” path as follows:**

```sh
vault policy write staging-app - <<EOF
  path "staging-secrets/*” {
    capabilities = ["read"]
 }
EOF
```
#### Create a role and attach the policy we created above

The policy is a set of permissions and a role needs access to secrets with given permissions declared in policies. As an example, we will create a role that will have “_staging-app’s_” policy permissions. This role will authenticate to the cluster and will only have read permissions in the given secrets. You can bind the authentication to specific Kubernetes service accounts or you can grant all namespaces and service accounts permission to authenticate.

**In this case below, applications in default namespace only will be able to successfully authenticate.**

```sh
vault write auth/staging-cluster/role/staging-web-app \
    bound_service_account_names=default \
    bound_service_account_namespaces=default \
    policies=staging-app \
    ttl=24h
```

**Or you can grant all namespaces and service accounts permission to authenticate as follows:**

```sh
vault write auth/staging-cluster/role/staging-web-app \
    bound_service_account_names=* \
    bound_service_account_namespaces=* \
    policies=staging-app \
    ttl=24h
```
## 3. Production Cluster

### Configuring Production cluster Kubernetes Auth method

Just like in our Dev and Staging clusters, before we can configure Kubernetes Auth, we need to have a few things in our basket. We need JWT reviewer token, Kubernetes ca certificate, Kubernetes service account and the Kubernetes host control plane. So we shall begin by gathering these things in each of the clusters.
### Get Production Control Plane

**Change your context to Production Kubernetes Cluster or whichever way you get access to your production cluster.**

```sh
kubectl config use-context production-cluster
```

**Then get the Production control plane URL.**

```sh
kubectl cluster-info
Kubernetes control plane is running at https://192.168.20.24
```
### Get/create Production service account

This service account is going to be used to generate the JWT reviewer token as we will see very soon. So let us create a ClusterRole and ClusterRoleBinding for the service account. Note that if there is a service account with the requisite permissions in the cluster already, you can use it here. The service account can be in any namespace.

```sh
vim vault-sa.yaml

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
   name: role-tokenreview-binding ##This!!
   namespace: default
roleRef:
   apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: vault-auth
  namespace: default
```

Create the Service Account

```sh
kubectl create sa vault-auth
```

REMEMBER: The service account must have role-tokenreview-binding ClusterRoleBinding permissions.
### Get Production Reviewer JWT Token

Once we have our service account, we can now generate the reviewer JWT Token as follows

```sh
export PROD_VAULT_SA_NAME=$(kubectl -n <namespace> get sa vault-auth --output jsonpath="{.secrets[*]['name']}")
export PROD_SA_JWT_TOKEN=$(kubectl -n <namespace> get secret $PROD_VAULT_SA_NAME --output 'go-template={{ .data.token }}' | base64 --decode)
echo $PROD_SA_JWT_TOKEN >> ~/prod-cluster-jwt-token
```

You will have to replace with the namespace where your service account can be found in.

The reviewer JWT Token will be stored in “_~/prod-cluster-jwt-token_” file.
### Get Production CA Certificate

Lastly, let us generate the ca certificate of the Production Cluster. Over to the terminal, run the commands below:

```sh
PROD_KUBE_CA_CERT=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.certificate-authority-data}' | base64 --decode)
echo $PROD_KUBE_CA_CERT >> ~/prod-ca.crt
```

The CA certificate will be stored in “_~/prod-ca.crt_” file.

Once the two files have been generated, I presume in your local machine, you will need to send them to the Vault server. Our server is in GCP in this example, and the files were transferred as follows:

```sh
scp ~/prod-cluster-jwt-token user@vault-server:~
scp ~/prod-ca.crt  user@vault-server:~
```

At this point, we have everything we need to configure Production cluster’s Kubernetes Auth Method
### Configure Production cluster’s Kubernetes Auth Method

To configure this, login to your same Vault instance and run the commands below.

**Create Kubernetes Production Auth method path**

```sh
vault auth enable --path=prod-cluster kubernetes
```

**Then configure the new path with the details we created as follows**

```sh
vault write auth/prod-cluster/config \
    token_reviewer_jwt="$(cat ~/prod-cluster-jwt-token)" \
    kubernetes_host=https://192.168.20.24 \
    kubernetes_ca_cert="$(cat ~/prod-ca.crt)" \
    issuer="https://kubernetes.default.svc.cluster.local"
```

Once we are at this point, our Staging Auth configuration is successfully done and the only thing remaining is the configuration of secrets, policies and roles.
#### Create Secrets for respective applications in each cluster

First, you can create a path that will hold secrets in Staging cluster then you can add secrets therein. An example is given below

```sh
vault secrets enable -path=prod-secrets kv-v2
vault kv put prod-secrets/prod-payments-service/config username=geeks-prod password=12345
```
#### Create a policy with requisite permissions for the secrets accessibility

Once the secret is created, you need the best policy/permissions for it. Here, we will give “_read_” permissions only for this policy. You can have many policies for different use-cases.

```sh
vault policy write prod-app - <<EOF
  path “prod-secrets/data/prod-payments-service/config" {
    capabilities = ["read"]
 }
EOF
```

**You can also permit the policy to read everything under “prod-secrets” path as follows:**

```sh
vault policy write prod-app - <<EOF
  path “prod-secrets/*“ {
    capabilities = ["read"]
 }
EOF
```
#### Create a role and attach the policy we created above

The policy is a set of permissions and a role needs access to secrets with given permissions declared in policies. As an example, we will create a role that will have “_staging-app’s_” policy permissions. This role will authenticate to the cluster and will only have read permissions in the given secrets. You can bind the authentication to specific Kubernetes service accounts or you can grant all namespaces and service accounts permission to authenticate.

**In this case below, applications in default namespace only will be able to successfully authenticate.**

```sh
vault write auth/prod-cluster/role/prod-web-app \
    bound_service_account_names=default \
    bound_service_account_namespaces=default \
    policies=prod-app \
    ttl=24h
```

**Or you can grant all namespaces and service accounts permission to authenticate as follows:**

```sh
vault write auth/prod-cluster/role/prod-web-app \
    bound_service_account_names=* \
    bound_service_account_namespaces=* \
    policies=prod-app \
    ttl=24h
```

**_Books For Learning Kubernetes Administration:_**

- [Best Kubernetes Study books](https://computingforgeeks.com/best-kubernetes-study-books/)
## End Note

In the end, we should now be able to segregate all of the secrets in any of your working clusters and manage all of them in one place where their tokens can be easily reviewed, revoked, added and modified. Now any application in any of your Kubernetes Clusters can connect to Vault and get secrets if the details they provide are correct. Next we shall look at how applications can be configured to get secrets from the ready Vault server.

Related guides you will enjoy:

- [Use Vault-Agent sidecar to inject Secrets in Vault to Kubernetes Pod](https://computingforgeeks.com/use-vault-agent-sidecar-to-inject-secrets-in-vault-to-kubernetes-pod/)
- [Install Vault Cluster in GKE via Helm, Terraform and BitBucket Pipelines](https://computingforgeeks.com/install-vault-cluster-gke-via-helm-terraform-bitbucket-pipelines/)
- [Install and Configure Hashicorp Vault Server on Ubuntu / CentOS / Debian](https://computingforgeeks.com/install-and-configure-vault-server-linux/)