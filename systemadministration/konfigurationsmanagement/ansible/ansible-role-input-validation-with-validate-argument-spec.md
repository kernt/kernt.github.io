---
tags:
  - konfigurationsmanagement
  - ansible
  - validation
  - input
  - argument
  - spec
---

# Ansible role input Validation

Ensuring the integrity of inputs in automation workflows is crucial, especially in a dynamic and modular framework like Ansible.
The Ansible `validate_argument_spec` function, part of the `ansible.builtin` collection, is a powerful tool for enforcing parameter validation directly within roles, improving both security and reliability.

This article provides an end-to-end guide to using `validate_argument_spec` in Ansible roles to validate parameters, avoid errors, and maintain consistent data standards across tasks. By integrating `validate_argument_spec`, you can ensure that your Ansible roles receive correct, type-validated, and requirement-compliant inputs before executing automation tasks.

# Why Use `validate_argument_spec`?

With `validate_argument_spec`, Ansible users can define parameter requirements such as types, default values, allowed choices, and required fields in a centralized argument specification. This provides several benefits:

1. **Error Prevention**: Catches invalid inputs early, preventing runtime failures.
2. **Readability and Consistency**: Centralizes the validation logic, making role parameters easier to understand and maintain.
3. **Reusable Validation Logic**: Creates modular validation that can be applied across multiple roles or tasks.

# How `validate_argument_spec` Works

The `validate_argument_spec` function validates inputs against a structured argument specification.
This spec defines the type, constraints, and optionality of each argument, ensuring that only appropriate data is passed to tasks within the role.
It simplifies validation by eliminating the need for custom validation logic within tasks, using a predefined specification file instead.

In this example, we’ll create a sample role that uses `validate_argument_spec` to validate parameters such as `name`, `state`, and `region`.

# Setting Up `validate_argument_spec` in an Ansible Role

Below is a guide for setting up a role with input validation using `validate_argument_spec`.

## Step 1: Define the Role Structure

First, set up the Ansible role with the following structure:

```sh
roles/  
├── my_role/  
│   ├── tasks/  
│   │   ├── main.yml  
│   │   └── validate.yml  # Separate validation task  
│   ├── defaults/  
│   │   └── main.yml  
│   └── argument_spec.yml
```

This structure includes:

- `**argument_spec.yml**`: Defines the expected arguments and their validation rules.
- `**tasks/main.yml**`: Contains the main logic of the role, including a call to the validation step.
- `**tasks/validate.yml**`: A separate task file to include `validate_argument_spec` for validation.
- `**defaults/main.yml**`: Provides default values for the arguments.

## Step 2: Define the Argument Specification (`argument_spec.yml`)

In `argument_spec.yml`, define the parameters the role expects, including types, required fields, default values, and any allowed choices.

*argument_spec.yml*

```yaml
my_role_args:  
  name:  
    type: str  
    required: true  
    description: "Name of the resource."  
  state:  
    type: str  
    default: "present"  
    choices: ["present", "absent"]  
    description: "Desired state of the resource."  
  region:  
    type: str  
    required: false  
    description: "Region where the resource should be managed."
```

Explanation:

- `**name**`: A required string parameter that represents the resource name.
- `**state**`: Optional, with a default value of `"present"` and limited to `"present"` or `"absent"`.
- `**region**`: An optional string that represents the resource's geographic region.

## Step 3: Set Up the Validation Task (`tasks/validate.yml`)

In `validate.yml`, use `validate_argument_spec` to validate the `my_role_args` input against `argument_spec.yml`.

```yaml
# tasks/validate.yml  
- name: Load argument specifications  
  include_vars:  
    file: "../argument_spec.yml"  
    name: arg_spec  
- name: Validate arguments with validate_argument_spec  
  validate_argument_spec:  
    argument_spec: "{{ arg_spec.my_role_args }}"  
    provided_arguments: "{{ my_role_args }}"  
  register: validation_output  
- name: Display validation results  
  debug:  
    msg: "{{ validation_output.msg }}"
```

