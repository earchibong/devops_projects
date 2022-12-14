# AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 4 – TERRAFORM CLOUD
### introduction
<br>

The sole aim of this project to build the infrastructure in AWS using terraform. But, instead of running the Terraform codes in 
<a href="https://github.com/earchibong/devops_training/terraform_03.md">project 18</a> from a command line, it will be executed from **Terraform cloud** console. 

The AMI is built differently with `packer` while Ansible is used to configure the infrastructure after its been provisioned by Terraform.

The following outlines the steps:

<br>

## STEP ONE: Migrate `.tf` codes to Terraform Cloud

<br>

- 1. Create `Terraform Cloud` Account : follow <a href="https://app.terraform.io/public/signup/account">this link</a> to do so
- 2. Create an organization:
  - Select `Start from scratch`, choose a name for your organization and create it. 

<br>

![organisation](https://user-images.githubusercontent.com/92983658/207548121-befda977-0466-4707-a40a-b09cd6b0e864.png)

<br>

- 3. Configure a workspace:
  - *note: understand the difference between `version control workflow`, `CLI-driven workflow` and `API-driven workflow`. Learn more about it <a   href="https://www.youtube.com/watch?v=m3PlM4erixY&t=287s">here</a>* 
  - As the most common and recommended way to run Terraform commands triggered from our git repository, `version control workflow` will be used   for this project.
  
 <br>

  - Create a new repository in GitHub and call it `terraform-cloud`.
  - push Terraform codes developed in the previous projects to the repository.: 
    - on terminal, change Git remote url with `git remote set-url` command: 
     - `git remote set-url origin <repo url>`
     - `git push`
     
 <br>
  
  - in terraform cloud, choose `version control workflow` and you will be promped to connect the GitHub account to your workspace – follow the prompt and add your newly created repository to the workspace. 
  
<br>

![workspace_1a](https://user-images.githubusercontent.com/92983658/207563508-32f25fb7-a91b-4eff-a9fe-61c2a4233a59.png)

<br>

![wordspace_1b](https://user-images.githubusercontent.com/92983658/207563539-30328ef2-de15-4e09-9ba1-0bcf6a31b85c.png)

<br>

- 4. Configure variables:

Terraform Cloud supports two types of variables: environment variables and Terraform variables: 
   - Set two environment variables: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, set the values that you used in <a href="https://github.com/earchibong/devops_training/blob/main/terraform_01.md">Project 16</a>
 
<br>

![variables_1a](https://user-images.githubusercontent.com/92983658/207566827-515a1506-24ce-4ad5-8fcc-82e617e2fdc4.png)

<br>

![variables_1b](https://user-images.githubusercontent.com/92983658/207566862-29e4f41a-b063-40c3-bc36-75ab374783e3.png)

<br>

Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources.

<br>

## STEP TWO: Build AMI images with Packer

- install `packer`: get instructions <a href="https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli"> here</a>
