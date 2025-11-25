# Introduction:

In this blogpost we are going to discuss on how to design end to end gitops workflows using Terraform,Gitlab, FluxCD and Kubernetes.

But before we delve into the details of implementation lets take a step back and try to understand what problem statement gitops addresses and how does terraform comes into the mix.

# What and the Why of GitOps

If we look at the way we build our infrastructure nowadays most of it is built using some form of infrastructure as code or at the very least almost all of cloud infrastructure is built that way. One of challenges of using the infrastructure as code is over a period of time as the codebase grows and more people get involved in the process of development it becomes cumbersome to maintain the codebase.

Thats where gitops comes in , at the core gitops utilises git to maintain the code base of infrastructure code and uses an automated way to match the configuration with the target environment. In this way we can deploy faster and also catch and recover from errors much faster than before since we have the complete history of what changed over time.

# Why Terraform for GitOps

At this point you might be wondering how does terraform comes in to the mix and how does it help. So , in the previous section we were talking about the automated way to match the configuration , usually we will use a tools such as fluxcd to do that for us in a typical kubernetes environment. But that requires additional configuration and setup , that’s where terraform comes in , you see with terraform we can not only build infrastructure but also can install and configure tools such as flux so that at one go you can build your kubernetes cluster and once that happens you can bootstrap flux and setup gitops workflows with it, in that way you are set with a fully functional gitops workflow from the get go.

# Tools Overview:

# FluxCD:

Flux is a tool for keeping Kubernetes clusters in sync with sources of configuration (like Git repositories), and automating updates to configuration when there is new code to deploy.

# Gitlab:

GItLab is a CI/CD tool which helps you to automate the application build and release process.

# Terraform:

Terraform allows users to define and provision infrastructure resources such as virtual machines, networks, storage, and other cloud services using a declarative configuration language.

# What we will be building:

In the solution we will be deploying an nginx app to a **eks** cluster , apart from that we will be using flux to manage the deployments to the eks cluster , and we will be using terraform for infrastructure provisioning and flux bootstrap configuration.

# Setup

# Prerequisites:

1) `An AWS Account`  
2) `Gitlab Cloud Account/ Self Hosted Gitlab`  
3) `Lastest Version of Terraform installed in local system`

# Environment Setup:

## AWS Setup:

Create the ECR Repository

`aws ecr create-repository --repository-name nginx-repo`

Log in to the Amazon ECR registry

```
aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com

```
Pull the Nginx image from Docker Hub

`docker pull nginx:latest`

Tag the Nginx image for your ECR repository

`docker tag nginx:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/nginx-repo:0.0.1`

Push the image to the ECR repository

`docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/nginx-repo:0.0.1`

## Gitlab Setup

PAT Token Generation

```
- Login to the GitLab instance  
- Click on the profile icon  
- Click on preferences and post that click on access tokens  
- Enter the name of the token and set an expiry date of your choice  
- Post that select the first checkbox under scopes called api  
- Click on create personal access token and then click to reveal the token  
- Save the token.
```

Project Setup:

```
- Click on the new project button on the cloud instance of gitlab.  
- Click on Create Blank Project  
- Provide the Project Name (Make a note of the name for later use)  
- Select the Group Name from the drop down next to the project url,  
   make a note of this group name we will be using this later.  
- Provide a name for the project.  
- Select the Visibility  
- Save the Project  
- Click on the Code Button and copy the url for the github for https make a   
  make a note of the url as we will be needing this later
```

## Terraform Setup

Prior to getting started with the terraform config and snippets to build our infrastructure we need to setup aws credentials by using the below commands

AWS Credential Setup

```sh
export AWS_ACCESS_KEY_ID="anaccesskey"  
export AWS_SECRET_ACCESS_KEY="asecretkey"  
export AWS_REGION="us-west-2"
```

Post the above setup we need to setup the gitlab token in our local environment.The below section outlines how to do that, paste the token which you copied from the pat token section in the gitlab setup section above.

