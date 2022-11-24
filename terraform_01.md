<br>

# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

This project demonstrates how the AWS infrastructure for 2 websites that was built manually in project 15 is automated with the use of Terraform.

The following outlines the steps taken:

<br>

![image](https://user-images.githubusercontent.com/92983658/203531844-0d082b08-87b1-4b95-ad5d-c921e0b61611.png)

<br>

## Pre-requisites:

- **Create an IAM user, name it `terraform` (ensure that the user has only programatic access to your AWS account) and grant this user 
 `Administrator` Access` permissions**
- **Copy the secret access key and access key ID. Save them in a notepad temporarily.**
 - search for `IAM` in AWS search console
 - click `user` on `IAM` dashboard and then `add users`
 - select `programmatic access` for AWS credential type
 
<br>

![create_user_1a](https://user-images.githubusercontent.com/92983658/203534121-91fdf847-6ff0-4d83-b25c-46a35521cb22.png)

<br>

- select `add user to group` -> `create group`
- select `adminitrator access`

<br>

![create_user_1b](https://user-images.githubusercontent.com/92983658/203535472-e4dc8957-1dc2-4e06-80fb-1c5f1c81ab29.png)

<br>

![create_user_ic](https://user-images.githubusercontent.com/92983658/203535500-8d445772-a3e3-4e0c-a8f1-fe162936a290.png)

<br>

![create_user_1d](https://user-images.githubusercontent.com/92983658/203535541-8b51d69d-86b6-40a2-862b-0b225234bc5b.png)

<br>

![create_user_1e](https://user-images.githubusercontent.com/92983658/203535596-9d0bdd18-8bf1-4fca-b98c-3aa958effc34.png)

<br>

![create_user_1f](https://user-images.githubusercontent.com/92983658/203535617-d5b1abf3-4a86-4b48-9879-3c74add84264.png)

<br>

- **Configure programmatic access from your workstation to connect to AWS using the access keys copied above and a Python 
SDK (boto3). You must have Python 3.6 or higher on your workstation.**

*If you are on Windows, use gitbash, if you are on a Mac, you can simply open a terminal.*
**For easier authentication configuration – `AWS CLI` with `aws configure` command will be used.**

- Create an S3 bucket to store Terraform state file. You can name it something like <yourname>-dev-terraform-bucket 
 (Note: S3 bucket names must be unique unique within a region partition)
 - search for `S3` in AWS conslole -> `create bucket`

<br>
 
![bucket_1a](https://user-images.githubusercontent.com/92983658/203540854-411bd131-323e-4a03-ad73-8d74b0c2a18b.png)

<br>
 
![bucket_1b](https://user-images.githubusercontent.com/92983658/203540915-6dd84aec-a8f2-48de-ae4a-6102ea9121ef.png)

 <br>
 
 ![bucket_1c](https://user-images.githubusercontent.com/92983658/203541069-89122e34-f691-4e5c-a2ae-f817b9022e31.png)

<br>
 
![cbucket_1g](https://user-images.githubusercontent.com/92983658/203542731-8825e666-4ddb-4e73-b3ee-f5c0e3f173f3.png)

<br>
 
![bucket_1e](https://user-images.githubusercontent.com/92983658/203541131-d116b60e-6b99-46bf-a5eb-8e9701686a8b.png)

<br>
 
- To install AWS SDK boto3 , it is recommended to upgrade the Python to latest version : `brew install python`
- install `pip`: `sudo easy_install pip`
- install `boto3`: `pip3 install boto3`
- install `AWS CLI`: `pip3 install awscli`
- configure credentials with `aws configure` command

<br>

![boto3_install](https://user-images.githubusercontent.com/92983658/203551877-3cebfcb7-27c3-4f87-ba8c-bc5225cb2151.png)

<br>
 
- ensure you can programmatically access AWS account by running following commands in `>python`:
```
python

import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
 
```

- *You should see your previously created S3 bucket name*
<br>

![import_boto](https://user-images.githubusercontent.com/92983658/203561094-043eb2fe-8d12-4492-b1b5-a7b43fc32571.png)

<br>
 
## Step ONE: VPC | SUBNETS | SECURITY GROUPS
- **create a directory structure**
 - on the desktop, create a new folder named `Terraform`
 - Open Visual Studio Code and:
  - Create a folder called `PBL` inside `Terraform`
  - Create a file in the folder, name it `main.tf`

<br>

![pbl](https://user-images.githubusercontent.com/92983658/203767834-666a27bf-88b8-42c3-a666-70567b16cd19.png)

<br>
 
### Provider and VPC resource section

- **Set up Terraform CLI**
 - follow instructuions <a href="https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli">here</a>
 - Add `AWS` as a provider, and a resource to create a VPC in the `main.tf` file.
 *The Provider block informs Terraform that we intend to build infrastructure within AWS.*
 *The Resource block is what creates a VPC.*

```
 
provider "aws" {
  region = "<your AWS region: e.g: eu-central-1>"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}
 
```
<br>
 
*Note: You can change the configuration above to create your VPC in other region that is closer to you. The same applies to all configuration snippets that will follow.*

 <br>
 
![main_1a](https://user-images.githubusercontent.com/92983658/203774406-f9d0006a-6fe7-412b-9815-a7845b58b63e.png)

<br>
 
- download necessary plugins for Terraform to work.
 - run `terraform init` command in `PBL` folder
 
<br>
 
![terraform_init](https://user-images.githubusercontent.com/92983658/203777454-df803779-59a5-4796-8c8f-b3d7524f07f3.png)

<br>
 
*Note that a new directory has been created: .terraform\.... This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that you must execute terraform init again, to download them.*
 
- check to see what terraform intends to create : in `PBL` run `terraform plan`

<br>
 
![plan](https://user-images.githubusercontent.com/92983658/203779799-a2b72e1e-ee52-4ae3-9255-951763623cd2.png)

<br>
 
- if happy with changes planned, execute `terraform apply`
 
<br>
 
![terraform_apply_1a](https://user-images.githubusercontent.com/92983658/203780754-070fac00-1c65-4af1-92c3-4512d18274d6.png)
![terraform_apply_1b](https://user-images.githubusercontent.com/92983658/203780763-6e8f548e-fc51-4639-a763-38cf451833d5.png)

<br>
 
 - *A new file terraform.tfstate is created as a result of the above command which Terraform uses to keeps itself up to date with the 
 exact state of the infrastructure and terraform.tfstate.lock.info file which Terraform uses to track who is running its code against the infrastructure at any point in time*
 
<br>
 
### Subnets resource section
 
According to the architectural design, we require 6 subnets:

- 2 public
- 2 private for webservers
- 2 private for data layer

<br>
 
- create the first 2 public subnets.
 - Add below configuration to the `main.tf` file:
 
```
 
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-west-2a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-west-2b"
}
 
