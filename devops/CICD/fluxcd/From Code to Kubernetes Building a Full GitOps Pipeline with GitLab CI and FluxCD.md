# What is GitOps and Why Do We Need It?

GitOps is a modern approach to continuous deployment that leverages Git as the single source of truth for both application and infrastructure code. It enables teams to manage and automate their Kubernetes deployments by using Git repositories to track and version control all changes. With GitOps, any change made to the Git repository automatically triggers an update to the live environment, ensuring that the state of the system is always consistent with what’s defined in Git. It’s particularly beneficial in Kubernetes environments where managing infrastructure and application updates can become complex and error-prone without a robust version control system.

In this article, I will guide you through the fundamentals of GitOps and demonstrate how to configure a complete end-to-end GitOps pipeline using GitLab CI and FluxCD. We’ll explore how these tools integrate seamlessly to automate your Kubernetes deployments, ensuring that your infrastructure and applications are always in sync with the desired state defined in your Git repositories. Whether you’re new to GitOps or looking to enhance your CI/CD pipeline, this step-by-step guide will provide you with the knowledge and code snippets needed to build a robust and efficient GitOps workflow.
## Requirements:

- GitLab repository
- Running Kubernetes cluster
- Kubectl and Flux CLI installed
- GitLab Runner configured
- Helm
- FluxCD installed on Kubernetes
- Access to a container registry (e.g., Docker Hub, GitLab Container Registry, Harbor)

## General Structure

![](lNrF1UpPy0dCFSytH1Tc_A.webp)

In this GitOps pipeline, we’ll be setting up a series of stages that ensure our application is built, tested, and deployed to our Kubernetes cluster in a secure and automated manner. The pipeline is designed to handle a typical CI/CD workflow, which integrates static analysis, containerization, security testing, and deployment. Here’s how the stages are structured:

**Build Stage**:

- This initial stage compiles the application and packages it.
- The output of this stage is a build artifact that will be used in subsequent stages.

**Test Stage**:

- In this stage, the application is tested using unit and integration tests to ensure code quality and functionality.
- After the tests pass, this stage splits into two parallel paths: (SAST and Docker build)

**SAST (Static Application Security Testing)**:

- This stage performs static code analysis to detect security vulnerabilities in your application’s source code.

**Build and Push Docker Image**:

- Simultaneously, the tested application is packaged into a Docker image and pushed to a container registry.

**DAST (Dynamic Application Security Testing)**:

- Once the Docker image is available, this stage runs dynamic security tests on the running application to find vulnerabilities in a live environment.
- This stage ensures that the application is secure before moving to deployment.

**Deploy Stage**:

- If both SAST and DAST stages are successful, the pipeline proceeds to deployment.
- In this stage, Kubernetes YAML files are generated from Helm charts (they will be in their own repository), which define how the application will be deployed in the Kubernetes cluster.
- These YAML files are then pushed to a dedicated Git repository monitored by FluxCD.
- FluxCD automatically detects these changes and updates the application in the Kubernetes cluster.

For this tutorial, I will be using a general Spring Boot application as an example to demonstrate each of these stages, ensuring that the app is securely tested and deployed using GitOps principles.

## Getting Started

To begin setting up our GitOps pipeline, we’ll need to install `kubectl` and FluxCD on our development machine. `kubectl` is the command-line tool for interacting with the Kubernetes cluster, allowing to manage and deploy applications. FluxCD is the GitOps operator that will monitor the Git repository for changes and ensure that our Kubernetes cluster is always in sync with the desired state defined in your repository.

**Install** `**kubectl**`: Follow the official Kubernetes documentation to download and install `kubectl` for your operating system. Once installed, configure it to connect to your Kubernetes cluster.

