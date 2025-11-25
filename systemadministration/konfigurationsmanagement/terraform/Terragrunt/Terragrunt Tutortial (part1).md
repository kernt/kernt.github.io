terragrunt is a thin wrapper for terraform that provides extra tools for keep ur configuration Dry,short for don’t repeat urself& managing remote state effectively.

Key Feature:

![](t04MnKo-AAR3JsKQNqcIAA.webp)

Use Cases:

![](NBGC4PoVicdtEPHYtxyRDQ.webp)

What problems does terragrunt slove?

![](R6XM_MKIamc9159xZDcz4g.webp)

The DRY Principle?  
Dry principle or don’t repeat principle

![](vO7Sr2n-_Otacx33wMEaCg.webp)

![](J9N6ceTooIWthoIiGiUiKw.webp)

![](pQRh5Rt5OW9NbWEKyBbwBg.webp)

![](gWqZZieJMc5CA2eLzNOasg.webp)

![](Y-avJIHJqh5A5iI80YGfMg.webp)

```
Linux:  
wget “release1.8.3 V of terragrunt”  
apt install -y unzip  
unzip terraform_1.8.3_linux_amd64.zip  
chmod +x terraform  
mv terraform /usr/bin  
terragrunt version  
====================================================

Part 2:

Terraform/OpenTofu and Terragrunt — Compatibility Chart
```

![](Lo55MhaEwEaVMouOJlqp5g.webp)

ii)terragrunt cache:

.terragrunt-cache dir

![](idbw_HhYniWyZYxn9ZhWgA.webp)

![](MPs-wSoRCcEZWLb4hBb2jw.webp)

To store cache directories in a centralized location using terragrunt, you can utilize the terragrunt download_dir Environment variable

![](H_7skGxYNp0hSr86E_JKjw.webp)

this feature allows you to specify a custom directory where terragrunt will store its cache directories.

iii)Terragrunt with aws:

![](OENSZ2cdZsRLeb5R20ToBA.webp)

![](QQzakCn-IgcOS9GVy9uxaw.webp)

=====================================================

***Part 3:

Terragrunt Configuration:


![Let’s talk about the heart of Terragrunt: the primary configuration file, often named terragrunt.hcl.  This file is where you define how Terragrunt should operate within your infrastructure setup. It’s like the control center for  your Terragrunt configurations.](S6Iqtz0EARKXupU2VzwFYQ.webp)

Let’s talk about the heart of Terragrunt: the primary configuration file, often named terragrunt.hcl. This file is where you define how Terragrunt should operate within your infrastructure setup. It’s like the control center for your Terragrunt configurations.

![](BZ0NzLgSP4YtPSgVjOFz_A.webp)

Terragrunt speaks the HashiCorp Configuration Language (HCL), a language designed for clarity and ease of use. With HCL, you’ll find yourself writing configurations that are not only machine-readable but also human-friendly, making it easier to understand and maintain your infrastructure setup.

![](Rch93X03oH_1WMSJ6zmg7g.webp)

Terragrunt configurations operate on an inheritance model, inheriting properties from underlying Terraform configurations.

Think of it like building blocks: each layer inherits properties from the one beneath it, creating a hierarchical structure that simplifies configuration management and ensures consistency across your projects.

![](qvC96Pd4mETAYYngepig7A.webp)

Terragrunt organizes configuration elements using blocks, providing a structured approach to defining settings.

Within these blocks, you’ll find various components like locals, remote_state, and include, each serving a specific role in enhancing and customizing your Terragrunt configurations.

![](Z8eGDcT497oayVZbMWRaag.webp)

Module configuration in Terragrunt is where we define how Terraform modules should be integrated and utilized within our infrastructure setup. It’s like providing instructions to Terragrunt on how to find, configure, and use the necessary modules effectively.

Within module configuration, we specify critical details such as the source of the Terraform module, version constraints, and any necessary variables. This step essentially provides Terragrunt with a clear roadmap, guiding it to the specific modules required for our infrastructure and ensuring that they are used correctly.