GitLab Token SetupLinux/MacOs

`export GITLAB_TOKEN=<Saved token from gitlab setup>`

Windows

`$env:GITLAB_TOKEN = "<Saved token from gitlab setup>"`

# Walkthrough:

# Clone the demo repo

`git clone https://gitlab.com/devops5480719/devops-samples.git`

Then navigate to the infra directory

`cd devops-samples/gitops-gitlab/infra/`

Once you are in the directory you will find the below files and folders.

`├── config_files`  
`│   ├── app-nginx.yml`  
`│   ├── ecr-sync.yml`  
`│   └── nginx.yml`  
`├── main.tf`  
`├── variables.tf`  
`└── versions.tf`

Inside you will find a folder called **config_files**(more on that later) and a few terraform files.

Lets have a closer look at terraform files to begin with.

> _Note: The resources defined in the below section are by no means meant for production deployments these are defined purely for testing purposes.In the code snippets substitute the_ **_<aws-account-id>_** _with your_ **_AWS Account Id_** _and_ **_<aws-region>_** _with the_ **_region_** _where you will deploy resources._

## Variable Configuration

First lets look at the variable configuration.

`**variables.tf**`

```yaml
variable "gitlab_group" {    
  type = string    
  default = <name of the gitlab group>    
}    
  
variable "gitlab_project" {    
  type = string    
  default = <name of the project in gitlab>    
}    
variable "aws_region" {    
  type = string    
  default = <name of the region in aws where you will deploy resources>    
}  
```

In the above snippet substitute the default value with the gitlab_group with the name copied earlier in the gitlab section and same is true for the gitlab_project as well.

For the aws_region choose a region of your choice where you will be deploying the resources to.

## Provider Configuration

Now lets look at the provider configuration in the **versions.tf** file

We have included the required providers and the required configurations for the respective provider blocks for authentication and authorisation.

`**versions.tf**`

```yaml
required_providers {    
    flux = {    
      source  = "fluxcd/flux"    
      version = ">= 1.0.0"    
    }    
    gitlab = {    
      source  = "gitlabhq/gitlab"    
      version = ">=15.10.0"    
    }    
    aws = {    
      source  = "hashicorp/aws"    
      version = ">= 5.0"    
    }    
    kubernetes = {    
      source  = "hashicorp/kubernetes"    
      version = ">= 2.16.1"    
    }    
  }    
}    
provider "gitlab" {    
  base_url = "https://gitlab.com/api/v4/"    
}  
provider "flux" {    
  kubernetes = {    
    host                   = module.eks.endpoint    
    cluster_ca_certificate = base64decode(module.eks.cluster_ca_certificate)    
    exec = {    
      api_version = "client.authentication.k8s.io/v1beta1"    
      args        = ["eks", "get-token", "--cluster-name", var.cluster_name]    
      command     = "aws"    
    }    
  }    
  git = {    
    url = "ssh://git@gitlab.com/${data.gitlab_project.this.path_with_namespace}.git"    
    ssh = {    
      username    = "git"    
      private_key = tls_private_key.flux.private_key_pem    
    }    
    branch = "main"    
  }    
}
```

## Resource Configuration:

Now lets take a look at the resource configuration.

VPC Configuration

`**main.tf**`

```yaml
module "vpc" {    
  source = "terraform-aws-modules/vpc/aws"    
  name = "flux-vpc"    
    cidr = "10.0.0.0/16"    
    azs             = ["<aws-region>a", "<aws-region>b", "<aws-region>c"]    
    private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24", "10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]    
    public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]    
    enable_nat_gateway = true    
    enable_vpn_gateway = false    
    single_nat_gateway = true    
    public_subnet_tags = {    
      "kubernetes.io/role/elb" = "1"    
    }    
    tags = {    
      Terraform = "true"    
      Environment = "dev"    
    }    
}
```

EKS Configuration

We will be defining the below resources for eks.

1. Eks Cluster
2. Null resource to add kubeconfig to your context

`**main.tf**`

