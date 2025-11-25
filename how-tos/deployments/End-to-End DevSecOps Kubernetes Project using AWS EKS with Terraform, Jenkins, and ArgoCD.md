![](NQoY_yla1QMVJq7F.gif)

# Project Introduction

This comprehensive guide will walk you through setting up a robust DevSecOps pipeline on AWS using Kubernetes.
The Example project is designed to deploy a Tetris game application on an EKS cluster, incorporating best practices for security and automation.
# Prerequisites

Before diving into the project, make sure you have the following prerequisites in place:

1. **Local Environment Setup**:

- **Terraform and AWS CLI**: Install and configure Terraform and AWS CLI on your local machine. Basic knowledge of these tools is necessary.
- **Basic Knowledge**: Ensure basic knowledge of Terraform, AWS CLI, and familiarity with cloud concepts.

2. **Jenkins Server Deployment:**

- **Git**: Basic knowledge of Git commands is required.
- **AWS EC2**: Understanding of AWS EC2 instances and security groups.

**3. Jenkins Configuration:**

- **Jenkins**: Familiarity with Jenkins and basic Jenkins pipeline concepts.
- **Docker, Sonarqube, Terraform, Kubectl, AWS CLI, Trivy**: Basic knowledge of these tools is necessary.

**4. ArgoCD Setup:**

- **Kubernetes**: Basic knowledge of Kubernetes concepts.
- **ArgoCD**: Familiarity with ArgoCD and understanding of continuous deployment concepts.

**5. Pipeline Configuration:**

- **Jenkins Plugins:** Understanding of Jenkins plugins, especially AWS Credentials, and Pipeline: AWS Steps.
- **Tools Configuration:** Basic knowledge of configuring tools like Docker, NodeJS, OWASP Dependency-Check, and SonarQube in Jenkins.

