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
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_pivate_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
}

```

<br>

*note: for CIDR_block, we use `count.index + 2` so terraform can continue the count after it has created 2 public subnets. THis will prevent
a conflict with both private and public subnets sharing the same cidr_block*

<br>

- update `variables.tf`

<br>

```
variable "preferred_number_of_private_subnets" {
  default = null
  type = number
  description: "number of private subnets"
}

```

<br>

- update `terraform.tfvars`: add the following -> `preferred_number_of_private_subnets = 4`
                                                             