```yaml
module "eks" {    
  source = "terraform-aws-modules/eks/aws"    
  version = "~> 20.0"    
  cluster_name = "flux-cluster"    
  cluster_version = "1.29"    
  cluster_endpoint_private_access = true    
  cluster_endpoint_public_access  = true    
  cloudwatch_log_group_retention_in_days = 7    
  cloudwatch_log_group_class = "INFREQUENT_ACCESS"    
  cluster_enabled_log_types = ["api"]    
  vpc_id                   = module.vpc.vpc_id    
  subnet_ids = module.vpc.private_subnets    
  cluster_addons = {    
    coredns = {    
      most_recent = true    
    }    
    kube-proxy = {    
      most_recent = true    
    }    
    aws-ebs-csi-driver = {    
      most_recent    = true    
    }    
    eks-pod-identity-agent = {    
      most_recent    = true    
    }    
    vpc-cni = {    
      most_recent    = true    
    }    
  }    
  eks_managed_node_group_defaults = {    
    ami_type               = "AL2_x86_64"    
    disk_size              = 50    
    instance_types = ["t3.large"]    
    capacity_type = "SPOT"    
    update_config = {    
      max_unavailable_percentage = 100    
    }    
  }    
  eks_managed_node_groups = {    
    ps-cluster-sample = {    
      min_size = 1    
      desired_size = 1    
      max_size = 4    
      instance_types = ["t3.large"]    
      capacity_type  = "SPOT"    
    }    
  }    
  enable_cluster_creator_admin_permissions = false    
  access_entries = {    
    admin_sso = {    
      kubernetes_groups = []    
      principal_arn     = "<arn of the role or the user to which you want to grant admin privileges>"    
      policy_associations = {    
        ClusterAdmin = {    
          policy_arn = "arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy"    
          access_scope = {    
            type       = "cluster"    
          }    
        }    
        EKSAdmin = {    
          policy_arn = "arn:aws:eks::aws:cluster-access-policy/AmazonEKSAdminPolicy"    
          access_scope = {    
            type       = "cluster"    
          }    
        }    
      }    
    }    
  }    
}    
resource "null_resource" "update_kubeconfig" {    
  triggers = {    
    eks_cluster_id = module.eks.cluster_id    
  }    
  provisioner "local-exec" {    
    command = "aws eks update-kubeconfig --name ${module.eks.cluster_name} --region ${var.aws_region}"    
  }    
}    
}
```

Flux BootStrap Configuration

Finally Lets look at the flux bootstrap configuration.

`**main.tf**`

```yaml
resource "tls_private_key" "flux" {    
  depends_on = [module.eks, module.vpc, null_resource.update_kubeconfig]    
  algorithm   = "ECDSA"    
  ecdsa_curve = "P256"    
}    
data "gitlab_project" "this" {    
  path_with_namespace = "${var.gitlab_group}/${var.gitlab_project}"    
}    
resource "gitlab_deploy_key" "this" {    
  depends_on = [module.eks, module.vpc]    
  project  = data.gitlab_project.this.id    
  title    = "Flux"    
  key      = tls_private_key.flux.public_key_openssh    
  can_push = true    
}    
resource "flux_bootstrap_git" "this" {    
  depends_on = [gitlab_deploy_key.this, module.eks, module.vpc, null_resource.update_kubeconfig]    
  path = "clusters/${module.eks.cluster_name}"    
  components_extra = ["image-reflector-controller","image-automation-controller"]    
}    
locals {    
  yaml_files = fileset("${path.module}/config_files", "*.yml")    
  yaml_content = { for file in local.yaml_files : file => file("${path.module}/config_files/${file}") }    
}    
resource "gitlab_repository_file" "yaml_files" {    
  depends_on = [flux_bootstrap_git.this, null_resource.update_kubeconfig]    
  for_each = local.yaml_content    
  project = data.gitlab_project.this.id // Use the project ID of an existing project or gitlab_project.example_project.id if you created a new project    
  file_path = "clusters/${module.eks.cluster_name}/${each.key}"    
  content   = base64encode(each.value)    
  commit_message = "added flux image configs"    
  branch    = "main"    
}
```

