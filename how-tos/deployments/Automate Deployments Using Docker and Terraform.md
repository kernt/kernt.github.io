The concept of containerization has gained popularity over the past years. This has made most organizations modernize their apps for cloud use. Today, there are many tools that can be used to run containers. These tools include Docker, Podman, Kubernetes etc.

Docker serves as an open-source container runtime engine that empowers you to construct, execute, and oversee containers. Initially launched in 2013, the Docker Engine emerged as a universally accepted solution for packaging applications. It has evolved into a pivotal tool in numerous enterprises, particularly for adapting and creating applications suitable for cloud environments.

Docker is made of the following:

- **Docker Engine**: At the core, this is the runtime engine responsible for the execution and oversight of Docker containers on the host system. It offers an API and a command-line interface (CLI) to interact with containers efficiently.
- **Docker Images**: These function as blueprints outlining the content and setup of a container. Created from declarative variables known as Dockerfiles, images can be versioned and shared across diverse environments.
- **Docker Registry**: A central repository that serves as a storehouse for sharing Docker images. While the default public registry is Docker Hub, private registries can also be established and utilized.
- **Docker Compose**: A valuable tool tailored for managing multi-container applications. Simplifying the process, it employs a single YAML file to define the configuration and interconnections among various containers.

### Introduction to Terraform

**Terraform** on the other hand is an open-source infrastructure as code (IaC) tool developed by HashiCorp. Managing resources on Terraform is done in a declarative manner. This is to say that you only need to describe the desired infrastructure state in the configuration file, and it will create and manage the resources necessary to achieve that state. When Docker and Terraform are used together, it is even easier to automate the deployment of containerized applications on various infrastructure providers.

Here are the benefits of using them together:

- **Container Registry**: Before deploying containers, you’ll typically need a container registry to store and distribute your Docker images. Terraform can automate the provisioning of a container registry like Docker Hub, Amazon Elastic Container Registry (ECR), or Google Container Registry (GCR).
- **Infrastructure Provisioning**: Terraform can be used to define and provision the necessary infrastructure resources required for running Docker containers. This includes virtual machines, networks, storage, load balancers, etc. Terraform supports multiple cloud providers (such as AWS, Azure, GCP) as well as on-premises infrastructure.
- **Deployment Configuration**: Terraform allows you to define the desired deployment configuration, including the number of containers, networking setup, resource requirements, and any other relevant settings. This information is typically specified in the Terraform configuration files (written in HashiCorp Configuration Language or HCL).
- **Container Orchestration**: Terraform can integrate with container orchestration platforms like Kubernetes, Docker Swarm, or Amazon Elastic Container Service (ECS). You can use Terraform to provision the necessary resources for running the container orchestration system and configuring it to manage your Docker containers.
- **Deployment Automation**: Once you have defined the infrastructure and container orchestration setup using Terraform, you can automate the deployment process. Terraform provides a command-line interface (CLI) and various automation options, allowing you to apply the infrastructure changes and deploy your containerized applications with a single command or as part of a CI/CD pipeline.

Today, we will learn how to automate Deployments on Docker with Terraform.

## 1. Install Docker on Your System

Before we proceed, you need to have the Docker Engines installed and running on your system. You can use the dedicated guide below to install Docker on your system.

