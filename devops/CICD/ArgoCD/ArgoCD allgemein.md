---
tags:
  - devops
  - argo
  - cicd
---

# ArgoWorkflow: First Deployment

Argo workflows is a workflow engine for executing DAGs of tasks on Kubernetes. 
It is implemented as Kubernetes CRDs (Custom Resource Definitions) and container-native workflows.

## Target Deployment

We will deploy argo workflows 0.41.4 with s3 access for artifacts. As of August 12th, 2024, the most recent version is 0.41.14. However, we will use 0.41.4. For the object storage, we will use Ceph RGW for this deployment. However, any s3-compatible object storage should be fine.

![](https://miro.medium.com/v2/resize:fit:700/1*UVfhp_tRCwp445tqNqxNdA.png)

Argo Workflow Installation

Namespace argo-workflows is where the argo workflows package (mainly argo workflows server and controller) will be installed. Namespace argo-events is where application developers will create actual workflows.

## Preparation

First, we need to set up an object storage. We used an Ceph RGW. Detailed instructions on how to set one up, please see Ceph RGW section in [this Ceph article](https://daegonk.medium.com/ceph-articles-ac918c367b6b). This step should provide an s3 storage access information: an endpoint, an access key and a secret key. They will be used for a Kubernetes configmap and secret in the next step.

Before helm installation, we need to prepare a config map and a secret. So, we will create namespaces and set up them first.

```yaml
---  
apiVersion: v1  
kind: Namespace  
metadata:  
  name: argo-workflows  
  labels:  
    app: argo-workflows  
---  
apiVersion: v1  
kind: Namespace  
metadata:  
  name: argo-events  
  labels:  
    app: argo-events

apiVersion: v1  
kind: ConfigMap  
metadata:  
  name: argo-workflows-controller-cm  
  namespace: argo-workflows  
data:  
  artifactRepository: |  
    s3:  
      bucket: argo-artifact-bucket  
      endpoint: [ip address]  
      insecure: true  
      accessKeySecret:  
        name: argo-artifact-secret  
        key: access-key  
      secretKeySecret:  
        name: argo-artifact-secret  
        key: secret-key
```

We will create a secret in `argo-events` namespace, not in `argo-workflows` .

`kubectl create secret -n argo-events generic argo-artifact-secret --from-literal=access-key='[access key]' --from-literal=secret-key='[secret key]'`

Please note that s3 storage access values need to be provided. Also, the object storage was set up with insecure mode. If it is in secure mode, `insecure` needs to be set to `false`.

We also use MetalLB for `LoadBalancer` service type.
See [this](https://daegonk.medium.com/metallb-befc757ccc9b) for installation steps.

## Helm Chart Installation

Let’s get argo workflows helm chart.

```sh
helm repo add argo https://argoproj.github.io/argo-helm  
helm fetch argo/argo-workflows --version 0.41.4 --untar
```

Now, we will override the following values.

```yaml
controller:  
  configMap:  
    create: false  
    name: argo-workflows-controller-cm  
server:  
  serviceType: "LoadBalancer"  
  loadBalancerIP: "192.168.63.216"  
  extraArgs: [ "--auth-mode=client" ]
```

The `argo-workflows-controller-cm` is what we created in the preparation steps. We expose workflow server using a load balancer. We will explain our choice of `auth-mode` in the next section. Please change `loadBalancerIP` value accordingly for your environment. Or you can change it to other service types. The default value is `ClusterIP` .

Now, we are ready to install the package.

`helm -n argo-workflows install argo-workflow ./argo-workflows -f argo-workflows-override.yaml`

![](https://miro.medium.com/v2/resize:fit:700/1*XHMXQn9uRgyM1ReMPrF-ZQ.png)

Kubernetes Components from Argo Workflows installation

Helm notes have some information for accessing servers. You will see them at the end of installation. Also, you can retrieve them using this command.

`helm get notes argo-workflow -n argo-workflows`

## Access Workflow Server

Once it is installed, we can access workflows server at http://192.168.63.216:2746. The ip address is what we provide in the override yaml file.

![](https://miro.medium.com/v2/resize:fit:700/1*9ZD0h2WV5YPVrOjCj23xHw.png)

Argo Workflows Server Website

We will use client authentication method in the middle. Please note that we set `auth-mode` to be `client` in the override yaml. To get a token, run these commands. Paste the output of `argo auth token` command starting _Bearer_.

```sh
argo_server=$(kubectl get pod -n argo-workflows -l app.kubernetes.io/name=argo-workflows-server --no-headers=true | awk '{printf $1}')  
kubectl -n argo-workflows exec $argo_server -- argo auth token
```

## Workflow Namespace Configuration

We need more work to do on the namespace where workflows will be deployed. In this article, we choose argo-events for that purpose. We already created the namespace and a secret for s3 storage access in it.

Now, we will create a service account with some RBAC permissions that workflows need to have.

```yaml
---  
apiVersion: v1  
kind: ServiceAccount  
metadata:  
  name: operate-workflow-sa  
  namespace: argo-events  
  
---  
apiVersion: rbac.authorization.k8s.io/v1  
kind: Role  
metadata:  
  name: operate-workflow-role  
  namespace: argo-events  
rules:  
  - apiGroups: [ "argoproj.io" ]  
    resources: [ "workflows" ]  
    verbs: [ "*" ]  
  - apiGroups: [ "argoproj.io" ]  
    resources: [ "workflowtaskresults" ]  
    verbs: [ "create", "patch" ]  
  - apiGroups: [ "" ]  
    resources: [ "pods" ]  
    verbs: [ "get", "patch" ]  
  
---  
apiVersion: rbac.authorization.k8s.io/v1  
kind: RoleBinding  
metadata:  
  name: operate-workflow-role-binding  
  namespace: argo-events  
roleRef:  
  apiGroup: rbac.authorization.k8s.io  
  kind: Role  
  name: operate-workflow-role  
subjects:  
  - kind: ServiceAccount  
    name: operate-workflow-sa  
    namespace: argo-events
```

## Testing: Workflow Creation

We will create a `workflowtemplates` and a `workflows` . It is just to verify the installed argo workflows package. We will not explain details of their specifications.

```yaml
---  
apiVersion: argoproj.io/v1alpha1  
kind: WorkflowTemplate  
metadata:  
  name: hw-templates  
  namespace: argo-events  
spec:  
  entrypoint: whalesay-template  
  templates:  
  - name: whalesay-template  
    inputs:  
      parameters:  
      - name: message  
        value: "Hello WorkflowTemplates"  
    container:  
      image: docker/whalesay  
      command: [cowsay]  
      args: ["{{inputs.parameters.message}}"]
```

This is a workflow template that prints message with a whale shape.

```yaml
apiVersion: argoproj.io/v1alpha1  
kind: Workflow  
metadata:  
  name: hello-world  
  namespace: argo-events  
spec:  
  serviceAccountName: operate-workflow-sa  
  entrypoint: whalesay  
  templates:  
  - name: whalesay  
    steps:  
      - - name: call-whalesay-template  
          templateRef:  
            name: hw-templates  
            template: whalesay-template  
          arguments:  
            parameters:  
            - name: message  
              value: "hello world"
```

This is a workflow that use the `hw-templates` workflow template. Please note that we use service account `operate-workflow-sa` that we created after the installation.

![](https://miro.medium.com/v2/resize:fit:700/1*ovrkI7eP94LlWBa5i0D-AQ.png)

The argo workflows server website also provides these information. It is more user-friendly as well.

![](https://miro.medium.com/v2/resize:fit:700/1*Vs0m3y7v5jxzYBMiVny40g.png)

Workflows Page on Argo workflows server website

## Afterwords

In this article, we focused on

- installing argo workflows packages
- verifying whether it is correctly installed by creating a workflow.

We still have not verified our s3 storage configuration. Next time, we will start with s3 storage configuration and present how to write workflow templates.
## More Articles

- [On Ceph](https://daegonk.medium.com/ceph-articles-ac918c367b6b)
- [On Kubernetes](https://daegonk.medium.com/articles-on-kubernetes-04be54ef5c7a)
https://blog.devgenius.io/argoworkflow-first-deployment-8ff95bfdd847

https://itnext.io/create-your-own-open-source-observability-platform-using-argocd-prometheus-alertmanager-a17cfb74bfcf#7c1d
## Argo cd Secrets
https://codefresh.io/blog/gitops-secrets-with-argo-cd-hashicorp-vault-and-the-external-secret-operator/
## ArgoCD RBAC
https://codefresh.io/blog/multi-tenant-argocd-with-application-projects/
https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/#rbac-model-structure

```
argocd admin initial-password -n argocd
```

```
argocd login <ARGOCD_SERVER>
```