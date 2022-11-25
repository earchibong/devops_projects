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

### Create A NAT Gateway
- Create a file called `nat_gateway.tf` and entering the following codes to create a NAT gateway and assign an elastic IP to it:
```

resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on    = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-Nat", var.name)
    },
  )
}

```

<br>

![nat_gateway](https://user-images.githubusercontent.com/92983658/203997774-e2de2a20-f7e4-42bb-bfbe-8d7103764a7c.png)

<br>

### AWS ROUTES
- Create a file called `route_tables.tf` and use it to create routes for both public and private subnets, create the below resources. Ensure they are properly tagged.

  - aws_route_table
  - aws_route
  - aws_route_table_association

<br>

```
# create private route table
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Private-Route-Table", var.name)
    },
  )
}

# associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
  count          = length(aws_subnet.private[*].id)
  subnet_id      = element(aws_subnet.private[*].id, count.index)
  route_table_id = aws_route_table.private-rtb.id
}

# create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table", var.name)
    },
  )
}

# create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
  route_table_id         = aws_route_table.public-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.ig.id
}

# associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
  count          = length(aws_subnet.public[*].id)
  subnet_id      = element(aws_subnet.public[*].id, count.index)
  route_table_id = aws_route_table.public-rtb.id
}

```
<br>

![route_tables](https://user-images.githubusercontent.com/92983658/203999319-cfe81fce-3225-48c2-9400-9e044946a21c.png)

<br>

- run `terraform plan` and `terraform apply` to add the following resources to `AWS` in multi-az set up:

  – the main vpc
  – 2 Public subnets
  – 4 Private subnets
  – 1 Internet Gateway
  – 1 NAT Gateway
  – 1 EIP
  – 2 Route tables

<br>

## Compute and Access Control configuration
