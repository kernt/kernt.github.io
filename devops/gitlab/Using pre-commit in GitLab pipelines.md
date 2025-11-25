As developers, we’ve all been there — you’ve spent hours working on a feature, only to have it break when you merge it into the main branch. Or, you’ve pushed code to production only to realize it’s riddled with errors and typos. [pre-commit](https://pre-commit.com/) is very convenient for enforcing high-quality commits and keeping the codebase tidy. I’ve been using it for a while now, and I appreciate how easy it is to set up and use with my projects. Using it with GitHub Actions is a breeze. However, we use GitLab for various projects at work.

In this article, we’ll explore how to automate pre-commit checks in GitLab [pipelines](https://docs.gitlab.com/ee/ci/pipelines/). We enforce quality and formatting standards with pre-commit [hooks](https://pre-commit.com/hooks.html).

# What are pre-commit hooks?

Pre-commit hooks are scripts that automatically execute before you commit code to your repository. They help enforce coding standards and check for syntax errors. These hooks run every time a commit is made, providing a thorough check. By performing these checks before the code is committed, you can identify and resolve issues early, preventing them from entering your remote branch.

It’s never fun to push your exciting feature only to have the first comment on your merge request say something like, “Please format the code.”

# How do pre-commit hooks work in GitLab pipelines?

In GitLab, you can use pre-commit hooks as part of your CI/CD pipeline. When you push code to your repository, GitLab will run the pre-commit hook before the code is committed. If the hook fails, the commit will be rejected, and you’ll receive an error message indicating what went wrong.

![](dQeumPnEaRA5AeSt5n8Qqw.webp)

A pipeline error in GitLab where pre-commit failed.

# How to set up pre-commit hooks in GitLab pipelines

Setting up pre-commit hooks in GitLab pipelines is relatively straightforward. Here are the steps:

## Create a `.pre-commit-config.yaml` file

In the root of your repository, create a new file called `**.pre-commit-config.yaml**`. This file will contain the scripts to run before the code is committed.

## Define your hooks

In the `**.pre-commit-config.yaml**` file, define the hooks you want to run. For example, run a linter to check for syntax errors or a formatter to enforce coding standards.

Here’s an example of what your `**.pre-commit-config.yaml**` file might look like:

```
repos:  
  - repo: https://github.com/pre-commit/pre-commit-hooks  
    rev: v5.0.0  
    hooks:  
      - id: end-of-file-fixer  
      - id: trailing-whitespace  
  - repo: https://github.com/astral-sh/ruff-pre-commit  
    rev: v0.8.5  
    hooks:  
      - id: ruff  
        args: [--fix]  
      - id: ruff-format  
  - repo: https://github.com/codespell-project/codespell  
    rev: v2.3.0  
    hooks:  
      - id: codespell  
  - repo: https://github.com/google/yamlfmt  
    rev: v0.14.0  
    hooks:  
      - id: yamlfmt

```

This will clean up any files that need their endings fixed, have extra whitespace at the end, enforce ruff formatting standards (this can be enhanced with [ruff configuration](https://docs.astral.sh/ruff/configuration/) settings), warn us of our typos, and format our yaml files.

## Configure your pipeline

To combine all of this to run automatically, we will need to set up a file so GitLab will know what to do. In your `**.gitlab-ci.yml**` configure your pipeline to run the pre-commit hook before the code is committed.

And here’s an example of what your `**.gitlab-ci.yml**` file might look like:

```
stages:  
  - build  
pre-commit:  
  stage: build  
  image: python:3.11  
  script:  
    - pip install pre-commit  
    - pre-commit run --all-files  
  rules:  
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"  
    - if: $CI_COMMIT_BRANCH == "main"
```

This will use Python to execute pre-commit anytime a commit occurs on the main branch, a merge request is created, or a commit is pushed to said merge request. We put this in the “build” stage, but GitLab has a few stages available out of the box. You might want to use the “test” or “deploy” stage for other actions.