![](XqwEQ8HTkCnGgrVoTm6ZCA.webp)

Variable definitions are a key component of module configuration in Terragrunt. By utilizing var blocks, we can define variables in a structured and modular manner, enhancing organization and promoting reusability within our Terragrunt configurations. This approach ensures that our configurations remain flexible and maintainable, allowing for easy adjustments and scalability as our infrastructure evolves.

![](c7SpqiHwTLJqqsQCuPXj_g.webp)

Now, let’s discuss the remote state configuration aspect within Terragrunt. This part is crucial as it allows us to configure how Terraform manages its state files remotely, ensuring consistency and security in our infrastructure setup.

In remote state configuration, we specify the backend settings for Terraform, such as S3, Azure Blob Storage, or Google Cloud Storage, along with any related parameters. This step essentially tells Terragrunt where to store the state files and how to access them, ensuring that our infrastructure’s state is managed securely and centrally. By configuring remote state settings effectively, we can enhance the reliability and scalability of our infrastructure deployments while minimizing the risk of state-related issues.

ii)Directory Structure:

![](https://miro.medium.com/v2/resize:fit:700/1*fLJILh-m2dF3R93-se_YRQ.png)

Let’s talk about directory Structure in Terragrunt The root directory of our project contains a primary terragrunt.hcl file, referred to as root terragrunt.hcl. This file serves as the central configuration point for Terragrunt, defining settings and behaviors for the entire project. Within our project, we have module directories or repositories containing reusable Terraform configurations specific to each module. These directories house .tf files defining infrastructure resources and variables, promoting code modularity and reuse across different parts of our infrastructure.

![](fLJILh-m2dF3R93-se_YRQ.webp)

Let’s talk about directory Structure in Terragrunt The root directory of our project contains a primary terragrunt.hcl file, referred to as root terragrunt.hcl. This file serves as the central configuration point for Terragrunt, defining settings and behaviors for the entire project. Within our project, we have module directories or repositories containing reusable Terraform configurations specific to each module. These directories house .tf files defining infrastructure resources and variables, promoting code modularity and reuse across different parts of our infrastructure.

Terragrunt supports a hierarchical structure for organizing modules. Modules can be arranged in subdirectories within module directories, providing better management and organization of our infrastructure components. Each environment in our project may have its directory, containing environment-specific Terragrunt configurations. This setup allows for customization and fine-tuning of configurations based on the requirements of each environment, ensuring flexibility and adaptability. We maintain common variable definitions in dedicated locations to be reused by Terragrunt across different environments. This approach encourages centralized management of variables, reducing redundancy and promoting consistency throughout our infrastructure configurations. We’ll see a visual representation of a typical directory structure encompassing these elements. The examples illustrate how our project’s directories and files are organized, providing a clear and structured framework for managing our infrastructure with Terragrunt.

![](https://miro.medium.com/v2/resize:fit:700/1*jJ0ht7EqywqbNrfeJ-G2wA.png)

In our Terragrunt setup, understanding the directory structure is crucial for effective organization and management. Let’s break down the key components:

> ●Root terragrunt.hcl: This file, residing in the root directory, acts as the primary Terragrunt configuration file for our entire project. It’s where we define global settings and behaviors.
> 
> ●Module Directories/Repositories: These directories house reusable Terraform configurations specific to individual modules. Inside, you’ll find .tf files defining infrastructure resources and variables. This setup promotes code modularity and encourages reuse across different parts of our infrastructure.
> 
> ●Organized Module Structure: Terragrunt supports a hierarchical structure for modules, allowing us to organize them into subdirectories for better management. This hierarchical approach streamlines the organization of our infrastructure components.
> 
> ●Environment-Specific Configurations: Each environment in our project may have its directory. Within these directories, we store environment-specific Terragrunt configurations, enabling customization and tailoring of configurations to meet the unique needs of each environment.
> 
> ●Common Variable Definitions: We maintain common variable definitions in dedicated locations for reuse by Terragrunt. This centralized management of variables promotes consistency across different environments and reduces redundancy.
> 
> ●Example Directory Structure: Here’s a visual representation of how these elements come together in a typical directory structure. This example provides a clear illustration of how our project’s directories and files are organized, facilitating efficient management and operation with Terragrunt.

iii)Supporting Files — {account,region,env,common}.hcl


![](bTRvn9deAeHHhMPWH68CFQ.webp)

Let’s explore how supporting files enhance our Terragrunt setup and promote reusability and consistency in our configurations:

Custom Variable Definitions: Supporting files such as {account, region, env, common}.hcl provide custom-tailored variable definitions, favoring reusability across our infrastructure configurations.

Utilization of Environment-Specific Common Configuration: These supporting files allow us to utilize environment-specific common configuration settings effectively. Each file serves a specific purpose: account.hcl: Contains account-specific configuration details like name or account ID.

region.hcl: Stores region-specific configuration such as region name or geolocation.

env.hcl: Houses environment-specific configuration like project name, tags, etc.

common.hcl: Defines global configuration settings common to all environments, like global tags, company name, resource prefixes, etc.

Supporting Files in YAML Format: Alternatively, these supporting files can be in YAML format, providing better third-party integration compared to HCL format. YAML’s readability and compatibility make it an attractive option for seamless integration with other tools and systems.

![](KzIa-_1k9zfidRX435DouQ.webp)

This visual representation illustrates how we incorporate the support files (account.hcl, region.hcl, env.hcl, common.hcl) into our existing directory structure:

• Each support file resides within its respective directory, alongside the Terragrunt configuration files.

• These files add granularity and customization to our infrastructure setup, allowing us to define specific configuration settings tailored to different aspects of our environment.

- By leveraging these support files, we enhance reusability, maintainability, and consistency across our Terragrunt configurations, ultimately streamlining our infrastructure management process.

=====================================================

****iv)Global Resources:

