Managing and presenting a clear, accessible changelog is essential for any software project. However, creating and updating these logs manually can be time-consuming, especially as a project grows. In this article, we’ll dive into an automated approach for generating changelogs and release pages in GitLab using GitLab CI and the GitLab API.

With GitLab CI, you can set up a seamless pipeline that automatically compiles your latest changes into a structured changelog and publishes it directly to your project’s release page. By the end of this guide, you’ll have a robust, reusable CI configuration that enables your team to track releases with minimal effort, keeps your documentation up to date, and reduces the need for manual intervention.

# Preparation

To start, we need a GitLab repository. In this example, we’ll create a repository named `gitlab-ci-changelog` and pull it locally to set up our pipeline configuration.

## Tagging the Repository

Automated changelog generation in GitLab relies on identifying changes based on the last tag. To initialize this process, we’ll create an initial tag that serves as the starting point for the changelog. Let’s tag this version as `v0.0.0`, which will act as a baseline for tracking future changes.

```sh
# Tag the repository  
git tag v0.0.0  
  
# Push the tags  
git push --tags
```

## Creating an API Access Token

Since the GitLab Runner’s token doesn’t support GitLab API access, we need to create a dedicated API access token to authenticate our CI pipeline. Here’s how to set it up:

1. Go to **Settings > Access Tokens** in your GitLab repository.
2. Create a new access token with the name `CI_API_TOKEN`.
3. Assign the **Maintainer** role to this token and grant it the **API** scope.

This token will allow our CI pipeline to interact with the GitLab API, enabling automated changelog generation.

![](K0bLhFxN58xsEAYh00Gygw.webp)

To securely store the token for use in the CI pipeline, add it as a GitLab CI/CD variable:

1. Go to **Settings > CI/CD** in your GitLab repository.
2. Open the **Variables** section and click **Add variable**.
3. Set the **Key** as `CI_API_TOKEN` and paste the access token in the **Value** field.
4. Enable **Masked** and **Protected** to secure the token, and ensure **Flags** are unchecked.

![](MLvtWTuaGczohNyvb-6rwA.webp)

## Creating the `.gitlab-ci.yml` File

The default location for `.gitlab-ci.yml` is the root of your project, but placing it in a dedicated `.gitlab` directory helps organize other project-specific configurations.

# Create a .gitlab directory:  
mkdir .gitlab  
# Create the .gitlab-ci.yml file within the new directory:  
touch .gitlab/.gitlab-ci.yml

Update the GitLab CI/CD configuration to recognize this new location. Go to **Settings > CI/CD** and change the **Custom CI/CD configuration path** to `.gitlab/.gitlab-ci.yml`.

![](vA8bqEDRKDc8F-X7yt0IYg.webp)

# Create the Pipeline

The pipeline for automating changelog and release-page creation consists of two main steps: **creating the changelog** and **publishing the release page**. The pipeline is triggered only when a new tag is pushed, as indicated by the `rules`condition, but you may need to change the rules for your needs.

```yaml
stages:  
  - release  
  
create-changelog:  
  stage: release  
  image: registry.gitlab.com/gitlab-ci-utils/curl-jq:latest  
  variables:  
    CHANGELOG_URL: "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/repository/changelog?version=${CI_COMMIT_TAG}"  
    CHANGELOG_COMMIT_MESSAGE: "Add changelog for ${CI_COMMIT_TAG}"  
  script:  
    - 'curl -H "PRIVATE-TOKEN: ${CI_API_TOKEN}" "${CHANGELOG_URL}" | jq -r .notes > release_notes.md'  
    - 'curl -H "PRIVATE-TOKEN: ${CI_API_TOKEN}" -X POST --data "message=${CHANGELOG_COMMIT_MESSAGE}; [skip ci]" "${CHANGELOG_URL}"'  
  artifacts:  
    paths:  
      - release_notes.md  
    expire_in: 1 hour  
  rules:  
    - if: $CI_COMMIT_TAG  
  
create-release-page:  
  stage: release  
  needs:  
    - job: create-changelog  
      optional: false  
  image: registry.gitlab.com/gitlab-org/release-cli:latest  
  script:  
    - echo "Creating gitlab release page for release version ${CI_COMMIT_TAG}"  
  release:  
    name: Release ${CI_COMMIT_TAG}  
    description: release_notes.md  
    tag_name: ${CI_COMMIT_TAG}  
    ref: ${CI_COMMIT_SHA}  
  rules:  
    - if: $CI_COMMIT_TAG
```

Here’s a breakdown of the `.gitlab-ci.yml` configuration:

```yaml
stages:  
  - release
```

This setup defines a single `release` stage, which will handle both the changelog generation and the release page creation.

## `create-changelog` Job

The first job, `create-changelog`, generates a changelog file based on recent commits and stores it as an artifact aswell.

