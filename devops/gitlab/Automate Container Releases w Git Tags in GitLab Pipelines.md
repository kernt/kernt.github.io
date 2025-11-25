TL;DR —  
* Pipeline should automate entire build, test, release and uprev versions  
* GitLab’s documentation is cryptic, or oddly very specific  
* Developers/Ops should operate with Merge Requests, Tags and app-level git-commits  
* Git commit inside of a Job can be done with a PAT and creative configuration  
* Full working example: [https://gitlab.com/mike-ensor/price-a-tray-dynamic-frontend/-/blob/main/.gitlab-ci.yml?ref_type=heads](https://gitlab.com/mike-ensor/price-a-tray-dynamic-frontend/-/blob/main/.gitlab-ci.yml?ref_type=heads)  
* Targeting intermediate to advanced developers and ops individuals

Deploying a new container for your app based on a git-tag is a clean method to release software. Tags provide a point-in-time, usually at a decisive moment in the code base. The CI/CD process should create a new container ONLY when there is a tag, or even more strict when the tag conforms to a specific format. All other commits should still continue to run the standard tests (ie: unit, functional, integration, etc). Ideally, the pipeline should up-rev the version automatically, based on the tag-name. This blog is aimed at an intermediate DevOps or developer and assumes basic CI/CD pipeline knowledge.

Gitlab’s documentation carries a lot of legacy functionality, has nuances between self-hosted, enterprise and SaaS and generally, I find uses terms that are not super intuitive. This post will describe some of the pitfalls discovered while building a pipeline that runs tests on all commits, modifies the source to set a “version”, builds & pushes a container to a registry, then finally commits the code representing the version to a branch for a Merge Request.

# Phases

Start with a simple set of 3 phases: test, versioning and release

![](F2EYlH_IV9EM6upEeG-Wg.webp)

**Tests** — All commits should still run the project’s bread&butter, the unit, functional and integration tests as desired. Plenty of other posts will go into the philosophical and often religious details about testing, so this post assumes you have some level of testing. Nothing special to note here other than remember to use proper hygiene with respect to caching based on your language of choice. This should run on all commits, so no `rules` or `when` or `only` or any other GitLab syntax tags.

**Versioning** — This is where things get interesting. The goal of this phase is to modify any files that represent the current “version” of the application. In the example I’m linking to, there are 5 files that are updated as a result of a version change. `/base/kustomization.yaml` , `/base/env-vars` , `package.json` , `package-lock.json` and `manifests/*.yaml` . This project uses Kustomize to generate KRM YAML files for Kubernetes so the `/base` needs to know about the new version. The `package` files are used by the Svelte app to version the artifact. And finally, the `manifests/` is generated to be used by downstream package repositories and is updated with the newest version. To accomplish this process, there is a script called `set-version.sh` at the root of the project ([https://gitlab.com/mike-ensor/price-a-tray-dynamic-frontend/-/blob/main/set-version.sh?ref_type=heads](https://gitlab.com/mike-ensor/price-a-tray-dynamic-frontend/-/blob/main/set-version.sh?ref_type=heads)). This simple script modifies the files using a combination of `kustomize`, `jq`, `yq` and `sed`.

Now comes the challenging part. The community does not have a Docker image with a combination of these tools, so the GitLab Job needs to install and set them up. Then the body can run the `set-version.sh` script file. After completing, the 5 files will have been updated in the CI workspace, but will not be carried on to other jobs unless saved as `artifacts`. Each item is individually listed. Use caution here, all artifacts are available in the `downloads` section of the pipeline, so do not store sensitive files in artifacts (regenerate them in each required job).

**NOTE**: at this time, the job would run on every commit, so we need to limit the job to only run when there is a Git Tag present. Documentation for this is sparse, and there are a lot of misinformation on the interwebs, this combination took a bit of time to figure out. (If you look at the source, I left out the generation of the .npmrc file for brevity)

```c
set-versions:  
  stage: versioning  
  image:  
    name: debian  
    entrypoint: [ "" ]  
  before_script:  
    # Setup the required tools (kustomize, yq and jq)  
    - apt-get update && apt-get install jq curl wget nodejs npm -y && jq --version  
    # YQ  
    - wget --no-verbose -O /usr/bin/yq https://github.com/mikefarah/yq/releases/download/v4.44.3/yq_linux_amd64 && chmod +x /usr/bin/yq && yq --version  
    # Kustomize  
    - wget --no-verbose -O kustomize.tar.gz "https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize/v5.5.0/kustomize_v5.5.0_linux_amd64.tar.gz"  
    - tar xvf kustomize.tar.gz && mv ./kustomize /usr/bin && kustomize version  
  script:  
    - ./set-version.sh "${VERSION}" # version set to CI_COMMIT_TAG (more later)  
    - ./generate.sh  
    - npm install # generate the package-lock.json file  
  artifacts:  
    paths:  
      - package.json  
      - package-lock.json  
      - base/kustomization.yaml  
      - base/env-vars  
      - manifests/*.yaml  
  rules:  
    - if: $CI_COMMIT_TAG  
      when: always  # Run always if there is a $CI_COMMIT_TAG
```

**Release** — The last phase is to create a new Docker image based on the new version of the application, then push those new changes to a branch allowing for a Merge Request to be created. Building a Docker image in GitLab CI can be difficult without some basic understanding. First, the `image` needs to have the `docker` and `docker-kit` binaries, but does not run the docker socket. The docker socket / service is run using the `services` directive. This means that the `image` can be a different container than the `services` container allowing for flexiblity. In this example, I intend on using `git` to save the changes made by the `versioning` phase if the `docker build` is completed.

A few challenges slowed down the progress of this job creation. First, `bash` is generally available in most `docker` or `docker-dind` containers, therefore I needed a different container. Since I was planning on `git` later, I chose the `bitnami/git:latest` image. This image has `apt` package manager and `bash` allowing me to run other `*.sh` scripts and install dependencies. I have created a bash script called `build.sh` that takes in the `version` and a `container-name`. This script is used in a majority of my projects so I can abstract how a container is built, regardless of the programming language used.

The second challenge was creating a method to `git-commit` the changes back to the repository. Gitlab has deprecated the `CI_JOB_TOKEN` or at least changed the usage of it enough times that the internet has conflicting information. I chose to use the Personal Access Token, but a Project-level token would work just fine for a production-based system. NOTE: the “scope” should be set to `write_repository` but the `role` needs to be `DEVELOPER` or `MAINTAINER` (preferably DEVELOPER) otherwise the “scope” does not matter (spent an hour debugging this). With this token saved as a GitLab CI variable I named`CI_DEPLOY_TOKEN` containing the value of the PAT token.

```yaml
release-version:
  stage: release
  before_script:
    # Dependencies for docker install
    - apt update && apt install wget apt-transport-https ca-certificates curl software-properties-common -y
    # Install docker
    - curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    - add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
    - apt install docker-ce docker-ce-cli docker-buildx-plugin docker-compose-plugin -y
    # Login to gitlab registry
    - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $CI_REGISTRY_USER --password-stdin
  image:
    name: bitnami/git:latest
    entrypoint: [ "" ]
  services:
    - name: docker:27.3.1-dind
  script:
    - ./build.sh "${VERSION}" "${CONTAINER_NAME}"
    - echo "Successful build, committing changes to a branch"
    - git config user.name "ci-bot"
    - git config user.email "ci-bot@example.cc"
    - git remote set-url origin "https://${GIT_USER}:${CI_DEPLOY_TOKEN}@gitlab.com/${CI_PROJECT_NAMESPACE}/${CI_PROJECT_NAME}.git">https://${GIT_USER}:${CI_DEPLOY_TOKEN}@gitlab.com/${CI_PROJECT_NAMESPACE}/${CI_PROJECT_NAME}.git"
    - git checkout -b "release/${VERSION}"
    - git add package.json package-lock.json base/kustomization.yaml base/env-vars manifests/
    - git commit -m "[skip ci] CI/CD release ${VERSION}" # NOTE: [skip ci] so we do not create a cycle of builds
    - git push origin "release/${VERSION}" # generate a Pull Request
  rules:
    - if: $CI_COMMIT_TAG
      when: always
```

**Releasing a version**

Now that we have the CI/CD pipeline in place, a release can be created by tagging the git-repo. We can generate a new release with the following example, assuming we want to release as version `1.0.0`.

```sh
git tag -a '1.0.0' -m 'Releasing 1.0.0 version'  
git push --tags
```

Upon completion, a new Docker image will be released to the configured registry. Additionally, a new `git-branch` will exist in the project in which to create a new Merge Request with. The merge request will contain the new versions of the files that hold the `version` information (ie: `package.json`, `base/kustomization.yaml`, etc).

![](pyvJyjwgaz_0wIwNQIH2Hg.webp)

**Conclusion**

This blogs covers how to create a versioned container using a GitLab CI/CD Pipeline. Any other changes to the repository will result in the standard test jobs, while a tag will create a versioned release. These principles can be used in conjunction with `releases` and other CI/CD techniques.

A full working example can be found in this project’s `.gitlab-ci.yml` file: [https://gitlab.com/mike-ensor/price-a-tray-dynamic-frontend/-/blob/f6763df20030648618b5c7293018ffcd441a6e5b/.gitlab-ci.yml](https://gitlab.com/mike-ensor/price-a-tray-dynamic-frontend/-/blob/main/.gitlab-ci.yml?ref_type=heads)