![](8-je24Bc_Gdq1oN0up5-wA.webp)

In our Terragrunt setup, we encounter certain services that don’t fit into the traditional region-based categorization. These are typically services that can be deployed once per account and aren’t tied to specific regions. Let’s take a closer look at how we handle these global services, using AWS as an example: Certain AWS services fall into this category. They are deployed once per AWS account and aren’t constrained by regionspecific configurations. To maintain a clean hierarchy and structure in our Terragrunt setup, we isolate these global services away from region subdirectories. Instead, we organize them at a higher level, separate from region-specific configurations. By separating global services from region-specific configurations, we ensure clarity and maintainability in our directory structure. This approach streamlines management and avoids clutter, allowing us to focus on configuring these global services independently from region-specific resources.

![](hJDVoVHdwpfGzHeqadLLKwA.webp)

Here are some example of global AWS services that are deployed once per account: IAM, Route53 (Public), CloudFront, WAF, and ACM

IAM:users,groups& roles  
Route53:for ur domains  
WAF:you have the web application firewall  
CloudFront:for your CDN  
ACM:for your certificates

M:for your certificates

![](c4ymQsd3PyXGvlg_oQIdaw.webp)

This visual representation demonstrates how we seamlessly integrate global services into our existing directory structure:

This visual representation demonstrates how we seamlessly integrate global services into our existing directory structure:

• Global services such as IAM, Route53 (Public), CloudFront, WAF, and ACM are placed at a higher level in our directory hierarchy, separate from region-specific configurations.

• This separation ensures clarity and organization in our Terragrunt setup, as it distinguishes global services from regionspecific resources.

• By isolating global services in their dedicated directory or directories, we maintain a clean hierarchy and structure, making it easier to manage and understand our infrastructure configurations.

Source: Some Data & Demo example’s are taken from “[https://kodekloud.com/](https://kodekloud.com/)”