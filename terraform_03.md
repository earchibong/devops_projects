# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 3 – REFACTORING

<br>

Continuing from <a href="https://github.com/earchibong/devops_training/blob/main/terraform_02.md"> Project 17</a>, 
the entire code is refactored inorder to simplify the code using a Terraform tool called **Module.**

## STEP 1: REFACTOR PROJECT USING MODULES

- Create a folder called `modules`
- Create the following folders inside the modules folder to combine resources of the similar type: `ALB`, `VPC`, `Autoscaling`, `Security`, `EFS`, `RDS`, `Compute`
- Create the following files for each of the folders: `main.tf`, `variables.tf` and `output.tf`
- Break down Terraform codes to have all resources in their respective modules. Combine resources of a similar type into directories within a ‘modules’ directory, for example, like this:

<br>

```

- modules
  - ALB: For Apllication Load balancer and similar resources
  - EFS: For Elastic file system resources
  - RDS: For Databases resources
  - Autoscaling: For Autosacling and launch template resources
  - compute: For EC2 and rlated resources
  - VPC: For VPC and netowrking resources such as subnets, roles, e.t.c.
  - security: for creating security group resources
  
```

<br>

 - move `routes-tables`, `roles`, `natgateway`, `internet-gateway` and `main.tf` into the `modules/VPC` folder
 - move `rds.tf` to `modules/RDS`
 - move `alb.tf`, `cert.tf` and `output.tf` to `modules/ALB`
 - move `efs.tf` to `modules/EFS`
 - move `asg-bastion`, `asg-wordpress`, `bastion.sh`, `nginx.sh`, `tooling.sh`, `wordpress.sh` into `modules/autoscaling`
 - move `security.tf` to `modules/security`
<br>


- Each module should contain following files:

<br>

```
- main.tf (or %resource_name%.tf) file(s) with resources blocks
- outputs.tf (optional, if you need to refer outputs from any of these resources in your root module)
- variables.tf (as we learned before - it is a good practice not to hard code the values and use variables)

```

<br>

- in `PBL` create the following files: `main.tf`, `providers.tf`

<br>

### Refactor Modules/ALB Folder:

**`output.tf` file refactor**:

```
output "alb_dns_name" {
  value       = aws_lb.ext-alb.dns_name
  description = "External load balance arn"
}

output "nginx-tgt" {
  description = "External Load balancaer target group"
  value       = aws_lb_target_group.nginx-tgt.arn
}


output "wordpress-tgt" {
  description = "wordpress target group"
  value       = aws_lb_target_group.wordpress-tgt.arn
}


output "tooling-tgt" {
  description = "Tooling target group"
  value       = aws_lb_target_group.tooling-tgt.arn
}

```

<br>

