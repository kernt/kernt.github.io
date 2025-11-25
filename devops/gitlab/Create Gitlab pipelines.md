This is the 3rd article in the [series “Gitlab pipeline”](https://medium.com/gitconnected/jenkins-to-gitlab-162739e96613). Here I want to share an example of a pipeline.

> Example :

We started off with 5 pipelines — 3 deploying to AWS and 2 to K8S. This snippet is of the pipeline deploying to K8S cluster.

```yaml
stages:  
  - cleanup  
  - prod-build  
  - prod-docker  
  - prod-deploy  
  
variables:  
  HELM_CHART_PATH: "helm"  
  IMAGE_NAME: <VALUE>  
  PROD_IMAGE_TAG: "$CI_COMMIT_SHORT_SHA-production"  
  PROD_FILE_NAME: "production.yaml"  
  PROD_NAMESPACE: <VALUE>  
  SLACK_CHANNEL: <VALUE>  
  SLACK_WEBHOOK_URL: <VALUE>  
  
cleanup:  
  stage: cleanup  
  script:  
    - "docker system prune -af"  
  
prod_build:  
  stage: prod-build  
  image: <VALUE>  
  before_script:  
    - ....  
  script:  
    - ....  
  # Define artifacts to preserve the "build" directory  
  artifacts:  
    paths:  
      - build/  
  rules:  
    - if: '$CI_COMMIT_BRANCH == "master"'  
      when: always  
  after_script:  
    - >  
      curl -X POST -H 'Content-type: application/json' --data "{\"channel\":\"$SLACK_CHANNEL\",\"text\":\"$CI_JOB_STATUS on branch $CI_COMMIT_REF_NAME for $CI_JOB_STAGE in <$CI_PIPELINE_URL>.\"}" $SLACK_WEBHOOK_URL  
  
prod_docker_image:  
  stage: prod-docker  
  image: docker:latest  
  script:  
    - docker build -t $CI_REGISTRY_ENDPOINT/$IMAGE_NAME:$PROD_IMAGE_TAG .  
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY_URL  
    - docker push $CI_REGISTRY_ENDPOINT/$IMAGE_NAME:$PROD_IMAGE_TAG  
  # Define dependencies and specify the job that generates artifacts  
  dependencies:  
    - prod_build  
  rules:  
    - if: '$CI_COMMIT_BRANCH == "master"'  
      when: always  
  
prod_deployment:  
  stage: prod-deploy  
  image: <VALUE>  
  before_script:  
    - doctl auth init --access-token $DO_PAT  
    - doctl kubernetes cluster kubeconfig save $DO_CLUSTER_ID  
  script:  
    - ....  
  rules:  
    - if: '$CI_COMMIT_BRANCH == "master"'  
      when: manual  
  after_script:  
    - >  
      curl -X POST -H 'Content-type: application/json' --data "{\"channel\":\"$SLACK_CHANNEL\",\"text\":\"$CI_JOB_STATUS on branch $CI_COMMIT_REF_NAME for $CI_JOB_STAGE in <$CI_PIPELINE_URL>.\"}" $SLACK_WEBHOOK_URL
```

> Explanation :

The [CI/CD YAML syntax](https://docs.gitlab.com/ee/ci/yaml/) (shown above) has been explained extensively in Gitlab docs. I will elaborate on a few things :

- We wanted to manually approve production deployments. Hence we added a [rule](https://docs.gitlab.com/ee/ci/yaml/#rules) - if the pipeline is running for master branch, the stage `prod_deployment` should be approved for execution.
- Output of a stage is not available for other stages to use. However you may need that sometimes. Hence you create [artefacts](https://docs.gitlab.com/ee/ci/jobs/job_artifacts.html) and make them accessible to another stage via [dependencies](https://docs.gitlab.com/ee/ci/yaml/#dependencies).
- You will see a `curl` command to notify about the pipeline in Slack. It has some variables starting with `$CI_` — they are [predefined variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) of Gitlab pipeline.
- To deploy to K8S cluster of Digital Ocean, we need the PAT (personal access token) and cluster ID. Just like other variables, these are stored in _Variables_ within _CI/CD Settings_ of the repository.
- As shown above, each stage has 1 step and all execute sequentially. However if you need to execute multiple steps of a stage in parallel :

```
stages:  
  - test  
  
unit-test:  
  stage: test  
  <....>  
  script:  
    - npm run utest  
  
integration-test:  
  stage: test  
  <....>  
  
e2e-test:  
  stage: test  
  <....>
```

> Troubleshooting

All our pipelines soon became unreliable due to shortage of resources.

Our Gitlab is hosted on Digital Ocean’s droplet. We had configured Gitlab to have the necessary archival / deletion policies for jobs, job artefacts etc.

- We first checked if there’s a resource crunch on the droplet. Memory consumption was more than 90% and disk consumption was more than 75%.
- We had added swap memory earlier. So, this time we ran a few [clean-up commands](https://docs.gitlab.com/ee/raketasks/cleanup.html). 2 of these helped reducing the memory and disk usage immediately :

```sh
gitlab-rake gitlab:cleanup:orphan_job_artifact_files DRY_RUN=false  
gitlab-rake gitlab:cleanup:sessions:active_sessions_lookup_keys
```

- We also manually cleaned the cache, but none of these helped resolve our issue.
- After further investigation we discovered that the EC2 instances (job-executors) were facing a resource crunch. That was because, during job execution a lot of large docker images were being pulled but never deleted.
- Hence we added a `cleanup` stage at the beginning of each pipeline (for consistency). We haven’t faced a resource crunch since. Our pipelines are running smoothly on the same instance type.
- Later we also enabled monitoring for our instances (as shown in the 1st article of the series).

> Benefits

Advantages with the current setup of Gitlab pipelines (compared to the earlier setup with Jenkins) :

- our pipeline execution time has reduced drastically (~ more than 80%)
- Sometimes with Jenkins resource crunch our pipelines would get indefinitely stuck or fail (with variety of errors) after a long execution time. Now we can depend on our pipelines to give accurate results on time.

So we have a bright future w.r.t. internal applications’ infrastructure management. No more complex / time consuming / misleading errors & better adoption of pipelines across projects.