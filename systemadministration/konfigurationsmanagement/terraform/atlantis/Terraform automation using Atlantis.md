As a DevOps Engineer you may need to deal with IAAC deployments using terraform. Terraform is one of the popular open-source IAAC platform. Running terraform locally can cause N number of issues and there are tons of ways to avoid.

Today I am going to introduce a tool called Atlantis, which is very easy way to manage Terraform deployments. With my experiences, I prefer to use Atlantis to automate terraform pipelines.

# What is Atlantis

Atlantis is an open-source DevOps tool designed written in Go language. This is to automate and streamline Terraform workflows through pull request (PR). Atlantis enables teams to manage Infrastructure as Code (IaC) collaboratively by triggering `terraform plan` and `terraform apply` commands directly from PR comments, ensuring visibility, security, and consistency.

# Atlantis Workflow

# Atlantis Workflow

![](LDNO5H_Ov5DIvMcy.webp)

Here the Atlantis workflow in very simple level.

**Webhook**: Configured at SCM end to trigger each and every event at repository side.
**Atlantis**: Push comments to SCM tool with outpu of terraform action.

# Key Features

These are the key features, why this is **go to tool** for Terraform Automation.

- Automated Terraform Workflows: Executes `terraform plan` on PR creation/update and `terraform apply` after approval.
- SCM Integration: Compatible with all SCM tools via webhook integration.
- State Locking: Prevents concurrent modifications to avoid conflicts (Eg: No need of having DynmoDB locks when storing state files in S3).
- Logging: Tracks changes through PR comments and Commits history.
- RBAC Support: Restricts command execution to authorized.
- Custom Workflows: Define custom workflows and setting for different folders in same repository.

Using this kind of open-source tool to connect your IAAC code and Cloud service provider, enables centralized platform to entire team members. Terraform outputs are visible in comments and records will not be deleted. This avoid human errors by executing manual executions. Terraform project and workflow mapping secure multiple plans/applies. atlantis.yaml is the manifest of the entire repository and it manages the alantis behaviour. Only repository read permission enough to connects atlantis.

# Atlantis Actions

![](uJcM_-Kh77_m_sHw.webp)

This is how Atlantis works with Webhook events sends by GIT/Bitbucket, and how Atlantis replies to GIT/Bitbucket with posting comments and AutoMerge events.

# How this works

To configure atlantis, you need to provide SCM URL, Repository, Token, Webhook. These details helps atlantis to connect with SCM tool to listen for every PR events/comments. Also this helps atlantis to push comments and auto merge to SCM.

State lock management isn’t neccessary, since once plan runs for a specific project, its state lock stores in Atlantis, which deny running any PRs agains that project. But allow to run PRs for any other projects. Once PR approved and apply runs, code will merge and users are free to run PRs for the project again.

![](us7fAq4XT6VqWPkECN2UPg.webp)

# Atlantis Deployment

Atlantis can be deployed as a systemd package, as a docker contianer or as a helm chart.

Refer official page for all deployment methods [https://www.runatlantis.io/docs/installation-guide.html](https://www.runatlantis.io/docs/installation-guide.html)

Here I have explained how to configure using **Helm chart**, since it is the most advanced scalable way to avoid running with dependencies.

**Refer official Helm Chart** [https://github.com/runatlantis/helm-charts](https://github.com/runatlantis/helm-charts)

To Test this, you need to have SCM tool (Git, Bitbucket, GitLab…). But Local testing I am using [**Gitea**](https://about.gitea.com/products/gitea/) open-source self-host Git server. (Main reason to use Gitea is to avoid route traffic over internet and not need of open ports from your local machine).

Steps:

1. Add chart and download values.yaml to local machine

```sh
helm repo add runatlantis https://runatlantis.github.io/helm-charts  
helm inspect values runatlantis/atlantis > values.yaml
```

1. Update values.yaml as below.

For test purpose I am allowing all the repositories under my gitea account to access by Atlantis. This is read-only access, Atlantis only can read (no write/update/delete actions perform by Atlantis)

```
# Your Gitea URL and Repo  
orgAllowlist: "gitea.localhost/*"
```

```
# To Authneticate and authorize to listen and comment to Gitea PRs  
gitea:  
  user: atlantis-bot  
  token: <CREATE NEW TOKEN in the Gitea and copy>  
  secret: <create your own secret to create webhook in Gitea repo>  
  baseUrl: "http://localhost:3001"# Atlantis frontend  
atlantisUrl: "http://localhost:4141"# Incoming requests forward to Pod  
service:  
  type: NodePort  
  port: 4141  
  nodePort: 4141# Disable due to local setup  
ingress:  
  enabled: false# Repository settings to atlantis  
repoConfig: |  
  ---  
  repos:  
  - id: /.*/  
    allow_custom_workflows: true  
    allowed_overrides: [workflow]  
    apply_requirements: [approved]  
  workflows:  
    default:  
      plan:  
        steps: [init, plan]  
      apply:  
        steps: [apply]# resource limitations  
resources:  
  requests:  
    memory: 1G  
    cpu: 100m  
  limits:  
    memory: 1G  
    cpu: 200m# PVC to store states  
volumeClaim:  
  enabled: true  
  storageClassName: standard  
  size: 1Gi
```

You need to create token at Gitea and add into Atalntis values.yaml to authenticate. Next you need to create secret key to configure with Webhook at Gitea end.

Above I have enabled all repos folders as **id: /.*/**

You can add a multiple repo folder as a lists under **repos:**

Workflows runs init, plan and apply accordingly.

Other resources and values can change as per your requirement.  
* All above settings are only for local setup. For production setup, you need to change **service, ingress** properties.

Above are the simplest configurations, you can add many more properties as per the default [values.yaml](https://github.com/runatlantis/helm-charts/blob/main/charts/atlantis/values.yaml).

1. Apply updated values.yaml and forward local 4141 to pod service 4141.

```sh
helm upgrade atlantis runatlantis/atlantis -f values.yaml  
kubectl port-forward pod/atlantis-0 4141:4141
```

1. Access the URL localhost:4141 to Atlantis frontend. Atlantis frontend is very basic page (we do not need it at all).

2. Create **atlantis.yaml** file in your repository.

```yaml
version: 3  
automerge: true  
  
################ PROJECTS ###################  
projects:  
- name: project_1  
  terraform_version: terraform1.10.5  
  dir: project_1/  
  workflow: project_1  
  
################ WORKFLOWS ###################  
workflows:  
  project_1:  
    plan:  
      steps:  
      - run: terraform1.10.5 init  
      - plan:  
           extra_args: <TFVARS file path>
```

You can add many projects and workflows according to your requirement.

1. Update terraform scripts and create PR

Here the coolest feature :) You could see atlantis starts listening to webhook and generate terraform plan output to the PR as a comment. This goes with terraform apply too.

If you make any changes to PR again, you might need to run **atlantis commands** to notify Atlantis to execute Terraform at Atlantis end.

```sh
atlantis plan  
atlantis apply
```

1. Last step

Once PR approved and ran **atlantis apply ,** after successful apply PR will automerge to master. (this can disable in atlantis.yaml)

I hope you have got some idea about this tool now. Give a try and let me know how you feel about this.