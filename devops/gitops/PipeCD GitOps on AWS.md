GitOps is a term that has been popular in recent years. A popular GitOps tool is ArgoCD. 
ArgoCD helps us deploy the GitOps model on Kubernetes. However, Cloud Native is not only Kubernetes; there are many other environments, such as Cloud AWS, GCP, and Azure. In this article, we will learn about a tool that helps us deploy the GitOps model in many different environments, PipeCD. We will make examples of PipeCD, Terraform, and AWS.

# GitOps

For a simple explanation of GitOps, let’s imagine the following typical case: We have a system, and everything is working well. Superiors and everyone else are happy. Suddenly, one day, our system stopped running. The frustrated superior cursed the entire team and demanded the cause.

After losing sleep for several days and nights, you discover that the error is due to someone editing the database configuration so that the current application cannot connect to the database. I corrected my mistake and asked again who did it, but no one accepted it. This is a very familiar story that happens every day.

GitOps helps us resolve this problem. With GitOps, our system has two states:

- Observable state
- Desire state

The desired state is defined by the system’s configuration declaration files. And Git is where we store these files. GitOps is the process of synchronizing the observable state with the desired state that is stored on Git.

Now, if there are any changes to the system, they are done through Git. If anyone wants to change the configuration, they must edit the code and create a pull request in Git. We have complete control over the system state we want and know who made the changes.

With GitOps, it does not require the use of any specific tool, but only tools that meet the following features:

- Connects to Git.
- Detect the difference between the observable state and the desired state.
- Perform synchronization to keep the current state the same as the desired state.

In this article, we use Terraform and PipeCD to implement GitOps on AWS.
# Prerequisites

