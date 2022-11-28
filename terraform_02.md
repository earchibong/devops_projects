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

## Compute and Access Control Configuration

### AWS Identity and Access Management: IaM and Roles
Pass an IAM role to EC2 instances to give them access to some specific resources.

- **Create `AssumeRole`**
Typically, `AssumeRole` is used within an `AWS` account or for cross-account access.
It uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token.

- Add the following code to a new file named `roles.tf` 
This will create `AssumeRole` with `AssumeRole policy`. It grants to an entity, in our case it is an EC2, permissions to assume the role.

<br>

```
resource "aws_iam_role" "ec2_instance_role" {
name = "ec2_instance_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      },
    ]
  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume role"
    },
  )
}

```

<br>

![assume_role](https://user-images.githubusercontent.com/92983658/204278531-c80e6cf2-0f40-4627-acac-fc21e073dbe6.png)

<br>

- Create `IAM` policy for this role

<br>

```
resource "aws_iam_policy" "policy" {
  name        = "ec2_instance_policy"
  description = "A test policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:Describe*",
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]

  })

  tags = merge(
    var.tags,
    {
      Name =  "aws assume policy"
    },
  )

}

```

<br>

![iam_policy](https://user-images.githubusercontent.com/92983658/204279005-9124a8dd-0edd-4d64-97e1-49eaef2eabf4.png)

<br>

- Attach the Policy to the `IAM Role`

<br>

```

resource "aws_iam_role_policy_attachment" "test-attach" {
        role       = aws_iam_role.ec2_instance_role.name
        policy_arn = aws_iam_policy.policy.arn
    }
    
```

<br>

- Create an `Instance Profile` and interpolate the `IAM Role`

<br>

```

 resource "aws_iam_instance_profile" "ip" {
        name = "aws_instance_profile_test"
        role =  aws_iam_role.ec2_instance_role.name
    }
    
```

<br>

![IAM_1c](https://user-images.githubusercontent.com/92983658/204279536-a179a165-81bc-4b91-bdeb-6580a1497833.png)

<br>

## Create Compute Resources
As per the architecture the following needs to be created:

- Security Groups
- Target Group for Nginx, WordPress and Tooling
- Certificate from AWS certificate manager
- an External Application Load Balancer and Internal Application Load Balancer.
- launch template for Bastion, Tooling, Nginx and WordPress
- an Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress
- Elastic Filesystem
- Relational Database (RDS)

<br>

### Create Security Groups

create all the security groups in a single file, then refrence this security group within each resources that needs it.

- Create a file and name it `security.tf`, copy and paste the code below

```

# security group for alb, to allow acess from any where for HTTP and HTTPS traffic
resource "aws_security_group" "ext-alb-sg" {
  name        = "ext-alb-sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow TLS inbound traffic"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

 tags = merge(
    var.tags,
    {
      Name = "ext-alb-sg"
    },
  )

}

# security group for bastion, to allow access into the bastion host from you IP
resource "aws_security_group" "bastion_sg" {
  name        = "vpc_web_sg"
  vpc_id = aws_vpc.main.id
  description = "Allow incoming HTTP connections."

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

   tags = merge(
    var.tags,
    {
      Name = "Bastion-SG"
    },
  )
}

#security group for nginx reverse proxy, to allow access only from the extaernal load balancer and bastion instance
resource "aws_security_group" "nginx-sg" {
  name   = "nginx-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

   tags = merge(
    var.tags,
    {
      Name = "nginx-SG"
    },
  )
}

resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ext-alb-sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

# security group for ialb, to have acces only from nginx reverser proxy server
resource "aws_security_group" "int-alb-sg" {
  name   = "my-alb-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "int-alb-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-ialb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.nginx-sg.id
  security_group_id        = aws_security_group.int-alb-sg.id
}

# security group for webservers, to have access only from the internal load balancer and bastion instance
resource "aws_security_group" "webserver-sg" {
  name   = "my-asg-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "webserver-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.int-alb-sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

# security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port
resource "aws_security_group" "datalayer-sg" {
  name   = "datalayer-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

 tags = merge(
    var.tags,
    {
      Name = "datalayer-sg"
    },
  )
}

resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

```

*note: the `aws_security_group_rule` was used to refrence another security group in a security group.*

<br>

![security_1a](https://user-images.githubusercontent.com/92983658/204284696-df288d29-e5d7-4da0-a6ae-69d46038db1b.png)
![security_1b](https://user-images.githubusercontent.com/92983658/204284721-1f2b77ad-ee5b-44a7-a4a7-a6c5e8e8c7ac.png)
![security_1c](https://user-images.githubusercontent.com/92983658/204284737-bc2d8a3d-290a-49c8-8ac0-4ddd1a4f86ee.png)
![security_1d](https://user-images.githubusercontent.com/92983658/204284752-d0cec85d-39d5-43b4-bef8-2b6822067ab2.png)
![security_1f](https://user-images.githubusercontent.com/92983658/204284781-06a5b656-6ea3-41cf-923e-933934c50a10.png)
![security_1g](https://user-images.githubusercontent.com/92983658/204285319-3e50f77f-c383-4662-9052-6614c6138ed7.png)
![security_1h](https://user-images.githubusercontent.com/92983658/204285339-ff1205e6-34b3-406d-b534-51161e95fd4c.png)
![security_1i](https://user-images.githubusercontent.com/92983658/204285352-f0a67cbb-fe4d-4e76-97f3-0ad1c5f79a2c.png)
![security_1j](https://user-images.githubusercontent.com/92983658/204285380-864162bd-c0fa-478e-934a-4c949832855a.png)

<br>

### Create Certificate From Amazon Certificate Manager

- Create `cert.tf` file and add the following code snippets to it.
*NOTE: Be sure to change the domain name to your own domain name and every other name that needs to be changed.*

```

# The entire section creates a certificate, public zone, and validates the certificate using DNS method

# Create the certificate using a wildcard for all the domains created in archibong.link
resource "aws_acm_certificate" "archibong" {
  domain_name       = "*.archibong.link"
  validation_method = "DNS"
}

# calling the hosted zone
data "aws_route53_zone" "archibong" {
  name         = "archibong.link"
  private_zone = false
}

# selecting validation method
resource "aws_route53_record" "archibong" {
  for_each = {
    for dvo in aws_acm_certificate.archibong.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.archibong.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "archibong" {
  certificate_arn         = aws_acm_certificate.archibong.arn
  validation_record_fqdns = [for record in aws_route53_record.archibong : record.fqdn]
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.archibong.zone_id
  name    = "tooling.archibong.link"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}

# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.archibong.zone_id
  name    = "wordpress.archibong.link"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}

```

<br>

### Create an external (Internet facing) Application Load Balancer (ALB)
create the ALB, then create the target group and lastly, create the lsitener rule.

<br>

- Create a file called `alb.tf`
- create an ALB to balance the traffic between the Instances:

```

resource "aws_lb" "ext-alb" {
  name     = "ext-alb"
  internal = false
  security_groups = [
    aws_security_group.ext-alb-sg.id,
  ]

  subnets = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

   tags = merge(
    var.tags,
    {
      Name = "ACS-ext-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}

```

<br>

- To inform the ALB to where to route the traffic, create a Target Group to point to its targets:

<br>

```
resource "aws_lb_target_group" "nginx-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }
  name        = "nginx-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}

```
<br>

- create a `Listner` for this target Group

<br>

```
resource "aws_lb_listener" "nginx-listner" {
  load_balancer_arn = aws_lb.ext-alb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.archibong.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}

```

<br>

- Add the following outputs to `output.tf` to print them on screen

<br>

```

output "alb_dns_name" {
  value = aws_lb.ext-alb.dns_name
}

output "alb_target_group_arn" {
  value = aws_lb_target_group.nginx-tgt.arn
}

```
<br>