- [How To Install Docker CE on Linux Systems](https://computingforgeeks.com/install-docker-ce-on-linux-systems/)

Once installed, add your user to the docker group:

```
sudo usermod -aG docker $USER
newgrp docker
```

Verify the installation.

```
$ docker version
Client: Docker Engine - Community
 Version:           24.0.1
 API version:       1.43
 Go version:        go1.20.4
 Git commit:        6802122
 Built:             Fri May 19 18:06:24 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          24.0.1
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.4
....
```

## 2. Install Terraform on Your System

After installing Docker, you also need Terraform installed. The below guides can be used to achieve that:

- [How to Install Terraform](https://computingforgeeks.com/?s=how+to+install+terraform)

Verify the installation with the command:

```
$ terraform  version
Terraform v1.4.6
on linux_amd64
```

## 3. Automate Deployments Using Docker and Terraform

Now with Terraform and Docker installed, we can automate the deployment of containerized applications on various infrastructure providers. For this guide, we will learn how you can automate container deployments with Terraform.

First, create the Terraform Configuration.

```
mkdir ~/terraform-demo-docker && cd ~/terraform-demo-docker
```

We will begin by adding the **_provider.tf_** file to define Docker so that Terraform knows how to interact with it. This configuration also uses the Docker API to manage the lifecycle of Docker containers.

```
vim provider.tf
```

In the file, add the below lines:

```
# Declaring the Required provider (Docker provider)
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}

# Specifying the Docker provider configuration
provider "docker" {
  host = "unix:///var/run/docker.sock"
}
```

For Docker on a **Windows** Machines, you can use the host as shown:

```
........
provider "docker" {
  host = "tcp://localhost:2375"
}
```

Now we can begin to automate deployments by adding more configuration files that define the deployments in the same directory.

### a. Create a Docker Image with Terraform

Creating a Docker Image requires one to have a DockerFile. For this demo, we will create Tesseract Images from the below DockerFile

```
$ vim Dockerfile
FROM ubuntu:20.04
RUN apt-get update -y
ENV DEBIAN_FRONTEND=noninteractive 
RUN apt-get install -y gnupg apt-transport-https apt-utils wget
RUN echo "deb https://notesalexp.org/tesseract-ocr5/focal/ focal main" \
|tee /etc/apt/sources.list.d/notesalexp.list > /dev/null
RUN wget -O - https://notesalexp.org/debian/alexp_key.asc | apt-key add -
RUN apt-get update -y
RUN apt-get install tesseract-ocr -y
RUN apt install imagemagick -y
ENTRYPOINT ["tesseract"]
RUN tesseract -v
```

Once the file has been created, create the **_main.tf_** file:

```
vim main.tf
```

In the file, add the lines below:

```
resource "null_resource" "docker_build" {
  provisioner "local-exec" {
    command = "docker build -t tesseract:5 ."
  }
}
```

The `docker build` command is used to build the Docker image, and `-t` specifies the image name and optionally a tag. The `.` indicates that the build context is the current directory.

To start the deployment use:

```
terraform init
```

Now apply the configuration:

```
terraform plan
terraform apply
```

Accept the process to proceed:

```
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

Sample Output:

![Automate Deployments Using Docker and Terraform](https://computingforgeeks.com/wp-content/uploads/2023/05/Automate-Deployments-Using-Docker-and-Terraform-1024x516.png?ezimgfmt=rs:696x351/rscb23/ng:webp/ngcb23 "Automate Deployments Using Docker and Terraform 1")

After this, you will have the images created.

```
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
tesseract    5         ee4b0010d7b0   29 seconds ago   311MB
```

### b. Run Docker Containers with Terraform

You can also spin docker containers using Terraform, all you need to do is update your **_main.tf_** file. For this guide, we will run two containers, one for Nginx with port 80 and the other for Apache with port 8080 exposed.

So open the file:

```
vim main.tf
```

Add the below lines to it:

```
resource "docker_container" "nginx" {
  name  = "nginx-container"
  image = "nginx"
  ports {
    internal = 80
    external = 80
  }
}

resource "docker_container" "apache" {
  name  = "apache-container"
  image = "httpd"
  ports {
    internal = 80
    external = 8080
  }
}
```

Now create the containers:

```sh
terraform init
terraform apply
```

Sample Output:

![Automate Deployments Using Docker and Terraform 1](https://computingforgeeks.com/wp-content/uploads/2023/05/Automate-Deployments-Using-Docker-and-Terraform-1-1024x268.png?ezimgfmt=rs:696x182/rscb23/ng:webp/ngcb23 "Automate Deployments Using Docker and Terraform 2")

Verify the creation:

```sh
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                  NAMES
f2c719a4e398   httpd     "httpd-foreground"       23 seconds ago   Up 22 seconds   0.0.0.0:8080->80/tcp   apache-container
1beb88d0164a   nginx     "/docker-entrypoint.…"   24 seconds ago   Up 22 seconds   0.0.0.0:80->80/tcp     nginx-container
```

### c. Create Docker Volume

To create a Docker volume, add the below lines to the main.tf file:

```sh
resource "docker_volume" "my_volume" {
  name       = "my-volume"
}
```

Apply the changes

```sh
terraform init
terraform apply
```

Verify with the command:

```sh
$ docker volume ls
DRIVER    VOLUME NAME
local     my-volume
```

### d. Destroy the Resources

You can remove/delete all the resources in the main.tf file using the terraform command below:

```sh
terraform destroy
```

Accept the prompt:

```
Do you really want to destroy all resources? yes
```

Now you will have the resources deleted as shown.

## Verdict

That is the end of this guide on how to automate Deployments Using Docker and Terraform. I hope this was informative.

See more;

- [Learn Terraform Automation in 3 days using Video Courses](https://computingforgeeks.com/learn-terraform-automation-using-video-courses/)
- [Automate Bitbucket Tasks via Terraform](https://computingforgeeks.com/automate-bitbucket-tasks-via-terraform/)
- [Create and grant privileges to users in CloudSQL Databases using Terraform](https://computingforgeeks.com/create-and-grant-privileges-to-users-in-cloudsql-databases-using-terraform/)