In the above code snippet we are adding two extra components to flux_bootstrap_git resource namely i**mage-reflector-controller** and **image-automation-controller.**

These two controllers are going to be responsible for pulling/updating the tags from the ecr repo.

In order for flux to perform this we need to define few additional components which you will find in the config_files folder in the cloned demo repo as seen in the image below .

```
├── config_files  
│   ├── app-nginx.yml  
│   ├── ecr-sync.yml  
│   └── nginx.yml  
├── main.tf  
├── variables.tf  
└── versions.tf
```

Now lets look at each of these files and what’s inside them, lets start with the file app-nginx.yaml file.

`**app-nginx.yaml**`

```yaml
---  
apiVersion: v1    
kind: Secret    
metadata:    
  name: nginx-auth-pat    
  namespace: flux-system    
type: Opaque    
data:    
  password: < gitlab token as base64 encoded string >   
  username: < username as base64 encoded string >---  apiVersion: source.toolkit.fluxcd.io/v1    
kind: GitRepository    
metadata:    
  name: nginx    
  namespace: flux-system    
spec:    
  interval: 1m0s    
  ref:    
    branch: main    
  url: <gitlab_repo_url>   
  secretRef:    
    name: nginx-auth-pat    
---    
apiVersion: kustomize.toolkit.fluxcd.io/v1    
kind: Kustomization    
metadata:    
  name: nginx    
  namespace: flux-system    
spec:    
  prune: true    
  interval: 1m0s    
  path: "./app/nginx/environments/production"    
  sourceRef:    
    kind: GitRepository    
    name: nginx    
---    
apiVersion: image.toolkit.fluxcd.io/v1beta2    
kind: ImageRepository    
metadata:    
  name: nginx    
  namespace: flux-system    
spec:    
  image: <aws-account-id>.dkr.ecr.<region>.amazonaws.com/<image name>    
  interval: 1m0s    
  secretRef:    
    name: ecr-credentials    
---    
apiVersion: image.toolkit.fluxcd.io/v1beta2    
kind: ImagePolicy    
metadata:    
  name: nginx    
  namespace: flux-system    
spec:    
  imageRepositoryRef:    
    name: nginx    
  policy:    
    semver:    
      range: 0.0.x    
---    
apiVersion: image.toolkit.fluxcd.io/v1beta1    
kind: ImageUpdateAutomation    
metadata:    
  name: nginx    
  namespace: flux-system    
spec:    
  interval: 1m0s    
  sourceRef:    
    kind: GitRepository    
    name: nginx    
  git:    
    checkout:    
      ref:    
        branch: main    
    commit:    
      author:    
        email: flux@example.com    
        name: flux    
      messageTemplate: "{{range .Updated.Images}}{{println .}}{{end}}"    
    push:    
      branch: main    
  update:    
    path: ./clusters/flux-cluster/nginx.yml  
    strategy: Setters  
```

In the above configuration we have defined the below components:

`**Secret**` : We have defined a base 64 encoded **pat token** (which we created in gitlab setup section) and **username** for our git repo in gitlab, the username i have used is **git** in accordance with the provider section configuration in terraform. `**GitRepository**`: This is the gitlab repo url which flux will poll for the yaml files, which you created in the gitlab setup section. `**ImageRepository**`: We define a repository to scan and store a specific set of tags in a database `**ImagePolicy**`: We defines rules for selecting a “latest” image from image repositories. `**ImageUpdateAutomation**`: We define an automation process that will update a git repository, based on image policy objects in the same namespace

In the above file under ImageUpdateAutomation we have defined the path for flux to update the image tag under the section **.spec.update.path** as **./clusters/flux-cluster/nginx.yml** which will tell flux where to update the image tag

