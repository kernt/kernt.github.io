---
tags:
  - cloudflare
  - terraform
  - bitbucket
---
The movement towards automation has become the de facto. Organizations around the globe demand automation with an alacrity that has never been there before. At the end of the day, automating manual and repetitive tasks brings golden fruits in the end but the process is never easy all the time. In case you are using Cloudflare in your Organization to manage your domain/zone records, then this guide provides something you can consider adding to your workflows.

We are going to use Terraform code to manage our records so that our teams can review and approve any changes that might happen anywhere in the organization. With one codebase that governs the entire DNS records, several benefits will accrue such as:

- The code will serve as documentation: Terraform code is pretty clear. So it will be easy to see all of the records at a glance
- We can enforce checks before records are added or altered
- Easy to implement CI/CD with code: This will cover the previous point as well because now we will be able to apply checks and reviews before code is merged.
- No more logging into the dashboard. All developers of operations team will ned is just a terminal
- Consistency can be easily achieved

Alright, at this juncture, we should go ahead and implement this setup. We will create a repository in Bitbucket that will hold all of our code and we shall also leverage Bitbucket pipelines to handle our CI/CD. Let us begin:
## Step 1: Create a Bitbucket repository

Simply login to your Bitbucket account and create a new repository. After that is done, clone it and then we can begin adding the necessary files to it.
## Step 2: Clone the repository and add terraform modules

Launch the terminal that you love and navigate to the directory you would wish to drop the cloned repo directory then supply the command below assuming that git is installed already.

```sh
git clone <the-repository-url>
```

For this example, we are going to use modules we created in a previous guide about generating Cloudflare records using cf-terraform. The structure of our files is as follows

```sh
├── cloudflare
│   ├── README.md
│   ├── bitbucket-pipelines.yml
└── entries
    ├── main.tf
    ├── vars.tf
    ├── modules
    │   ├── computingforgeeks-com
    │   │   ├── cloudflare_records.tf
    │   │   ├── provider.tf
    │   │   └── vars.tf
    │   ├── computinggeeks-com
    │   │   ├── cloudflare_records.tf
    │   │   ├── provider.tf
    │   │   └── vars.tf
         └── vars.tf
```
## Step 3: Adding Enchantments

From the tree decorated above, you can see we have files in each directory. In the root _snapshoter_ as well as in the modules directories. We need to fill their barns in this Step.

Inside _main.ft_ file, we had the following:

```sh
vim main.ft

provider "cloudflare" {
  alias = "dns_records"
  email   = var.cloudflare_email
  api_key = var.cloudflare_api_key
}

terraform {
  required_providers {
    cloudflare = {
      source = "cloudflare/cloudflare"
      version = "~> 3.0"
    }
  }
}

#1. computingforgeeks.com

module "computingforgeeks-com" {
  source                     = "./modules/computingforgeeks-com"
  providers = {
    cloudflare = cloudflare.dns_records
  }
}

#2. computinggeeks.com

module "computinggeeks-com" {
  source                     = "./modules/computinggeeks-com"
  providers = {
    cloudflare = cloudflare.dns_records
  }
}

terraform {
  backend "gcs" {
    bucket      = "geeks-terraform-state-bucket"
    prefix  = "terraform/cloudflare_state"
  }
}
```

**And inside _vars.tf_ file under _snapshoter_ directory, we had:**

```sh
vim vars.tf

variable "cloudflare_api_key" {
  type        = string
  sensitive   = true
  default     = "e5de3ef7282fde61ed8cfeca65788db4d31aa"
}

variable "cloudflare_email" {
  type        = string
  default     = "admin@computingforgeeks.com"
}
```

**Moreover, we created a _”providers.tf”_ file that have this configuration in each module directory**

```sh
vim ~/cloudflare/entries/modules/computingforgeeks-com/provider.tf 

terraform {
  required_providers {
    cloudflare = {
      source = "cloudflare/cloudflare"
      version = "~> 3.0"
    }
  }
}
```

And inside each module directory, we have variables associated with the module. We shall put them in a file _vars.tf_ in each module directory. An example of one is a zone_id variable in _computingforgeeks-com module_ as follows:

```sh
vim ~/cloudflare/entries/modules/computingforgeeks-com/vars.tf 

variable "geeks_com_zone_id" {
  type        = string
  default     = "158ab570cd45512ffcbc5ebcd282d410"
  description = "This is computingforgeeks.com domain Zone ID"
  sensitive   = true
}
```
## Step 4: Adding New Records

Most of the work is done thus far. The only part remaining is just adding records to Cloudflare via Terraform. We have designated a file in each of the modules known as _cloudflare_records.tf_. Inside this file, which already contains imported records from Cloudflare, proceed to add a record of your choosing (A, AAAA, MX, SRV, TXT, CNAME) as follows.