```c
create-changelog:  
  stage: release  
  image: registry.gitlab.com/gitlab-ci-utils/curl-jq:latest  
  variables:  
    CHANGELOG_URL: "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/repository/changelog?version=${CI_COMMIT_TAG}"  
    CHANGELOG_COMMIT_MESSAGE: "Add changelog for ${CI_COMMIT_TAG}"  
  script:  
    - 'curl -H "PRIVATE-TOKEN: ${CI_API_TOKEN}" "${CHANGELOG_URL}" | jq -r .notes > release_notes.md'  
    - 'curl -H "PRIVATE-TOKEN: ${CI_API_TOKEN}" -X POST --data "message=${CHANGELOG_COMMIT_MESSAGE}; [skip ci]" "${CHANGELOG_URL}"'  
  artifacts:  
    paths:  
      - release_notes.md  
    expire_in: 1 hour  
  rules:  
    - if: $CI_COMMIT_TAG
```

- **Image**: This job uses a Docker image with `curl` and `jq`, which allow us to interact with GitLab’s API and parse JSON data.
- **Variables**: The `CHANGELOG_URL` variable constructs the API endpoint URL for the changelog, while `CHANGELOG_COMMIT_MESSAGE` holds a custom commit message for the changelog entry.
- **Script**:  
    - The first `curl` command fetches the changelog from GitLab’s API, filtering relevant notes via `jq`, and saves them to a file named `release_notes.md`.  
    - The second `curl` command commits the changelog, adding `[skip ci]` to prevent unnecessary CI pipeline runs.
- **Artifacts**: The generated `release_notes.md` file is stored as an artifact for one hour, allowing it to be passed to the next job.
- **Rules**: This job runs only when a new tag is created.

## `create-release-page` Job

The `create-release-page` job takes the generated changelog and publishes a release page in GitLab, making it available in the repository's Releases section.

```c
create-release-page:  
  stage: release  
  needs:  
    - job: create-changelog  
      optional: false  
  image: registry.gitlab.com/gitlab-org/release-cli:latest  
  script:  
    - echo "Creating gitlab release page for release version ${CI_COMMIT_TAG}"  
  release:  
    name: Release ${CI_COMMIT_TAG}  
    description: release_notes.md  
    tag_name: ${CI_COMMIT_TAG}  
    ref: ${CI_COMMIT_SHA}  
  rules:  
    - if: $CI_COMMIT_TAG
```

- **Needs**: This job depends on `create-changelog`, ensuring that the changelog is generated first.
- **Image**: It uses GitLab’s `release-cli` image, which provides tools for creating GitLab releases directly.
- **Script**: Logs a message indicating the release creation process.
- **Release**: Defines the release details:  
    - **Name**: Sets the release title, including the tag version.  
    - **Description**: Includes the contents of `release_notes.md` as the release description.  
    - **Tag Name**: Uses the pushed tag name for the release.  
    - **Ref**: Refers to the specific commit associated with the release.
- **Rules**: Similar to the first job, this runs only when a new tag is pushed.

This pipeline ensures that each time a new tag is created, a changelog is generated and a release page is published, streamlining the process and reducing manual tasks.

# Commit to your Repository

With everything set up, you’re ready to start creating changelog entries automatically in your pipeline. To use the new pipeline efficiently, you’ll need to leverage **Git trailers**. Git trailers add structured metadata to your commit messages, which Git can interpret using the `interpret-trailers` command.

You can add trailers manually to your commit messages. For instance, to categorize a commit as a “feature” in the changelog, you can include the following trailer in your commit message:

```c
<Commit message subject>  
  
<Commit message description>  
  
################################################################  
### GIT TRAILERS -- THESE MUST BE LAST IN THE COMMIT MESSAGE ###  
################################################################  
# added: New feature  
# fixed: Bug fix  
# changed: Feature change  
# deprecated: New deprecation  
# removed: Feature removal  
# security: Security fix  
# performance: Performance improvement  
# other: Other  
  
Changelog: feature
```

Alternatively, you can add trailers directly from the command line. Here’s how:

## Option 1: Using `--trailer` with `git commit`

If you want to add a trailer directly in the command, you can use the `--trailer` flag like this:

```sh
git commit --message "Added new Feature" \  
           --trailer "Changelog: added"
```

## Option 2: Adding Trailers in the Commit Message

If you prefer not to use the `--trailer` flag, you can include trailers directly in the commit message body. Here’s an example:

```sh
git commit -m <title> -m <description>  
git commit -m "Added new Feature" -m "Changelog: added"
```

By using Git trailers consistently in your commits, you’ll ensure that your pipeline can accurately categorize and document changes, making it easier for your team to track and understand project updates.

**Note:** The changelog generation runs once per version tag. After the changelog is generated, you’re free to edit it to suit your needs — for instance, if you forgot to set a trailer in the commit message.

## (OPTIONAL) Setup Git Commit Templates

While not strictly part of this article, setting up a commit template can be invaluable when working with Git trailers and automated changelog generation. By creating a commit template, you ensure that changelog entries are consistently added to your commits, even when you’re focused on other details. Many editors, like IntelliJ IDEA and VS Code, support loading these templates directly into their UI, making it easier to structure commit messages without missing key details.

You can create the template in your home directory for global use, or within your project repository for project-specific configuration. Here’s how to set one up:

