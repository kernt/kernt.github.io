---
tags:
  - aws
  - cloudstack
---
**Cloud computing** can be defined as the delivery of computing resources on demand over the internet. This eliminates the need to host applications, services and data locally on personal computers or servers; instead, third-party providers maintain the computing resources remotely. These computing resources are normally hosted in data centres and can be accessed over the Internet. This allows users to easily run applications, access data and use any other resources without the need for physical infrastructure. Today, there are many cloud providers, the most popular ones are Microsoft Azure, Amazon Web Services, Google Cloud Platform, Oracle Cloud, Huawei Cloud, Digital Ocean, Alibaba Cloud, IBM Cloud, OVHcloud, Linode etc.

**Amazon Web Services** abbreviated as **_AWS_** is one of the prominent and influential players in the cloud computing field. It provides a suite of cloud-based services, which offers organizations the ability to scale their infra, therefore, enhancing agility as well as minimizing costs. The main feature of AWS is its large and globally distributed infrastructure that has data distributed across different regions in the world.

Some other cool benefits of AWS include:

- **Extensive Service Portfolio**: It provides a range of services that include computing, storage, analytics, databases, machine learning, IoT etc.
- **Integration Capabilities**: It integrates with other platforms and services which allows businesses to connect and interact with other existing systems and third-party applications.
- **Community and Support**: There is a large and active community for AWS that provides extensive documentation, training resources, and support, making it easier for businesses to learn, implement, and troubleshoot AWS services.
- **Security**: It implements comprehensive security measures that protect customer data and infrastructure, including encryption, access controls, and compliance certifications.
- **Cost-effectiveness**: It operates on a pay-as-you-go model, here, businesses only pay for the services they use. This eliminates upfront costs and allows for better cost management.

**LocalStack** is the AWS cloud service emulator which can be run locally on your laptop or in your CI environment. It allows you to run AWS applications or Lambdas on your local environment without having to connect to a remote cloud provider. This can be so vital for those who want to develop and test complex CDK apps, or those who want to learn about AWS. It gives you the great freedom to manipulate the environment which is not available in the corporate environment.

The advantages associated with LocalStack are:

- **Cost and time saving**: here, you do not need to use paid features when testing and deploying resources. You also save a lot of time spent waiting for the long resource deployment cycles against the cloud and dev environment cleanups.
- **Extensive Service Portfolio**: It supports a number of AWS services, such as AWS Lambda, S3, Dynamodb, Kinesis, SQS, SNS etc.
- **Accelerate your Dev and Test Loop**: it makes it easier for users to test cloud apps from the first line of code and get immediate feedback. You also get the chance to use generic, reusable local API emulators.

Although Localstack comes with several features that are free, not all AWS features are free. But you can architect an environment that is very close to the real cloud environment. Join me in this guide as we walk through how to run local AWS Cloud Stack using LocalStack on Linux.

## Prepare your Environment

For LocalStack work by running a container on your local environment. For that to work, you need Docker installed. Instal Docker using any of the guides below.