Let as add one record in _computingforgeeks-com_ domain:

```sh
vim ~/cloudflare/entries/modules/computingforgeeks-com/cloudflare_records.tf

resource "cloudflare_record" "terraform_sample_dashboard" {
  name    = “terraform.item”
  proxied = false
  ttl     = 3600
  type    = "A"
  value   = "192.168.20.21”
  zone_id = geeks_com_zone_id
}
```

As you have noticed, we are referencing the _zone_id_ variable we declared in _vars.tf_.

Once that is satisfactorily done, we are ready to deliver the spell. Save up your files and then go back to your terminal and navigate to _snapshoter_ directory where _main.tf_ file is then initialise terraform.

```sh
cd ~/cloudflare/entries/
export TF_VAR_cloudflare_api_key= < your_cloudflare_api_key >
export TF_VAR_cloudflare_email= < the_email >
terraform init
```

After that, do a _terraform plan_ to see that we are going to add to Cloudflare. You should see an output similar to the one below the command.

```sh
terraform plan -var cloudflare_email=${TF_VAR_cloudflare_email} -var cloudflare_api_key=${TF_VAR_cloudflare_api_key}

Terraform will perform the following actions:

  # module.computingforgeeks-com.cloudflare_record.terraform_sample_dashboard will be created
  + resource "cloudflare_record" "terraform_sample_dashboard" {
      + allow_overwrite = false
      + created_on      = (known after apply)
      + hostname        = (known after apply)
      + id              = (known after apply)
      + metadata        = (known after apply)
      + modified_on     = (known after apply)
      + name            = "terraform.example"
      + proxiable       = (known after apply)
      + proxied         = false
      + ttl             = 3600
      + type            = "A"
      + value           = "192.168.20.21"
      + zone_id         = (sensitive)
    }
```
## Step 5: Adding Bitbucket Pipeline

To this point, we have made tremendous progress. The only problem is that we are still running the commands locally and no one will get notified in the organization if we would like to add new records. If your use use does not need this, you can work with your local setup and all will be fine.

To add Bitbucket pipeline, navigate to the _cloudflare_snapshoter_ directory and create the required bitbucket-pipeline.yml file that Bitbucket reads.

```sh
cd 
vim bitbucket-pipelines.yml

image: penchant/cf-terraform:latest
pipelines:
  branches:
    main:
      - step:
          name: Deploy to Cloudflare
          deployment: production
          script:
            - cd entries
            - echo $GCLOUD_API_KEYFILE | base64 -d  > ./your-gcp-api-key.json
            - gcloud auth activate-service-account --key-file your-gcp-api-key.json
            - terraform init
            - terraform plan -out create_record -var cloudflare_email=${TF_VAR_cloudflare_email} -var cloudflare_api_key=${TF_VAR_cloudflare_api_key}
            - terraform apply -auto-approve create_record
          services:
            - docker
```

The image above _penchant/cf-terraform:latest_ has terraform and Google Cloud SDK installed, so it will do everything for you.

Ensure that you have the following environment variables in your Bitbucket already setup:

- CLOUD_API_KEYFILE: For authenticating to your GCP in order to store the tfstate file. You can use any cloud here of your choice
- TF_VAR_cloudflare_api_key: The Cloudfalre Api Key
- TF_VAR_cloudflare_email: The email that will authenticate terraform

We are ready to roll. Simply commit the directories and files to Bitbucket repository and you can launch your Pipelines in the “_main_” branch. You can create branches, edit the files, commit, create pull requests, add reviewers and once everything is okay, merge the files to main and let Bitbucket Shine.
## Concluding Remarks

We have successfully created new records and created Bitbucket Pipeline to handle our Cloudflare automation. Note that you can use other platforms like Jenkins to achieve the same results. A Wonderful day as we hope you continue to keep safe. Share the Love and let us all be kind to each other. Thank you all for the tremendous support throughout the year.

Other guides you will love include:

- [Install Vault Cluster in GKE via Helm, Terraform and BitBucket Pipelines](https://computingforgeeks.com/install-vault-cluster-gke-via-helm-terraform-bitbucket-pipelines/)
- [Deploy VM instance on OpenStack using Terraform](https://computingforgeeks.com/deploy-vm-instance-on-openstack-with-terraform/)
- [How To Deploy EKS Cluster on AWS using Terraform](https://computingforgeeks.com/how-to-deploy-eks-cluster-on-aws-using-terraform/)
- [Install Terraform on CentOS 8 / Rocky Linux 8](https://computingforgeeks.com/how-to-install-terraform-on-centos-linux/)
- [How To Store Terraform State in Consul KV Store](https://computingforgeeks.com/how-to-store-terraform-state-in-consul-kv-store/)