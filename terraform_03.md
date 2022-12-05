# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 3 – REFACTORING

<br>

Continuing from <a href="https://github.com/earchibong/devops_training/blob/main/terraform_02.md"> Project 17</a>, 
the entire code is refactored inorder to simplify the code using a Terraform tool called **Module.**

## STEP 1: Configuring A Backend On S3 Bucket
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

# Note: The bucket name may not work for you since buckets are unique globally in AWS, so you must give it a unique name.

resource "aws_s3_bucket" "terraform_state" {
  bucket = "<your s3 bucket>"
  # Enable versioning so we can see the full revision history of our state files
  versioning {
    enabled = true
  }
  # Enable server-side encryption by default
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

```

<br>

*note: Terraform stores secret data inside the state files. Passwords, and secret keys processed by resources are always stored in there.
Hence, consider to always enable encryption. This is achieved that with `server_side_encryption_configuration.`*

<br>

![s3](https://user-images.githubusercontent.com/92983658/205692365-af4dff02-c063-430d-a96a-96d695991f65.png)

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


