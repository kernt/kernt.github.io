n this article, I will show how to store Terraform state in GitLab while deploying Terraform resources with GitLab CI/CD pipeline.

> First, add HTTP part to your Terraform block

```c
terraform {  
  backend "http" {}  
}
```
> Second, set two global environment variables in GitLab. These will be used for storing Terraform state inside your GitLab

```sh
GITLAB_USER_NAME=<your-gitlab-username>  
GITLAB_ACCESS_TOKEN=<gitlab-personal-access-token>
```

Do not forget to select **_api_** scope while creating the access token

not forget to select **_api_** scope while creating the access token

![](YEn7c3fHDS-040gr5QueMw.webp)

> Then configure your pipeline accordingly

```yaml
image: registry.gitlab.com/gitlab-org/terraform-images/stable:latest

cache:
  key: "${TF_ROOT}"
  paths:
    - ${TF_ROOT}/.terraform

before_script:
  - cd ${TF_ROOT}

variables:
  GIT_DEPTH: 30
  TF_STATE_NAME: default
  TFVARS_FILE: terraform.tfvars
  TF_ROOT: ${CI_PROJECT_DIR}
  TF_HTTP_RETRY_WAIT_MIN: 5
  TF_HTTP_LOCK_METHOD: POST
  TF_HTTP_UNLOCK_METHOD: DELETE
  TF_HTTP_USERNAME: ${GITLAB_USER_NAME}
  TF_HTTP_PASSWORD: ${GITLAB_ACCESS_TOKEN}

stages:
  - init
  - validate
  - check-format
  - plan
  - apply

.plan:
  rules:
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"'
      changes:
        - *.tf
      when: always  
    - if: '$CI_COMMIT_BRANCH == "master"'
      changes:
        - *.tf
      when: always

.check:  
  rules:
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"'
      changes:
        - *.tf
      when: always

.deploy:
  rules:
    - if: '$CI_COMMIT_BRANCH == "master"'
      changes:
        - *.tf
      when: always

init:
  extends: .check
  stage: init
  retry: 2
  tags:
    - local
  script:
    - |
      gitlab-terraform init -reconfigure \
      -backend-config="address=https://${CI_SERVER_HOST}/api/v4/projects/${CI_PROJECT_ID}/terraform/state/${TF_STATE_NAME}" \
      -backend-config="lock_address=https://${CI_SERVER_HOST}/api/v4/projects/${CI_PROJECT_ID}/terraform/state/${TF_STATE_NAME}/lock" \
      -backend-config="unlock_address=https://${CI_SERVER_HOST}/api/v4/projects/${CI_PROJECT_ID}/terraform/state/${TF_STATE_NAME}/lock" \
      -backend-config="username=${TF_HTTP_USERNAME}" \
      -backend-config="password=${TF_HTTP_PASSWORD}" \
      -backend-config="lock_method=${TF_HTTP_LOCK_METHOD}" \
      -backend-config="unlock_method=${TF_HTTP_UNLOCK_METHOD}" \
      -backend-config="retry_wait_min=${TF_HTTP_RETRY_WAIT_MIN}"
validate:
  extends: .check
  stage: validate
  retry: 2
  tags:
    - local
  script:
    - gitlab-terraform validate

check-format:
  extends: .check
  stage: check-format
  retry: 2
  tags:
    - local
  script:
    - gitlab-terraform fmt -check -recursive -diff

plan:
  extends: .plan
  stage: plan
  retry: 2
  tags:
    - local
  script:
    - gitlab-terraform plan -var-file=${TFVARS_FILE} -parallelism=10
    - gitlab-terraform plan-json -var-file=${TFVARS_FILE} -parallelism=10
  resource_group: ${TF_STATE_NAME}
  artifacts:
    public: false
    paths:
      - ${TF_ROOT}/plan.cache
    reports:
      terraform: ${TF_ROOT}/plan.json

apply:
  extends: .deploy
  stage: apply
  retry: 2
  tags:
    - local
  dependencies:
    - plan
  script:
    - gitlab-terraform apply -parallelism=10 -auto-approve
  resource_group: ${TF_STATE_NAME}
```

> After the pipeline successfully runs, you can check the Terraform state in GitLab under Infrastructure/Terraform states

![](PayZF749px9dB1LqZMKiQA.webpg)

Terraform States in GitLab

RESOURCES:  
[https://docs.gitlab.com/ee/user/infrastructure/iac/terraform_state.html](https://docs.gitlab.com/ee/user/infrastructure/iac/terraform_state.html)