- [How To Install Docker on Debian](https://computingforgeeks.com/how-to-install-docker-on-debian-12-bookworm/)
- [How To Install Docker CE on RHEL 7 Linux](https://computingforgeeks.com/install-docker-ce-on-rhel-7-linux/)
- [How To Install Docker CE on Linux Systems](https://computingforgeeks.com/install-docker-ce-on-linux-systems/)

Ensure your user is added to the Docker group:

```sh
sudo usermod -aG docker $USER
newgrp docker
```

For those who want the **_LocalStack Pro Edition_** you need an API key. This is a unique identifier to activate your LocalStack license. This can be obtained on the [LocalStack](https://app.localstack.cloud/sign-in) page under **Account Settings** → **API Keys section**

[![Run Local AWS Cloud Stack using LocalStack on Linux](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-1024x714.png "Run Local AWS Cloud Stack using LocalStack on Linux 1")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux.png)
You can use the 14-day trial without any credit card required
## Running LocalStack on Linux

There are a number of ways to run LocalStack on Linux. Here are some of the ways to get LocalStack running on Linux:

- Using Localstack CLI
- Using Docker/ Docker Compose

Choose one of the below methods that best works in your environment.
### Option 1: Run LocalStack using LocalStack CLI

This is the quickest way to get started with LocalStack. This allows you to start and manage LocalStack using the CLI. For this method, you need to have a working Docker environment before you proceed.

There are several ways to install LocalStack CLI. These are:

- **Method 1: Install LocalStack CLI** **Using Binaries**

You can download the latest binary from the [Github release](https://github.com/localstack/localstack-cli/releases/latest) page. Alternatively, you can use wget as shown:

**Export the version:**
```sh
VER=$(curl -s https://api.github.com/repos/localstack/localstack/releases/latest|grep tag_name|cut -d '"' -f 4|sed 's/v//')
```

**Now download the version:**
```sh
##For AMD64
wget https://github.com/localstack/localstack-cli/releases/download/v$VER/localstack-cli-$VER-linux-amd64-onefile.tar.gz

##For ARM64
wget https://github.com/localstack/localstack-cli/releases/download/v$VER/localstack-cli-$VER-linux-arm64-onefile.tar.gz
```

**Once downloaded, extract the archive:**

```sh
tar -xvf localstack-cli-$VER-linux-*-onefile.tar.gz
```

**Now copy the binary to your PATH:**

```sh
sudo cp localstack /usr/local/bin
```

Now you will have the LocalStack CLI installed.

- **Method 2: Install LocalStack CLI** **Using PIP**

Another way of installing LocalStack is by using Python PIP. To proceed with this method, you need Python installed, the supported versions are Python 3.7 up to 3.11. Use any of these guides to install Python:

```sh
### Debian based systems ###
sudo apt update
sudo apt install -y python3 python3-pip

### RHEL based systems ###
sudo dnf -y install python3 python3-pip
```

**Now use PIP to install Localstack CLI:**

```sh
python3 -m pip install --upgrade localstack
```

- **Method 3: Install LocalStack CLI using Homebrew**

**Another way of installing the LocalStack CLI is by using Homebrew:**

```sh
brew install localstack/tap/localstack-cli
```

**Once the LocalStack CLI has been installed using any ot the methods, check the installed version:**

```sh
$ localstack --version
2.1.0
```
#### Run LocalStack Using LocalStack CLI

To run LocalStack using the CLI, first, export your API key:

```sh
export LOCALSTACK_API_KEY=<YOUR_API_KEY>
```

Now run LocalStack using the CLI.

```sh
localstack start -d
```

This will start a LocalStack container in the background.

[![Run Local AWS Cloud Stack using LocalStack on Linux 1](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-1.png "Run Local AWS Cloud Stack using LocalStack on Linux 2")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-1.png)
**You can view the container:**

```sh
docker ps
CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS                    PORTS                                                                                     NAMES
7b04f3859841   localstack/localstack-pro   "docker-entrypoint.sh"   49 seconds ago   Up 47 seconds (healthy)   53/tcp, 127.0.0.1:4510-4559->4510-4559/tcp, 443/tcp, 5678/tcp, 127.0.0.1:4566->4566/tcp   localstack_main
```
### Option 2: Run LocalStack using Docker/Docker-Compose

You can also run LocalStack directly using the Docker Engine. To achieve that, use the command with the below syntax:

- **For LocalStack** **Community Edition**

```sh
docker run -d \
  --rm -it \
  -p 4566:4566 \
  -p 4510-4559:4510-4559 \
  -e AWS_ACCESS_KEY_ID=admin \
  -e AWS_SECRET_ACCESS_KEY=${AWSACCESKEY} \
  localstack/localstack
```

- **For LocalStack** **LocalStack Pro:**

```sh
docker run -d \
  --rm -it \
  -p 4566:4566 \
  -p 4510-4559:4510-4559 \
  -e LOCALSTACK_API_KEY=LOCALSTACK_API_KEY \
  -e AWS_ACCESS_KEY_ID=admin \
  -e AWS_SECRET_ACCESS_KEY=${AWSACCESKEY} \
  localstack/localstack-pro
```

In the pro version, you provide the API key and use the **_localstack-pro_** image.

You can also make these deployments using Docker-compose. First, ensure that Docker-compose has been installed.

- [How To Install Docker Compose on Linux](https://computingforgeeks.com/how-to-install-latest-docker-compose-on-linux/)

**Create a persistent directory:**

```sh
sudo mkdir /localstack
sudo chmod 775 -R  /localstack
sudo chown -R $USER:docker /localstack
```

**On Rhel-based systems, configure SELinux:**

```sh
sudo chcon -R -t svirt_sandbox_file_t /localstack
```

**Next, create a Docker Compose YAML file.**

```sh
vim docker-compose.yml
```

In the file, add the below lines for the desired edition.

- **For Localstack Community:**

```sh
version: "3.8"

services:
  localstack:
    container_name: "home_localstack_main"
    image: localstack/localstack
    ports:
      - "0.0.0.0:4566:4566"            # LocalStack Gateway
      - "0.0.0.0:4510-4559:4510-4559"  # external services port range
    environment:
       - AWS_ACCESS_KEY_ID=admin
       - AWS_SECRET_ACCESS_KEY=Passw0rd
#      - DEBUG=${DEBUG-}
       - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "/localstack:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
```

- **For Localstack Pro**

```sh
version: "3.8"

services:
  localstack:
    container_name: "home_localstack_main"
    image: localstack/localstack-pro  # required for Pro
    ports:
      - "0.0.0.0:4566:4566"            # LocalStack Gateway
      - "0.0.0.0:4510-4559:4510-4559"  # external services port range
      - "127.0.0.1:53:53"                # DNS config (required for Pro)
      - "127.0.0.1:53:53/udp"            # DNS config (required for Pro)
      - "0.0.0.0:443:443"              # LocalStack HTTPS Gateway (required for Pro)
    environment:
      - AWS_ACCESS_KEY_ID="admin"
      - AWS_SECRET_ACCESS_KEY="Passw0rd"
#      - DEBUG=${DEBUG-}
      - PERSISTENCE=1
      - LOCALSTACK_API_KEY="your-API-KEY-here"  # required for Pro
      - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "/localstack:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
```

In the above command, set your AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY and LOCALSTACK_API_KEY.

**Start the container with the command:**

```sh
docker-compose up -d
```

Now you will have Localstack running. You can allow ports **4566** and **443** through your firewall:

```sh
##For UFW
sudo ufw allow 4566/tcp
sudo ufw allow 443/tcp

##For Firewalld
sudo firewall-cmd --add-port=4566/tcp --permanent
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```
## Integrate and Use LocalStack on Linux

Once installed with any of the above methods, you can proceed and use LocalStack as your Local AWS Cloud Stack. There are several ways to manage LocalStack.
### Option 1: Integrate LocalStack with AWS CLI

The simplest way to use it is from the AWS CLI. First, install the AWS CLI, this can be done using the below guide:

- [Install and Use AWS CLI on Linux Mint | macOS](https://computingforgeeks.com/install-and-use-aws-cli-on-linux-mint-macos/)

Once installed, you need to configure the CLI as shown:

```sh
aws configure
AWS Access Key ID [None]: admin
AWS Secret Access Key [None]: Passw0rd
Default region name [None]: us-east-2
Default output format [None]: json
```

If you did not set the **_AWS Access Key ID_** and the **_AWS Secret Access Key_**, you can just provide anything to be used here.

To demonstrate if everything is okay, we will create a test SQL queue with the command:

```sh
$ aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name hello-world --region us-east-2
{
    "QueueUrl": "http://localhost:4566/000000000000/hello-world"
}
```

**We can now send a message queue:**

```sh
aws --endpoint-url=http://localhost:4566 sqs send-message --queue-url http://localhost:4566/000000000000/hello-world --message-body "This is my first Message" --region us-east-2
{
    "MD5OfMessageBody": "fd25e7ee57d96b82e8db20d6c0c47ac5",
    "MessageId": "cb8ed23d-516b-427c-a3ba-e7bbfb735ab9"
}
```

**View the messages in the queue:**

```sh
aws --endpoint-url=http://localhost:4566 sqs receive-message --queue-url ttp://localhost:4566/000000000000/hello-world --max-number-of-messages 10 --region us-east-2
```

Sample Output:

[![Run Local AWS Cloud Stack using LocalStack on Linux 2](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-2.png "Run Local AWS Cloud Stack using LocalStack on Linux 3")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-2.png)
### Option 2: Using LocalStack AWS CLI (awslocal)

You can also use the LocalStack AWS CLI known as `awslocal`. This makes it possible to run the AWS command against LocalStack.

**To install it, use PIP as shown:**

```sh
pip install awscli-local
```

Once installed, you can use it as desired. For example, instead of the long AWS CLI command, we will have a short one when listing SQS queues:

```sh
awslocal sqs list-queues  --region us-east-2
{
    "QueueUrls": [
        "http://localhost:4566/000000000000/hello-world"
    ]
}
```

This method has **limitations** as it relies on several dependencies as well as AWS CLI versions. To work around this you need to downgrade the AWS CLI version

```sh
sudo pip install awscli
```
### Option 3: Manage LocalStack on the Web

This option can be used by those who have the **_API key_** configured. First, make configurations for your endpoint. The default domain name is

```sh
sudo vim /etc/hosts
192.168.200.51	localhost.localstack.cloud
```

Replace the IP address of your LocalStack machine appropriately. Then proceed and access the [LocalStack web UI](https://app.localstack.cloud/dashboard). You should see your instance listed as shown:

[![Run Local AWS Cloud Stack using LocalStack on Linux 8](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-8.png "Run Local AWS Cloud Stack using LocalStack on Linux 4")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-8.png)
Now you can click on it and manage the resources.

[![Run Local AWS Cloud Stack using LocalStack on Linux 3](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-3-1024x875.png "Run Local AWS Cloud Stack using LocalStack on Linux 5")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-3.png)
You can now click on the resources tab

[![Run Local AWS Cloud Stack using LocalStack on Linux 4](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-4-1024x875.png "Run Local AWS Cloud Stack using LocalStack on Linux 6")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-4.png)
You are free to deploy desired resources, say the S3 bucket.

[![Run Local AWS Cloud Stack using LocalStack on Linux 6](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-6-1024x875.png "Run Local AWS Cloud Stack using LocalStack on Linux 7")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-6.png)
Once created, UI will appear as shown.

[![Run Local AWS Cloud Stack using LocalStack on Linux 5](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-5-1024x875.png "Run Local AWS Cloud Stack using LocalStack on Linux 8")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-5.png)
Now upload data and use it as desired.

[![Run Local AWS Cloud Stack using LocalStack on Linux 7](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-7-1024x738.png "Run Local AWS Cloud Stack using LocalStack on Linux 9")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-7.png)
You can also configure your endpoint domain name under settings as shown:

[![Run Local AWS Cloud Stack using LocalStack on Linux 9](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-9-1024x687.png "Run Local AWS Cloud Stack using LocalStack on Linux 10")](https://computingforgeeks.com/wp-content/uploads/2023/06/Run-Local-AWS-Cloud-Stack-using-LocalStack-on-Linux-9.png)
Now this method makes it easier to manage the resources for those who are not yet comfortable with the AWS CLI.
## Verdict

Today, we have walked through how to run Local AWS Cloud Stack using LocalStack on Linux. I hope this was helpful to you.

See more:

- [How To Acquire Linux Skills for DevOps and Cloud Engineers](https://computingforgeeks.com/acquire-linux-skills-for-devops-cloud-engineers/)
- [Manage VM Instances on Hetzner Cloud using hcloud CLI](https://computingforgeeks.com/manage-vm-instances-on-hetzner-cloud-using-hcloud-cli/)
- [How To Upgrade VMware vCloud Usage Meter from 4.5 to 4.6](https://computingforgeeks.com/how-to-upgrade-vmware-vcloud-usage-meter/)
- [Create and grant privileges to users in CloudSQL Databases using Terraform](https://computingforgeeks.com/create-and-grant-privileges-to-users-in-cloudsql-databases-using-terraform/)