- Terraform
- Having a Kubernetes cluster and connecting to it via [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
# Terraform

Let’s do a simple example of declaring EC2 configuration with Terraform code and storing it on github. Then use PipeCD to connect to github and synchronize the EC2 configuration saved in Git to the actual AWS infrastructure.

Clone this [repo](https://github.com/hoalongnatsu/pipecd). First, we need to prepare the [Terraform Backend](https://developer.hashicorp.com/terraform/language/settings/backends/s3). Move to the backend folder and run:

`terraform init && terraform apply -auto-approve`

```sh
Apply complete! Resources: 11 added, 0 changed, 0 destroyed.  
  
Outputs:  
  
config = {  
  "bucket" = "pipecd-s3-backend"  
  "dynamodb_table" = "pipecd-s3-backend"  
  "region" = "ap-southeast-1"  
  "role_arn" = "arn:aws:iam::111937018111:role/PipecdS3BackendRole"  
}
```

Move to the terraform folder and update the S3 backend with the above value.

```sh
terraform {  
  backend "s3" {  
    bucket         = "pipecd-s3-backend"  
    key            = "pipecd"  
    region         = "ap-southeast-1"  
    encrypt        = true  
    role_arn       = "arn:aws:iam::111937018111:role/PipecdS3BackendRole"  
    dynamodb_table = "pipecd-s3-backend"  
    shared_credentials_files = ["/etc/piped-secret/credentials"]  
  }  
}  
  
provider "aws" {  
  region = "ap-southeast-1"  
  shared_credentials_files = ["/etc/piped-secret/credentials"]  
}  
  
data "aws_ami" "ubuntu" {  
  most_recent = true  
  
  filter {  
    name   = "name"  
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]  
  }  
  
  owners = ["099720109477"] # Canonical Ubuntu AWS account id  
}  
  
resource "aws_instance" "ubuntu" {  
  ami           = data.aws_ami.ubuntu.id  
  instance_type = "t2.micro"  
  
  tags = {  
    Name = "Hello PipeCD"  
  }  
}
```

Next step, we install PipeCD.
# PipeCD

[PipeCD](https://pipecd.dev/) consists of two main components: Control Plane and Piped.
## Control Plane

The control plane is a centralized part of PipeCD. It contains several services as below to manage the application, and deployment data and handle all requests from Piped (the agent to run the synchronization process). In the simple case, you only need to install a single Control Plane for Piped management.

We use Helm to install the PipeCD Control Plane on the Kubernetes Cluster. Create a file `piped-values.yaml`:

```yaml
config:  
  data: |  
    apiVersion: "pipecd.dev/v1beta1"  
    kind: ControlPlane  
    spec:  
      datastore:  
        type: MYSQL  
        config:  
          url: root:test@tcp(pipecd-mysql:3306)  
          database: quickstart  
      filestore:  
        type: MINIO  
        config:  
          endpoint: http://pipecd-minio:9000  
          bucket: quickstart  
          accessKeyFile: /etc/pipecd-secret/minio-access-key  
          secretKeyFile: /etc/pipecd-secret/minio-secret-key  
          autoCreateBucket: true  
      projects:  
        - id: quickstart  
          staticAdmin:  
            username: hello-pipecd  
            passwordHash: "$2a$10$ye96mUqUqTnjUqgwQJbJzel/LJibRhUnmzyypACkvrTSnQpVFZ7qK" # bcrypt value of "hello-pipecd"  
mysql:  
  database: quickstart  
  image: mysql  
  rootPassword: test  
quickstart:  
  enabled: true  
secret:  
  encryptionKey:  
    data: encryption-key-just-used-for-quickstart  
  minioAccessKey:  
    data: quickstart-access-key  
  minioSecretKey:  
    data: quickstart-secret-key
```

Run Helm CLI:

`helm upgrade -i pipecd oci://ghcr.io/pipe-cd/chart/pipecd --version v0.44.2 --create-namespace --namespace=pipecd -f piped-values.yaml`

After installing successfully, run the following CLI to open the PipeCD UI:

`kubectl -n pipecd port-forward svc/pipecd 8080`

Open `localhost:8080` and login with username and password are `hello-pipecd`.

After logging in successfully, the browser will redirect you to the PipeCD console settings page in `piped` the settings tab. You will find the `+ADD` button on the top of this page, click here and insert information to register the deployment runner for PipeCD.

Click on the `Save` button, and then you can see the `piped-id` and `secret-key`.
## Connect to GitHub

Next, we create Piped to connect to Github. Create file `piped-values.yaml`

```
config:  
  data: |  
    apiVersion: pipecd.dev/v1beta1  
    kind: Piped  
    spec:  
      projectID: quickstart  
      pipedID: <pipe-id>  
      pipedKeyFile: /etc/piped-secret/piped-key  
      syncInterval: 1m  
      apiAddress: pipecd:8080  
      repositories:  
        - repoId: pipecd  
          remote: https://github.com/hoalongnatsu/pipecd.git  
          branch: main  
      platformProviders:  
        - name: terraform  
          type: TERRAFORM  
  
args:  
  insecure: true  
  
secret:  
  data:  
    piped-key: <pipe-key>
```

Replace `<pipe-id>` and `<pipe-key>` with the above value. Run Helm:

`helm upgrade -i dev oci://ghcr.io/pipe-cd/chart/piped --version=v0.44.2 --namespace=pipecd -f piped-values.yaml`

```sh
NAME: dev  
LAST DEPLOYED: Wed Aug 16 11:11:21 2023  
NAMESPACE: pipecd  
STATUS: deployed  
REVISION: 1  
TEST SUITE: None  
NOTES:  
Now, the installed piped is connecting to.
```

After connecting to GitHub, The next thing we need to do is specify the folder containing the Terraform configuration so that PipeCD can synchronize it on AWS. To do this, we need to create an application.
# Application

Navigate to the `Applications` page, click on the `+ADD` button on the top left corner. Go to the `ADD MANUALLY` tab:

Filling the form as the image below:

![](P5qv56JwKsRaRj4f)

Then click the `SAVE` button to register. After a bit, the first deployment would be complete automatically to sync the application to the state specified in the current Git commit.

However, after a while, it returns an error:

![](wPhPK0K1mqqSt4B4)
# Specify AWS Credentials

The reason is that we have not configured AWS credentials so that PipeCD knows which AWS account to deploy to. To configure Credentials, edit the file `piped-values.yaml` as follows:


```yaml
secret:  
  data:  
    piped-key: t14hagknd38cjkdzrq5c9fdgma1h2cdsrsn72kk50jpzr74xcc  
    credentials: |  
      [default]  
      aws_access_key_id=<access_key>  
      aws_secret_access_key=<secret_key>
```

Replace `<pipe-id>` and `<pipe-key>` with your value. Then run the Helm CLI again :

`helm upgrade -i dev oci://ghcr.io/pipe-cd/chart/piped --version=v0.44.2 --namespace=pipecd -f piped-values.yaml`

```sh
Release "dev" has been upgraded. Happy Helming!  
NAME: dev  
LAST DEPLOYED: Wed Aug 16 11:15:12 2023
NAMESPACE: pipecd
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES: 
Now, the installed piped is connecting to.
```

Note: this is only for demo purposes, if working on a real project you should use [Secret management](https://pipecd.dev/docs-v0.44.x/user-guide/managing-application/secret-management/).

Go back to the `Applications` page, navigate to the `dev` application and click the `SYNC` button.

![](vd6JYJn59s_zxBAR)

Open the AWS Console and you will see EC2 has just been created.

![](61C43YStiJe22Lch)
# Detect Diff

If you edit the EC2 configuration on the AWS Console, for example, change Tags to Hello.

![](wVCrJTLS1isilerr)

PipeCD will automatically detect and notify you.

![](IdhFaK6OHTRwoI7k)

If we click the `SYNC` button, PipeCD will synchronize the EC2 configuration to be similar to Git. This step can be configured automatically using the [DeploymentTrigger](https://pipecd.dev/docs-v0.44.x/user-guide/configuration-reference/#deploymenttrigger) property. But with Terraform, we should not let this process be automatic because it is risky.

_Remember to delete all resources after completing the exercise._
# Conclude

So we have learned how to deploy the GitOps model on AWS with PipeCD. The project is still in the process of development and additions. Please try it out and send your comments.

Give the project a star if you see it helpful: [https://github.com/pipe-cd/pipec](https://github.com/pipe-cd/pipecd)