Post the above config we also need to define the below ecr sync file which will automate the authentication to ecr and thus enabling the **image automation controller** to pull the image tags from the ecr repos.The below file sets up a cron job with the necessary permissions to authenticate from ecr every 6 hours and update secret accordingly.

`**ecr-sync.yaml**`

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: ecr-credentials-sync
  namespace: flux-system
rules:
- apiGroups: [""]
  resources:
  - secrets
  verbs:
  - get
  - create
  - patch
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: ecr-credentials-sync
  namespace: flux-system
subjects:
- kind: ServiceAccount
  name: ecr-credentials-sync
roleRef:
  kind: Role
  name: ecr-credentials-sync
  apiGroup: ""
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ecr-credentials-sync
  namespace: flux-system
  # Uncomment and edit if using IRSA
  # annotations:
  #   eks.amazonaws.com/role-arn: <role arn>
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ecr-credentials-sync
  namespace: flux-system
spec:
  suspend: false
  schedule: 0 */6 * * *
  failedJobsHistoryLimit: 1
  successfulJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: ecr-credentials-sync
          restartPolicy: Never
          volumes:
          - name: token
            emptyDir:
              medium: Memory
          initContainers:
          - image: amazon/aws-cli
            name: get-token
            imagePullPolicy: IfNotPresent
            # You will need to set the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY environment variables if not using
            # IRSA. It is recommended to store the values in a Secret and load them in the container using envFrom.
            # envFrom:
            # - secretRef:
            #     name: aws-credentials
            env:
            - name: REGION
              value: us-east-1 # change this if ECR repo is in a different region
            volumeMounts:
            - mountPath: /token
              name: token
            command:
            - /bin/sh
            - -ce
            - aws ecr get-login-password --region ${REGION} > /token/ecr-token
          containers:
          - image: ghcr.io/fluxcd/flux-cli:v0.25.2
            name: create-secret
            imagePullPolicy: IfNotPresent
            env:
            - name: SECRET_NAME
              value: ecr-credentials
            - name: ECR_REGISTRY
              value: <account id>.dkr.ecr.<region>.amazonaws.com # fill in the account id and region
            volumeMounts:
            - mountPath: /token
              name: token
            command:
            - /bin/sh
            - -ce
            - |-
              kubectl create secret docker-registry $SECRET_NAME \
                --dry-run=client \
                --docker-server="$ECR_REGISTRY" \
                --docker-username=AWS \
                --docker-password="$(cat /token/ecr-token)" \
                -o yaml | kubectl apply -f -
```

Now lets take a look at the last file which is called **nginx.yml** , this files deploys a sample nginx app.

`**nginx.yaml**`

```yaml
apiVersion: apps/v1    
kind: Deployment    
metadata:    
  name: nginx-deployment    
spec:    
  replicas: 1    
  template:    
    spec:    
      containers:    
        - name: nginx    
          image: <aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com/nginx:0.0.0 # {"$imagepolicy": "flux-system:nginx"}
```

The snippet `**# {"$imagepolicy": "flux-system:nginx"}**` is the marker in which flux-system is the **namespace** and nginx is the name of the **image policy** this should match the automation configuration defined above in the **app-nginx.yml** file.

This snippet tells flux which policy to use when updating the container image

All of these files will be copied to the flux folder for it to apply it to the eks cluster via the below code snippet in the **main.tf** file

```yaml
locals {    
  yaml_files = fileset("${path.module}/config_files", "*.yml")    
  yaml_content = { for file in local.yaml_files : file => file("${path.module}/config_files/${file}") }    
}    
resource "gitlab_repository_file" "yaml_files" {    
  depends_on = [flux_bootstrap_git.this, null_resource.update_kubeconfig]    
  for_each = local.yaml_content    
  project = data.gitlab_project.this.id // Use the project ID of an existing project or gitlab_project.example_project.id if you created a new project    
  file_path = "clusters/${module.eks.cluster_name}/${each.key}"    
  content   = base64encode(each.value)    
  commit_message = "added flux image configs"    
  branch    = "main"    
}
```

