How to build custom “my_ee” Ansible Execution Environment specifying some custom System (git), Python (boto), and collection (community.aws) dependencies using the ansible-builder command-line tool.

# How to build a custom Ansible Execution Environment?

Using an Ansible Execution Environment is the latest technology to maintain up-to-date Python dependency of the Ansible collections without interfering with your Linux system. It’s the evolution of Python Virtual Environment.

This initial configuration sometimes is a roadblock for some Ansible users.

I’m Luca Berton and welcome to today’s episode of Ansible Pilot.

# Ansible Execution Environment

- Ansible Execution Environment
- `ansible-builder` command-line tool
- `ansible-runner` command-line tool

Let’s talk about the Ansible Execution Environment.

The Ansible Execution Environment is container images that can be utilized as Ansible control nodes.

It’s the latest technology developed by Red Hat to simplify the automation process.

The main advantage is a standard environment for Development and Production images using container technology creating portable automation runtimes.

This technology superseded manual Python Virtual Environments, Ansible module dependencies, and bubblewrap.

Experienced users are probably familiar with a lot of challenges managing custom Python Virtual Environments and Ansible module dependencies. Enterprise users of Ansible Automation Platform were familiar with limiting execution jobs under bubblewrap in order to isolate processes

The creation is performed by the Ansible Builder tool.

Ansible Builder produces a directory that acts as the build context for the container image build, containing the ``Containerfile``, along with any other files that need to be added to the image.

On the other end, the execution is performed by the Ansible Runner tool.

The Ansible Runner enables you to run the Execution Environment as a container in the current machine. It is basically taking care that the content runs as expected.

# Links

- [https://www.ansible.com/products/execution-environments](https://www.ansible.com/products/execution-environments)
- [https://docs.ansible.com/automation-controller/latest/html/userguide/execution_environments.html](https://docs.ansible.com/automation-controller/latest/html/userguide/execution_environments.html)

# demo

Build an Ansible Execution Environment using ansible-builder tool  
- Execution Environment name: _my_ee_  
- System dependency: git  
- Python dependency boto3  
- Collection dependency: community.aws

How to Build an Ansible Execution Environment using ansible-builder tool.

I’m going to show you how to Build a custom “`my_ee`” Ansible Execution Environment using the `ansible-builder` tool specifying some custom System, Python, and collection dependency.

For example, let’s build a custom Ansible Execution Environment named “`my_ee`” With System requirements ``git``, Python libraries ``boto3`` and Amazon Collection ``community.aws``.

# code

- execution-environment.yml

```yaml
---  
version: 1  
dependencies:  
  galaxy: requirements.yml  
  python: requirements.txt  
  system: bindep.txt  
additional_build_steps:  
  prepend: |  
        RUN pip3 install --upgrade pip setuptools  
  append:  
    - RUN ls -al /
```

- requirements.yml

```yaml
---  
collections:  
  - name: community.aws  
requirements.txtbotocore>=1.18.0  
boto3>=1.15.0  
boto>=2.49.0
```

- bindep.txt

```sh
git [platform:rpm]  
git [platform:dpkg]
```

# execution

```sh
ansible-builder build -t my_ee -v 3  
Running command:  
  podman build -f context/Containerfile -t my_ee context  
Complete! The build context can be found at: /home/devops/ee/context
```

When the ansible-builder tool is not installed you should install it via DNF command:

`[root@demo ee]# dnf install ansible-runner`

The tool needs access to the Red Hat Container Registry available with your Red Hat Ansible Automation subscription (username and password of Red Hat Portal).

`podman login registry.redhat.io`

A successful build produces the following `context/Containerfile`:

```sh
ARG _EE_BASE_IMAGE_=registry.redhat.io/ansible-automation-platform-22/ee-minimal-rhel8:latest  
ARG _EE_BUILDER_IMAGE_=registry.redhat.io/ansible-automation-platform-22/ansible-builder-rhel8:latest  
FROM _$EE_BASE_IMAGE_ as galaxy  
ARG _ANSIBLE_GALAXY_CLI_COLLECTION_OPTS_=  
USER root  
ADD _build /build  
WORKDIR /build  
RUN ansible-galaxy role install -r requirements.yml --roles-path "/usr/share/ansible/roles"  
RUN _ANSIBLE_GALAXY_DISABLE_GPG_VERIFY_=1 ansible-galaxy collection install _$ANSIBLE_GALAXY_CLI_COLLECTION_OPTS_ -r requirements.yml --collections-path "/usr/share/ansible/collections"  
FROM _$EE_BUILDER_IMAGE_ as builder  
COPY --from=galaxy /usr/share/ansible /usr/share/ansible  
ADD _build/requirements.txt requirements.txt  
ADD _build/bindep.txt bindep.txt  
RUN ansible-builder introspect --sanitize --user-pip=requirements.txt --user-bindep=bindep.txt --write-bindep=/tmp/src/bindep.txt --write-pip=/tmp/src/requirements.txt  
RUN assemble  
FROM _$EE_BASE_IMAGE_  
USER root  
RUN pip3 install --upgrade pip setuptools  
COPY --from=galaxy /usr/share/ansible /usr/share/ansible  
COPY --from=builder /output/ /output/  
RUN /output/install-from-bindep && rm -rf /output/wheels  
RUN ls -la /
```

https://github.com/lucab85/ansible-pilot?source=post_page-----b05e35f402e8---------------------------------------

