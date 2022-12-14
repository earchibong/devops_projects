# AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 4 – TERRAFORM CLOUD
## introduction
<br>

The sole aim of this project to build the infrastructure in AWS using terraform. But, instead of running the Terraform codes in 
<a href="https://github.com/earchibong/devops_training/terraform_03.md">project 18</a> from a command line, it will be executed from **Terraform cloud** console. 

The AMI is built differently with `packer` while Ansible is used to configure the infrastructure after its been provisioned by Terraform.

The following outlines the steps:

<br>

## STEP ONE: Migrate `.tf` codes to Terraform Cloud

<br>

- Create `Terraform Cloud` Account : follow <a href="https://app.terraform.io/public/signup/account">this link</a> to do so
- 