# Deployment:

Post that initialise terraform by running the below command

`terraform init`

Then run the plan command to check all resources getting spun up

`terraform plan`

Post that apply terraform configuration

`terraform apply --auto-approve`

Post this terraform will do the following.

```
Create VPC and Supporting resources  
Create EKS Cluster and supporting resources  
Bootstrap Flux with eks and store all the config and application files the gitlab repo
```

Once the terraform completes the execution post that run the below command

`kubectl get po -n flux-system`

You should get the below output with all the flux components running.

```sh
NAME                                           READY   STATUS    RESTARTS   AGE  
helm-controller-5f7457c9dd-6svk8               1/1     Running   0          40s  
image-automation-controller-79447887bb-blqp4   1/1     Running   0          40s  
image-reflector-controller-65df777f5c-nrgsx    1/1     Running   0          40s  
kustomize-controller-5f58d55f76-tpgs6          1/1     Running   0          40s  
notification-controller-685bdc466d-mp46f       1/1     Running   0          40s  
source-controller-86b8b57796-vl2fp             1/1     Running   0          39s
```

Now that we have all the components running , lets check if we have ecr secret cron job executed, for that execute the below command in the terminal

`kubectl get cronjob -n flux-system`

You should get the below output

```
NAME                   SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE  
ecr-credentials-sync   0/1 * * * *   False     0        51s             2m42s
```

Now lets check if the secret is present in the flux-system namespace you should see the below output:

`k get secret -n flux-system`  
  
```
NAME              TYPE                             DATA   AGE  
ecr-credentials   kubernetes.io/dockerconfigjson   1      3m7s  
flux-system       Opaque                           3      5m39s  
nginx-auth-pat    Opaque                           2      4m19s
```

As you can see from the above output we have the secret called ecr-credentials present.

Now lets check if the flux image controllers were able to pull the tag from our ecr repo

```sh
flux get image all  
NAME                   LAST SCAN                       SUSPENDED       READY   MESSAGE  
imagerepository/nginx   2024-05-23T16:55:46+05:30       False           True    successful scan: found 2 tags  
NAME                    LATEST IMAGE                                                    READY   MESSAGE  
imagepolicy/nginx       <aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com/nginx:0.0.1       True    Latest image tag for '<aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com/nginx' resolved to 0.0.1  
NAME                            LAST RUN                        SUSPENDED       READY   MESSAGE  
imageupdateautomation/nginx     2024-05-23T16:55:39+05:30       False           True    repository up-to-date
```

As you can see from the output the image policy was able to scan and fetch the latest image tag from ecr.

Now lets check the same has been updated in our deployment or not by issuing the below command:

`kubectl get deployment nginx-deployment -o=jsonpath='{.spec.template.spec.containers[*].image}' | awk -F '[:@]' '{print $1, $2}'`

Below is output we got , as you can see the image has been updated

`<aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com/nginx:0.0.1`

Now lets move to the gitlab side and validate if flux has made the necessary changes.

You can check the commit history of your repo from the ui or you can clone the repo and run the below command.

Run the below command :

```sh
git log --author="flux" --since="today 00:00" --until="today 23:59" --pretty=format:"%h %s" -- clusters/flux-cluster/nginx.yml
```

Below is the output you get :

`85e5b48 <aws-account-d>.dkr.ecr.<aws-region>.amazonaws.com/nginx:0.0.1`

As you can see from the above output flux updated the image in the nginx.yml file from a random image number to the actual tag and the same was updated in deployment.

Lets try to push a new revision of the image and lets see if flux automatically updates it

Tag the Nginx image for your ECR repository

`docker tag nginx:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/nginx-repo:0.0.2`

Now lets check the same has been updated in our deployment or not by issuing the below command:

```
kubectl get deployment nginx-deployment -o=jsonpath='{.spec.template.spec.containers[*].image}' | awk -F '[:@]' '{print $1, $2}'
```

Below is the output you get :

`<aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com/nginx:0.0.2`

