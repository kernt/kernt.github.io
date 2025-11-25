# Overview — What Is Atlantis?

[Atlantis](https://www.runatlantis.io/) is an application for automating Terraform pull requests. It is deployed as a standalone application in your infrastructure.

Atlantis listens for GitHub webhooks from Terraform pull requests. It then runs `terraform plan` and comments with the output back on the pull request so that engineers can review and discuss changes from a central place.

When you want to apply, comment `atlantis apply` on the pull request and Atlantis will run `terraform apply` and comment back with the output. A successful `apply` will automatically merge the pull request.

This means that engineers no longer need to run terraform on their machines locally in order for them to apply changes to their infrastructure. The concept should, in theory, reduce config drift from unapplied changes.

In this article, I’m going to demonstrate how I implemented the Atlantis tool to enable GitOps for our Terragrunt/Terraform code. For the purposes of this article, I need to provide some background into our existing Terragrunt/Terraform repo.

![](R7OqwRktlf1ByGb9VhpaYg.webp)

# Background — Before Atlantis

[Terragrunt](https://terragrunt.gruntwork.io/) is a thin wrapper that provides extra tools for keeping your configurations DRY, working with multiple Terraform modules, and managing remote state. My Terragrunt code lives in a repo we’ll call `infrastructure`. This terragrunt code calls Terraform modules that live in another remote repo that we’ll call `terraform` for the purposes of this article.

We have a multi-account AWS infrastructure. The accounts we have are `nonprod`, `prod` and `mgmt`.

Therefore, my `infrastructure` repo structure looks something like the below:

```sh
aws/accounts/  
├─ nonprod/  
│  ├─ ap-northeast-1/  
│  │  ├─ eks/  
│  │  │  ├─ terragrunt.hcl  
│  │  ├─ .envrc  
│  ├─ eu-west-1/  
│  ├─ .envrc  
├─ prod/  
│  ├─ ap-northeast-1/  
│  │  ├─ ec2/  
│  │  │  ├─ terragrunt.hcl  
│  │  ├─ eks/  
│  │  │  ├─ terragrunt.hcl  
│  │  ├─ .envrc  
│  ├─ .envrc  
├─ mgmt/  
│  ├─ ap-northeast-1/  
│  │  ├─ eks/  
│  │  │  ├─ terragrunt.hcl  
│  │  ├─ .envrc  
│  ├─ eu-west-1/  
│  │  ├─ s3/  
│  │  │  ├─ terragrunt.hcl  
│  │  ├─ ecr/  
│  │  │  ├─ terragrunt.hcl  
│  │  ├─ .envrc  
│  ├─ .envrc
```

In each account directory above, we also have an `.envrc` file which allows `direnv` to set environmental variables, based on the directory you’re in. In my case, `AWS_PROFILE` would be set to either `prod`, `nonprod` or `mgmt`. This means engineers simply have to run terraform in whichever directory they want, and the correct role from each account would be assumed automatically according to the named profiles in their local `~/.aws/config` file.

# Objectives

- Deploy Atlantis to our Kubernetes Cluster.  
    – The deployment will use our own custom image.  
    – We’ll pass in the server side config via helm values.  
    – Create a Kubernetes Service Account to pass in IAM credentials to the Atlantis Pod.
- Have Atlantis run `terragrunt plan` on all PR’s as soon as they are created, with the output posted as a comment.  
    – Dynamically create the second config file known as the Repo Level Config with the `terragrunt-atlantis-config` tool.  
    – Give Atlantis permissions to comment on our GitHub repo.

# Deploying Atlantis to Kubernetes

I deployed Atlantis to Kubernetes using the official Helm Chart that you can find here: [https://github.com/runatlantis/helm-charts](https://github.com/runatlantis/helm-charts)

I will not cover GitHub permissions in this article, you can read about those here: [https://www.runatlantis.io/docs/access-credentials.html#generating-an-access-token](https://www.runatlantis.io/docs/access-credentials.html#generating-an-access-token) Note, you will be able to use `values.loadEnvFromSecrets` in the Helm Chart to specify Kubernetes Secrets that should be loaded as environmental variables into your Pod.

## IAM Permissions

As stated previously, I have 3 AWS accounts. The IAM role assigned to the Atlantis Pod comes from a Kubernetes Service Account. The Service Account is annotated with the name of the IAM role we created.

```json
serviceAccount:  
  create: true  
  annotations:  
    eks.amazonaws.com/role-arn: "arn:aws:iam::(( account )):role/eks-(( environment ))-(( region ))-atlantis"
```

For more information on creating a role that you can use with a Service Account, see the documentation here: [https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)

The purpose of this role is to [cross-account assume](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html) a new `terraform` role that we will create in each of the AWS accounts described earlier. Therefore the role must have `sts:AssumeRole` permissions, and the Terraform roles in each account must have a Trust Relationship or assume role policy that allows the Atlantis Pods role to assume it.

![](5sCLRdaoF_6wrPtcEA36ng.webp)

Atlantis uses a role in the mgmt account, that can assume the Terraform role in each of the mgmt/prod/nonprod accounts. The Terraform role is used by terraform to create infrastructure.

This `terraform` role is the role which we’ll place in the provider blocks of the Terragrunt code. To do this, without hardcoding your roles in the terraform code, you can use a `[generate](https://terragrunt.gruntwork.io/docs/reference/config-blocks-and-attributes/#generate)` terragrunt block in a parent `terragrunt.hcl` file as outlined here:

```sh
_generate_ "provider" {  
  path      = "provider.tf"  
  if_exists = "skip"  contents = <<EOF  
provider "aws" {  
  assume_role {  
    role_arn = "arn:aws:iam::${get_env("TF_VAR_account_id", "")}:role/terraform"  
  }  
  default_tags {  
    tags = local.common_tags  
  }  
}  
EOF  
}
```

## Dockerfile

I pointed the Helm chart to my own custom Atlantis Docker image, based on my own Dockerfile that pulls from the official Atlantis image and adds the tools below.

- yq
- direnv
- terragrunt
- terragrunt-atlantis-config
- jq
- aws-cli

## Atlantis Configuration — Repo Level Atlantis.yaml Config

There are two configs that are required for Atlantis. The first configuration is called a Repo Level `atlantis.yaml` config: [https://www.runatlantis.io/docs/repo-level-atlantis-yaml.html](https://www.runatlantis.io/docs/repo-level-atlantis-yaml.html). It is responsible for enabling the self-explantory feature known as `autoplan`. This configuration should exist in the root of your Terragrunt repo. A sample of what this file should look like is below:

```sh
automerge: true
parallel_apply: true  
parallel_plan: true  
projects:  
  - autoplan:  
      enabled: true  
      when_modified:  
        - '*.hcl'  
        - '*.tf*'  
        - 'files'  
    dir: aws/accounts/nonprod/eu-west-1/ec2  
  - autoplan:  
      enabled: true  
      when_modified:  
        - '*.hcl'  
        - '*.tf*'  
        - 'files'  
    dir: aws/accounts/nonprod/eu-west-1/eks
```

The config enables some global features such as `automerge` (automatically merges after a successful `atlantis apply`) and enables `autoplan` on the directories specified within it.

The important thing to note here is that all of your directories that contain a `terragrunt.hcl` file (the tens or hundreds of them) will need to be listed in this config file under the `projects:` key. Fortunately, a tool known as `[terragrunt-atlantis-config](https://github.com/transcend-io/terragrunt-atlantis-config)` can generate this configuration for us before each pipeline run.

## Atlantis Configuration — Server Side Config

The second configuration you’ll need is known as the Server Side Config: [https://www.runatlantis.io/docs/server-side-repo-config.html](https://www.runatlantis.io/docs/server-side-repo-config.html)

This config is where we’ll define what actually happens to the Terragrunt code we push to Github. i.e the plan/apply commands, and any steps we want to run pre, or post plan/apply. You can use the `values.repoConfig` to parse in your Server Side Config.

## Pre-workflow

In our `pre_workflow`, we generate the repo level `atlantis.yaml`config described in the previous section, using `[terragrunt-atlantis-config](https://github.com/transcend-io/terragrunt-atlantis-config)`, as shown below. We then use `yq` to manually add a further folder to ensure that any files within that folder, such as IAM policy template files, are also monitored for modifcations.


```Python
pre_workflow_hooks:  
  - run: terragrunt-atlantis-config generate --automerge --ignore-dependency-blocks --ignore-parent-terragrunt true --filter aws/accounts/ --autoplan --output atlantis.yaml  
  - run: yq e -i '.projects[].autoplan.when_modified += "files"'   
atlantis.yaml
```

The main `workflow` will consist of the `plan` and `apply` steps which utilises the built in Environment Variables provided by Atlantis.

```
workflows:  
  terragrunt:  
    plan:  
      steps:  
        - env:  
            name: TERRAGRUNT_TFPATH  
            command: 'echo "terraform${ATLANTIS_TERRAFORM_VERSION}"'  
        - run: direnv-allow-all  
        - run: direnv exec . terragrunt plan -input=false -out=$PLANFILE  
        - run: direnv exec . terragrunt show -json $PLANFILE > $SHOWFILE    apply:  
      steps:  
        - env:  
            name: TERRAGRUNT_TFPATH  
            command: 'echo "terraform${ATLANTIS_TERRAFORM_VERSION}"'  
        - run: direnv-allow-all  
        - run: direnv exec . terragrunt apply -input=false $PLANFILE
```

`direnv-allow-all` is a custom bash script that runs `direnv allow` in all of the directories with an `.envrc` file.

The above workflow should be enough to have Atlantis automatically plan PR’s and apply when you comment `atlantis apply`.

## Conclusion

If you’ve followed the steps correctly, you should start seeing comments on your PR’s from the GitHub user you allocated to Atlantis:

![](CjFQGSmTLhyd6eeA.webp)

Note:

If you are using a `provider` block that references a role, this role will be assumed using whichever role the `atlantis` pod is using.

# Further Reading

Stay tuned for a follow up article where we deploy `infracost` to get a complete cost breakdown on every one of your terraform PR’s.

![](CJMo520b4HMBrqcw.webp)