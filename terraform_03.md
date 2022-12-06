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

![alb_variable](https://user-images.githubusercontent.com/92983658/205919443-b6181ae4-90c2-42f9-ae61-2dff33193c3a.png)
![ab_variable](https://user-images.githubusercontent.com/92983658/205919461-b92236c2-2c4f-4941-8332-a079fa2c2fcf.png)
![alb_variable_1c](https://user-images.githubusercontent.com/92983658/205919483-d4846e3d-65d6-441e-84ed-c3fd0e33705f.png)
![alb_variable_1d](https://user-images.githubusercontent.com/92983658/205919503-34a68434-f24f-49c5-b13f-1e4cda3869b0.png)

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


