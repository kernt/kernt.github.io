# Introduction

In the realm of DevOps and infrastructure as code (IaC), testing automation is a critical component to ensure the reliability and efficiency of configurations. Ansible, a powerful open-source automation tool, allows users to define and manage infrastructure as code through playbooks. Molecule, an extension of Ansible, facilitates the testing of Ansible roles in an isolated environment providing a consistent and reproducible testing workflow.

One common scenario involves using Docker containers as test hosts within Molecule. Docker containers offer lightweight, portable environments that can be easily spun up and torn down, making them ideal for testing purposes. In this article, we will explore a Molecule setup utilizing Docker containers for the create, converge, and destroy steps.
# Molecule Configuration

The Molecule configuration is defined in the `molecule.yml` file. This configuration specifies the test platforms and dependencies. In this example, the dependency is set to the Galaxy role, and the platforms include a Docker container based on the Ubuntu 22.04 image.

```yaml
dependency:  
  name: galaxy  
  options:  
    requirements-file: requirements.yml  
platforms:  
  - name: molecule-ubuntu  
    image: ubuntu:22.04
```

The `requirements.yml` file lists the necessary Ansible collections, in this case, the `community.docker` collection.

```yaml
collections:
  - community.docker
```
# Create Playbook

The `create.yml` playbook is responsible for creating Docker containers based on the defined platforms. It uses the `community.docker.docker_container` Ansible module to start containers with a specified image and other parameters. If the containers are not running, the playbook fails, and detailed information is printed.

The playbook also dynamically adds the created containers to the Molecule inventory.

```yaml
- name: Create  
  hosts: localhost  
  gather_facts: false  
  vars:  
    molecule_inventory:  
      all:  
        hosts: {}  
        molecule: {}  
  tasks:  
    - name: Create a container  
      community.docker.docker_container:  
        name: "{{ item.name }}"  
        image: "{{ item.image }}"  
        state: started  
        command: sleep 1d  
        log_driver: json-file  
      register: result  
      loop: "{{ molecule_yml.platforms }}"  
  
    - name: Print some info  
      ansible.builtin.debug:  
        msg: "{{ result.results }}"  
  
    - name: Fail if container is not running  
      when: >  
        item.container.State.ExitCode != 0 or  
        not item.container.State.Running  
      ansible.builtin.include_tasks:  
        file: tasks/create-fail.yml  
      loop: "{{ result.results }}"  
      loop_control:  
        label: "{{ item.container.Name }}"  
  
    - name: Add container to molecule_inventory  
      vars:  
        inventory_partial_yaml: |  
          all:  
            children:  
              molecule:  
                hosts:  
                  "{{ item.name }}":  
                    ansible_connection: community.docker.docker  
      ansible.builtin.set_fact:  
        molecule_inventory: >  
          {{ molecule_inventory | combine(inventory_partial_yaml | from_yaml, recursive=true) }}  
      loop: "{{ molecule_yml.platforms }}"  
      loop_control:  
        label: "{{ item.name }}"  
  
    - name: Dump molecule_inventory  
      ansible.builtin.copy:  
        content: |  
          {{ molecule_inventory | to_yaml }}  
        dest: "{{ molecule_ephemeral_directory }}/inventory/molecule_inventory.yml"  
        mode: "0600"  
  
    - name: Force inventory refresh  
      ansible.builtin.meta: refresh_inventory  
  
    - name: Fail if molecule group is missing  
      ansible.builtin.assert:  
        that: "'molecule' in groups"  
        fail_msg: |  
          molecule group was not found inside inventory groups: {{ groups }}  
      run_once: true # noqa: run-once[task]

# we want to avoid errors like "Failed to create temporary directory"  
- name: Validate that inventory was refreshed  
  hosts: molecule  
  gather_facts: false  
  tasks:  
    - name: Check uname  
      ansible.builtin.raw: uname -a  
      register: result  
      changed_when: false  
  
    - name: Display uname info  
      ansible.builtin.debug:  
        msg: "{{ result.stdout }}"
```

The `tasks/create-fail.yml` file is included in case of container creation failure. It retrieves and displays the container logs.

```yaml
- name: Retrieve container log  
  ansible.builtin.command:  
    cmd: >-  
      {% raw %}  
      docker logs  
      {% endraw %}  
      {{ item.stdout_lines[0] }}  
  changed_when: false  
  register: logfile_cmd  
  
- name: Display container log  
  ansible.builtin.fail:  
    msg: "{{ logfile_cmd.stderr }}"
```
# Converge Playbook

The `converge.yml` playbook checks for the existence of the "molecule" group in the inventory and then performs the convergence steps on the created containers.

```yaml
- name: Fail if molecule group is missing  
  hosts: localhost  
  tasks:  
    - name: Print some info  
      ansible.builtin.debug:  
        msg: "{{ groups }}"  
  
    - name: Assert group existence  
      ansible.builtin.assert:  
        that: "'molecule' in groups"  
        fail_msg: |  
          molecule group was not found inside inventory groups: {{ groups }}  
  
- name: Converge  
  hosts: molecule  
  gather_facts: false  
  tasks:  
    - name: Check uname  
      ansible.builtin.raw: uname -a  
      register: result  
      changed_when: false  
  
    - name: Print some info  
      ansible.builtin.assert:  
        that: result.stdout | regex_search("^Linux")
```
# Destroy Playbook

The `destroy.yml` playbook is responsible for stopping and removing the Docker containers and cleaning up the dynamic Molecule inventory.

```yaml
- name: Destroy molecule containers  
  hosts: molecule  
  gather_facts: false  
  tasks:  
    - name: Stop and remove container  
      delegate_to: localhost  
      community.docker.docker_container:  
        name: "{{ inventory_hostname }}"  
        state: absent  
        auto_remove: true  
  
- name: Remove dynamic molecule inventory  
  hosts: localhost  
  gather_facts: false  
  tasks:  
    - name: Remove dynamic inventory file  
      ansible.builtin.file:  
        path: "{{ molecule_ephemeral_directory }}/inventory/molecule_inventory.yml"  
        state: absent
```
