In this series, I’ll first show you what Flux is and how it works, and later show you how to put all the basic Kubernetes configuration into a Git repository to add load balancer, ingress with cert-manager, and persistent storage with a single command.

**Edit: Added missing Gateway-API CRDs which prevents installing of cert-manager, Added github repo with example**

**I attach great importance to the fact that I have tested and carried out the processes described here myself.**

## What has happened so far

In [Part 1](https://medium.com/@michael-tissen/unlock-lightning-fast-raspberrypi-kubernetes-cluster-setup-turbocharge-initialization-with-fluxcd-ef3692cb8e4d)

- We prepared a git-repository for flux
- We install the flux-cli
- We installed flux to our cluster
- We created manifests for **metallb** in our git-repository
- We watched Flux automatically apply our manifests to the cluster.

## What is missing

To have a fully usable kubernetes cluster we need also an ingress, a manager for our certificates and persistence to have stateful workloads.

According to the Gitops pattern, we want to have our entire definition of the cluster in our Git repository in order to restore the cluster from the Git repository.

However, the above components require secrets that you should not store in your Git repository.

We solve this problem by using the excellent sealed-secrets package. This allows us to have the secrets encrypted in the Git repository.

## Add Sealed-Secrets Helm-Chart

In our fluxcd repo we need to define the manifests for the sealed-secrets controller. Similar to metallb we use a helm-chart.

1. Add a new folder “sealed-secrets” to the infrastructure folder.

![](nkRXfJilv1tigd8QwH_M8A.webp)

1. In the new folder add the helmchart repo to a **source.yaml:**

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta1  
kind: HelmRepository  
metadata:  
  name: sealed-secrets  
  namespace: flux-system  
spec:  
  interval: 30m  
  url: "https://bitnami-labs.github.io/sealed-secrets"
```

1. Add a Flux HelmRelease in a **release.yaml**:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1  
kind: HelmRelease  
metadata:  
  name: sealed-secrets  
  namespace: flux-system  
spec:  
  releaseName: sealed-secrets-controller  
  targetNamespace: kube-system  
  chart:  
    spec:  
      chart: sealed-secrets  
      sourceRef:  
        kind: HelmRepository  
        name: sealed-secrets  
      version: ">=1.15.0-0"  
  interval: 1h0m0s  
  install:  
    remediation:  
      retries: 3  
    crds: Create  
  upgrade:  
    crds: CreateReplace
```

    crds: CreateReplace

> It is important that **metadata.name** is **sealed-secrets-controller** because there may be problems with the sealed-secret cli later on! It is also important to deploy sealed-secrets to the kube-system namespace. This simplifies fetching the public certificate.

1. Add a Kustomization for the two yaml-files in **kustomization.yaml:**

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
resources:  
  - source.yaml  
  - release.yaml

```
1. Extend our **clusters/staging/infrastructure.yaml** with:

```yaml
---  
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2  
kind: Kustomization  
metadata:  
  name: sealed-secrets  
  namespace: kube-system  
spec:  
  interval: 1m  
  sourceRef:  
    kind: GitRepository  
    name: flux-system  
    namespace: flux-system  
  path: ./infrastructure/sealed-secrets  
  prune: true  
---  
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2  
kind: Kustomization  
metadata:  
  name: metallb  
  namespace: metallb-system  
spec:  
  dependsOn:  
    - name: sealed-secrets  
      namespace: flux-system  
  interval: 1m  
  sourceRef:  
    kind: GitRepository  
    name: flux-system  
    namespace: flux-system  
  path: ./infrastructure/metallb  
  prune: true  
  wait: true  
  timeout: 10m  
---
```

We put our reference to the **sealed-secrets** definitions **after** all namespace declarations and **before** the **metallb-system**. This order is only for better readability.

> Define the kube-system namespace for sealed-secrets in the infrastructure.yaml also

The important change lies in the metallb-Kustomization and is the **dependsOn**-definition.

Sealed secrets should be initialized as early as possible so that all further deployments can benefit from them. In the definition above, metallb is dependent on sealed-secrets.

1. Commit and push the new changes to your git-repository and flux-cd should apply the new helm-chart.

If everything works fine you should see a sealed-secrets pod when you execute “kubectl get pods — all-namespaces”

```
NAMESPACE        NAME                                      READY   STATUS    RESTARTS        AGE  
flux-system      notification-controller-ddf44665d-rxp6t   1/1     Running   2 (3h23m ago)   21d  
kube-system      coredns-77ccd57875-c92g8                  1/1     Running   1 (3h24m ago)   21d  
flux-system      helm-controller-b957fcf89-wfmkf           1/1     Running   2 (3h23m ago)   21d  
metallb-system   metallb-speaker-czd8f                     1/1     Running   2 (3h23m ago)   21d  
kube-system      metrics-server-648b5df564-kxfz8           1/1     Running   2 (3h23m ago)   21d  
metallb-system   metallb-controller-6f6787d9cf-wqfcq       1/1     Running   3 (3h23m ago)   21d  
flux-system      source-controller-9dfbc5cd-bh2zn          1/1     Running   2 (3h23m ago)   21d  
metallb-system   metallb-speaker-ln6zp                     1/1     Running   2 (3h23m ago)   21d  
flux-system      kustomize-controller-644f79985c-g4qx6     1/1     Running   0               3h18m  
metallb-system   metallb-speaker-8pnvx                     1/1     Running   1 (170m ago)    21d  
kube-system      sealed-secrets-548785469d-rjxw5           1/1     Running   0               116s
```

## How Sealed-Secrets works

Now we get to the interesting part. The creation of a secret.

![](iUJJhEH5yCDuzeegHrQmg.webp)

On the local machine we use the **sealed-secret-cli** to encrypt the actual secret. A public key is used. The encrypted sealed secret can be safely uploaded to the git repository.

> The private key was created during the installation of sealed-secrets and is located in the kubernetes cluster.

![](AUFz15d9-0HbrOXoiCFSg.webp)

The sealed secret which is uploaded to the git-repository, is pulled from flux. The Sealed-Secrets Controller decrypts and creates a normal secret that Kubernetes can use and process.

The sealed secret which is uploaded to the git-repository, is pulled from flux. The Sealed-Secrets Controller decrypts and creates a normal secret that Kubernetes can use and process.

## Install the Sealed-Secrets CLI

Follow the official instructions to install **kubeseal:** [https://github.com/bitnami-labs/sealed-secrets#kubeseal](https://github.com/bitnami-labs/sealed-secrets#kubeseal)

On my mac i’am using brew:

```sh
apt install kubeseal
```

Get an encryption key with:

```
kubeseal --fetch-cert > pub-sealed-secrets.pem
```

> Because we deployed sealed-secrets to the kube-system namespace we can use the fetch-cert subcommand without problems

## Example

Generate a Kubernetes secret manifest with kubectl:

```sh
kubectl -n default create secret generic basic-auth \  
--from-literal=user=admin \  
--from-literal=password=change-me \  
--dry-run=client \  
-o yaml > basic-auth.yaml
```

Encrypt the secret with kubeseal:

```sh
kubeseal --format=yaml --cert=pub-sealed-secrets.pem \  
< basic-auth.yaml > basic-auth-sealed.yaml
```

## Add to repository

Create a **sealed-secrets-secrets** folder in the **infrastructure** directory

![](Gj6_qH0yZcDsjwNBErd7mA.webp)

With the following files:

```yaml
---  
apiVersion: bitnami.com/v1alpha1  
kind: SealedSecret  
metadata:  
  creationTimestamp: null  
  name: basic-auth  
  namespace: default  
spec:  
  encryptedData:  
    password: AgAFI6Kk8gnVLEJ9tsqQxWsGYTZ5P9JvQNtoQixHGsxYLTbc3LHrlf8sl0pWiKKq14si+WekqhFA5cvjkqzgTxL0V5ye3wbon45NwD1Ly5z9EKeF/83Wkij9VlKVo5DoFSOoVgGpSiPJWd1+ymyVk6suftEbsvgkpDx63ycXBfcfCOoTma9V5tKqL30sU8ppRHGVXqDOYE/AhqhTwcmAcYu2zRe1wNPdLUAF1dUwvXrV5U37o3k5XDXZCCuuAIfFqEiau2x6xujBbOF+NunN7XWuDVVeeb/N23VvncVgMznceryt98796+s3KeLuKFcvxugQvwbGpYYYAct2VA1295Fauqnm5GtWg+pUIT/Z9F/tLX7+MxrkhQtmATeJ5g1C+jVp6annzQZ/GK0dJvumwFYnX0MK1T/BwX3d8WCGCFVpkiUlhBwZMh5ieER5wCkq9kDxoq29otoRrp2INEwCSVcWKcEePHEuPBPJMQTp5qwDu0LDnK7D4U1ZlC2gKOFyv/IlDnDl9daLAUUCFhJDgmfPdMJpqgZ60SJcZG93TOb8GQFgSLSpW9m2nYCbJ0p3U/mY2z7N3dIVupbQhzYtpZTiyMqr9foKkFf6+/NWsngcmbkuFMdy1F1Mc+nG2pWHWHdL6ToT2gdejY7iKCGPBkucu6mFoyaeTlf8sa+8Sdvs6Drqt2USy9B8jIjLJpZktHZ6+nhRb7+2264=  
    user: AgBfOUaVn9Wq5G+JxxAXRf39YE/VVYF5/wbdBuLsTDRDRBBJSxPfB2uEWkMNHLQQL8obdyGgQ0wOjUmOgJiGe45fih6Rx208QYxRL3z6oxHmRgBg13NJKvM8f1Zu1ZOYriT1VtaTuKVxQWcKWwXsq/NL02Clm0SalMpFqfKkdZyGoRAqsHNuANZ+118QCJDoqdneYn/MmuAxIVP6VXx7PECU8OSOP5lYDN160wH37m/pyY/pgltWzS2tm4/NZGiAu7jYtUs262i1GBcrQpUvBzs+EWZvJvtLdhfBIbQJvmGjw+wmmSaMbi8fK0/Qh5OVg5gVX5mzx7yHeDpamElwwV6yk3D/uxvlWH0eH80cxE7jKnrGZ7jK2z3O8dnkrburJXCKGQbRh3+YwCA//Ep3w/RjZnR/NJJLVyUbbJaPSw+NCTD7VMi46NEqp3z2FNo/oP8/YCdssvNyavYQ74JwhFq4bUWJmGXnL0viqt3x3OjceqCkeNx9bCS0i+EDbVyZBGFi1/o+EsgrxxirAjrcAj7uDIe2kMDcN3pbm5zcljBBVavRUp9qZdOAPfkIsJ3aK65y0O+IOgusQp2cSQZu029a207USRfMlsWmwDJTzzGjzWSFpEA0Ggexg2UUvGL7E67sIWDUSoQq7an8J7qcH/D+ydbcBtJxx7EUQ/bRcIBS9p3WKvyn1rMS79fCWDn0J3mr202htA==  
  template:  
    metadata:  
      creationTimestamp: null  
      name: basic-auth  
      namespace: default
```

**kustomization.yaml**

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
resources:  
  - basic-auth-sealed.yaml
```

Extend our **clusters/staging/infrastructure.yaml** with:

```yaml
---  
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2  
kind: Kustomization  
metadata:  
  name: sealed-secrets-secrets  
  namespace: kube-system  
spec:  
  dependsOn:  
    - name: metallb-policies  
      namespace: metallb-system  
  interval: 1m  
  sourceRef:  
    kind: GitRepository  
    name: flux-system  
    namespace: flux-system  
  path: ./infrastructure/sealed-secrets-secrets  
  prune: true  
  wait: true  
  timeout: 10m
```

Push changes to your git repo. Fluxcd should decrypt the **SealedSecret** ressource to a plain old **Secret**.

## Verify the decrypted key

```yaml
kubectl -n default get secret  
NAME                  TYPE                DATA   AGE  
basic-auth            Opaque              2      4m31s
```

# Add cert-manager to our cluster

1. Add a traefik folder to infrastructure

![](SySZ-NFfrkXU3zbsngT2tw.webp)

1. Create a **source.yaml**

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta1  
kind: HelmRepository  
metadata:  
  name: jetstack  
  namespace: cert-manager  
spec:  
  interval: 5m  
  url: "https://charts.jetstack.io"
```

1. Create a **kustomizeconfig.yaml** to reference configurations from a HelmRelease:

```yaml
nameReference:  
- kind: ConfigMap  
  version: v1  
  fieldSpecs:  
  - path: spec/valuesFrom/name  
    kind: HelmRelease  
- kind: Secret  
  version: v1  
  fieldSpecs:  
  - path: spec/valuesFrom/name  
    kind: HelmRelease
```

1. Create a **release.yaml**

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1  
kind: HelmRelease  
metadata:  
  name: cert-manager-controller  
  namespace: cert-manager  
spec:  
  releaseName: cert-manager  
  chart:  
    spec:  
      chart: cert-manager  
      sourceRef:  
        kind: HelmRepository  
        name: jetstack  
        namespace: cert-manager  
      version: "v1.14.1"  
  interval: 1h0m0s  
  install:  
    remediation:  
      retries: 3  
    crds: CreateReplace  
  upgrade:  
    crds: CreateReplace  
  valuesFrom:  
    - kind: ConfigMap  
      name: cert-manager-values
```

A special feature of the upper release.yaml is a reference to a ConfigMap cert-manager-values.

1. Create a **kustomization.yaml**

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
namespace: cert-manager  
resources:  
  - source.yaml  
  - release.yaml  
  
configMapGenerator:  
  - name: cert-manager-values  
    files:  
      - values.yaml=values.yaml  
configurations:  
  - kustomizeconfig.yaml
```

Here we define our resources like for sealed-secrets. In addition, it is defined here that a ConfigMap called cert-manager-values is created from a values.yaml and is available to the HelmRelease.

1. Create a **values.yaml**

```yaml
# Default values for cert-manager.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.
global:
  ## Reference to one or more secrets to be used when pulling images
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  imagePullSecrets: []
  # - name: "image-pull-secret"

  # Optional priority class to be used for the cert-manager pods
  priorityClassName: ""
  rbac:
    create: true

  podSecurityPolicy:
    enabled: false
    useAppArmor: true

  # Set the verbosity of cert-manager. Range of 0 - 6 with 6 being the most verbose.
  logLevel: 2

  leaderElection:
    # Override the namespace used to store the ConfigMap for leader election
    namespace: "kube-system"

    # The duration that non-leader candidates will wait after observing a
    # leadership renewal until attempting to acquire leadership of a led but
    # unrenewed leader slot. This is effectively the maximum duration that a
    # leader can be stopped before it is replaced by another candidate.
    # leaseDuration: 60s

    # The interval between attempts by the acting master to renew a leadership
    # slot before it stops leading. This must be less than or equal to the
    # lease duration.
    # renewDeadline: 40s

    # The duration the clients should wait between attempting acquisition and
    # renewal of a leadership.
    # retryPeriod: 15s

installCRDs: true

replicaCount: 1

strategy: {}
  # type: RollingUpdate
  # rollingUpdate:
  #   maxSurge: 0
  #   maxUnavailable: 1

# Comma separated list of feature gates that should be enabled on the
# controller pod.
featureGates: "ExperimentalGatewayAPISupport=true"

image:
  repository: quay.io/jetstack/cert-manager-controller
  # You can manage a registry with
  # registry: quay.io
  # repository: jetstack/cert-manager-controller

  # Override the image tag to deploy by setting this variable.
  # If no value is set, the chart's appVersion will be used.
  # tag: canary

  # Setting a digest will override any tag
  # digest: sha256:0e072dddd1f7f8fc8909a2ca6f65e76c5f0d2fcfb8be47935ae3457e8bbceb20
  pullPolicy: IfNotPresent

# Override the namespace used to store DNS provider credentials etc. for ClusterIssuer
# resources. By default, the same namespace as cert-manager is deployed within is
# used. This namespace will not be automatically created by the Helm chart.
clusterResourceNamespace: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  # name: ""
  # Optional additional annotations to add to the controller's ServiceAccount
  # annotations: {}
  # Automount API credentials for a Service Account.
  automountServiceAccountToken: true

# Optional additional arguments
extraArgs: []
  # Use this flag to set a namespace that cert-manager will use to store
  # supporting resources required for each ClusterIssuer (default is kube-system)
  # - --cluster-resource-namespace=kube-system
  # When this flag is enabled, secrets will be automatically removed when the certificate resource is deleted
  # - --enable-certificate-owner-ref=true
  # Use this flag to enabled or disable arbitrary controllers, for example, disable the CertificiateRequests approver
  # - --controllers=*,-certificaterequests-approver

extraEnv: []
# - name: SOME_VAR
#   value: 'some value'

resources: {}
  # requests:
  #   cpu: 10m
  #   memory: 32Mi

# Pod Security Context
# ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
securityContext:
  runAsNonRoot: true
# legacy securityContext parameter format: if enabled is set to true, only fsGroup and runAsUser are supported
# securityContext:
#   enabled: false
#   fsGroup: 1001
#   runAsUser: 1001
# to support additional securityContext parameters, omit the `enabled` parameter and simply specify the parameters
# you want to set, e.g.
# securityContext:
#   fsGroup: 1000
#   runAsUser: 1000
#   runAsNonRoot: true

# Container Security Context to be set on the controller component container
# ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
containerSecurityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true

volumes: []

volumeMounts: []

# Optional additional annotations to add to the controller Deployment
# deploymentAnnotations: {}

# Optional additional annotations to add to the controller Pods
# podAnnotations: {}

podLabels: {}

# Optional additional labels to add to the controller Service
# serviceLabels: {}

# Optional additional annotations to add to the controller service
# serviceAnnotations: {}

# Optional DNS settings, useful if you have a public and private DNS zone for
# the same domain on Route 53. What follows is an example of ensuring
# cert-manager can access an ingress or DNS TXT records at all times.
# NOTE: This requires Kubernetes 1.10 or `CustomPodDNS` feature gate enabled for
# the cluster to work.
# podDnsPolicy: "None"
# podDnsConfig:
#   nameservers:
#     - "1.1.1.1"
#     - "8.8.8.8"

nodeSelector: {}

ingressShim: {}
  # defaultIssuerName: ""
  # defaultIssuerKind: ""
  # defaultIssuerGroup: ""

prometheus:
  enabled: true
  servicemonitor:
    enabled: false
    prometheusInstance: default
    targetPort: 9402
    path: /metrics
    interval: 60s
    scrapeTimeout: 30s
    labels: {}
    honorLabels: false

# Use these variables to configure the HTTP_PROXY environment variables
# http_proxy: "http://proxy:8080"
# https_proxy: "https://proxy:8080"
# no_proxy: 127.0.0.1,localhost

# expects input structure as per specification https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.11/#affinity-v1-core
# for example:
#   affinity:
#     nodeAffinity:
#      requiredDuringSchedulingIgnoredDuringExecution:
#        nodeSelectorTerms:
#        - matchExpressions:
#          - key: foo.bar.com/role
#            operator: In
#            values:
#            - master
affinity: {}

# expects input structure as per specification https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.11/#toleration-v1-core
# for example:
#   tolerations:
#   - key: foo.bar.com/role
#     operator: Equal
#     value: master
#     effect: NoSchedule
tolerations: []

webhook:
  replicaCount: 1
  timeoutSeconds: 10

  # Used to configure options for the webhook pod.
  # This allows setting options that'd usually be provided via flags.
  # An APIVersion and Kind must be specified in your values.yaml file.
  # Flags will override options that are set here.
  config:
    # apiVersion: webhook.config.cert-manager.io/v1alpha1
    # kind: WebhookConfiguration

    # The port that the webhook should listen on for requests.
    # In GKE private clusters, by default kubernetes apiservers are allowed to
    # talk to the cluster nodes only on 443 and 10250. so configuring
    # securePort: 10250, will work out of the box without needing to add firewall
    # rules or requiring NET_BIND_SERVICE capabilities to bind port numbers <1000.
    # This should be uncommented and set as a default by the chart once we graduate
    # the apiVersion of WebhookConfiguration past v1alpha1.
    # securePort: 10250

  strategy: {}
    # type: RollingUpdate
    # rollingUpdate:
    #   maxSurge: 0
    #   maxUnavailable: 1

  # Pod Security Context to be set on the webhook component Pod
  # ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  securityContext:
    runAsNonRoot: true

  # Container Security Context to be set on the webhook component container
  # ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  containerSecurityContext: {}
    # capabilities:
    #   drop:
    #   - ALL
    # readOnlyRootFilesystem: true
    # runAsNonRoot: true

  # Optional additional annotations to add to the webhook Deployment
  # deploymentAnnotations: {}

  # Optional additional annotations to add to the webhook Pods
  # podAnnotations: {}

  # Optional additional annotations to add to the webhook MutatingWebhookConfiguration
  # mutatingWebhookConfigurationAnnotations: {}

  # Optional additional annotations to add to the webhook ValidatingWebhookConfiguration
  # validatingWebhookConfigurationAnnotations: {}

  # Optional additional annotations to add to the webhook service
  # serviceAnnotations: {}

  # Optional additional arguments for webhook
  extraArgs: []

  resources: {}
    # requests:
    #   cpu: 10m
    #   memory: 32Mi

  ## Liveness and readiness probe values
  ## Ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
  ##
  livenessProbe:
    failureThreshold: 3
    initialDelaySeconds: 60
    periodSeconds: 10
    successThreshold: 1
    timeoutSeconds: 1
  readinessProbe:
    failureThreshold: 3
    initialDelaySeconds: 5
    periodSeconds: 5
    successThreshold: 1
    timeoutSeconds: 1

  nodeSelector: {}

  affinity: {}

  tolerations: []

  # Optional additional labels to add to the Webhook Pods
  podLabels: {}

  # Optional additional labels to add to the Webhook Service
  serviceLabels: {}

  image:
    repository: quay.io/jetstack/cert-manager-webhook
    # You can manage a registry with
    # registry: quay.io
    # repository: jetstack/cert-manager-webhook

    # Override the image tag to deploy by setting this variable.
    # If no value is set, the chart's appVersion will be used.
    # tag: canary

    # Setting a digest will override any tag
    # digest: sha256:0e072dddd1f7f8fc8909a2ca6f65e76c5f0d2fcfb8be47935ae3457e8bbceb20

    pullPolicy: IfNotPresent

  serviceAccount:
    # Specifies whether a service account should be created
    create: true
    # The name of the service account to use.
    # If not set and create is true, a name is generated using the fullname template
    # name: ""
    # Optional additional annotations to add to the controller's ServiceAccount
    # annotations: {}
    # Automount API credentials for a Service Account.
    automountServiceAccountToken: true

  # The port that the webhook should listen on for requests.
  # In GKE private clusters, by default kubernetes apiservers are allowed to
  # talk to the cluster nodes only on 443 and 10250. so configuring
  # securePort: 10250, will work out of the box without needing to add firewall
  # rules or requiring NET_BIND_SERVICE capabilities to bind port numbers <1000
  securePort: 10250

  # Specifies if the webhook should be started in hostNetwork mode.
  #
  # Required for use in some managed kubernetes clusters (such as AWS EKS) with custom
  # CNI (such as calico), because control-plane managed by AWS cannot communicate
  # with pods' IP CIDR and admission webhooks are not working
  #
  # Since the default port for the webhook conflicts with kubelet on the host
  # network, `webhook.securePort` should be changed to an available port if
  # running in hostNetwork mode.
  hostNetwork: false

  # Specifies how the service should be handled. Useful if you want to expose the
  # webhook to outside of the cluster. In some cases, the control plane cannot
  # reach internal services.
  serviceType: ClusterIP
  # loadBalancerIP:

  # Overrides the mutating webhook and validating webhook so they reach the webhook
  # service using the `url` field instead of a service.
  url: {}
    # host:

cainjector:
  enabled: true
  replicaCount: 1

  strategy: {}
    # type: RollingUpdate
    # rollingUpdate:
    #   maxSurge: 0
    #   maxUnavailable: 1

  # Pod Security Context to be set on the cainjector component Pod
  # ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  securityContext:
    runAsNonRoot: true

  # Container Security Context to be set on the cainjector component container
  # ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  containerSecurityContext: {}
    # capabilities:
    #   drop:
    #   - ALL
    # readOnlyRootFilesystem: true
    # runAsNonRoot: true

  # Optional additional annotations to add to the cainjector Deployment
  # deploymentAnnotations: {}

  # Optional additional annotations to add to the cainjector Pods
  # podAnnotations: {}

  # Optional additional arguments for cainjector
  extraArgs: []

  resources: {}
    # requests:
    #   cpu: 10m
    #   memory: 32Mi

  nodeSelector: {}

  affinity: {}

  tolerations: []

  # Optional additional labels to add to the CA Injector Pods
  podLabels: {}

  image:
    repository: quay.io/jetstack/cert-manager-cainjector
    # You can manage a registry with
    # registry: quay.io
    # repository: jetstack/cert-manager-cainjector

    # Override the image tag to deploy by setting this variable.
    # If no value is set, the chart's appVersion will be used.
    # tag: canary

    # Setting a digest will override any tag
    # digest: sha256:0e072dddd1f7f8fc8909a2ca6f65e76c5f0d2fcfb8be47935ae3457e8bbceb20

    pullPolicy: IfNotPresent

  serviceAccount:
    # Specifies whether a service account should be created
    create: true
    # The name of the service account to use.
    # If not set and create is true, a name is generated using the fullname template
    # name: ""
    # Optional additional annotations to add to the controller's ServiceAccount
    # annotations: {}
    # Automount API credentials for a Service Account.
    automountServiceAccountToken: true

# This startupapicheck is a Helm post-install hook that waits for the webhook
# endpoints to become available.
# The check is implemented using a Kubernetes Job- if you are injecting mesh
# sidecar proxies into cert-manager pods, you probably want to ensure that they
# are not injected into this Job's pod. Otherwise the installation may time out
# due to the Job never being completed because the sidecar proxy does not exit.
# See https://github.com/jetstack/cert-manager/pull/4414 for context.
startupapicheck:
  enabled: true

  # Pod Security Context to be set on the startupapicheck component Pod
  # ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
  securityContext:
    runAsNonRoot: true

  # Timeout for 'kubectl check api' command
  timeout: 1m

  # Job backoffLimit
  backoffLimit: 4

  # Optional additional annotations to add to the startupapicheck Job
  jobAnnotations:
    helm.sh/hook: post-install
    helm.sh/hook-weight: "1"
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded

  # Optional additional annotations to add to the startupapicheck Pods
  # podAnnotations: {}

  # Optional additional arguments for startupapicheck
  extraArgs: []

  resources: {}
    # requests:
    #   cpu: 10m
    #   memory: 32Mi

  nodeSelector: {}

  affinity: {}

  tolerations: []

  # Optional additional labels to add to the startupapicheck Pods
  podLabels: {}

  image:
    repository: quay.io/jetstack/cert-manager-ctl
    # You can manage a registry with
    # registry: quay.io
    # repository: jetstack/cert-manager-ctl

    # Override the image tag to deploy by setting this variable.
    # If no value is set, the chart's appVersion will be used.
    # tag: canary

    # Setting a digest will override any tag
    # digest: sha256:0e072dddd1f7f8fc8909a2ca6f65e76c5f0d2fcfb8be47935ae3457e8bbceb20

    pullPolicy: IfNotPresent

  rbac:
    # annotations for the startup API Check job RBAC and PSP resources
    annotations:
      helm.sh/hook: post-install
      helm.sh/hook-weight: "-5"
      helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded

  serviceAccount:
    # Specifies whether a service account should be created
    create: true

    # The name of the service account to use.
    # If not set and create is true, a name is generated using the fullname template
    # name: ""

    # Optional additional annotations to add to the Job's ServiceAccount
    annotations:
      helm.sh/hook: post-install
      helm.sh/hook-weight: "-5"
      helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded

    # Automount API credentials for a Service Account.
    automountServiceAccountToken: true
```

This is a default values.yaml which is shipped with the helm chart. One addition is `featureGates: "ExperimentalGatewayAPISupport=true"` because in the next part we will use the gateway-api which is now available in the GA version.

For the installation of cert-manager with gateway-api to work we need flux to install the gateway-crd first:

1. Create a gateway-crd folder in infrastructure:

![](aUrLeigxxbo7wi5l7MOHEw.webp)

1. Download the gateway-api-crd definitions from `[https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml](https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml)`

2. Create a new **install.yaml** and add the content of **standard-install.yaml** to this file.

3. Create a kustomization.yaml with the following content:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
resources:  
  - install.yaml
```

Extend our **clusters/staging/infrastructure.yaml** with (as the last two blocks):

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: gateway-crds
  namespace: default
spec:
  dependsOn:
    - name: sealed-secrets-secrets
      namespace: kube-system
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: flux-system
    namespace: flux-system
  path: ./infrastructure/gateway-crd
  prune: true
  wait: true
  timeout: 10m
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: cert-manager
  namespace: cert-manager
spec:
  dependsOn:
    - name: gateway-crds
      namespace: default
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: flux-system
    namespace: flux-system
  path: ./infrastructure/certmanager
  prune: true
  wait: true
  timeout: 10m
```

So previously to installing certmanager flux will apply the gateway-api-crd to our kubernetes cluster.

Push your changes to git and let flux apply this changes.

If everything works fine you should see something like this:

![](H9W-wdavucrZ0dBBj1rfig.webp)

Here you can find the complete example on [https://github.com/rubiktubik/flux-metallb-secrets-certmanager](https://github.com/rubiktubik/flux-metallb-secrets-certmanager)

the only thing i removed is the **sealed-secret-secrets** folder because on your cluster you will have your own private key and cannot decrypt my sealed-secrets.

# Summary

- Installed sealed-secrets
- Installed sealed-secrets cli
- Encrypted secret
- Uploaded secret safely to github
- Decrypted secret in kubernetes
- Installed cert-manager to the cluster

# What’s next

In the next part, we will install an ingress.

We will use the recently stabilized Gateway API instead of Ingress. We will use kong as an implementation. We also will generate let’s encrypt certificates for https routes on demand with cert-manager.