**Create a Commit Template File**:  
For a global template, use a command like:

`vim ~/.git-commit-template`

For a project-specific template, place the file in the `.gitlab` folder we created earlier, for example:

`vim .gitlab/.git-commit-template`

**Sample Commit Template:**

Below is an example template structure. **Comments (**lines starting with `**#**`**) will not be committed** but serve as helpful reminders while you write your commit messages.

```sh
##################################################  
# Write a title summarizing what this commit does.  
# Start with an uppercase imperative verb, such as  
# Add, Drop, Fix, Refactor, Bump; see ideas below.  
# Think of your title as akin to an email subject,  
# so you don't need to end with a sentence period.  
# Use 50 char maximum, which is this line's width.  
##################################################  
  
########################################################################  
# Why is this change happening?  
# Describe the purpose, such as a goal, or use case, or user story, etc.  
# For every line, use 72 char maximum width, which is this line's width.  
########################################################################  
  
################################################################  
### GIT TRAILERS -- THESE MUST BE LAST IN THE COMMIT MESSAGE ###  
################################################################  
# added: New feature  
# fixed: Bug fix  
# changed: Feature change  
# deprecated: New deprecation  
# removed: Feature removal  
# security: Security fix  
# performance: Performance improvement  
# dependencies: Dependency updates  
# other: Other  
Changelog: other
```

You can find additional template examples, such as this one [on GitHub](https://github.com/joelparkerhenderson/git-commit-template), which offers more ideas for structuring commit messages.

**Set Up Your Template with Git:**

To use this template for all your commits globally, configure it with:

`git config --global commit.template ~/.git-commit-template`

Alternatively, for project-specific configurations, navigate to your project directory and set the template path there:

`git config commit.template .gitlab/.git-commit-template`

By establishing a commit template, you’ll make it easier to stay consistent, add changelog entries, and ensure that all metadata needed for automation is always included.

# Create a Release/Version

Once you’ve pushed your changes to the repository, it’s time to create a new release version by tagging it. This tag will trigger the changelog generation and release creation pipeline.

```sh
# Tag the repository  
git tag v0.0.1  
  
# Push the tags  
git push --tags
```

Head to **Build > Pipelines** in your GitLab project to Check the Pipeline Status. Once the pipeline completes, you should see a green “Passed” status. At this point, a `CHANGELOG.md` file will be generated and available in your repository. Open it to view the automatically generated changelog, showcasing all the changes you’ve tagged—magic at your fingertips!

# Handling Merge Requests

Wondering how this setup works with merge requests? 😊 Let’s go over it!

If you avoid squashing commits when merging, there’s no need for extra steps — just ensure each commit you want in the changelog includes a trailer.

However, if you use squash commits, you’ll need to configure a `Squash Commit Message Template`. This template allows you to include a changelog trailer even for squash merges. Here’s how to set it up:

1. Go to **Settings > Merge Requests > Squash commit message template** in your GitLab repository.
2. Add the a template, for example the following:

```c
%{title}  
  
Changelog: other  
  
# Please choose one of the following:  
  
# added: New feature  
# fixed: Bug fix  
# changed: Feature change  
# deprecated: New deprecation  
# removed: Feature removal  
# security: Security fix  
# performance: Performance improvement  
# dependencies: Dependency updates  
# other: Other
```

By setting `Changelog: other` as the default trailer, you’ll ensure that every squash merge is accounted for in the changelog, even if no specific trailer is chosen. You can customize this template to better fit your project’s needs.

# Conclusion

Automating your changelog and release page generation with GitLab CI simplifies release tracking and keeps documentation up-to-date with minimal manual intervention. With this setup, you ensure that each commit and feature is consistently documented, making it easy for your team and users to see what’s new with every release. Implementing structured commit messages, Git trailers, and templates also adds consistency, helping you capture all critical metadata. This guide equips you to maintain a smooth release process that scales effortlessly, so you can focus more on development and less on documentation.

Now you’re ready to set up and streamline changelog and release automation in your GitLab projects — happy coding!

# Example Repository

- [https://gitlab.com/0hlov3/gitlab-ci-changelog](https://gitlab.com/0hlov3/gitlab-ci-changelog)
- **Changelog**: [https://gitlab.com/0hlov3/gitlab-ci-changelog/-/blob/main/CHANGELOG.md](https://gitlab.com/0hlov3/gitlab-ci-changelog/-/blob/main/CHANGELOG.md?ref_type=heads)
- **Release**: [https://gitlab.com/0hlov3/gitlab-ci-changelog/-/releases](https://gitlab.com/0hlov3/gitlab-ci-changelog/-/releases)

# Helpful Links

- [https://docs.gitlab.com/ee/user/project/changelogs.html](https://docs.gitlab.com/ee/user/project/changelogs.html)
- [https://docs.gitlab.com/ee/development/changelog.html](https://docs.gitlab.com/ee/development/changelog.html)
- [https://github.com/joelparkerhenderson/git-commit-template](https://github.com/joelparkerhenderson/git-commit-template)