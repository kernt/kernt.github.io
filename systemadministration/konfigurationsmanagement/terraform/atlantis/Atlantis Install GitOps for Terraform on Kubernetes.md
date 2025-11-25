---
tags:
  - terraform
  - kubernetes
  - gitops
---
# Introduction

Infrastructure as Code (IaC) has become a cornerstone of modern software development, allowing teams to manage and provision infrastructure through code. [Terraform](https://terraform.com/?ref=8grams.tech), a popular IaC tool developed by [HashiCorp](https://hashicorp.com/?ref=8grams.tech), enables users to define infrastructure configurations in a declarative manner. However, as organizations scale, manually executing Terraform commands becomes cumbersome, error-prone, and inefficient. This is where [**_Atlantis_**](https://www.runatlantis.io/?ref=8grams.tech) comes into play. Atlantis integrates GitOps principles with Terraform, automating workflows and enhancing collaboration, consistency, and security.

![](zXtbQnWa-9unm2ew.webp)

# History and Background

## **Origins of Terraform**

Terraform was introduced by [HashiCorp](https://www.terraform.io/?ref=8grams.tech) to simplify the process of defining and provisioning infrastructure. It uses a high-level configuration language (HCL) to describe the desired state of infrastructure, which Terraform then uses to create, update, and manage resources across various providers. Despite its powerful capabilities, Terraform’s manual execution presents several challenges: _Human Error, Inconsistent Practices, Security Risks._ Handling sensitive data manually increases the risk of exposing critical information.

## **Inception of Atlantis**

Recognizing these challenges, Atlantis was created to automate Terraform workflows and integrate them with version control systems (VCS). Atlantis was designed to provide a consistent, automated approach to managing infrastructure changes, reducing human error, and enhancing collaboration and security.

## **Why Atlantis Was Created**

Atlantis was created to address the pain points associated with manual Terraform execution by leveraging GitOps principles. GitOps is a methodology that uses Git as the single source of truth for infrastructure definitions and deployments, ensuring that changes are auditable and reproducible.

Common Pain Points in Terraform Management:

1. **Manual Execution Errors**: Manually running Terraform commands increases the likelihood of errors, leading to inconsistencies and potential outages.
2. **Lack of Collaboration and Consistency**: Different team members may follow varying practices, causing configuration drift and making it difficult to maintain a unified infrastructure state.
3. **Security Concerns**: Handling sensitive data manually exposes it to potential security risks.

## **Need for Automation**

To mitigate these issues, there was a need for a tool that could automate Terraform commands, integrate with VCS, and provide a standardized workflow. Atlantis was created to fulfill this need, offering a streamlined, automated approach to managing Terraform workflows.

# Key Features of Atlantis

## **Automated Workflows**

Atlantis automates the Terraform plan and apply steps by integrating with your VCS. When a pull request is opened, modified, or merged, Atlantis automatically runs the appropriate Terraform commands, ensuring that changes are applied consistently and reducing the need for manual intervention.

4. **Terraform Plan**: Atlantis runs `terraform plan` when a pull request is created or updated, providing a detailed preview of the changes that will be made. This output is posted as a comment on the pull request, allowing team members to review and discuss the proposed changes.
5. **Terraform Apply**: Once the pull request is approved and merged, Atlantis runs `terraform apply` to apply the changes to the infrastructure, ensuring a consistent and automated deployment process.

## Pull Request Automation

Atlantis facilitates better collaboration through pull request automation. When a pull request is created, Atlantis automatically runs `terraform plan` and posts the output as a comment. This allows team members to review the proposed changes and discuss potential issues before they are applied. Upon approval, Atlantis can automatically apply the changes when the pull request is merged.

![](EejAulW7oDOGcGwj.webp)

## Security and Compliance

Atlantis enhances security by managing Terraform execution without exposing sensitive information. It uses role-based access control (RBAC) to ensure that only authorized users can apply changes. Additionally, audit logs provide a detailed history of changes, helping teams maintain compliance with organizational policies.

- **Role-Based Access Control (RBAC)**: Atlantis restricts access to sensitive operations, ensuring that only authorized users can make changes to the infrastructure.
- **Audit Logs**: Detailed logs of all actions performed by Atlantis provide a comprehensive audit trail, enhancing accountability and compliance.

## Scalability and Flexibility

Atlantis supports multiple Terraform projects and environments, making it scalable for organizations of any size. It allows customizable workflows and configurations, enabling teams to tailor Atlantis to their specific needs.

- **Multiple Projects and Environments**: Atlantis can manage multiple Terraform projects and environments, ensuring that infrastructure changes are isolated and managed independently.
- **Customizable Workflows**: Teams can customize Atlantis workflows to suit their specific requirements, providing flexibility and adaptability.

## User-Friendly Interface

With its intuitive user interface, Atlantis simplifies the setup and management of Terraform workflows. Users can easily monitor and manage infrastructure changes, improving overall efficiency. Atlantis provides straightforward setup instructions, enabling teams to get up and running quickly. The user-friendly interface makes it easy to manage and monitor Terraform workflows, enhancing productivity and efficiency.

# Benefits of Using Atlantis

## Automation and Consistency

Atlantis ensures that infrastructure changes are applied consistently by automating Terraform commands. This reduces human error and enhances the reliability of infrastructure deployments. **_Automation_** ensures that infrastructure changes are applied consistently, reducing the risk of errors and downtime. By automating repetitive tasks, Atlantis minimizes the potential for human error, improving overall reliability.

## Improved Collaboration

By integrating with VCS, Atlantis facilitates better collaboration among team members. Pull request workflows enable code reviews and discussions, ensuring that changes are thoroughly vetted before being applied. Pull request workflows enable team members to review and discuss changes, improving collaboration and quality. Also, Integration with VCS provides enhanced visibility into infrastructure changes, making it easier to track and manage modifications.

## Enhanced Security

Managing Terraform execution through Atlantis reduces the risk of exposing sensitive information. Role-based access control and audit logs enhance security and compliance, providing peace of mind for organizations.

- **Secure Execution**: Atlantis handles sensitive operations securely, reducing the risk of data exposure.
- **Compliance and Auditing**: Detailed audit logs ensure compliance with organizational policies and provide a comprehensive record of changes.

## Scalability and Efficiency

Atlantis can handle multiple Terraform projects and environments, making it suitable for organizations of any size. Automation reduces the time and effort required for infrastructure management, increasing overall efficiency. Support for multiple projects and environments ensures that Atlantis can scale with your organization. Automation reduces the time and effort required for managing infrastructure, freeing up resources for other tasks.

## Ease of Use

Atlantis offers a user-friendly interface and easy setup, making it accessible for teams with varying levels of expertise. Its intuitive design simplifies the management of Terraform workflows, allowing teams to focus on delivering value. The intuitive interface makes it easy to manage and monitor Terraform workflows, improving overall user experience. Easy setup instructions enable teams to get started with Atlantis quickly, reducing the time to value.

# Architecture of Atlantis

Atlantis is designed to integrate seamlessly with VCS, automating Terraform workflows. Its architecture includes several key components that work together to provide a robust and scalable solution for managing infrastructure changes.

![](kIc-RzAhjgSMBJ77.webp)

Components and Their Roles:

1. **Server**: The Atlantis server is the core component that handles all Terraform operations. It listens for webhook events from the VCS, runs Terraform commands, and posts the results back to the pull request.
2. **VCS Integrations**: Atlantis integrates with various VCS platforms (e.g., GitHub, GitLab, Bitbucket) to receive webhook events and post comments on pull requests.
3. **Webhooks and Event Handling**: Atlantis relies on webhooks to receive notifications about pull request events. When a pull request is created or updated, Atlantis triggers the appropriate Terraform commands and posts the results back to the pull request.

## Workflow Processing and Automation

![](D0wNTNNf8vuS_e8l.webp)

Atlantis automates the Terraform workflow by processing pull request events and running the necessary Terraform commands. The typical workflow includes:

1. **Pull Request Created/Updated**: When a pull request is created or updated, a webhook event is sent to the Atlantis server.
2. **Terraform Plan**: Atlantis runs `terraform plan` and posts the output as a comment on the pull request, allowing team members to review the proposed changes.
3. **Pull Request Approved**: Once the pull request is approved and merged, Atlantis runs `terraform apply` to apply the changes to the infrastructure.
4. **Results Posted**: The results of the `terraform apply` command are posted back to the pull request, providing a complete record of the changes.

# Install Atlantis on Kubernetes

Atlantis can be easily installed in Kubernetes using the standard Deployment method. Before installing Atlantis on Kubernetes, certain prerequisites must be met. In this example, we will install Atlantis to run Terraform against Google Cloud Platform (GCP). The process is also applicable to other public cloud providers like AWS. Additionally, we will use GitHub to host our Terraform code.

First, we need to create a Google service account with the proper permissions to manage GCP resources. In this example, we will use the “Owner” role, although the “Editor” role should suffice in most cases.

```c
# in order to be able to run Atlantis: https://www.runatlantis.io/
# https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/google_service_account
resource "google_service_account" "atlantis-sa" {
  account_id = "atlantis-sa"
  project = "my-project"
}

# https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/google_project_iam
resource "google_project_iam_member" "atlantis-im" {
  project = "my-project"
  role    = "roles/owner"
  member  = "serviceAccount:${google_service_account.atlantis-sa.email}"
}
```

Download the Service Account key from the Google Cloud Console by navigating to **_IAM & Admin > Service Accounts_**. Choose the Service Account you created and download its key, then copy the content.

Next, prepare the following environment variables:

- `ATLANTIS_GH_TOKEN` and `ATLANTIS_GH_USER`: Obtain these from [GitHub settings tokens](https://github.com/settings/tokens).
- `ATLANTIS_GH_WEBHOOK_SECRET`: Create a long and random string to protect your Atlantis webhook endpoint.

Prepare a Persistent Volume Claim (PVC) for Atlantis. Atlantis requires a small amount of storage to save Terraform state files.

```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: atlantis-kubeconf-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Gi

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: atlantis-dir-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Gi
```

And then let’s create the Deployment

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: atlantis
  labels:
    app: atlantis
spec:
  selector:
    matchLabels:
      app: atlantis
  template:
    metadata:
      labels:
        app: atlantis
    spec:
      containers:
        - name: atlantis
          image: glendmaatita/atlantis:latest
          imagePullPolicy: Always
          envFrom:
            - configMapRef:
                name: atlantis-config
          volumeMounts:
          - name: google-json-volume
            mountPath: /app/google.json
            subPath: google.json
          - name: atlantis-dir
            mountPath: /app/atlantis
          securityContext:
            runAsUser: 0
      volumes:
      - name: google-json-volume
        configMap:
          name: atlantis-gcp-key
      - name: atlantis-dir
        persistentVolumeClaim:
          claimName: atlantis-dir-pvc
--- 

apiVersion: v1
kind: ConfigMap
metadata:
  name: atlantis-config
data:
  ATLANTIS_ALLOW_COMMANDS: "version,plan,apply,unlock"
  ATLANTIS_ALLOW_FORK_PRS: "true"
  ATLANTIS_API_SECRET: supersecretrandomstring
  ATLANTIS_ATLANTIS_URL: "https://atlantis.example.com"
  ATLANTIS_GH_TOKEN: fromyourgithubtoken
  ATLANTIS_GH_USER: fromyourgithubtokenname
  ATLANTIS_GH_WEBHOOK_SECRET: supersecretrandomstring
  ATLANTIS_REDIS_HOST: "10.0.0.1"
  ATLANTIS_REPO_ALLOWLIST: "github.com/your-organization/your-terraform-repository"
  ATLANTIS_WEB_BASIC_AUTH: "true"
  ATLANTIS_WEB_USERNAME: admin
  ATLANTIS_WEB_PASSWORD: supersecretrandomstring
  ATLANTIS_PORT: "4141"
  ATLANTIS_DATA_DIR: "/app/atlantis"
  GOOGLE_APPLICATION_CREDENTIALS: "/app/google.json"

---

apiVersion: v1
kind: ConfigMap
metadata:
  name: atlantis-gcp-key
data:
  google.json: |
< Content of Service Account key >
  
---

apiVersion: v1
kind: Service
metadata:
  name: atlantis
  labels:
    app: atlantis
spec:
  selector:
    app: atlantis
  ports:
    - protocol: TCP
      port: 80
      targetPort: 4141

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: atlantis
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-issuer
spec:
  ingressClassName: nginx
  rules:
    - host: atlantis.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service: 
                name: atlantis
                port: 
                  number: 80
  tls:
    - hosts:
        - atlantis.example.com
      secretName: tls-ops-atlantis
```

lease make adjustments, especially in the ConfigMap section where we define some configurations. Now, we can apply those manifests.

```sh
kubectl -n atlantis volume.yaml  
kubectl -n atlantis deployment.yaml
```

If everything is set up, try updating your Terraform code and creating a Pull Request. You will see the magic happen.