**GitHub Repository for the Project** [https://github.com/cookiesikeri/End-to-End-Kubernetes-DevSecOps-Tetris-Project](https://github.com/cookiesikeri/End-to-End-Kubernetes-DevSecOps-Tetris-Project.git)

# Step 1: We need to create an IAM user and generate the AWS Access key

Create a new IAM User on AWS and give it to the AdministratorAccess for testing purposes (not recommended for your Organization’s Projects)

Go to the AWS IAM Service and click on **Users.**
o to the AWS IAM Service and click on **Users.**

![](qJoYYE81CqhC61_J.webp)

Click on **Create user**

![](PCUjN7nNJ2j2T1o1.webp)

Provide the name to your user and click on **Next.**

![](dUhnMycFnAc7IZoD.webp)

Select the **Attach policies directly** option and search for **AdministratorAccess** then select it**.**

Click on the **Next.**

![](Sipf8N9lf4KdO9qH.webp)

Click on **Create user**

![](T914Id6uh3Kzz9yP.webp)

Now, Select your created user then click on **Security credentials** and generate access key by clicking on **Create access key.**

![](bmVmS5ddZW06wV1.webp)

Select the **Command Line Interface (CLI)** then select the checkmark for the confirmation and click on **Next.**

![](oDLGEDMGHUXyaa2T.webp)

Provide the **Description** and click on the **Create access key.**

![](2aXVth6fepyn1r2xc.webp)

Here, you will see that you got the credentials and also you can download the CSV file for the future.

![](2O7yHe6X8e0X_Dqv6.webp)

# Step 2: We will install Terraform & AWS CLI to deploy our Jenkins Server(EC2) on AWS

Install & Configure Terraform and AWS CLI on your local machine to create Jenkins Server on AWS Cloud

**Terraform Installation Script**

```sh
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg - dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg  
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list  
sudo apt update  
sudo apt install terraform -y
```

**AWSCLI Installation Script**

```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"  
sudo apt install unzip -y  
unzip awscliv2.zip  
sudo ./aws/install
```

Now, Configure both the tools

**Configure Terraform**

Edit the file /etc/environment using the below command and add the highlighted lines and add your keys at the blur space.

`sudo vim /etc/environment`

![](k3g9U-LIDiqLQMRs.webp)

After doing the changes, restart your machine to reflect the changes of your environment variables.

**Configure AWS CLI**

Run the below command, and add your keys
`aws configure`

![](YEPNTWzGBAe9Zbdr.webp)
# Step 3: Deploy the Jenkins Server(EC2) using Terraform

Clone the Git repository- [https://github.com/cookiesikeri/End-to-End-Kubernetes-DevSecOps-Tetris-Project](https://github.com/cookiesikeri/End-to-End-Kubernetes-DevSecOps-Tetris-Project)

Navigate to the **Jenkins-Server-TF**

Do some modifications to the backend.tf file such as changing the **bucket** name and **dynamodb** table(make sure you have created both manually on AWS Cloud).

![](Kj9u7vukLLZH-tFP.webp)

Now, you have to replace the Pem File name as you have some other name for your Pem file. To provide the Pem file name that is already created on AWS

![](ynBjxTXtxcmuPx5J.webp)
Initialize the backend by running the below command
`terraform init`

![](RG7t0Rig325Na96r.webp)

Run the below command to check the syntax error

`terraform validate`  

`terraform fmt`

Run the below command to get the blueprint of what kind of AWS services will be created.

`terraform plan -var-file=variables.tfvars`

![](rOHulgLVwwcAZcbg.webp)

Now, run the below command to create the infrastructure on AWS Cloud which will take 3 to 4 minutes maximum

`terraform apply -var-file=variables.tfvars --auto-approve`

![](KL1lhs2b5bhSJqv7.webp)

Now, connect to your Jenkins-Server by clicking on Connect.

![](ecBB_snpIkKPkA6J.webp)
opy the **ssh** command and paste it on your local machine.

![](jrBB_VsoZqSp0ZPH.webp)
# Step 4: Configure the Jenkins

Now, we logged into our **Jenkins server.**

![](koSES2qXaFde8vMx.webp)

We have installed some services such as Jenkins, Docker, Sonarqube, Terraform, Kubectl, AWS CLI, and Trivy.

Let’s validate whether all our installed or not.

```
jenkins --version  
docker --version  
docker ps  
terraform --version  
kubectl version  
aws --version  
trivy --version

```
![](2x1bNYSBKG095_wR.webp)

Now, we have to configure Jenkins. So, copy the public IP of your Jenkins Server and paste it on your favorite browser with an 8080 port.

ow, run the below command to get the administrator password and paste it on your Jenkins.

`systemctl status jenkins.service`

OR

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

Click on **Install suggested plugins**

After installing the plugins, continue as admin

Click on **Save and Finish**
Click on **Start using Jenkins**
# Step 5: We will deploy the EKS Cluster using Jenkins

Now, go back to your Jenkins Server **terminal** and configure the AWS.

![](DH-lRIcVpZ5O6Nrj.webp)

Click on **Manage Jenkins**

![](tL2UuWtxYbLFbwEu.webp)

Click on **Plugins**

![](Gw6s5eri0K3aCLdj.webp)

Select the **Available plugins** and install the following plugins and click on **Install**

```sh
AWS Credentials  
Pipeline: AWS Steps
```

![](Mge_5kjaLnU0z7Al.webp)

Once, both the plugins are installed, restart your Jenkins service.

Now, we have to set our AWS credentials on Jenkins

Go to **Manage Plugins** and click on **Credentials**

![](FVfoAtbx2ariIVgq.webp)

Click on **global.**

![](R5vEjVGrZEbZidYl.webp)

Click on **Add credentials**

![](iEJiN-UHqF_fVRG5.webp)

Select **AWS Credentials** as **Kind** and add **the ID** same as shown in the below snippet except for your AWS Access Key & Secret Access key and click on **Create.**

![](FyHWhpsxjk-XadP5.webp)

Now, Go to the **Dashboard** and click **Create a job**

![](jhG2-m4PWB4ZhDnH.webp)

Select the **Pipeline** and provide the name to your **Jenkins Pipeline** then click on **OK.**

![](QObodFIf4ung88DY.webp)

Now, Go to the GitHub Repository in which the Jenkins Pipeline code is located to deploy the EKS service using Terraform.

[https://github.com/cookiesikeri/End-to-End-Kubernetes-DevSecOps-Tetris-Project/blob/master/Jenkins-Pipeline-Code/Jenkinsfile-EKS-Terraform](https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-DevSecOps-Tetris-Project/blob/master/Jenkins-Pipeline-Code/Jenkinsfile-EKS-Terraform)

Copy the entire code and paste it here

**Note:** There will be some configurations like backend.tf files that need to be updated from your side. Kindly do that to avoid errors.

EKS-Terraform Code- [https://github.com/cookiesikeri/End-to-End-Kubernetes-DevSecOps-Tetris-Project/tree/master/EKS-TF](https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-DevSecOps-Tetris-Project/tree/master/EKS-TF)

After pasting the Jenkinsfile code, click on **Save** & **Apply.**

![](LE-rYEn7a0zvmyrX.webp)

Click on **Build**

![](6c9I0TZEK87bejZw.webp)

You can see our **Pipeline** was **successful**

![](krM04dmmRQ57u9_m.webp)

You can validate how many resources have been created by going to the **console logs**

![](MmmF_EzzqwUsn2.webp)

Now, we will configure the **EKS** **Cluster** on the **Jenkins Server**

Run the below command to configure the EKS Cluster

`aws eks update-kubeconfig --region us-east-1 --name Tetris-EKS-Cluster`

To validate whether the EKS Cluster was successfully configured or not. Run the below command

`kubectl get nodes`

![](UFKmwjunQITOmQid.webp)

# Step 6: We will install ArgoCD Controller on EKS Cluster and make publicly available

Create tetris namespace

`kubectl create namespace tetris`

![](EIczaChlZdvKQ4t1.webp)

Now, we will configure the argoCD controller on our EKS cluster.

Create a new namespace named argocd and apply the manifest file on the EKS cluster.

```sh
kubectl create namespace argocd  
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml
```

![](eqtTYx3VOBYtQkMk.webp)

Validate whether the argocd controller is deployed or not using the below command.

`kubectl get pods -n argocd`

![](JpQ5OlcY2JskY01y.webp)

As we have to access our ArgoCD controller through GUI, we need to set up the LoadBalancer for it.

Run the below command to set up the **load balancer** which will expose the argoCD controller publicly.

`kubectl patch svc argocd-server -n argocd -p ‘{“spec”: {“type”: “LoadBalancer”}}’`

![](43oeznJqGgDX_JMv.webp)

Now, you will see **Load Balancer** on the AWS console.

![](haxLlicrjXwdEPpQ.webp)

Copy the LoadBalancer DNS and paste it into your favorite browser.

Click on **Advanced.**

![](JnkVTILm2abk0x8h.webp)

Click on the highlighted link

![](C0alyJy4fmugGBCn.webp)

Here, you can see the ArgoCD login page. But we can’t do anything because we don’t know the password of the admin user.

![](7vpx-fXEldh-UEOU.webp)

Now, we need an admin password to log in to our argoCD.

Now, we need an admin password to log in to our argoCD.

There is one pre-requisite which is **jq** to get the password by using filtration.

`sudo apt install jq -y`

![](aidu1utJxHi5g32W.webp)

Store the ArgoCD DNS name in the variable

`export ARGOCD_SERVER=kubectl get svc argocd-server -n argocd -o json | jq — raw-output ‘.status.loadBalancer.ingress[0].hostname’`

![](V8AZJcEUboP08yIn.webp)

Run the below command to get the password.

Enter the username and password in **argoCD** and click on **SIGN IN.**

![](c8vLoGPyTHKvPJ7N.webp)

Here is our **ArgoCD Dashboard.**

Click on **CREATE APPLICATION.**

![](c6LDxO7kBdZrtlF0.webp)

Provide the name of your **Application name, the Project name** will be **default**, and **SYNC POLICY** will be **Automatic** then scroll down.

![](qtW0jXLaGEk3-202.webp)

Provide the **GitHub Repo,** enter the path where your deployment file is located, then, **Cluster URL** will be [https://kubernetes.default.svc](https://kubernetes.default.svc/) and in the end, **namespace** will be **tetris** but you can write another namespace name.

Click on **CREATE.**

![](ylIWwabWEFX13n8T.webp)

# Step 7: Install the required plugins and configure the plugins to deploy our Tetris Application Version1

Install the following plugins by going to **Dashboard -> Manage Jenkins -> Plugins -> Available Plugins**

```
Docker  
Docker Commons  
Docker Pipeline  
Docker API  
docker-build-step  
Eclipse Temurin installer  
NodeJS  
OWASP Dependency-Check  
SonarQube Scanner
```

![](hyYj_xrWvlxJDKcE.webp)

Now, we have to configure our Sonarqube.

To do that, copy your Jenkins Server public IP and paste it on your favorite browser with a 9000 port

The username and password will be **admin**

Click on **Log In.**

![](SVT7pF1IdKdDQSuT.webp)

Update the password

![](wdBVrDQ2Aft3J9Nj.webp)

Click on **Administration** then **Security,** and select **Users**

![](0Whv9sKXmLftTSSU.webp)

Click on **Update tokens**

![](VUTRQXHA8FIBGwy-.webp)

Click on **Generate**

![](jA8QXbZ4vGPFLvjO.webp)

Copy the **token** and keep it somewhere safe and click on **Done.**

![](KDLwSn3IL33LdNft.webp)

Now, we have to configure **webhooks** for quality checks.

Click on **Administration** then, **Configuration** and select **Webhooks**

![](NCpAmVt-6up3BjDg.webp)

Click on **Create**

![](Wy28ttGm4GGzgcwj.webp)

Provide the name to your project and in the **URL,** provide the jenkins server public ip with port 8080 and add sonarqube-webhook in suffix and click on **Create.**

```sh
[http://<jenkins-server-public-ip>:8080/sonarqube-webhook/](http://34.224.69.67:8080/sonarqube-webhook/)
```

![](uQERAv67vYC_NKaL.webp)

Here, you can see the **webhook.**

![](W_QFHpEEeR7R1Qux.webp)

Now, we have to configure the installed plugins.

Go to **Dashboard -> Manage Jenkins -> Tools**

We are configuring jdk

Search for jdk and provide the configuration like below snippet.

![](O-WRVI-XRoy0-sjf.webp)

Now, we will configure nodejs

Search for node and provide the configuration like the below snippet.

![](5CiUmwxVF22FZMnp.webp)

Now, we will configure the OWASP Dependency check

Search for Dependency-Check and provide the configuration like the below snippet.

![](tLKVO2HrMizAPKQZ.webp)

Now, we will configure the docker

Search for docker and provide the configuration like the below snippet.

![](z1VeBdKH8BOfH6R0.webp)

Now, we will configure sonarqube

Search for sonarqube and provide the configuration like the below snippet.

![](2MvtN9OgOHPyPpi2.webp)

This is it, now click on **Apply** and **Save.**

Now, we have to store our generated token for Sonarqube in Jenkins

Go to **Dashboard -> Manage Jenkins -> Credentials**

Select the kind as **Secret text** and paste your token in **Secret** and keep other things as it is.

Click on **Create**

![](nSB6ckv_L-VUvsi6.webp)

Token is created

![](ORf8njeCPhQQFaiS.webp)

Now, we have to set the path for Sonarqube in Jenkins

Go to **Dashboard -> Manage Jenkins -> System**

Search for **SonarQube installations**

Provide the **name** as it is, then in **Server URL** copy the sonarqube public IP (same as jenkins) with port 9000 and select the sonar token that we have added recently and click on **Apply** & **Save**.

![](JMWqQWdCk7L0hHJh.webp)

Now, In our Jenkinsfile we are pushing our docker image to Dockerhub and then, we have to update the same image name with the new tag in the deployment file which is present on GitHub.

To do that, we need to store our Docker & GitHub credentials in Jenkins.

Go to **Dashboard -> Manage Jenkins -> Credentials**

Add your docker hub username and password in the respective field with ID **docker.**

Click on **Create**

![](sHHU2JAvQgPW-4jw.webp)

Add GitHub credentials

Select the kind as **Secret text** and paste your GitHub Personal access token(not password) in **Secret** and keep other things as it is.

Click on **Create**

![](2sHHU2JAvQgPW-4jw.webp)

Add GitHub credentials

Select the kind as **Secret text** and paste your GitHub Personal access token(not password) in **Secret** and keep other things as it is.

Click on **Create**

**Note:** If you haven’t generated your token then, you have it generated first then paste it in the Jenkins

![](cH6zanBOufiXwrGC.webp)

Now, we are ready to create our Jenkins Pipeline to deploy our Tetris Application.

Go to **Jenkins Dashboard**

Click on **New Item**

![](tFrhiUi9dHCxFM9l.webp)

Provide the name of your **Pipeline** and click on **OK.**

![](z1gAzqTaxSMtb6w_.webp)

This is the Jenkinsfile to deploy Tetris Application Version 1 on EKS.

Copy and paste it into the Jenkins

[https://github.com/cookiesikeri/End-to-End-Kubernetes-DevSecOps-Tetris-Project/blob/main/Jenkins-Pipeline-Code/Jenkinsfile-TetrisV](https://github.com/cookiesikeri/End-to-End-Kubernetes-DevSecOps-Tetris-Project/blob/main/Jenkins-Pipeline-Code/Jenkinsfile-TetrisV1)

Click **Apply & Save.**

![](Jk3DWHbHk0m7SJsE.webp)

Now, click on the build.

Our pipeline was successful.

![](KGk_2wAPgm7TcVXX.webp)

Now, Go to the **ArgoCD Console.** You will see your application has been deployed or deploying

Now, Go to the **ArgoCD Console.** You will see your application has been deployed or deploying

![](0_8-prFdunKBA5i0uO.webp)

This is our Sonarqube which will show you the Code Smells and vulnerabilities in your source code.

![](0_qC6eAnl4ZJipfVbN.webp)

This is our Dependency Check Results

![](h0_50grZhvlZkmAp48j.webp)

Copy the DNS name of your load balancer from ArgoCD Console or you can go to AWS Console and copy the Load Balancer and hit the DNS on your favorite browser to enjoy the **Tetris Game**.

![](0_JDqxjFJm6UShzZBK.webp)

# Step 8: We will deploy our Tetris Application Version 2

Now, suppose we have done some modifications to our previous version to make it more good in the sense of GUI or anything else. Then, we will have to deploy our **Version** **2** of our same application.

To do that, we will create a new pipeline. We can do it in the existing pipeline as well but this way you will be able to understand clearly.

We have a separate code for our Tetris Version 2. In which Dockerfile is present, so we will build the image and push it on docker and then update the same manifest file instead of v1 we will replace it with v2 manually first.

**Hope you get the high overview, what are we going to do next?**

Let’s make it and finish our project.

Go to **Jenkins** -> **Dashboard** and click on **New item**

![](0_INHlL2jazp2DHkg0.webp)

Provide the name to your **Pipeline name** and click on **OK.**

![](0_jBa-pE8Ul2mFws9d.webp)

This is the Jenkinsfile to deploy Tetris Application Version 2 on EKS.

Copy and paste it in the Jenkins

Click **Apply & Save.**

![](0_dlhHOa5DECpqS_5v.webp)

Before going to **build** the pipeline, update the manifest file.

Replace the v1 with v2 in the image section.

Now, Once you click on the build to deploy our Tetris Application Version 2.  
You will see our **pipeline** was **successful**.

![](0_k4WIoCIyNYIWlRVW.webp)

ArgoCD deployed our Application. Now, you can click on the **service** to get the **LoadBalancer** **DNS** name

![](0_PZp_oqXUp_1Ppi2B.webp)

Once you hit the DNS, you will see our Game is live and you need to just click on **Start.**

![](2_PZp_oqXUp_1Ppi2B.webp)

Now, you can enjoy the game.

![](0_KPZ112BvJyCdbEpb.webp)

# Cleanup

# Step 9: We will destroy the created AWS Resources

Delete both the created **LoadBalancer** manually.

![](0_eAfx1_rCcyYqoEVN.webp)

Select the **EKS-Terraform-Deploy** Pipeline.

Click on **Build with Parameters** and select the **destroy** and click on **Build.**

![](0_RDynRMQhxfCf9t4N.webp)

The Pipeline ran successfully which means the EKS Cluster has been deleted.

You can validate from the console as well.

![](0_sQatnwgDe5805EJ3.webp)

Now, we have to delete our **Jenkins Server**.

To do that, just run the below command on your local machine from where you create Jenkins Server.

terraform destroy -var-file=variables.tfvars --auto-approve

![](0_zzhSRCi_cT5SxWkc.webp)

# Conclusion

# Conclusion

Congratulations on completing the End-to-End DevSecOps Kubernetes Project! This project provides hands-on experience with DevOps practices, AWS services, Kubernetes, and continuous integration and deployment.

Stay connected on **LinkedIn**: [LinkedIn Profile](https://www.linkedin.com/in/ikeri-ebenezer-384059b1/)

Stay up-to-date with **GitHub**: [Profile](https://github.com/cookiesikeri)