```
 
<br>
 
![subnets_2](https://user-images.githubusercontent.com/92983658/203783514-08232075-253e-4c3e-9a77-1963b69e7e72.png)

<br>
 
*We are creating 2 subnets, therefore declaring 2 resource blocks – one for each of the subnets.*
*We are using the vpc_id argument to interpolate the value of the VPC id by setting it to aws_vpc.main.id. This way, Terraform knows inside which VPC to create the subnet.*
 
 - Run `terraform plan` and `terraform apply`

<br>
 
![terraform_apply_2a](https://user-images.githubusercontent.com/92983658/203784819-108a95ba-7d21-49a0-8b25-155f53d3c9a7.png)
![terraform_apply_2b](https://user-images.githubusercontent.com/92983658/203784824-f368395f-cd29-4f03-917b-8072913073f0.png)
![terraform_aaply_2c](https://user-images.githubusercontent.com/92983658/203784838-e6d1e217-073f-4fa9-b966-bd6e7edc193f.png)

<br>

### REFACTOR CODES
- **improve the code by refactoring it.**
 - *Hardcoded values: Both the `availability_zone` and `cidr_block arguments` are hard coded. We should always endeavour to make our work dynamic.*
 - *Multiple Resource Blocks: are declared for each subnet in the code. A single resource block that can dynamically create resources without specifying multiple blocks is needed instead*
 
<br>
 
*First, destroy the current infrastructure. Since we are still in development, this is totally fine. Otherwise, **DO NOT DESTROY** an infrastructure that has been deployed to production.*
 
<br>
 
- To destroy whatever has been created run `terraform destroy` command, and type `yes` after evaluating the plan.

<br>
 
![destroy_1a](https://user-images.githubusercontent.com/92983658/203786782-9c4dd97d-3647-4bf7-9157-292ff6192daa.png)
![destroy_1b](https://user-images.githubusercontent.com/92983658/203786792-672dbeec-5bd5-41c2-93a1-b8a69d608120.png)
![destroy_1c](https://user-images.githubusercontent.com/92983658/203786808-3c9ba2f4-1846-4aec-a4e3-60e74fb8dc68.png)
![destroy_1d](https://user-images.githubusercontent.com/92983658/203786847-3d3ed65b-572c-4b37-9396-d7f157227a10.png)

<br>
 
- In order to make the work dynamic, hard coded values are removed by introducing variables
 - Starting with the provider block, declare a variable named region, give it a default value, and update the provider section by 
 referring to the declared variable.
 
```
 
 variable "region" {
        default = "eu-central-1"
    }

    provider "aws" {
        region = var.region
    }
 
 ```
 
 <br>
 
 - Do the same to `cidr value` in the `vpc` block, and all the other arguments.
 
 ```
 
    variable "region" {
        default = "eu-central-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink

    }
 
 ```
 
 <br>
 
 ![refactir_1a](https://user-images.githubusercontent.com/92983658/203791476-b907efcd-f614-4dc2-8f22-e37ca156c608.png)

 ![refactor_1b](https://user-images.githubusercontent.com/92983658/203791484-8bdd80b4-0dc0-4738-9279-c1cbef1955b0.png)

<br>
 
- **fixing multiple resource blocks:**
 - fetch `Availability zones` from AWS, and replace the hard coded value in the subnet’s availability_zone section.

<br>
 
```
       # Get list of availability zones
        data "aws_availability_zones" "available" {
        state = "available"
        }
 
 ```
 
 <br>
 
 - Introduce a count argument in the subnet block to make use of the new data resource:
 
 ```
 
 # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
 
 ```
 
 *The count tells terraform to invoke a loop to create 2 subnets; and the data resource will return a list object that contains a list of AZs.
But if Terraform is being run with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because the cidr_block still has to be hard coded because the same cidr_block cannot be created twice within the same VPC.*
 
 <br>
 
 - introduce a function cidrsubnet() that accepts 3 parameters to make the cidr block dynamic
 
 ```
 
  # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
 
 ```
 
- introuducing a length() function, which basically determines the length of a given list, map, or string solves the problem of the hard coded count value and makes it more dynamic.
```
 
# Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = length(data.aws_availability_zones.available.names)
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
 
```

<br>

- But since this returns the value of 3 instead of 2 that is preferred, a variable for preferred number of public subnets will need to be set:

```

variable "preferred_number_of_public_subnets" {
  default = 2
}
 
```
 
<br>
 
- Next, update the count argument with a condition. Terraform needs to check first if there is a desired number of subnets. Otherwise, use the data returned by the lenght function. See how that is presented below.
 
```
 
# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}
 
```
 
<br>
 
![config_1a](https://user-images.githubusercontent.com/92983658/203797111-599a165e-eba5-4398-9f38-9649cdaf0326.png)
![config_1b](https://user-images.githubusercontent.com/92983658/203797134-ba75b8a3-f8ac-4642-8a1e-aa972d101596.png)
![config_1c](https://user-images.githubusercontent.com/92983658/203797179-51b75e28-d55f-4150-adb1-cc0bb562d94a.png)

<br>
 

