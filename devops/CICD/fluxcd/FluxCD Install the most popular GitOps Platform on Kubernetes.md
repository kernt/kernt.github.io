# Introduction

Modern software delivery is no longer just about writing code; it’s about continuous integration, deployment, and feedback loops. This dynamic landscape is rapidly shifting towards a model where Git isn’t just a versioning tool but a holistic platform for operations — enter [GitOps](https://www.gitops.tech/?ref=8grams.tech). Before examining FluxCD, it’s vital to understand the foundational principles of this movement.

## About GitOps

_GitOps_ is an operational framework that leverages Git as the singular source of truth for both application code and infrastructure.

![](x7QZYCiNotq791n.webp)

**Infrastructure as Code (IaC)**

Here, every element, from server configurations, networking protocols, to application deployment manifests, is defined as code and stored in a Git repository. This allows for versioning, rollback, and collaboration. For example, imagine a scenario where a cloud infrastructure consists of load balancers, VMs, and databases. In GitOps, configurations for these components would reside in a Git repository, enabling developers to propose changes via pull requests.

**Automated Sync**  
Continuous synchronization tools monitor the Git repository. Any change to the repository — like a merged pull request — triggers an automated rollout of those changes in the infrastructure. For example, A developer wishes to scale a Kubernetes deployment from 3 replicas to 5. They would update the deployment manifest in Git. Once merged, automated tools like FluxCD detect this change and adjust the running state in the cluster.

## GitOps with FluxCD

![](YXot0mz_G_VHC-TB.webp)

Source: [https://medium.com/cloudnloud](https://medium.com/cloudnloud)

[**FluxCD**](https://fluxcd.io/?ref=8grams.tech) is a leading tool in this GitOps movement. As an open-source project, its primary function is to monitor Git repositories for changes and synchronize those changes in a Kubernetes environment, ensuring a consistent state.

_FluxCD_ is intrinsically designed with GitOps in mind:

- **Continuous Deployment**: It continually monitors the associated Git repository. When a commit gets merged, _FluxCD_ initiates the process to deploy those changes to the designated environment, be it staging or production.
- **Feedback Loops**: Via notifications and logs, _FluxCD_ offers insights if discrepancies arise between the desired state (in Git) and the actual state (in the cluster). This ensures that developers and operations teams are always aligned.

# FluxCD Architecture

_FluxCD_, in essence, provides a continuous delivery solution, tailored specifically to fit within the GitOps paradigm. Its architecture is designed to seamlessly and continually ensure that the state of your Kubernetes cluster matches the configurations defined in a Git repository. Let’s break down the primary architectural components and understand their interplay.

## Flux Agent

The _Flux Agent_ is essentially the workhorse of the _FluxCD_ system. It’s a _daemon_ that runs inside your Kubernetes cluster, actively watching for changes in the associated Git repository.

**Responsibilities**:

1. **Polling and Syncing**: The agent frequently polls the designated Git repository to detect any new commits. Upon identifying a change, it orchestrates the deployment of the updated configurations into the Kubernetes cluster.
2. **Auto-release Mechanism**: If configured, the agent can also monitor container registries for new image versions and automatically update the Git repository and the cluster to use the latest images.

## Flux API and CLI

They serve as the primary interfaces for users to interact with FluxCD.

**Responsibilities**:

1. **Config Management**: Administrators and users can set or modify various configurations, such as specifying which Git repository to monitor or adjusting the polling frequency.
2. **Manual Syncing**: If immediate synchronization is required, users can trigger manual syncs via the CLI, bypassing the regular polling intervals.
3. **Status Queries**: The API and CLI allow users to fetch the synchronization status, understand discrepancies, if any, and retrieve logs for debugging.

## Source Controller

The _Source Controller_ is a relatively newer addition in FluxCD’s architecture, particularly becoming more prominent in Flux v2.

**Responsibilities**:

1. **Fetching Code**: The Source Controller is responsible for obtaining the latest code/configuration from various Git repositories.
2. **Event Notifications**: It signals the Flux Agent about any changes or updates in the Git repositories. Instead of the agent continuously polling the repository, the Source Controller optimizes this by actively notifying the agent of changes, thus acting as an efficient mediator.

# Working with Kustomize and Helm

![](h6DJOnEv7pUleWKP.webp)

Source: contentstack.io

## FluxCD and Helm

[_Helm_](https://helm.sh/?ref=8grams.tech) is often dubbed as the _“package manager for Kubernetes”_. It allows users to define, install, and upgrade complex Kubernetes applications using `charts` (pre-configured Kubernetes resources).

1. **HelmRelease Custom Resource (CRD)**: FluxCD introduces a custom resource known as `HelmRelease`. This CRD allows users to declare Helm chart releases and their specifications right within their Git repositories.
2. **Helm Controller in Flux**: FluxCD has a dedicated Helm controller that watches for changes to `HelmRelease` resources. Upon detecting changes in the Git repository related to Helm charts or configurations, it triggers the Helm operation, be it an install, upgrade, or rollback.
3. **Decoupled Operations**: With FluxCD, the actual Helm chart can reside in a Helm repository, and only the `HelmRelease` specification with values is maintained in the Git repository. This decouples the chart development from its deployment.
4. **Automated Image Updates**: Flux can be configured to automatically update the image versions in the Helm values whenever a new image is pushed to a container registry. This ensures that applications are always running the latest version without manual intervention.

## FluxCD and Kustomize

[_Kustomize_](https://kustomize.io/?ref=8grams.tech) offers a way to customize raw, template-free YAML files for Kubernetes, enabling multiple variants of configurations without altering the base resources.

1. **Kustomization Controller in Flux**: FluxCD has a controller dedicated to Kustomize called the Kustomization controller. This controller watches for changes in `Kustomization` custom resources and applies the customized YAML manifests to the Kubernetes cluster.
2. **Repository Structure**: In the Git repository, you can have a base directory containing common resources and multiple overlays (variants) representing different environments like staging and production. Each overlay can contain a `kustomization.yaml` file that specifies how to modify the base resources.
3. **Kustomize in Action with Flux**: When changes are detected in the Git repository, the Kustomization controller reads the associated `kustomization.yaml` files, processes the overlays on the base resources, and deploys the customized manifests to the Kubernetes cluster.
4. **Image Update Automation**: Just like with Helm, FluxCD can automatically update image tags in Kustomize overlays when a new image version is detected in a container registry. This allows for automated, GitOps-driven deployments of the latest application versions.

FluxCD’s deep integrations with _Helm_ and _Kustomize_ are a testament to its flexibility. Users can:

- Use _Helm_ for packaging and managing complex applications, leveraging the simplicity of Helm charts with the power of GitOps via FluxCD.
- Utilize _Kustomize_ for maintaining environment-specific customizations, ensuring that configurations for different environments (e.g., dev, staging, production) are declarative and stored in version control, while also enjoying the benefits of automated deployments with Flux.

Together, these integrations ensure that FluxCD is not just a tool for simple deployments but is versatile enough to handle complex, multi-environment, and large-scale Kubernetes applications with ease.

# FluxCD vs ArgoCD

In the world of GitOps, both FluxCD and [ArgoCD](https://argo-cd.readthedocs.io/?ref=8grams.tech) stand out as dominant tools. Let’s break down their differences and similarities:

![](F0k-8jkBKNjZrIRsRwKgCA.webp)

Suppose a company needs a GitOps tool with a strong UI component for visual tracking. In this case, they might lean towards ArgoCD due to its rich, integrated UI. Conversely, if a company heavily uses Helm and Kustomize, they might find FluxCD’s deep integrations more advantageous.

# Install FluxCD on Kubernetes

FluxCD is a Kubernetes-native application, which means it is explicitly designed to operate within a Kubernetes environment. This inherent design compatibility ensures that FluxCD seamlessly integrates with Kubernetes, making its installation and operation on a Kubernetes cluster straightforward and efficient.

## Preparation

To fully embody the principles of GitOps, the initial step is to maintain a dedicated repository for our Kubernetes configuration files. This repository will house all the Kubernetes manifest files. Assume our repository is located at `https://github.com/MyCompany/gitops`. FluxCD will treat this repository as the single source of truth for the desired state of our cluster.

Next, we need to generate a GitHub Token. Navigate to `[https://github.com/settings/tokens](https://github.com/settings/tokens?ref=8grams.tech)` and create a token, ensuring it has all the permissions under the `repo` section. Once generated, you can load the GitHub token using the `export` command:

`~$ export GITHUB_TOKEN=<gh-token>`

## Install FluxCD CLI

FluxCD CLI is the command-line interface for interacting with FluxCD. It’s a vital tool that facilitates the installation, bootstrap, configuration, and daily operations with FluxCD. Install it by executing this command below:

`~$ curl -s https://fluxcd.io/install.sh | sudo bash`

## Bootstrapping FluxCD

Bootstrapping will set up all the necessary components and configurations to run FluxCD on a Kubernetes cluster. This often involves setting up the required controllers, custom resource definitions (CRDs), and other dependencies.

Assume our GitOps repository is on `https://github.com/MyCompany/gitops`, bootstrap FluxCD with following command below:

```sh
~$ flux bootstrap github \  
  --owner=MyCompany \                    
  --repository=gitops \               
  --branch=main \                                         
  --path=./clusters/retro \  
  --personal
```

## Working with Kustomize

Kustomize is a standalone tool to customize Kubernetes manifests `the Kubernetes native way` without the need for templates or placeholders. It became an integral part of `kubectl` in version 1.14 and onwards.

You can learn about Kustomize through this article below

Assuming you’ve set up a Kustomize configuration as described in the article above, you can leverage FluxCD to continuously monitor this configuration. FluxCD will detect any changes made to your Kustomize setup and automatically update your cluster’s state to reflect the most recent configuration committed to the repository.

Execute this command below to make FluxCD manage your Kustomize:

```sh
flux create kustomization simple-web-server-dev \  
    --source=GitRepository/flux-system \  
    --target-namespace=dev \  
    --path="./overlays/dev" \  
    --prune=true \  
    --interval=1m
```

Now, whenever you modify the `overlays/dev/deployment-dev.yaml`, all you need to do is push the changes to `https://github.com/MyCompany/gitops`, and FluxCD will handle the rest.

## Setting an Alert

Frequently, it’s essential to know when FluxCD has successfully executed our changes. To stay informed, we can set up notifications to be sent to our Discord or Slack channels once FluxCD completes its tasks. In this example, we’ll establish a notification system that allows FluxCD to send updates to a specific Discord channel after synchronizing your Kubernetes cluster with the latest committed configuration.

Create two files: `alert.yaml` and `notification.yaml`. The `alert.yaml` file will house the built-in alert configurations, while `notification.yaml` will be set up with configurations for the Discord Webhook.

```yaml
# alert.yaml  
---  
apiVersion: notification.toolkit.fluxcd.io/v1beta1  
kind: Alert  
metadata:  
  name: discord  
  namespace: flux-system  
spec:  
  providerRef:  
    name: discord  
  eventSeverity: info  
  eventSources:  
  - kind: Kustomization  
    name: '*'  
  - kind: HelmRelease  
    name: '*'  
  - kind: HelmChart  
    name: '*'  
  - kind: ImageRepository  
    name: '*'
```

```yaml
# notification.yaml  
---  
apiVersion: notification.toolkit.fluxcd.io/v1beta1  
kind: Provider  
metadata:  
  name: discord  
  namespace: flux-system  
spec:  
  type: discord  
  address: https://discord.com/api/webhooks/<Discord Webhooks>
```

Install Alert System

```sh
kubectl -n flux-system apply -f alert.yaml  
kubectl -n flux-system apply -f notification.yaml
```

Voilà! You now have a powerful GitOps platform running in your Kubernetes cluster.