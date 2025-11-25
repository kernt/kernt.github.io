## Streamlining Ansible Development with Molecule: A Comprehensive Installation Guide

# Introduction

Ansible, a powerful open-source automation tool, has become a staple for IT administrators and developers alike. With its widespread adoption, there’s a growing need for a robust testing and development framework to ensure the reliability and efficiency of Ansible content collections. Molecule, the testing framework for Ansible roles and playbooks, now has a developer preview that aligns closely with Ansible content collection development.

In this article, we will guide you through the installation of Molecule developer preview and demonstrate how it can be integrated seamlessly into your Ansible content collection development workflow.
# Installing Molecule

To install Molecule, the open-source testing framework for Ansible roles and playbooks, follow these steps. Firstly, ensure you have Python installed, preferably version 3.9 or higher, along with Ansible-core version 2.12 or later. Depending on your chosen driver, additional OS packages may be required. For CentOS and Ubuntu, you can install the necessary packages using the following:

`sudo dnf install -y gcc python3-pip python3-devel openssl-devel python3-libselinux  # For CentOS`

Next, use pip, the only supported installation method, to install Molecule and Ansible-core:

`python3 -m pip install molecule ansible-core`

Please note that Ansible is not listed as a direct dependency of the Molecule package, as it is called a command-line tool. If needed, install Ansible using your distribution’s package installer separately. Also, be cautious if using SELinux-supporting systems, as issues may arise even if SELinux is not enabled or is configured to be permissive.

To ensure a successful installation, it is recommended to install Molecule in a virtual environment. This can be done by running:

```sh
python3 -m pip install --upgrade --user setuptools  # Upgrade setuptools in a virtual environment  
python3 -m pip install --user molecule
```

Molecule does not include ansible-lint, but you can install it separately with:

`python3 -m pip install --user molecule ansible-lint`

By default, Molecule uses the “delegated” driver. Install other drivers separately from PyPI if needed, such as podman:

`python3 -m pip install --user "molecule-plugins[podman]"`

Remember that if you’re upgrading Molecule from previous versions, remove previously installed drivers like `molecule-podman` or `molecule-vagrant` since they are now available in the molecule-plugins package. Once installed, the Molecule script is typically available in the PATH. Still, it can also be called a Python module using `python -m molecule ...`. This alternative method offers benefits such as explicit control over the Python interpreter used by Molecule and installation at the user level without needing the script in the PATH. With these steps, you have successfully installed Molecule and are ready to enhance your Ansible role and playbook development workflow.
## Molecule Developer Preview

The molecule is conveniently packaged as part of the Ansible Automation Platform. To install Molecule, use the following command:

`dnf install --enablerepo=ansible-automation-platform-2.4-for-rhel-8-x86_64-rpms molecule`

This command fetches and installs Molecule, making it ready for use in your Ansible projects.
# Getting Started with Molecule

## Creating a Collection

When developing Ansible content collections, it’s recommended to organize them under the `collections/ansible_collections` directory. Let's create a sample collection called `foo.bar`:

`ansible-galaxy collection init foo.bar`

Navigate to the roles directory within your new collection:

`cd <path to your collection>/foo.bar/roles/`

Initialize a new role for this collection:

`ansible-galaxy role init my_role`

# Adding a Task to the Role

Add a task under `my_role/tasks/main.yml` to test Molecule within the role:

```yaml
---  
- name: Task is running from within the role  
  ansible.builtin.debug:  
    msg: "This is a task from my_role."
```

# Integrating Molecule into the Content Collection

To integrate Molecule into your content collection, follow these steps:

1. Create a new directory named `extensions` in your collection:

```sh
mkdir <path to your collection>/extensions/
```

1. Navigate to the new `extensions` directory:

`cd <path to your collection>/extensions/`

1. Initialize the default Molecule scenario:

molecule init scenario

1. Edit the `molecule.yml` file within `<path_to_your_collection>/extensions/molecule/default/` and add the following entry:

```yaml
provisioner:  
  name: ansible  
  config_options:  
    defaults:  
      collections_path: ${ANSIBLE_COLLECTIONS_PATH}
```

1. Set the `ANSIBLE_COLLECTIONS_PATH` environment variable before running Molecule:

`export ANSIBLE_COLLECTIONS_PATH=/home/user/working/collections`

Ensure that the path reflects the location up to the `collections` directory, not the `ansible_collections` directory.

With these steps, you’ve successfully integrated Molecule into your Ansible content collection development environment. Now, you can leverage Molecule to test and validate your roles and playbooks more effectively, ensuring a seamless automation experience.
# Links

- [https://ansible.readthedocs.io/projects/molecule/](https://ansible.readthedocs.io/projects/molecule/)
