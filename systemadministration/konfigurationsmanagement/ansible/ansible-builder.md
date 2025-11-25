---
tags:
  - konfigurationsmanagement
  - ansible
  - ansible-builder
---

# Ansible Builder

The example in `test/data/pytz` requires the `awx.awx` collection in the execution environment definition. The lookup plugin `awx.awx.schedule_rrule` requires the PyPI `pytz` and another library to work. If `test/data/pytz/execution-environment.yml` file is given to the `ansible-builder build` command, then it will install the collection inside the image, read `requirements.txt` inside of the collection, and then install `pytz` into the image.

The image produced can be used inside of an `ansible-runner` project by placing these variables inside the `env/settings` file, inside of the private data directory.

```yaml
---
container_image: image-name
process_isolation_executable: podman # or docker
process_isolation: true
```

# [Passing Secrets](https://ansible.readthedocs.io/projects/builder/en/stable/scenario_guides/scenario_secret_passing/#passing-secrets "Permalink to this heading")

When creating an Execution Environment, it may be useful to use [build secrets](https://docs.docker.com/build/building/secrets/). This can be done with a combination of the use of [additional_build_steps](https://ansible.readthedocs.io/projects/builder/en/stable/definition/#additional-build-steps) within the EE definition file, and the [--extra-build-cli-args](https://ansible.readthedocs.io/projects/builder/en/stable/usage/#extra-build-cli-args) CLI option.

Use the [--extra-build-cli-args](https://ansible.readthedocs.io/projects/builder/en/stable/usage/#extra-build-cli-args) CLI option to pass a build CLI argument that defines the secret:

`ansible-builder build --extra-build-cli-args="--secret id=mytoken,src=my_secret_file.txt"`

Then, use a custom `RUN` command within your EE definition file that references this secret:

```yaml
---
version: 3

images:
  base_image:
    name: quay.io/centos/centos:stream9

additional_build_steps:
  prepend_base:
    - RUN --mount=type=secret,id=mytoken TOKEN=$(cat /run/secrets/mytoken) some_command

options:
  skip_ansible_check: true
```
  
* [builder](https://ansible.readthedocs.io/projects/builder/en/latest/)