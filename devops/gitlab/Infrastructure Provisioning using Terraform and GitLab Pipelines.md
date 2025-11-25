With the introduction of GitLab 13, the GitLab team added the support of Managed Terraform backend, right out of the box for Terraform state management. Terraform is compatible with multiple remote storage backends such as AWS S3, Azure Blob Storage, Google Cloud Storage bucket, and many more; most Enterprises select one of these since they really love to move closer to their platform Cloud Service Provider. In some cases, Organizations chose the alternatives. GitLab being an all-in-one DevOps Platform, it is a good approach to include everything in the same place due to ease of maintenance and support. So in this short article, we will create a Terraform pipeline with GitLab Managed Terraform backend as state storage embedding a conditional approval mechanism before the deployment phase.

Let's see how it is done with an example of provisioning an S3 bucket using GitLab pipelines

## Pre-requisites

1. Gitlab Account
2. AWS Account and Credentials

## Setup

1. **Code**

We are trying out the provisioning with an S3 bucket and the whole codebase is located [here](https://gitlab.com/renjithvr11/terraform-gitlab-pipelines). One of the key things that we have to note in the code is the `http backend` provider. If we wanted to use a GitLab Managed Terraform Remote backend, we have to mention the below code block in the backend file
```c
terraform {  
  backend "http" {  
  }  
}
```

There are a few variables that we can mention in the `backend` block, which can be found in the Terraform [doc](https://www.terraform.io/language/settings/backends/http#http). For the implementation purpose, I have inserted the required environment variables in the GitLab pipeline itself.

**2. Pipeline Structure**

Basically, the Terraform pipeline looks similar to the below and it has 5 different phases

![](bvvBjDpHvjv7NoLMMurNUA.webp)

1. Prepare — The providers will be downloaded from the Hashicorp registry and a backend will be initialized
2. Validate — It runs formatting of TF and runs a validate
3. Build — Executes the TF scripts and creates a plan
4. Deploy — The created plan in the previous step will be executed. There is an approval mechanism associated with this to have two eyes check on the plan.
5. Destroy — This is a manual trigger, which will destroy the infrastructure created.

Below is the YAML pipeline used for the provisioning

```yaml
image: registry.gitlab.com/gitlab-org/terraform-images/stable:latest
variables:
  TF_ROOT: ${CI_PROJECT_DIR}/terraform
  TF_ADDRESS: ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${CI_PROJECT_NAME}
  TF_HTTP_LOCK_METHOD: "POST"
  TF_HTTP_UNLOCK_METHOD: "DELETE"

cache:
  key: "${TF_ROOT}"
  paths:
    - ${TF_ROOT}/.terraform/

before_script:
  - cd ${TF_ROOT}

stages:
  - prepare
  - validate
  - build
  - deploy
  - destroy

init:
  stage: prepare
  script:
    - gitlab-terraform init

validate:
  stage: validate
  script:
    - gitlab-terraform init
    - gitlab-terraform validate

plan:
  stage: build
  script:
    - gitlab-terraform plan
    - gitlab-terraform plan-json
  artifacts:
    name: plan
    paths:
      - ${TF_ROOT}/plan.cache
    reports:
      terraform: ${TF_ROOT}/plan.json

# Separate apply job for manual launching Terraform 
apply:
  stage: deploy
  environment:
    name: production
  script:
    - gitlab-terraform apply
  dependencies:
    - plan
  when: manual
  only:
    - main

destroy:
  stage: destroy
  script:
    - gitlab-terraform destroy
  dependencies:
    - apply
  when: manual 
  only:
    - main
```

lso, note that the secret variables for AWS and Remote backend are mapped in the CI/CD variable section

![](Syf2udVBGcBMoo4f9gVsQA.webp)

**3. Pipeline in Action**

Once you trigger your pipeline, terraform backend on GitLab would be created and initialized on the fly.

![](QagWAxwZmk3qYIKUdsWUFQ.webp)

The newly created Terraform backend can be found in GitLab at the below location. Also, it is possible to download the state file to your local machine as well by downloading it.

![](TI1XrUffrhVrSCFAd6aiVg.webp)

The build stage generates the plan file during the execution and it will also output it to the local runner directory. If everything looks good, then it is good for deployment. The deploy stage is set with an option for the manual trigger, as the reviewer needs to verify the plan, before an apply. Terraform is quite powerful, one wrong `apply` can be catastrophic;).

If you want to tear down the newly created cloud resources, go ahead and click destroy. That's all folks, for now. Hope you find the article useful. Thanks for reading.

In case of any queries, please feel to connect me via the below links

- [LinkedIn](https://www.linkedin.com/in/rvr88/)
- [Twitter](https://twitter.com/mysticrenji)
- [Medium](https://renjithvr11.medium.com/)

## References

- [https://gitlab.com/renjithvr11/terraform-gitlab-pipelines](https://gitlab.com/renjithvr11/terraform-gitlab-pipelines)
- [https://docs.gitlab.com/ee/user/infrastructure/iac/terraform_state.html](https://docs.gitlab.com/ee/user/infrastructure/iac/terraform_state.html)
- [https://www.terraform.io/language/settings/backends/http#http](https://www.terraform.io/language/settings/backends/http#http)