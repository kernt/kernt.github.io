In this article, we will learn the integration of Atlantis with GitHub and Terraform.

Terraform and GitHub has played an important role in our **DevOps** job and its simplex structure has made complex commands run in a simpler way making Terraform more flexible and a popular tool.

> Have you ever faced challenges like managing your multiple projects/repos in Terraform, facing difficulty while enforcing peer review, versioning control and too many sets of privileged credentials, even if we have [click-up](https://clickup.com/), [Gmail](https://mail.google.com/mail/u/0/#inbox), [slack](https://slack.com/) environment integrated with each other OR when you have already deployed the code and after the review is done, we face challenges in altering them? _😟_Then you should definitely try this solution called [Atlantis](https://www.runatlantis.io/)!

Atlantis helps you to run commands using GitOps where you can merge Atlantis, create a Pull Request, get approvals and also run the terraform commands so that your entire team is aware of what changes are being made to the infrastructure.

To deploy Atlantis, you need to know the basics of [Terraform](https://www.terraform.io/), [Kubernetes](https://kubernetes.io/), [Helm](https://helm.sh/), [GitHub](https://github.com/), and [Webhook Relay](https://webhookrelay.com/).

By the end of this blog, you will be able to know what is Atlantis, how to deploy it in [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) using Helm, how to use Atlantis and why Atlantis is preferred more️?

# So, let's start with what is Atlantis?

Atlantis is an open-source technology that allows the engineer/approver to review the changes in the infrastructure as well as evaluate that the proposed change is the actual change that will be executed on your infrastructure before applying it. Atlantis used with [_Terraform_](https://www.terraform.io) can be used as a [CICD](https://en.wikipedia.org/wiki/CI/CD#:~:text=In%20software%20engineering%2C%20CI%2FCD,testing%20and%20deployment%20of%20applications.) for Infrastructure as Code (IAC).

Atlantis is compatible with [**GitHub**](https://github.com/)**,** [**GitLab**](https://about.gitlab.com/)**,** [**Bitbucket Cloud,**](https://bitbucket.org/product) **and** [**Azure DevOps.**](https://azure.microsoft.com/en-in/services/devops/)

[](https://azure.microsoft.com/en-in/services/devops/)

![](guSr-MCtSaCSGpdNhYegUw.webp)
Atlantis Overview

# How does Atlantis work with Terraform?

Atlantis is an automation application that responds to [Terraform](https://www.terraform.io/) pull requests via web callback or webhooks and automates the terraform init, plan, and apply the workflow.

The following sequence diagram illustrates the sequence of actions described above:

![](3KS05b0LueiFAagJ.webp)

When you need to review the code before making any final changes, you can just do an “**_Atlantis plan_**” and check for the infrastructure changes.

It runs a terraform plan and comments with the output back on the pull request. And after your evaluation has been done, you can just comment “**_Atlantis apply_**” on the pull request, and Atlantis will run terraform apply and comment back with the output.

The Atlantis installation process then adds hooks to the repository which allows communication to the Atlantis server during the Pull Request (PR) process.

You don’t need heavy packages and large VM’s to deploy Atlantis, it can be done in a **container or a small virtual machine** — the only requirement is that the Terraform instance should communicate with both your version control (e.g., GitHub) and infrastructure (e.g., [AWS](https://aws.amazon.com/)/[GCP](https://cloud.google.com/)/[Azure](https://azure.microsoft.com/en-in/)) you’re changing.

Once Atlantis is configured for a particular repository, it then executes as follows:

1. _A developer generates a feature branch in git, builds some changes, and_ **_creates a Pull Request (GitHub) or Merge Request (GitLab)_**_._
2. _The developer writes “_**_Atlantis plan_**_” in a Pull Request (PR) comment._
3. _Atlantis locally_ **_runs terraform plans_** _via the installed_ [_webhooks_](https://en.wikipedia.org/wiki/Webhook)_. If there are no other Pull Requests in progress, Atlantis adds the resulting plan as a comment to the Merge Request and if there are any other Pull Requests in progress, the resulting command fails as we can’t assure that the plan will be valid once applied._
4. _The developer checks if the plan executes, provides the output as expected, and_ **_adds reviewers to the Merge Request_**_._
5. _Once the Pull Request has been approved by the approver, the developer writes “_**_Atlantis apply_**_” to apply in a PR comment. This will trigger Atlantis to run terraform apply and then the changes will be deployed to your infrastructure._

The command fails if the [Merge Request](https://docs.github.com/en/github/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/merging-a-pull-request) has not been approved.

![](oNZwSHgoqdnjL2pn4XOuKQ.webp)

Having known the process of Atlantis, let's check how to deploy Atlantis and make use of it. There are many ways via which you can deploy Atlantis, but in this blog, we are using GKE and GitHub. You can refer to their [official doc](https://www.runatlantis.io/docs/deployment.html) for more deployment options.

# Deploying Atlantis on Google Kubernetes Engine Using Helm

# Pre-requisites:

- [Kubernetes](https://kubernetes.io/) environment and kubectl configured on your machine. For this blog, I am using GKE Cluster, but you can also use Mini Kube or EKS
- [Helm](https://helm.sh/) CLI — a Kubernetes package manager.
- [Webhook Relay account](https://my.webhookrelay.com/) — webhook forwarding solution to our internal Kubernetes environment.
- [GitHub](https://github.com/) Organizational Account
- [Git host access credentials](https://www.runatlantis.io/docs/access-credentials.html#create-an-atlantis-user-optional) — Make sure you save the access token; we will need it in the next step.
- [GitHub Secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) — Make sure you save the secret; we will need it in the next step.
- [Webhook Relay operator](https://my.webhookrelay.com/tokens) — Make sure you save the Relay Key and Relay secret; we will need it in the next step.

There are multiple ways for deploying Atlantis, the official document for the same is in this [link](https://www.runatlantis.io/docs/deployment.html#deployment)

Let's follow the GitHub Deployment on GKE:

1. Create a namespace

`kubectl create namespace atlantis` 

2. Switch context to Atlantis namespace

`kubectl config set-context $(kubectl config current-context) --namespace=atlantis`

3. Add repositories

`helm repo add stable https://kubernetes-charts.storage.googleapis.com`

4. Update helm repository

`helm repo update`

5. Use the following command to run Atlantis

```sh
helm upgrade --install atlantis stable/atlantis --version 3.12.2 \  
  --set=github.user=GitHub-Username \  
  --set=github.token=******* \  
  --set=github.secret=***** \  
  --set=service.type=ClusterIP \  
  --set=ingress.enabled=false \  
  --set=orgWhitelist="github.com/atlantis/*"
```

Here, the variables are:

- github.user — your GitHub username
- [github.token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-token) — your GitHub personal access token
- github.secret — shared secret that will have to be shared with Atlantis
- [service.type](https://cloud.google.com/kubernetes-engine/docs/concepts/service) — making sure Atlantis is not exposed as a [Node Port service](https://cloud.google.com/kubernetes-engine/docs/concepts/service#service_of_type_nodeport)
- [ingress.enabled](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress) —We have set it to false, i.e. disabled ingress
- orgWhitelist — which repositories should be processed

![](1G_PQ_AjcE2e0mRMsgwtKA.webp)

Install Atlantis using Helm

1. Get the status of the pod and service

`kubectl get podskubectl get svc`

2. Export your Webhook Operator

```sh
export RELAY_KEY=xxxxxxxxxxxx  
export RELAY_SECRET=xxxxx
```

3. Add the webhook relay operator to your helm repository and update the helm repo

```sh
helm repo add webhookrelay https://charts.webhookrelay.com  
helm repo update
```

4. Install Helm

```sh
helm upgrade --install webhookrelay-operator webhookrelay/webhookrelay-operator \  
  --set credentials.key=$RELAY_KEY \  
  --set credentials.secret=$RELAY_SECRET
```

![](JBp2bpNPFyAzTTkRGpCVFg.webp)
Install Webhook Relay Operator

1. We should now see two pods running in our Atlantis namespace:

2. Next step is to configure webhook forwarding to the Atlantis service:

> _Use vim to create a new yaml file & press i to insert the following deployment file and :wq! to save the_ **atlantis-cr.yaml** _file_

```yaml
apiVersion: forward.webhookrelay.com/v1  
kind: WebhookRelayForward  
metadata:  
  name: forward-to-atlantis  
spec:  
  buckets:  
  - name: github-to-atlantis  
    inputs:  
    - name: public-endpoint  
      description: "Endpoint for GitHub"  
      responseBody: "OK"  
      responseStatusCode: 200  
    outputs:  
    - name: atlantis-pod  
      destination: [http://atlantis:80](http://atlantis:80)
```

3. Create the deployment using the apply command

`kubectl apply -f atlantis-cr.yaml`

1. We should now see three pods running in our atlantis namespace:

```sh
kubectl get pods  
kubectl get svc
```

> To check the public webhooks URL refer this [link](https://my.webhookrelay.com/buckets) and copy the Atlantis bucket URL

![](CKGmOJz5Dzxe-vbKEP5_sg.webp)

Webhook Relay URL

1. Go to [**GitHub**](https://github.com/) → Your Repo → Settings →Webhooks  
Set payload URL to the bucket URL and make sure to add **/events** in the end. Set the content type to **application/JSON;** set the **secret** to webhook secret got in point no.7.

Check the following checkboxes ✅

✔️_Pushes_
✔️_Pull request reviews_
✔️_Issue comments_
✔️_Pull requests_

_Make sure the_ **_Active checkbox_** _remains checked and click on Save._

![](rbvUsOBwse92S4mP_t1ftw.webp)

You can also refer to [GitHub Repository](https://github.com/sethvargo/atlantis-on-gke), where you can Clone the repo in your local system and run Terraform Commands, and configure Atlantis.

Once Atlantis is configured, you can try it out on GitHub by creating a Pull Request and commenting on the Atlantis plan and Atlantis apply.

# Benefits of Atlantis:

- **_Manage Multiple Projects_**⚡️ We have to manage multiple projects in AWS/GCP/Azure and other clouds. A single [Pull Request](https://docs.github.com/en/github/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) can run terraform plans in each directory in which it will copy and paste the planned output into the PR comment, and then apply locally before merging the Pull Request. Automating that process with Atlantis immediately improves productivity and thus we can manage multiple projects at the same time.
- **_No More “It worked on my machine”_** _👻_ We always end up with this quote that “it worked on my machine”, it should work on yours as well, here we don’t need to do anything locally, everything can be tested on a single platform.
- **_Increased Audit Trail:_** 🔏 Each pull request which has been raised, holds detailed logs of what infrastructure changes were made when the changes were made, who made the change, and who approved it.
- **_Secure Workflow:_** 🔐 Atlantis can be configured to require a Pull Request (PR) to be approved and merged before the changes can be applied. Atlantis can add an extra layer of protection that prevents engineers from accidentally applying changes before they’re approved by the approver
- **_Reduction of_** [**_IAM Permissions_**](https://cloud.google.com/iam/docs/permissions-reference) **_provided to Engineers:_** ✋ We are often required to provide highly privileged credentials to perform Terraform actions, and the greater the number of principals you have with privileged access, there is a higher chance of attacking risks.
- **_Formalize the delivery process:_** 👍 Atlantis makes sure that it locks the directory/workspace until the pull request is merged or the lock is deleted by manual intervention. This makes sure that changes are applied in the order as they are expected.

# Limitations

- Atlantis excels at all the things that we have seen till now, like the terraform init / plan / apply workflow, but it is not designed to handle more complex tasks, like importing terraform resources or running terraform state commands. Those operations still require manual intervention as of now.
- Atlantis lets us work efficiently and effectively, increasing productivity. We do not have to wait for opening a pull request, executing terraform commands. We are now able to make infrastructure changes to n number of environments in analogy.

> Note: If you’re new to Atlantis, you can run [**Test Drive of Atlantis**](https://www.runatlantis.io/guide/test-drive.html) where you can test the Atlantis working using the Atlantis executable files. Also, you can go through the [**official walkthrough**](https://www.youtube.com/watch?v=TmIPWda0IKg&t=135s) of Atlantis to gain more insights.