![alb_output](https://user-images.githubusercontent.com/92983658/205914879-29bbfdad-b46a-4466-b39c-f46fbd4159f1.png)

<br>

**`variable.tf` file refactor:**

```
# The security froup for external loadbalancer
variable "public-sg" {
  description = "Security group for external load balancer"
}


# The public subnet froup for external loadbalancer
variable "public-sbn-1" {
  description = "Public subnets to deploy external ALB"
}
variable "public-sbn-2" {
  description = "Public subnets to deploy external  ALB"
}


variable "vpc_id" {
  type        = string
  description = "The vpc ID"
}


variable "private-sg" {
  description = "Security group for Internal Load Balance"
}

variable "private-sbn-1" {
  description = "Private subnets to deploy Internal ALB"
}
variable "private-sbn-2" {
  description = "Private subnets to deploy Internal ALB"
}

variable "ip_address_type" {
  type        = string
  description = "IP address for the ALB"

}

variable "load_balancer_type" {
  type        = string
  description = "te type of Load Balancer"
}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}


variable "name" {
  type        = string
  description = "name of the loadbalancer"

}

```

<br>

![ab_variable](https://user-images.githubusercontent.com/92983658/205919461-b92236c2-2c4f-4941-8332-a079fa2c2fcf.png)
![alb_variable_1c](https://user-images.githubusercontent.com/92983658/205919483-d4846e3d-65d6-441e-84ed-c3fd0e33705f.png)
![alb_variable_1d](https://user-images.githubusercontent.com/92983658/205919503-34a68434-f24f-49c5-b13f-1e4cda3869b0.png)

<br>

### Refactor for `Modules/AutoScaling`:

**refactor `variables.tf`:**

```

variable "ami-web" {
  type        = string
  description = "ami for webservers"
}

variable "instance_profile" {
  type        = string
  description = "Instance profile for launch template"
}


variable "keypair" {
  type        = string
  description = "Keypair for instances"
}

variable "ami-bastion" {
  type        = string
  description = "ami for bastion"
}

variable "web-sg" {
  type        = list(any)
  description = "security group for webservers"
}

variable "bastion-sg" {
  type        = list(any)
  description = "security group for bastion"
}

variable "nginx-sg" {
  type        = list(any)
  description = "security group for nginx"
}

variable "private_subnets" {
  type        = list(any)
  description = "first private subnets for internal ALB"
}


variable "public_subnets" {
  type        = list(any)
  description = "Seconf subnet for ecternal ALB"
}


variable "ami-nginx" {
  type        = string
  description = "ami for nginx"
}

variable "nginx-alb-tgt" {
  description = "nginx reverse proxy target group"
}

variable "wordpress-alb-tgt" {
  description = "wordpress target group"
}


variable "tooling-alb-tgt" {
  description = "tooling target group"
}


variable "max_size" {
  type        = number
  description = "maximum number for autoscaling"
}

variable "min_size" {
  type        = number
  description = "minimum number for autoscaling"
}

variable "desired_capacity" {
  type        = number
  description = "Desired number of instance in autoscaling group"

}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}

```

<br>

![auto_variable_1a](https://user-images.githubusercontent.com/92983658/205923155-429decfa-6e03-4962-bd8c-6c63aa154f22.png)
![auto_variable_1b](https://user-images.githubusercontent.com/92983658/205923167-9c04a784-7339-4d08-8a0c-bf5fd7f026d2.png)
![auto_variable_1c](https://user-images.githubusercontent.com/92983658/205923195-0d0a714d-de22-4ba0-ab5b-fedc35240261.png)
![auto_variable_1d](https://user-images.githubusercontent.com/92983658/205923209-0c9fdc37-15d8-4f32-ab0d-caf7a1120862.png)

<br>

### Refactor for `Modules/EFS`:

**refactor `variables.tf`:**

```

variable "efs-subnet-2" {
  description = "Second subnet for the mount target"
}

variable "efs-subnet-1" {
  description = "First subnet for the mount target"
}

variable "efs-sg" {
  type        = list
  description = "security group for the file system"

}

variable "account_no" {
  type        = string
  description = "account number for the aws"
} 


variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}

```

<br>

![efs_variables](https://user-images.githubusercontent.com/92983658/206646492-24c5bf3a-13a4-4d36-b7d5-53ca22789cd9.png)

<br>

### Refactor for `Modules/RDS`:

**refactor `variables.tf`:**

```


variable "db-username" {
  type        = string
  description = "The master user name"
}


variable "db-password" {
  type        = string
  description = "Master password"
}

variable "db-sg" {
  type = list
  description = "The DB security group"
}

variable "private_subnets" {
  type        = list
  description = "Private subnets fro DB subnets group"
}


variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}

```

<br>

![rds_variables](https://user-images.githubusercontent.com/92983658/206651757-3bca2e45-7e81-41b4-a9e1-f26fbe6d94fd.png)

<br>

### Refactor for `Modules/Security`:
**refactor `main.tf`:**

```

# create all security groups dynamically
resource "aws_security_group" "ACS" {
  for_each    = local.security_groups
  name        = each.value.name
  description = each.value.description
  vpc_id      = var.vpc_id

 
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = each.value.name
    },
  )
}

```

<br>

![security_main](https://user-images.githubusercontent.com/92983658/206652402-aa769abe-41d5-4e09-ad0e-cd427fea44d6.png)

<br>

**refactor `main.tf`:**

```
output "ALB-sg" {
  value = aws_security_group.ACS["ext-alb-sg"].id
}


output "IALB-sg" {
  value = aws_security_group.ACS["int-alb-sg"].id
}


output "bastion-sg" {
  value = aws_security_group.ACS["bastion-sg"].id
}


output "nginx-sg" {
  value = aws_security_group.ACS["nginx-sg"].id
}


output "web-sg" {
  value = aws_security_group.ACS["webserver-sg"].id
}


output "datalayer-sg" {
  value = aws_security_group.ACS["datalayer-sg"].id
}

```

<br>

![security_outputs](https://user-images.githubusercontent.com/92983658/206652660-eb18a91b-84a2-46dd-8534-70f955287e05.png)

<br>

### Refactor for `Modules/VPC`:

**refactor `outputs.tf`:**

```

output "public_subnets-1" {
  value       = aws_subnet.public[0].id
  description = "The first public subnet in the subnets"
}

output "public_subnets-2" {
  value       = aws_subnet.public[1].id
  description = "The first public subnet"
}


output "private_subnets-1" {
  value       = aws_subnet.private[0].id
  description = "The first private subnet"
}

output "private_subnets-2" {
  value       = aws_subnet.private[1].id
  description = "The second private subnet"
}


output "private_subnets-3" {
  value       = aws_subnet.private[2].id
  description = "The third private subnet"
}


output "private_subnets-4" {
  value       = aws_subnet.private[3].id
  description = "The fourth private subnet"
}


output "vpc_id" {
  value = aws_vpc.main.id
}


output "instance_profile" {
  value = aws_iam_instance_profile.ip.id
}

```

<br>

![vpc_outputs](https://user-images.githubusercontent.com/92983658/206654574-18081904-203f-4b88-bf9a-d7ec422777bb.png)

<br>

**refactor `variables.tf`:**

```


variable "region" {
}

variable "vpc_cidr" {
  type = string
}

variable "enable_dns_support" {
  type = bool
}

variable "enable_dns_hostnames" {
  type = bool
}

variable "enable_classiclink" {
  type = bool
}


variable "preferred_number_of_public_subnets" {
  type = number
}

variable "preferred_number_of_private_subnets" {
  type = number
}

variable "private_subnets" {
  type        = list(any)
  description = "List of private subnets"
}

variable "public_subnets" {
  type        = list(any)
  description = "list of public subnets"

}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}

variable "name" {
  type    = string
  default = "ACS"

}
variable "environment" {
  default = "true"
}

```

<br>

![vpc_variables](https://user-images.githubusercontent.com/92983658/206655223-30fcc2d7-947b-43b5-b07c-0a3704a65d9c.png)

<br>

### Refactor for `Modules/Compute`:

**refactor `main.tf`:**

```

# create instance for jenkins
resource "aws_instance" "Jenkins" {
  ami                         = var.ami-jenkins
  instance_type               = "t2.micro"
  subnet_id                   = var.subnets-compute
  vpc_security_group_ids      = var.sg-compute
  associate_public_ip_address = true
  key_name                    = var.keypair

 tags = merge(
    var.tags,
    {
      Name = "ACS-Jenkins"
    },
  )
}


#create instance for sonbarqube
resource "aws_instance" "sonbarqube" {
  ami                         = var.ami-sonar
  instance_type               = "t2.medium"
  subnet_id                   = var.subnets-compute
  vpc_security_group_ids      = var.sg-compute
  associate_public_ip_address = true
  key_name                    = var.keypair


   tags = merge(
    var.tags,
    {
      Name = "ACS-sonbarqube"
    },
  )
}

# create instance for artifactory
resource "aws_instance" "artifactory" {
  ami                         = var.ami-jfrog
  instance_type               = "t2.medium"
  subnet_id                   = var.subnets-compute
  vpc_security_group_ids      = var.sg-compute
  associate_public_ip_address = true
  key_name                    = var.keypair


  tags = merge(
    var.tags,
    {
      Name = "ACS-artifactory"
    },
  )
}

```

<br>

![compute_main](https://user-images.githubusercontent.com/92983658/206655589-d6fff2fa-3218-4fc0-8faf-00e414e19380.png)

<br>

**refactor `variables.tf`:**

```

variable "subnets-compute" {
    description = "public subnetes for compute instances"
}
variable "ami-jenkins" {
    type = string
    description = "ami for jenkins"
}
variable "ami-jfrog" {
    type = string
    description = "ami for jfrob"
}
variable "ami-sonar" {
    type = string
    description = "ami foir sonar"
}
variable "sg-compute" {
    description = "security group for compute instances"
}
variable "keypair" {
    type = string
    description = "keypair for instances"
}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}

```

<br>

![compute_variables](https://user-images.githubusercontent.com/92983658/206656067-c81f38fb-184e-47aa-88f6-e9f311723fcc.png)

<br>

*Notes: Access the complete code structure in this <a href="https://github.com/earchibong/terraform-refactor.git">Repository</a>

<br>

- configure `providers` and `backends` sections in separate files but place both in the root module

<br>

![providers](https://user-images.githubusercontent.com/92983658/206660092-13a4d293-9a3c-47ec-8358-5f6068ae7f86.png)

<br>





Configuring A Backend On S3 Bucket
<br>
THe following step will be used Re-initialize Terraform to use S3 backend:

- Add S3 and DynamoDB resource blocks before deleting the local state file
- Update terraform block to introduce backend and locking
- Re-initialize terraform
- Delete the local tfstate file and check the one in S3 bucket
- Add outputs
- terraform apply

<br>

By default the Terraform state is stored locally, to store it remotely on AWS using S3 bucket as the backend and also make use of  
DynamoDB as the State Locking the following setup is done:

- Create a file and name it `backend.tf`. Add the below code and replace the name of the S3 bucket you created in `Project-16`.

<br>

```

resource "aws_s3_bucket" "terraform-state" {
  bucket = "libby-dev-terraform-bucket    "
  force_destroy = true
}

resource "aws_s3_bucket_versioning" "version" {
  bucket = aws_s3_bucket.terraform-state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "first" {
  bucket = aws_s3_bucket.terraform-state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

```

<br>

*note: Terraform stores secret data inside the state files. Passwords, and secret keys processed by resources are always stored in there.
Hence, consider to always enable encryption. This is achieved that with `server_side_encryption_configuration.`*

<br>

![s3_1a](https://user-images.githubusercontent.com/92983658/205866873-54d1c060-9b0a-4cec-8414-37f7d80cf5f5.png)


<br>

- create a `DynamoDB table` to handle locks and perform consistency checks.

```

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}

```

<br>

![terraform_locks](https://user-images.githubusercontent.com/92983658/205694662-cfbeff8e-f56f-4e10-8d53-ae8c15120d45.png)

<br>



*note: Terraform expects that both S3 bucket and DynamoDB resources are already created before we configure the backend. 
run `terraform apply` to provision resources.*

<br>

- Configure S3 Backend

```

terraform {
  backend "s3" {
    bucket         = "<your s3-dev-terraform-bucket>"
    key            = "global/s3/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

```

<br>

- confirm `dynamo table` and `s3 backend`

<br>

![terraform_locks_dynamo](https://user-images.githubusercontent.com/92983658/205701130-1912fbba-3a11-4d3f-83fd-252f74504ffc.png)

<br>