Explanation:

- **Load Argument Spec**: The first task loads the argument specification (`arg_spec.my_role_args`) from `argument_spec.yml`.
- **Validation**: `validate_argument_spec` checks if `my_role_args` meets the defined argument specification.
- **Result Output**: Displays the validation results to confirm if the arguments meet the specification.

## Step 4: Main Task File (`tasks/main.yml`)

In `main.yml`, use `include_tasks` to load `validate.yml` for validation, followed by other tasks if validation is successful.

```yaml
# tasks/main.yml  
- name: Validate role arguments  
  include_tasks: validate.yml  
  when: arg_spec is defined  
- name: Perform main task after validation  
  debug:  
    msg: "Resource {{ my_role_args.name }} is set to {{ my_role_args.state }} in region {{ my_role_args.region | default('not specified') }}"
```

Here:

- **Validation Inclusion**: Uses `include_tasks` to include `validate.yml` and perform validation.
- **Main Task Execution**: Executes main tasks only if validation passes.

## Step 5: Define Default Values (`defaults/main.yml`)

Defaults provide baseline values that users can override when calling the role.

```yaml
# defaults/main.yml  
my_role_args:  
  name: "default_name"  
  state: "present"  
  region: "us-west-1"
```

## Step 6: Example Playbook

The following playbook demonstrates how to call `my_role` and pass custom arguments.

```yaml
# example_playbook.yml
- hosts: localhost
  roles:
    - role: my_role
      my_role_args:
        name: "example_resource"
        state: "absent"
        region: "us-east-1"
```

This playbook passes specific values to `my_role`, which `validate_argument_spec` will validate before executing any tasks within the role.

# Benefits of Using `validate_argument_spec`

Implementing `validate_argument_spec` in Ansible roles provides several key advantages:

1. **Error Prevention**: Ensures only valid data reaches critical tasks, reducing the likelihood of runtime failures.
2. **Simplified Code**: Validation logic is encapsulated in a single argument specification file, making the codebase cleaner and more maintainable.
3. **Reusable Validation**: The argument spec can be reused across multiple roles, providing consistent validation across projects.
4. **Enhanced Debugging**: Clear error messages from `validate_argument_spec` make it easy to identify and correct input issues.

# Conclusion

The `validate_argument_spec` function in Ansible offers a robust way to enforce input validation, ensuring that roles receive correct and standardized inputs.
By using `validate_argument_spec` within roles, you can create automation workflows that are both reliable and maintainable.
Integrating it into your Ansible roles can significantly improve both the security and functionality of your infrastructure as code.

Whether building roles for internal use or sharing them on Ansible Galaxy, `validate_argument_spec` is an essential tool for creating dependable automation in complex environments.

# Video Course

