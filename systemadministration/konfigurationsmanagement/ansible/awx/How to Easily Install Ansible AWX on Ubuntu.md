Ansible AWX is an open-source web-based user interface, REST API, and task engine designed to make IT automation simple. It serves as the upstream project for Red Hat’s Ansible Tower, offering users a powerful, centralized platform to manage Ansible playbooks and inventories, monitor jobs, and streamline workflows.

If you’re looking for a lightweight yet feature-rich automation tool for your DevOps needs, AWX is an excellent choice.

This guide will explain the steps to easily install Ansible AWX on Ubuntu, covering all necessary configurations and commands.

# Understanding Ansible AWX

Ansible AWX provides all the functionality of Ansible Tower without the cost, making it ideal for small teams and individual developers. It allows you to:

- Manage playbooks and inventories visually.
- Set up role-based access control.
- Monitor real-time job execution and logs.

Before proceeding with the installation, ensure your system meets the requirements and has the necessary software pre-installed.

# Pre-requisites for Installing AWX on Ubuntu

## **System Requirements**

1. **Supported Ubuntu Version:** Ubuntu 20.04 or later.
2. **Hardware Requirements:**

- CPU: Dual-core processor or higher.
- RAM: At least 4GB (8GB recommended).
- Disk Space: At least 20GB of free space.

## **Software Requirements**

The following tools must be installed before you begin:

- **Docker and Docker Compose:** Required to containerize AWX.
- **Python and Pip:** Essential for running Ansible.
- **Other Tools:** git, curl, and ansible.

Run these commands to install the prerequisites:

```sh
sudo apt update && sudo apt upgrade -y  
sudo apt install -y python3 python3-pip docker.io docker-compose git curl ansible
```

Verify the Docker installation:

```sh
docker --version  
docker-compose --version
```

# Installing Ansible AWX on Ubuntu

## **Step 1: Update and Prepare the System**

Begin by ensuring your system is up-to-date:

`sudo apt update && sudo apt upgrade -y`

Install essential tools if they are not already installed:

`sudo apt install -y git curl`

## **Step 2: Install Docker and Docker Compose**

Install Docker if it’s not already installed:

`sudo apt install -y docker.io`

Start and enable the Docker service:

```sh
sudo systemctl start docker  
sudo systemctl enable docker
```

Install Docker Compose:

`sudo apt install -y docker-compose`

Verify the installations:

```sh
docker --version  
docker-compose --version
```

## **Step 3: Clone the AWX Repository**

Clone the official AWX repository:

```sh
git clone https://github.com/ansible/awx.git  
cd awx/installer
```

## **Step 4: Configure AWX Inventory File**

The `inventory` file is used to define the configuration settings for your AWX instance. Open the file for editing:

`nano inventory`

Customize the following variables as per your requirements:

- **Postgres Settings:**  
    Replace the default values with your preferred database username and password:

```
postgres_data_dir=/var/lib/pgdocker  
postgres_username=awx  
postgres_password=securepassword
```

- **Admin Credentials:**  
    Set the default admin username and password:

```
admin_user=admin  
admin_password=adminpassword
```

Save and close the file.

## **Step 5: Install Ansible AWX**

Run the installation playbook using Ansible:

`ansible-playbook -i inventory install.yml`

This command will download and set up the necessary Docker containers for AWX, including Postgres, Redis, and the AWX application itself.

# Accessing the AWX Web Interface

Once the installation is complete, you can access the AWX dashboard through your web browser. Use the IP address or hostname of your server followed by port 80. For example:

http://your-server-ip/

Log in using the credentials you defined in the inventory file (`admin_user` and `admin_password`).

# Common Troubleshooting Tips

- **Docker Errors:** Ensure the Docker service is running:

`sudo systemctl restart docker`

- **Firewall Issues:** Allow access to AWX via port 80:

`sudo ufw allow 80/tcp`

- **Missing Dependencies:** Reinstall Python dependencies with Pip if you encounter errors:

`pip3 install -r requirements.txt`

