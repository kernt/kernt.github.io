Elevating Ansible Development through Systematic Testing and Iterative Refinement in Molecule Scenarios
https://youtu.be/RKk2HilVOn8
# Introduction

Molecule, the testing framework for Ansible roles and playbooks, introduces powerful functionality through its concept of scenarios.
Think of a scenario as a test suite for roles or playbooks within an Ansible collection.
This article delves into Molecule scenarios, their layout, and how to harness their capabilities for efficient testing and development.
# Ansible Collection

Adding Molecule to your Ansible collection is a straightforward process that enhances your development and testing workflow.
Start by creating a new directory within your collection named “`extensions`.”
Navigate to this newly created directory using the command line and then initialize a new default Molecule scenario with the following:

```sh
cd "{{ ansible_collection_name }}"/extensions/  
molecule init scenario
```

This step sets the foundation for incorporating Molecule into your collection, allowing you to seamlessly integrate testing and development practices. The newly created scenario within the “extensions” directory becomes a pivotal component in orchestrating Molecule’s testing lifecycle for your Ansible roles and playbooks.
# Understanding the Scenario Layout

The scenario layout is crucial for organizing Molecule’s testing components. Within the `molecule/default` folder, several key files play distinct roles:

- `create.yml`: This playbook file creates instances and stores data in the instance-config.
- `destroy.yml`: Contains Ansible code for destroying instances and removing them from the instance-config.
- `molecule.yml`: The central configuration entry point for Molecule per scenario, allowing configuration of each tool for testing roles.
- `converge.yml`: A playbook file invoking your role. Molecule uses this file with `ansible-playbook` to run against an instance created by the driver.
# Inspecting the molecule.yml Configuration

The `molecule.yml` file is pivotal for configuring Molecule. It's a YAML file with keys representing high-level components:

1. Dependency Manager: Molecule defaults to using the Galaxy development guide to resolve role dependencies.
2. Platforms Definitions: Specifies instances to create, their names, and group assignments. Useful for testing roles against multiple distributions.
3. Provisioner: Molecule’s Ansible provisioner manages instance lifecycles based on this configuration.
4. Scenario Definition: Controls the sequence order of scenarios.
5. Verifier Framework: Defaults to using Ansible, providing a way to write state-checking tests on the target instance.
# Running a Full Test Sequence

Molecule offers commands for manually managing instance, scenario, development, and testing tools. Alternatively, automate these tasks within a scenario sequence using `molecule test`. The full lifecycle sequence includes dependency resolution, cleanup, instance creation, preparation, convergence, idempotence, side effect, verification, and final cleanup.

```sh
Usage: molecule [OPTIONS] COMMAND [ARGS]...
  Molecule aids in the development and testing of Ansible roles.
  To enable autocomplete for a supported shell execute command below after replacing
  SHELL with either bash, zsh, or fish:

eval "$(_MOLECULE_COMPLETE=SHELL_source molecule)"
  
Options:
  --debug / --no-debug    Enable or disable debug mode. Default is disabled.  
  -v, --verbose           Increase Ansible verbosity level. Default is 0.  
  -c, --base-config TEXT  Path to a base config (can be specified multiple times). If
                          provided, Molecule will first load and deep merge the  
                          configurations in the specified order, and deep merge each
                          scenario's molecule.yml on top. By default Molecule is  
                          looking for '.config/molecule/config.yml' in current VCS  
                          repository and if not found it will look in user home.  
                          (None).  
  -e, --env-file TEXT     The file to read variables from when rendering molecule.yml.
                          (.env.yml)  
  --version
  --help                  Show this message and exit.  
  
Commands:  
  check        Use the provisioner to perform a Dry-Run (destroy, dependency,...  
  cleanup      Use the provisioner to cleanup any changes made to external systems...  
  converge     Use the provisioner to configure instances (dependency, create,...  
  create       Use the provisioner to start the instances.  
  dependency   Manage the role's dependencies.  
  destroy      Use the provisioner to destroy the instances.  
  drivers      List drivers.  
  idempotence  Use the provisioner to configure the instances and parse the output...  
  init         Initialize a new scenario.
  list         List status of instances.
  login        Log in to one instance.
  matrix       List matrix of steps used to test instances.
  prepare      Use the provisioner to prepare the instances into a particular...  
  reset        Reset molecule temporary folders.
  side-effect  Use the provisioner to perform side-effects to the instances.  
  syntax       Use the provisioner to syntax check the role.
  test         Test (dependency, cleanup, destroy, syntax, create, prepare,...  
  verify       Run automated tests against instances.
```
# Testing the Collection Role

The `converge.yml` file, created during initialization, is a playbook to run your role from start to finish. Modify as needed, but it's an excellent starting point for Molecule novices. Create an isolated test environment and utilize it for live development with `molecule converge`. This command stops after the converge action, enabling iterative testing while making changes to your collection or the converge play.

As an example, consider testing the role with the following code in `converge.yml`:

```yaml
---  
- name: Include a role from a collection  
  hosts: localhost  
  gather_facts: false  
  tasks:  
    - name: Testing role  
      ansible.builtin.include_role:  
        name: foo.bar.my_role  
        tasks_from: main.yml
```

Run `molecule converge` to execute the steps in the default scenario while maintaining the infrastructure for ongoing development and testing. This iterative process ensures a robust and reliable Ansible automation workflow.
# Links
- [https://ansible.readthedocs.io/projects/molecule/](https://ansible.readthedocs.io/projects/molecule/)