![](https://miro.medium.com/v2/resize:fit:700/0*7qM3pCSS1P_AJYhJ)

- [Udemy: Learn Ansible Automation in 250+examples & practical lessons: Learn Ansible with some real-life examples of how to use the most common modules and Ansible Playbook](https://click.linksynergy.com/deeplink?id=euGmLrdj*Ec&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fansible-by-examples-devops%2F%3FreferralCode%3D8E065F6D6F8622A3DEC8)

![](https://miro.medium.com/v2/resize:fit:700/0*6czvITprkoZcMWEV.png)

- [Udemy: Terraform for Beginners: Code, Deploy, and Scale: A Practical Approach for Beginners to Learn Cloud Infrastructure with Terraform](https://click.linksynergy.com/deeplink?id=euGmLrdj*Ec&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fterraform-for-beginners-code-deploy-and-scale%2F%3FreferralCode%3D39F3B3F1A91F00BE8EFD)

# Printed Book

![](https://miro.medium.com/v2/resize:fit:100/0*00gvOYtoFwLezjNO)

- [Ansible For VMware by Examples](https://amzn.to/3qes2hm)

![](https://miro.medium.com/v2/resize:fit:100/0*5F7ueyTizDTQwGcX)

- [Ansible for Kubernetes by Example](https://amzn.to/3OlAuU5)

![](https://miro.medium.com/v2/resize:fit:100/0*SgwPSesmohVDcQUP)

- [Hands-on Ansible Automation](https://amzn.to/3qoLQyy)

![](https://miro.medium.com/v2/resize:fit:99/0*Fjl3_QcAa_HTPmkb)

- [Red Hat Ansible Automation Platform](https://amzn.to/41K0cbm)

# eBooks

![](https://miro.medium.com/v2/resize:fit:100/0*6bTLlWdLHfTi65V8.png)

- [Buy on Leanpub Terraform By Example: A Practical Approach for Beginners to Learn Cloud Infrastructure with Terraform](https://leanpub.com/terraform-by-example)

![](https://miro.medium.com/v2/resize:fit:100/0*zWQW2tMJOZZ4QHg1)

- [Ansible by Examples: 200+ Automation Examples For Linux and Windows System Administrator and DevOps](https://leanpub.com/ansiblebyexamples)

![](https://miro.medium.com/v2/resize:fit:100/0*sSyoz5P_idMcm4a-)

- [Ansible Cookbook: A Comprehensive Guide to Unleashing the Power of Ansible via Best Practices, Troubleshooting, and Linting Rules with Luca Berton](https://leanpub.com/ansible-cookbook)

![](https://miro.medium.com/v2/resize:fit:100/0*nAALGoXM8fYFiBRa)

- [Ansible For Windows By Examples: 50+ Automation Examples For Windows System Administrator And DevOps](https://leanpub.com/ansibleforwindowsbyexamples)

![](https://miro.medium.com/v2/resize:fit:100/0*K_7AH1TGx7paJL8d)

- [Ansible For Linux by Examples: 100+ Automation Examples For Linux System Administrator and DevOps](https://leanpub.com/ansibleforlinuxbyexamples)

![](https://miro.medium.com/v2/resize:fit:100/0*1bNTL48qo9TuOOHW)

- [Ansible Linux Filesystem By Examples: 40+ Automation Examples on Linux File and Directory Operation for Modern IT Infrastructure](https://leanpub.com/linuxfileanddirectorybyansibleexamples)

![](https://miro.medium.com/v2/resize:fit:100/0*ucXD6kcIp1boo9kD)

- [Ansible For Security by Examples: 100+ Automation Examples to Automate Security and Verify Compliance for IT Modern Infrastructure](https://leanpub.com/ansibleforsecuritybyexamples)

![](https://miro.medium.com/v2/resize:fit:100/0*BxXPASNUQb-wxE9p)

- [Ansible Tips and Tricks: 10+ Ansible Examples to Save Time and Automate More Tasks](https://leanpub.com/ansible-tips-and-tricks)

![](https://miro.medium.com/v2/resize:fit:100/0*lrH07K6zTDHttClD)

- [Ansible Linux Users & Groups By Examples: 20+ Automation Examples on Linux Users and Groups Operation for Modern IT Infrastructure](https://leanpub.com/ansiblelinuxusersandgroupsbyexamples)

![](https://miro.medium.com/v2/resize:fit:100/0*EhRl9f_4WV7wFYDW)

- [Ansible For PostgreSQL by Examples: 10+ Examples To Automate Your PostgreSQL database](https://leanpub.com/ansible-for-postgresql-by-examples)

![](https://miro.medium.com/v2/resize:fit:100/0*uJMJ3EYESfeGms2w)

- [Ansible For Amazon Web Services AWS By Examples: 10+ Examples To Automate Your AWS Modern Infrastructure](https://leanpub.com/ansible-for-aws-by-examples)

![](https://miro.medium.com/v2/resize:fit:100/0*JfsPzbt4vnt_pqM3)

- [Ansible Automation Platform By Example: A step-by-step guide for the most common user scenarios](https://leanpub.com/Ansible-Automation-Platform/)