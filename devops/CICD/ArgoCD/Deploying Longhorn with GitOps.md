## Why GitOps?

Managing Kubernetes clusters can become overwhelming. GitOps transforms this complexity into simplicity by using Git as the **single source of truth**. This approach ensures consistency, automation, and version control for all your configurations.

There are several tools to implement GitOps, such as Flux and ArgoCD. For my homelab, I chose **ArgoCD** because of its intuitive interface and seamless integration with Kubernetes.

## Key features of ArgoCD:

- **Automated Sync**: Keeps your cluster in sync with Git automatically or on-demand.
- **Declarative Management**: Ensures your cluster’s state always matches what’s defined in Git.
- **User-Friendly UI**: A clean interface for managing applications and troubleshooting issues.
- **Multi-Cluster Support**: Easily manages multiple clusters from one place.

Let’s start by setting up ArgoCD to manage our cluster.

## Step 1: Install ArgoCD

First, add the ArgoCD Helm repository and create a namespace:

helm repo add argo https://argoproj.github.io/argo-helm  
helm repo update  
  
`kubectl create namespace argocd`

Next, install ArgoCD using Helm:

`helm install argocd argo/argo-cd - n argocd`

To access the ArgoCD UI:

`kubectl port-forward svc/argocd-server -n argocd 8080:443`

Navigate to [https://localhost:8080](https://localhost:8080) and retrieve the initial admin password:

`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

You’re now ready to use ArgoCD!

## Step 2: Deploy Longhorn

To install Longhorn, start by adding its Helm repository:

```sh
helm repo add longhorn https://charts.longhorn.io  
helm repo update
```

## Custom Configuration

Create a `custom-values.yaml` file to store Longhorn configurations in your Git repository. Here’s an example:

```sh
persistence:  
  defaultClass: true  
defaultSettings:  
  backupTarget: "s3://k8s-backups@us-east-1/"  
  backupTargetCredentialSecret: "minio-credentials"  
service:  
  ui:  
    type: LoadBalancer  
    port: 80
```

Adding a dummy AWS region (e.g., us-east-1) in the `backupTarget` configuration is necessary because many S3-compatible systems, including MinIO, emulate the Amazon S3 API. The AWS region plays a role in how clients interpret and validate the S3 endpoint.

## MinIO Credentials Secret

Define the MinIO credentials in a Kubernetes secret:

```yaml
apiVersion: v1  
kind: Secret  
metadata:  
  name: minio-credentials  
  namespace: longhorn-system  
type: Opaque  
data:  
  AWS_ACCESS_KEY_ID: <base64-encoded-access-key>     # MinIO username in base64  
  AWS_SECRET_ACCESS_KEY: <base64-encoded-secret-key> # MinIO password in base64  
  AWS_ENDPOINTS: <base64-encoded-endpoint>           # http://<ip-loadbalancer-minio>:90
```

Apply the secret:

`kubectl apply -f minio-credentials.yaml`

Push the changes to your Git repository:

```sh
git add apps/longhorn/  
git commit -m "Add custom values for Longhorn"  
git push
```

## Deploy Longhorn with ArgoCD

Create an ArgoCD application manifest for Longhorn:

```yaml
apiVersion: argoproj.io/v1alpha1  
kind: Application  
metadata:  
  name: longhorn  
  namespace: argocd  
spec:  
  project: default  
  sources:  
    - repoURL: 'https://charts.longhorn.io/'  
      chart: longhorn  
      targetRevision: 1.7.2  
      helm:  
        valueFiles:  
          - $values/apps/longhorn/custom-values.yaml  
    - repoURL: 'https://github.com/pablodelarco/kubernetes-homelab'  
      targetRevision: main  
      ref: values  
  destination:  
    server: 'https://kubernetes.default.svc'  
    namespace: longhorn-system  
  syncPolicy:  
    automated:  
      prune: true  
      selfHeal: true  
    syncOptions:  
      - CreateNamespace=true
```

Save this file as `longhorn.yaml` and apply it:

`kubectl apply -f longhorn.yaml`

> Leveraging **multiple sources** in ArgoCD adds flexibility by integrating the latest Helm charts with custom configurations, enabling seamless updates while maintaining a scalable and adaptable setup.

