creating a custom execution environment

In this blog, we will look at AWX execution environments and how to create a custom execution environment if we need one. Let’s begin!

# What is AWX and Execution Environment?

AWX is the upstream project of Ansible Tower/Ansible Automation Platform based on the explanation [here](https://www.ansible.com/faq). We are using AWX because it takes care of some basic needs when using Ansible.

The term **Execution Environments** [_based on the explanation_](https://ansible.readthedocs.io/projects/runner/en/latest/execution_environments/), refers to the container runtime execution of **Ansible** within an [OCI Compliant Container Runtime](https://github.com/opencontainers/runtime-spec) using an [OCI Compliant Container Image](https://github.com/opencontainers/image-spec/) that appropriately bundles [Ansible Base](https://github.com/ansible/ansible), [Ansible Collection Content](https://github.com/ansible-collections/overview), and the runtime dependencies required to support these contents. The build tooling provided by [Ansible Builder](https://github.com/ansible/ansible-builder) aids in the creation of these images.

Each execution environment allows you to have a customized image to run jobs, and each of them contain only what you need when running the job.

If you need a specific command, module or python package that is not available in the default [execution environment image](https://quay.io/repository/ansible/awx-ee), you can easily create your own custom execution environment with ansible-builder.

# Configuring The Dependencies

I’ve created a [**github repository**](https://github.com/kcloud2023/awx-ee-builder/tree/main) just for the execution environments. I will be using this repository for the rest of this post.

**NOTE:** I am using AWX operator 2.5.0

After you’ve cloned the repository you will have a folder structure like below:

```sh
awx-ee-builder  
├── LICENSE  
├── README.md  
└── builder  
 ├── dependencies  
 │ ├── bindep.txt  
 │ ├── requirements.txt  
 │ └── requirements.yml  
 ├── execution-environment_v1.yml  
 ├── execution-environment_v3.yml  
 └── files  
 └── ansible.cfg
```

**NOTE: There is detailed information in the README of the repository.**

To create your custom execution environment, you will need:

- Docker
- Python3
- Ansible Builder

**NOTE:** If you want to use ansible-builder v3, you need to install Python3.9 on your system.

After installing docker and python3, you can install ansible-builder with the command:

`python3 -m pip install ansible-builder`

_Based on your python3 version, ansible-builder v1 or v3 will be installed. If you have python3.9 or upper ansible-builder v3 will be installed and if you have a lower python3 version, ansible-builder v1 will be installed._

After you are done with the prerequisites, you can begin to work with the customization. The repository contains folders/files that can help you customize your image. Under the builder/dependencies folder there are 3 different files with the following purpose:

- **bindep.txt:** Is used for RPM packages that need to be installed on the image
- **requirements.txt:** Is used for python packages/libraries that needs to be installed on the image
- **requirements.yml:** Is used for ansible-galaxy modules that needs to be installed on the image

After modifying requirements files, you need to modify your main file which is **execution-environment.yml**.

The repository includes examples for both ansible-builder v1 and v3, respectively

- execution-environment_v1.yml
- execution-environment_v3.yml

You can edit these execution-environment.yml files based on your specific needs, and **change their name to execution-environment.yml**

# Creating The Execution Environment

After you modify the above files based on your needs, you can create your EE with below with:

```sh
ansible-builder build --tag <your_registry_name>/awx/ee:<your_base_ee_version>-custom --container-runtime docker --verbosity 3
```

This command will create your execution environment and it will take some time, let’s take a look at the options:

- **— tag** is used for tagging the newly created docker image
- **<your_base_ee_version>** is used here to know which image is your image based on
- **— container-runtime** is docker, so our images will be docker images
- **— verbosity** is just the configuration of the command about how verbose should it be.

**NOTE:** There is _caching_ when creating an image, if your build fails, cached steps can be used again. But if you don’t want to use that cache, you can remove it by clearing the folder **context**, in your folder structure.

After you’ve successfully created your image, you can push the image to your image registry and create an execution environment in AWX.

To push your image:

`docker push <your_registry_name>/awx/ee:<your_base_ee_version>-custom`

**NOTE:** If your registry needs authentication, you can authenticate with the `_docker login_` command.

# Adding EE to AWX

After creating your EE image and pushing it to your image registry, your EE is ready to use. You can create a new EE in AWX from Administration -> Execution Environments -> Add

![](X1wLq-MZ1A2UGhDC.webp)

In the new EE creation form you need to fill the fields with the required values.

![](8DPVU0kwBEMRrIya.webp)

- **Name:** You can write your image name
- **Image:** This is the link to your image, in our example it is`<your_registry_name>/awx/ee:<your_base_ee_version>-custom`
- **Pull:** There are 3 options here: Always pull, Only if not present pull, Never pull. You can choose Only if not present pull.
- **Description:** This is the description of the EE
- **Organization:** This is the organization that can use the EE, leave it blank if you want to make the EE globally available.
- **Registry credential:** If your registry needs authentication, you need to create a _Credential_ with the type of _Container Registry_ and select it here_._

After creating the EE, you can edit one of your _templates_ and select the _execution environment_ for your newly created EE image.