- Windows: [https://kubernetes.io/docs/tasks/tools/install-kubectl-windows](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows)
- Linux: [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux)
- MacOS: [https://kubernetes.io/docs/tasks/tools/install-kubectl-macos](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos)

**Install FluxCD**: Next, install the FluxCD CLI by following the instructions on the official FluxCD documentation. This tool will help you bootstrap FluxCD into your Kubernetes cluster and manage your GitOps deployments.

- Homebrew install for MacOS and Linux:

```sh
brew install fluxcd/tap/flux
```

- Chocolately install for Windows:

`choco install flux`

## Setting Up

To set up our GitOps pipeline, we’ll need to create three separate repositories, each serving a specific purpose in our workflow:

**Spring App Repository**: This repository will contain our Spring Boot application’s source code and the `.gitlab-ci.yml` file that defines our CI/CD pipeline. This is where we will manage the application’s development and continuous integration processes.

**Helm Charts Repository**: This repository will store our Helm charts, which define the Kubernetes resources needed to deploy our application. These charts will be used in the deploy stage of our pipeline to generate Kubernetes manifests.

**FluxCD Repository**: This repository will be monitored by FluxCD and will contain the Kubernetes manifest files generated from our Helm charts. Any changes pushed to this repository will trigger FluxCD to update our application in the Kubernetes cluster, ensuring that our deployments stay in sync with the desired state defined in our Git repositories.

## Installing FluxCD to Kubernetes

With the necessary repositories in place, the next step is to install FluxCD on our Kubernetes cluster. FluxCD will act as the GitOps operator, continuously monitoring the FluxCD repository for changes and applying those changes to our cluster. To install FluxCD, we’ll use the Flux CLI, which simplifies the setup process.

To install FluxCD on Kubernetes, run the following command using the Flux CLI:

```sh
flux bootstrap gitlab \  
  --hostname= <your gitlab host> \  
  --owner= <repo location group> \  
  --repository= <repository name> \  
  --branch= main\  
  --path= <Flux install location> \  
  --namespace= <Kubernetes namespace to install> \  
--token-auth
```

When setting up FluxCD with GitLab, we use the `flux bootstrap gitlab` command to connect our GitLab repository to FluxCD. Each flag in this command has a specific purpose:

- `--hostname=<your gitlab host>`: Specifies the hostname of our GitLab instance. If we're using a self-hosted GitLab, this will be our custom domain. For GitLab.com, it would be `gitlab.com` or you could remove the flag.
- `--owner=<repo location group>`: Defines the group or user that owns the repository in GitLab. This is the top-level namespace where our repository is located.
- `--repository=<repository name>`: Indicates the name of the GitLab repository that FluxCD will monitor for changes. This is where our Kubernetes manifest files will be stored.
- `--branch=main`: Specifies the branch of the repository that FluxCD should track. The `main` branch is commonly used, but we can specify another branch if needed.
- `--path=<Flux install location>`: Determines the directory path within the repository where FluxCD will install its manifests.
- `--namespace=<Kubernetes namespace to install>`: Indicates the Kubernetes namespace where FluxCD will be installed. This helps in isolating FluxCD's resources from other applications in the cluster.
- `--token-auth`: Enables token-based authentication to connect to our GitLab repository. This flag ensures that FluxCD can securely access our GitLab repository using a personal access token.
## Creating The Pipeline

In this section, we will set up our GitLab CI pipeline to automate the build, testing, Dockerization, and deployment of our Spring Boot application. Below is the `.gitlab-ci.yml` configuration, which defines the stages and jobs necessary for our pipeline. We'll break down each stage and explain its purpose.
## Pipeline Stages Overview

```yaml
stages:  
  - build  
  - test  
  - dockerize  
  - deploy
```

Our pipeline is organized into four stages:

1. **Build**: Compiles the application and packages it as a JAR file.
2. **Test**: Runs unit and integration tests to ensure code quality.
3. **Dockerize**: Packages the application into a Docker image and pushes it to a container registry.
4. **Deploy**: Generates Kubernetes manifest files from Helm charts and pushes them to the FluxCD repository.
## Build Stage

```yaml
build:  
  stage: build  
  image: maven:3-jdk-8-alpine  
  script:  
    - mvn clean package -DskipTests  
  artifacts:  
    paths:  
      - target/*.jar
```

In the **Build** stage, we use a Maven Docker image to compile our Spring Boot application. The `mvn clean package -DskipTests` command packages the application into a JAR file while skipping the tests (which will be handled in the next stage). The compiled JAR file is stored as an artifact, which will be used in subsequent stages.
## Test Stage

```yaml
test:  
  stage: test  
  image: maven:3-jdk-8-alpine  
  script:  
    - mvn test  
  allow_failure: true
```

The **Test** stage runs the unit and integration tests using Maven. This stage is marked with `allow_failure: true`, meaning the pipeline will continue even if the tests fail. This is useful when you want to ensure that the build and deployment proceed while still capturing test results for later review.
## Dockerize Stage

```yaml
dockerize:  
  stage: dockerize  
  image: docker:24.0.5  
  services:  
    - docker:24.0.5-dind  
  before_script:  
    - docker login -u $REGISTRY_USER -p $REGISTRY_PASSWORD $IMAGE_REPO  
  script:  
    - cp target/*.jar .  
    - docker build -t ${IMAGE_REPO}${IMAGE_NAME}:${CI_COMMIT_SHORT_SHA} .  
    - docker push ${IMAGE_REPO}${IMAGE_NAME}:${CI_COMMIT_SHORT_SHA}  
  artifacts:  
    paths:  
      - target/*.jar  
  dependencies:  
    - build  
  needs:  
    - job: build  
      artifacts: true
```

In the **Dockerize** stage, we build a Docker image from the JAR file generated in the build stage. The Docker image is then pushed to our specified container registry. The `docker:24.0.5-dind` service is used to run Docker in Docker, allowing us to build and push images within the CI environment. The image is tagged with the commit hash for versioning.
# Configuring Helm

Before we proceed with the **Manifest** stage, we need to configure Helm by creating a Helm chart that will define the Kubernetes resources for our Spring Boot application. Helm allows us to manage these resources as reusable and configurable templates, making it easier to deploy our application across different environments.
## Step 1: Create a Helm Chart

First, let’s create a new Helm chart named `spring-app`. This chart will contain the templates for our Kubernetes resources. You can create the Helm chart using the following command:

`helm create spring-app`

This command will generate a basic directory structure for the `spring-app` Helm chart, including default templates and a `values.yaml` file where we can define the default values for our templates.
## Stp 2: Add Templates for Deployment, Service, Namespace, and HPA

Next, we’ll add four specific templates to the `spring-app` chart to manage our application’s deployment, service, namespace, and horizontal pod autoscaler (HPA). Here’s how to structure each template:

**Deployment Template**

The Deployment template defines how our Spring Boot application will be deployed and managed within the Kubernetes cluster. It specifies the number of replicas, the container image to use, and the ports the application will expose.

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: {{ include "spring-app.fullname" . }}  
  namespace: {{ include "spring-app.namespace" . | default "default-namespace" }}  
  labels:  
    app: {{ include "spring-app.name" . }}
```

- **apiVersion**: Specifies the API version of the resource. Here, we use `apps/v1` for the Deployment resource.
- **kind**: Defines the type of Kubernetes resource, which in this case is a `Deployment`.
- **metadata**: Contains information about the deployment, including the name, namespace, and labels. The name is generated using the `fullname` template helper, and the namespace is either provided or defaults to `default-namespace`.

```yaml
spec:  
  replicas: {{ include "spring-app.replicas" . | default 1 }}  
  selector:  
    matchLabels:  
      app: {{ include "spring-app.name" . }}  
  template:  
    metadata:  
      labels:  
        app: {{ include "spring-app.name" . }}  
    spec:  
      containers:  
      - name: {{ include "spring-app.name" . }}  
        image: {{ include "spring-app.image.repository" . }}:{{ include "spring-app.image.tag" . | default "latest" }}  
        ports:  
        - containerPort: {{ include "spring-app.containerPort" . | default 8080 }}
```

- **replicas**: Specifies the desired number of pod replicas for the deployment. It defaults to 1 if not provided.
- **selector**: Ensures that the deployment matches pods with the specified labels, tying the deployment to the correct pods.
- **template**: Defines the pod template that the deployment will use to create pods. It includes metadata (labels) and the specification for the container(s).
- **containers**: Specifies the container configuration, including the name, image, and the port on which the application will listen.

**HPA (Horizontal Pod Autoscaler) Template**

The HPA template configures an autoscaler for our deployment, which automatically adjusts the number of pod replicas based on CPU utilization or other metrics.

```yaml
apiVersion: autoscaling/v2  
kind: HorizontalPodAutoscaler  
metadata:  
  name: {{ include "spring-app.fullname" . }}-hpa  
  namespace: {{ include "spring-app.namespace" . | default "default-namespace" }}
```

- **apiVersion**: Specifies the API version, here it’s `autoscaling/v2`.
- **kind**: Indicates that this resource is a `HorizontalPodAutoscaler`.
- **metadata**: Contains the name and namespace of the HPA, similar to the deployment resource.

```yaml
spec:  
  scaleTargetRef:  
    apiVersion: apps/v1  
    kind: Deployment  
    name: {{ include "spring-app.fullname" . }}  
  minReplicas: {{ include "spring-app.hpa.minReplicas" . | default 1 }}  
  maxReplicas: {{ include "spring-app.hpa.maxReplicas" . | default 5 }}  
  metrics:  
  - type: Resource  
    resource:  
      name: cpu  
      target:  
        type: Utilization  
        averageUtilization: {{ include "spring-app.hpa.targetCPUUtilizationPercentage" . | default 70 }}
```

- **scaleTargetRef**: Defines the target resource that the HPA will manage, which in this case is the deployment we created earlier.
- **minReplicas** and **maxReplicas**: Define the minimum and maximum number of replicas that the HPA can scale to.
- **metrics**: Specifies the metric used to trigger scaling. Here, we are using CPU utilization, and scaling will occur if the CPU usage exceeds 70% on average.

**Service Template**

The Service template defines how our application will be exposed to other services or the outside world. It maps an internal port (the container port) to an external port.

```yaml
apiVersion: v1  
kind: Service  
metadata:  
  name: {{ include "spring-app.fullname" . }}-service  
  namespace: {{ include "spring-app.namespace" . | default "default-namespace" }}
```

- **apiVersion**: Specifies the API version, here it’s `v1` for the Service resource.
- **kind**: Indicates that this resource is a `Service`.
- **metadata**: Contains the name and namespace of the service.

```yaml
spec:  
  selector:  
    app: {{ include "spring-app.name" . }}  
  ports:  
    - protocol: TCP  
      port: {{ include "spring-app.containerPort" . | default 8080 }}  
      targetPort: {{ include "spring-app.containerPort" . | default 8080 }}  
      nodePort: {{ include "spring-app.service.nodePort" . | default 30001 }}  
  type: NodePort
```

- **selector**: Links the service to the pods with the specified label.
- **ports**: Defines the port mappings. The `port` is the external port, and the `targetPort` is the port the container listens on.
- **type**: Specifies the type of service. `NodePort` exposes the service on each node’s IP at a static port.

**Namespace Template**

The Namespace template ensures that our application’s namespace is created.

```yaml
apiVersion: v1  
kind: Namespace  
metadata:  
  name: {{ include "spring-app.namespace" . | default "default-namespace" }}  
  labels:  
    name: {{ include "spring-app.namespace" . | default "default-namespace" }}
```

- **apiVersion**: Specifies the API version, here it’s `v1`.
- **kind**: Indicates that this resource is a `Namespace`.
- **metadata**: Contains the name and labels for the namespace. Labels are optional but useful for organizing and selecting resources within the namespace.
## Step 3: Update `values.yaml`

In the `values.yaml` file, define the default values for our templates. Here's an example:

```yaml
replicaCount: 1  
image:  
  repository: "your-repository/spring-app"  
  tag: "latest"  
namespace: "default-namespace"  
containerPort: 8080  
hpa:  
  minReplicas: 1  
  maxReplicas: 5  
  targetCPUUtilizationPercentage: 70  
service:  
  nodePort: 30001
```

This configuration sets the default values for the deployment, such as the number of replicas, the Docker image to use, the active namespace, the container port, and the HPA settings. These values can be overridden when deploying the Helm chart to different environments.

With these templates in place, our Helm chart is now configured to generate the necessary Kubernetes manifests for deploying our Spring Boot application. These templates will be utilized in the Manifest stage to create YAML files that will be pushed to the FluxCD repository for deployment.
# Deploy Stage

The **Deploy** stage is a crucial part of our GitLab CI pipeline where we generate Kubernetes manifests from Helm charts and push them to the FluxCD repository. This ensures that any changes to our application’s configuration are automatically deployed to the Kubernetes cluster by FluxCD. Here’s how this stage is structured and what each part of the script does.
## Deploy Stage Overview

```yaml
deploy:  
  stage: deploy  
  image:   
    name: alpine/helm  
    entrypoint: [""]  
  variables:  
    NAMESPACE: spring-app-namespace  
  before_script:  
    - git clone --depth 1 https://gitlab-ci-token:${CI_JOB_TOKEN}@<helm-repo-url>.git  
    - git clone --depth 1 https://gitlab-ci-token:${CI_JOB_TOKEN}@<flux-repo-url>.git  
  script:  
    - helm template release helm/spring-app --set namespace=${NAMESPACE}, --set image.repository=${IMAGE_REPO}${IMAGE_NAME}, --set image.tag=${CI_COMMIT_SHORT_SHA}, --set fullnameOverride=${CI_PROJECT_NAME} --output-dir ./  
    - rm -rf flux-cd/apps/spring-app/*  
    - mv ./spring-app/templates/* flux-cd/apps/spring-app/  
    - cd flux-cd  
    - git config --global user.email "ci_cd@example.com"  
    - git config --global user.name "CI/CD Bot"  
    - git add .  
    - git commit -m "Update spring-app commit ${CI_COMMIT_SHORT_SHA}"  
    - git remote set-url origin https://gitlab-ci-token:${ACCESS_TOKEN}@<flux-repo-url>.git  
    - git push origin main
```

1. **Image and Variables**

```yaml
image:   
  name: alpine/helm  
  entrypoint: [""]  
variables:  
  NAMESPACE: spring-app-namespace
```

- **image**: We use an Alpine-based Helm image to execute Helm commands in this stage.
- **NAMESPACE**: This variable defines the Kubernetes namespace where the application will be deployed. It can be customized as needed.

**Before Script**

```yaml
before_script:  
  - git clone --depth 1 https://gitlab-ci-token:${CI_JOB_TOKEN}@<helm-repo-url>.git  
  - git clone --depth 1 https://gitlab-ci-token:${CI_JOB_TOKEN}@<flux-repo-url>.git
```

- **git clone**: These commands clone the Helm and FluxCD repositories into the CI environment. We use the GitLab CI token for authentication, ensuring secure access to these repositories.

**Script**

```yaml
script:  
  - helm template release helm/spring-app --set namespace=${NAMESPACE}, --set image.repository=${IMAGE_REPO}${IMAGE_NAME}, --set image.tag=${CI_COMMIT_SHORT_SHA}, --set fullnameOverride=${CI_PROJECT_NAME} --output-dir ./  
  - rm -rf flux-cd/apps/spring-app/*
```

- **helm template**: This command generates Kubernetes manifests from the Helm chart using the provided variables. The output is directed to the current directory.
- **rm -rf flux-cd/apps/spring-app/**: This command clears the old manifest files in the FluxCD repository to avoid conflicts with the new deployment.

**Moving Files and Committing Changes**

```
- mv ./spring-app/templates/* flux-cd/apps/spring-app/  
- cd flux-cd  
- git config --global user.email "ci_cd@example.com"  
- git config --global user.name "CI/CD Bot"  
- git add .  
- git commit -m "Update spring-app commit ${CI_COMMIT_SHORT_SHA}"  
- git remote set-url origin https://gitlab-ci-token:${ACCESS_TOKEN}@<flux-repo-url>.git  
- git push origin main

```
- **mv**: Moves the generated YAML files into the appropriate directory in the FluxCD repository.
- **git config**: Sets the Git user email and name for committing changes.
- **git add** and **git commit**: Stages and commits the new manifest files.
- **git push**: Pushes the changes to the main branch of the FluxCD repository. FluxCD will automatically detect this push and apply the new manifests to the Kubernetes cluster.

This stage automates the generation and deployment of Kubernetes manifests using Helm, ensuring that any code changes are consistently and automatically reflected in the cluster via FluxCD.

After the deploy job definition runs, the line for the image definition in the `deployment.yaml` file should be updated in the repository where the Kubernetes manifest files of the application are located. This ensures that the application is deployed with the correct Docker image version.

![](nyq0pp9Baljd2jspHV5Ow.webp)
FluxCD Repository Update

## Configuring FluxCD with Kustomization File

In the final stage of our GitOps pipeline, we will set up a FluxCD `Kustomization` file to point to the Kubernetes manifest files generated and pushed in the previous stages. This file will be used by FluxCD to manage the deployment of our Spring Boot application. Since we’ve already configured the Git repository using the `flux bootstrap` command, we only need to create and configure the `Kustomization` file to direct FluxCD to the correct location of our Kubernetes manifests.

The `Kustomization` files should be located inside the directory specified by the `--path=<Flux install location>` parameter that we defined in the `flux bootstrap` command. This ensures that FluxCD knows exactly where to find and apply the Kubernetes manifests for our application.
## Creating the Kustomization File

The `Kustomization` file informs FluxCD where to find the Kubernetes manifest files in our Git repository and how often it should check for updates. Here’s what the file looks like:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1  
kind: Kustomization  
metadata:  
  name: spring-app  
  namespace: flux-system  
spec:  
  interval: 5m0s  
  path: apps/spring-app  
  prune: true  
  sourceRef:  
    kind: GitRepository  
    name: flux-system  
  targetNamespace: spring-app-namespace
```
## Explanation of the Kustomization File

**apiVersion**: Specifies the API version used by FluxCD for `Kustomization` resources. Here, it’s `kustomize.toolkit.fluxcd.io/v1`.

**kind**: Indicates that this resource is a `Kustomization`. This tells FluxCD that it will manage a set of Kubernetes resources based on this configuration.

**metadata**: Contains the metadata for the Kustomization resource.

- **name**: The name of this Kustomization resource, which is `spring-app` in this example.
- **namespace**: The namespace where the FluxCD components are installed, typically defined during the Flux bootstrap process. Here, it’s `flux-system`.

**spec**: Defines the behavior and scope of the Kustomization.

- **interval**: Sets the frequency at which FluxCD checks the Git repository for updates. In this case, it checks every 5 minutes (`5m0s`).
- **path**: Specifies the path within the Git repository where the Kubernetes manifest files are located. For this setup, it’s `apps/spring-app`.
- **prune**: When set to `true`, FluxCD will automatically remove Kubernetes resources that are no longer defined in the manifest files, ensuring that the cluster state is always aligned with the Git repository.
- **sourceRef**: Defines the source Git repository that FluxCD will monitor.
- **kind**: Specifies that the source is a `GitRepository`.
- **name**: The name of the GitRepository resource as configured during the Flux bootstrap. In this case, it’s `flux-system`.
- **targetNamespace**: Specifies the Kubernetes namespace where the application will be deployed. For this setup, the application is deployed into the `spring-app-namespace`.

By configuring this final stage, we ensure that FluxCD continuously monitors and synchronizes the state of our Kubernetes cluster with the manifests defined in our Git repository. With this Kustomization file in place, any changes pushed to the repository will automatically trigger an update in the cluster, ensuring that the application remains in sync with the desired state. This completes the end-to-end GitOps pipeline, automating the deployment process from code changes to live updates in Kubernetes.