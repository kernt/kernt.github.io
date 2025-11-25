# Introduction

In the rapidly evolving world of software development, Continuous Integration/Continuous Deployment (CI/CD) has become a cornerstone of modern DevOps practices. Jenkins, one of the most popular open-source automation servers, facilitates CI/CD by automating the build, test, and deployment phases of the software development process. However, setting up Jenkins can be a repetitive and time-consuming task, which is where Ansible, an automation tool for configuration management, comes into play.

This article provides a step-by-step guide on how to automate the installation and initial setup of Jenkins on a server using Ansible, making it a reproducible and error-free process.

# Prerequisites

Before we dive into the Ansible playbook, ensure you have the following prerequisites met:

- An Ansible control node configured to manage your servers.
- A target server, which we refer to as `jenkins_server` in our inventory.
- The target server must be running a Ubuntu/Debian-based operating system since the playbook uses `apt` for package management.

# The Ansible Playbook for Jenkins Installation

An Ansible playbook is a blueprint of automation tasks, which are executed in the order they are defined. The playbook we are discussing is composed of a series of tasks to install Java, add the Jenkins repository, install Jenkins, and ensure the service is running.

## Step 1: Installing Java

Jenkins is a Java-based application, so the first task in our playbook installs Java:

```yaml
- name: Install Java  
  ansible.builtin.apt:  
    name: default-jdk  
    state: present
```

This task uses the `ansible.builtin.apt` module to install the default Java Development Kit (JDK) package.

## Step 2: Adding the Jenkins Repository Key

To install the latest stable version of Jenkins, we need to add the repository key to our system:

```yaml
- name: Add Jenkins repository key  
  ansible.builtin.apt_key:  
    url: https://pkg.jenkins.io/debian/jenkins.io.key  
    state: present
```

This ensures that the packages downloaded from the Jenkins repository are verified and trusted.

## Step 3: Adding the Jenkins Repository

Next, we add the actual Jenkins repository from where the Jenkins package will be downloaded:

```yaml
- name: Add Jenkins repository  
  ansible.builtin.apt_repository:  
    repo: deb https://pkg.jenkins.io/debian-stable binary/  
    state: present
```

This makes the latest Jenkins releases available to our package manager.

## Step 4: Installing Jenkins

With the repository in place, we can proceed to install Jenkins:

```yaml
- name: Install Jenkins  
  ansible.builtin.apt:  
    name: jenkins  
    state: present  
    update_cache: true
```

This task not only installs Jenkins but also updates the package cache to ensure we get the latest version available.

## Step 5: Starting and Enabling Jenkins

Finally, we need to ensure that the Jenkins service is started and enabled to start on boot:

```yaml
- name: Start and enable Jenkins  
  ansible.builtin.service:  
    name: jenkins  
    state: started  
    enabled: true

```
This task uses the `ansible.builtin.service` module to manage the Jenkins service.

# Playbook

```yaml
---  
- name: Install and start Jenkins  
  hosts: jenkins_server  
  become: true  
  tasks:  
    - name: Install Java  
      ansible.builtin.apt:  
        name: default-jdk  
        state: present  
  
    - name: Add Jenkins repository key  
      ansible.builtin.apt_key:  
        url: https://pkg.jenkins.io/debian/jenkins.io.key  
        state: present  
  
    - name: Add Jenkins repository  
      ansible.builtin.apt_repository:  
        repo: deb https://pkg.jenkins.io/debian-stable binary/  
        state: present  
  
    - name: Install Jenkins  
      ansible.builtin.apt:  
        name: jenkins  
        state: present  
        update_cache: true  
  
    - name: Start and enable Jenkins  
      ansible.builtin.service:  
        name: jenkins  
        state: started  
        enabled: true
```

