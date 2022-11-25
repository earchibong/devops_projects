# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 2

continuting from <a href="https://github.com/earchibong/devops_training/blob/main/terraform_01.md ">project 16</a>

### Networking
### Creating Private Subnets
- Create 4 private subnets keeping in mind following principles:

  - Make sure you use variables or length() function to determine the number of AZs
  - Use variables and cidrsubnet() function to allocate vpc_cidr for subnets
  - Keep variables and resources in separate files for better code structure and readability
  - Tags all the resources you have created so far. 
   - Explore how to use format() and count functions to automatically tag subnets with its respective number.

<br>

- update `main.tf`

<br>

```
resource "random_shuffle" "az_list" {
  input        = data.aws_availability_zones.available.names
  result_count = var.max_subnets
}

```
*note: as the AZ of the region is not up to 4 that we require to create the private subnets, an error will occur. Therefore `random_shuffle` resource is introduced and then specifying the maximum subnet in the `variable.tf` file.

<br>

```
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = random_shuffle.az_list.result[count.index]
  
tags = merge(
    var.tags,
    {
      Name = format("PrivateSubnet-%s", count.index)
    },
  )
}

```

<br>

*note 2: for CIDR_block, we use `count.index + 2` so terraform can continue the count after it has created 2 public subnets. THis will prevent
a conflict with both private and public subnets sharing the same cidr_block*

<br>

![main_1a](https://user-images.githubusercontent.com/92983658/203994675-757ff84a-4fb6-43aa-8c50-6eeed13ef718.png)

<br>

- update `variables.tf`: add the following to the file...

<br>

```
variable "preferred_number_of_private_subnets" {
  default = null
  type = number
  description = "number of private subnets"
}

variable "max_subnets" {
  default = 4
}

variable "Name" {
  default = "ACS"
  type = string
}

variable "tags" {
  default = {}
  type = map(string)
  description = "a mapping of tags to assign to all resources"
}

```

<br>

![variables_1a](https://user-images.githubusercontent.com/92983658/203994989-9242296f-de7b-4fed-bfc8-84af38689cbf.png)

<br>

- update `terraform.tfvars`: add the following...

<br>

```

preferred_number_of_private_subnets = 4

tags = {
  Enviroment      = "production" 
  Owner-Email     = "hello+@mintedcreative.com"
  Managed-By      = "Terraform"
  Billing-Account = "1234567890"
}

```

<br>

![tfvars](https://user-images.githubusercontent.com/92983658/203995247-b8d3a718-fe29-4e94-b819-a3649c5e63c9.png)

<br>

### Create Internet Gateway
- Create a file called `internet_gateway.tf` and entering the following codes:
```

resource "aws_internet_gateway" "ig" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-%s!", aws_vpc.main.id,"IG")
    } 
  )
}

```

<br>

*note: the use of format() function to dynamically generate a unique name for this resource. The first part of the %s takes the interpolated value of aws_vpc.main.id while the second %s appends a literal string IG and finally an exclamation mark is added in the end.*

<br>

![internet_gateway](https://user-images.githubusercontent.com/92983658/203996593-6024ae62-dbd7-4a89-8314-ab3b8a1b31db.png)

<br>



