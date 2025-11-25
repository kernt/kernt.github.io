---
tags:
  - konfigurationsmanagement
  - ansible
  - ansible-policy
---

# ansible-policy

Introduction

There’s a new Ansible feature in town, and I’m thrilled to announce the Ansible Policy utility, which can directly interact with the Open Policy Agent (OPA). 
If you’re familiar with Kubernetes, you might already know OPA’s popularity for specifying rules and validating projects against them.

This utility is particularly beneficial for organizations that need to ensure their projects comply with specific policies. 
For instance, in this example, I’ll validate my playbook against a policy that restricts AWS EC2 machine allocation to the US East 1 and US East 2 regions. 
As you’ll see, my playbook isn’t compliant, as specified on lines 15 and 29, because it attempts to allocate machines in different regions.

By flagging non-compliance, we prevent the execution of configurations that don’t meet our standards. So, why haven’t you heard about Ansible Policy? Because it’s a new prototype implementation that allows you to define and set OPA rules within your application using the Ansible Policy utility.

nstallation Guide
To install the Ansible Policy command, follow these steps:

Set Up a Python Virtual Environment: Since this is an experimental feature, it’s better to avoid cluttering your system with additional libraries.

python3 -m venv ansible-policy-env
source ansible-policy-env/bin/activate

ansible-policy Manual Installation: The Ansible Policy utility isn’t available in the PyPI repository yet, so you’ll need to install it manually.

pip install -e .
This command will download all necessary libraries and install them.

Dependency Management: You might encounter a few roadblocks if all dependencies aren’t satisfied. First, deactivate and reactivate the virtual environment.
deactivate
source ansible-policy-env/bin/activate
Install Ansible: Ensure Ansible is installed within your virtual environment.

pip install ansible

Install Open Policy Agent: The OPA must be installed on your machine. MacOS: Use Homebrew.

brew install opa

Now, the ansible-policy command should be available. If you invoke it without parameters, ensure to use --help or the correct options to avoid exceptions.

Using Ansible Policy
With Ansible Policy installed, you can specify configurations like ansible.config, the target directory, and the policy directory. Here’s an example configuration file and a non-validated playbook specifying packages below the region in my AWS instance:

*policybook.yml*

```yaml
policies:
  - name: "AWS Region Policy"
    description: "Ensure instances are in allowed regions"
    rules:
      - name: "allowed-regions"
        conditions:
          - key: "region"
            values: ["us-east-1", "us-east-2"]
```

*playbook.yml*

```sh
- name: Create EC2 instance
  hosts: localhost
  tasks:
    - name: Launch instance
      ec2:
        region: "{{ region }}"
        ...
```

Executing the policy validation against this playbook would result in an error if the region isn’t compliant.

`ansible-policy --policy-dir policies --project-dir project`

output

(venv) lberton@Lucas-MBP policy % ansible-policy --policy-dir policies --project-dir project
TASK [Create EC2 instance] project/playbook.yml L15-29 ************************
... Check_for_ec2_instance_region Not Validated
    The instance region {{ region }} is not allowed, you can only deploy to one of these ["us-east-1", "us-east-2"]

---------------------------------------------------------------------
SUMMARY
... Total files: 1, Validated: 0, Not Validated: 1

Violations are detected! in 1 task

(venv) lberton@Lucas-MBP policy %

Conclusion
The Ansible Policy utility is a powerful tool for enforcing compliance and ensuring your configurations meet organizational standards. By integrating with OPA, it provides flexible, fine-grained control over your automation scripts. I can’t wait to hear your automation stories and use cases with the policy book.

Let’s automate more!

For more information, check out:

Open Policy Agent

* [github ansible-policy](https://github.com/ansible/ansible-policy)