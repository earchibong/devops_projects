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

![alb_1a](https://user-images.githubusercontent.com/92983658/204292778-44e45bae-dc6d-4f48-a4b3-56917f53752c.png)
![alb_1b](https://user-images.githubusercontent.com/92983658/204292790-ad68c4d8-acf3-47a3-b882-7a4498c80519.png)

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

![output](https://user-images.githubusercontent.com/92983658/204292916-2808bb3b-fcb9-465a-b011-a6158396d865.png)

<br>

### Create an Internal (Internal) Application Load Balancer (ALB)

For the Internal Load balancer, the concepts as with the external load balancer.

<br>

- Add the code snippets inside the `alb.tf` file

<br>

```
# ----------------------------
#Internal Load Balancers for webservers
#---------------------------------

resource "aws_lb" "ialb" {
  name     = "ialb"
  internal = true
  security_groups = [
    aws_security_group.int-alb-sg.id,
  ]

  subnets = [
    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "ACS-int-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}

```

<br>

- To inform the ALB to where route the traffic, create a Target Group to point to its targets:

<br>

```

# --- target group  for wordpress -------

resource "aws_lb_target_group" "wordpress-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "wordpress-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}

# --- target group for tooling -------

resource "aws_lb_target_group" "tooling-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "tooling-tgt"
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
# For this aspect a single listener was created for the wordpress which is default,
# A rule was created to route traffic to tooling when the host header changes

resource "aws_lb_listener" "web-listener" {
  load_balancer_arn = aws_lb.ialb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.archibong.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.wordpress-tgt.arn
  }
}

# listener rule for tooling target

resource "aws_lb_listener_rule" "tooling-listener" {
  listener_arn = aws_lb_listener.web-listener.arn
  priority     = 99

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tooling-tgt.arn
  }

  condition {
    host_header {
      values = ["tooling.archibong.link"]
    }
  }
}

```

<br>

![internal_1a](https://user-images.githubusercontent.com/92983658/204294479-53646d2e-24bc-454b-a908-c6065991696d.png)
![internal_1b](https://user-images.githubusercontent.com/92983658/204294487-3c476c36-1ac9-4446-a9ea-9300ed7203ce.png)
![internal_1c](https://user-images.githubusercontent.com/92983658/204294501-47ba5f02-6172-4c00-9f17-edc31154cfc7.png)
![internal_alb](https://user-images.githubusercontent.com/92983658/204519871-a6dbc955-65e8-487d-8095-fcee7e697c0e.png)

<br>

### Create Auto Scaling Groups  
Based on the Architetcture, Auto Scaling Groups for bastion, nginx, wordpress and tooling are needed. So, two files will be created; `asg-bastion-nginx.tf` will contain Launch Template and Austoscaling froup for Bastion and Nginx, then `asg-wordpress-tooling.tf` will contain Launch Template and Austoscaling group for wordpress and tooling.

<br>

- Create `asg-bastion-nginx.tf` and add the code snippet below:

<br>

```

#### creating sns topic for all the auto scaling groups
resource "aws_sns_topic" "david-sns" {
name = "Default_CloudWatch_Alarms_Topic"
}

```

<br>

- create notification for all the auto scaling groups

<br>

```

resource "aws_autoscaling_notification" "david_notifications" {
  group_names = [
    aws_autoscaling_group.bastion-asg.name,
    aws_autoscaling_group.nginx-asg.name,
    aws_autoscaling_group.wordpress-asg.name,
    aws_autoscaling_group.tooling-asg.name,
  ]
  notifications = [
    "autoscaling:EC2_INSTANCE_LAUNCH",
    "autoscaling:EC2_INSTANCE_TERMINATE",
    "autoscaling:EC2_INSTANCE_LAUNCH_ERROR",
    "autoscaling:EC2_INSTANCE_TERMINATE_ERROR",
  ]

  topic_arn = aws_sns_topic.david-sns.arn
}

```

<br>

![notification](https://user-images.githubusercontent.com/92983658/204522498-24805a41-f278-462e-bfe3-6a4a8aa2cf8c.png)

<br>

#### Create Launch Template For Bastion
- add the following to `asg-bastion-nginx.tf`:

<br>

```

resource "random_shuffle" "az_list" {
  input        = data.aws_availability_zones.available.names
}

resource "aws_launch_template" "bastion-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.bastion_sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

   tags = merge(
    var.tags,
    {
      Name = "bastion-launch-template"
    },
  )
  }

  user_data = filebase64("${path.module}/bastion.sh")
}

# ---- Autoscaling for bastion  hosts

resource "aws_autoscaling_group" "bastion-asg" {
  name                      = "bastion-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  launch_template {
    id      = aws_launch_template.bastion-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "bastion-launch-template"
    propagate_at_launch = true
  }

}

# launch template for nginx

resource "aws_launch_template" "nginx-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.nginx-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name =  var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
    var.tags,
    {
      Name = "nginx-launch-template"
    },
  )
  }

  user_data = filebase64("${path.module}/nginx.sh")
}

# ------ Autoscslaling group for reverse proxy nginx ---------

resource "aws_autoscaling_group" "nginx-asg" {
  name                      = "nginx-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  launch_template {
    id      = aws_launch_template.nginx-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "nginx-launch-template"
    propagate_at_launch = true
  }

}

# attaching autoscaling group of nginx to external load balancer
resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
  autoscaling_group_name = aws_autoscaling_group.nginx-asg.id
  alb_target_group_arn   = aws_lb_target_group.nginx-tgt.arn
}

```

<br>

#### Create Launch Template For Wordpress and Tooling 

- Create `asg-wordpress-tooling.tf` and paste the following code:

<br>

```
  
# launch template for wordpress

resource "aws_launch_template" "wordpress-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.webserver-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
    var.tags,
    {
      Name = "wordpress-launch-template"
    },
  )

  }

  user_data = filebase64("${path.module}/wordpress.sh")
}

# ---- Autoscaling for wordpress application

resource "aws_autoscaling_group" "wordpress-asg" {
  name                      = "wordpress-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1
  vpc_zone_identifier = [

    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  launch_template {
    id      = aws_launch_template.wordpress-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "wordpress-asg"
    propagate_at_launch = true
  }
}

# attaching autoscaling group of  wordpress application to internal loadbalancer

resource "aws_autoscaling_attachment" "asg_attachment_wordpress" {
  autoscaling_group_name = aws_autoscaling_group.wordpress-asg.id
  alb_target_group_arn   = aws_lb_target_group.wordpress-tgt.arn
}

# launch template for toooling

resource "aws_launch_template" "tooling-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.webserver-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

  tags = merge(
    var.tags,
    {
      Name = "tooling-launch-template"
    },
  )

  }

  user_data = filebase64("${path.module}/tooling.sh")
}

# ---- Autoscaling for tooling -----

resource "aws_autoscaling_group" "tooling-asg" {
  name                      = "tooling-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [

    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  launch_template {
    id      = aws_launch_template.tooling-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "tooling-launch-template"
    propagate_at_launch = true
  }
}

# attaching autoscaling group of  tooling application to internal loadbalancer

resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
  autoscaling_group_name = aws_autoscaling_group.tooling-asg.id
  alb_target_group_arn   = aws_lb_target_group.tooling-tgt.arn
}

```

<br>

- add new input variables to `variables.tf`:

<br>

```


variable "ami" {
  type        = string
  description = "AMI ID for the launch template"
}

variable "keypair" {
  type        = string
  description = "key pair for the instances"
}

```

<br>

![var_ami](https://user-images.githubusercontent.com/92983658/204540800-3aae27a5-d5f2-44ec-87d3-a62b85d8e18d.png)

<br>

- Create four files that will be used as user data for launching the four different servers:

<br>

**For the bastion server `bastion.sh`**

<br>

```

#!/bin/bash 
yum install -y mysql 
yum install -y 
git tmux 
yum install -y ansible

```

<br>

![bastion_sh](https://user-images.githubusercontent.com/92983658/204525306-fa673b26-0244-429d-9e8f-cb8ea97bc0ce.png)

<br>

**Nginx server `nginx.sh`**:

<br>

```
#!/bin/bash
yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone https://github.com/earchibong/ACS-project-config.git
mv /ACS-project-config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /ACS-project-config

```

<br>

![nginx_sh](https://user-images.githubusercontent.com/92983658/204531345-5535684d-e8df-48f1-ac6c-5756598725de.png)

<br>

**For the Wordpress server `wordpress.sh`**:

<br>

```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-0f9364679383ffbc0 fs-8b501d3f:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/ACSadmin/g" wp-config.php 
sed -i "s/password_here/admin12345/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

```

<br>

![wordpress_sh](https://user-images.githubusercontent.com/92983658/204531823-b14ad4e0-d62e-4a9b-a44c-22c7870abd6e.png)

<br>

**for Tooling Webserver `tooling.sh`**:

<br>

```

#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-01c13a4019ca59dbe fs-8b501d3f:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/Livingstone95/tooling-1.git
mkdir /var/www/html
cp -R /tooling-1/html/*  /var/www/html/
cd /tooling-1
mysql -h acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com -u ACSadmin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com', 'ACSadmin', 'admin12345', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

```

<br>

![tooling_sh](https://user-images.githubusercontent.com/92983658/204532939-37f24cc2-00eb-46bf-8073-2f628e913a61.png)

<br>

### Storage And Database

<br>

**Create KMS Key**
- create a file `efs.tf` and add the following code:

<br>

```

# create key from key management system
resource "aws_kms_key" "ACS-kms" {
  description = "KMS key "
  policy      = <<EOF
  {
  "Version": "2012-10-17",
  "Id": "kms-key-policy",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::${var.account_no}:user/<iam user created at the start of project>" },
      "Action": "kms:*",
      "Resource": "*"
    }
  ]
}
EOF
}

# create key alias
resource "aws_kms_alias" "alias" {
  name          = "alias/kms"
  target_key_id = aws_kms_key.ACS-kms.key_id
}

```

<br>

![kms](https://user-images.githubusercontent.com/92983658/204541406-71ad42d0-88af-4257-bee7-a634ea9b92d3.png)

<br>

**Create Elastic File System (EFS) and mount targets**

- add the following code to `efs.tf`:

<br>

```

 # create Elastic file system
resource "aws_efs_file_system" "ACS-efs" {
  encrypted  = true
  kms_key_id = aws_kms_key.ACS-kms.arn

  tags = merge(
    var.tags,
    {
      Name = "ACS-efs"
    },
  )
}

# set first mount target for the EFS 
resource "aws_efs_mount_target" "subnet-1" {
  file_system_id  = aws_efs_file_system.ACS-efs.id
  subnet_id       = aws_subnet.private[2].id
  security_groups = [aws_security_group.datalayer-sg.id]
}

# set second mount target for the EFS 
resource "aws_efs_mount_target" "subnet-2" {
  file_system_id  = aws_efs_file_system.ACS-efs.id
  subnet_id       = aws_subnet.private[3].id
  security_groups = [aws_security_group.datalayer-sg.id]
}

# create access point for wordpress
resource "aws_efs_access_point" "wordpress" {
  file_system_id = aws_efs_file_system.ACS-efs.id

  posix_user {
    gid = 0
    uid = 0
  }

  root_directory {
    path = "/wordpress"

    creation_info {
      owner_gid   = 0
      owner_uid   = 0
      permissions = 0755
    }

  }

}

# create access point for tooling
resource "aws_efs_access_point" "tooling" {
  file_system_id = aws_efs_file_system.ACS-efs.id
  posix_user {
    gid = 0
    uid = 0
  }

  root_directory {

    path = "/tooling"

    creation_info {
      owner_gid   = 0
      owner_uid   = 0
      permissions = 0755
    }

  }
}

```

<br>

**Create MySQL RDS**
- create the `RDS` itself using this snippet of code in `rds.tf` file:

<br>

```

# This section will create the subnet group for the RDS  instance using the private subnet
resource "aws_db_subnet_group" "ACS-rds" {
  name       = "acs-rds"
  subnet_ids = [aws_subnet.private[2].id, aws_subnet.private[3].id]

 tags = merge(
    var.tags,
    {
      Name = "ACS-rds"
    },
  )
}

# create the RDS instance with the subnets group
resource "aws_db_instance" "ACS-rds" {
  allocated_storage      = 20
  storage_type           = "gp2"
  engine                 = "mysql"
  engine_version         = "5.7"
  instance_class         = "db.t2.micro"
  name                   = "libbydb"
  username               = var.master-username
  password               = var.master-password
  parameter_group_name   = "default.mysql5.7"
  db_subnet_group_name   = aws_db_subnet_group.ACS-rds.name
  skip_final_snapshot    = true
  vpc_security_group_ids = [aws_security_group.datalayer-sg.id]
  multi_az               = "true"
}

```

<br>

![rds_2d](https://user-images.githubusercontent.com/92983658/204552306-c50eddb1-ed37-4b4c-a761-514b1369ca48.png)

<br>

- update `variables.tf` with new input variables:

<br>

```

variable "account_no" {
  type        = number
  description = "the account number"
}

variable "db-username" {
  type        = string
  description = "RDS admin username"
}

variable "db-password" {
  type        = string
  description = "RDS master password"
}

```

<br>

![variables_4a](https://user-images.githubusercontent.com/92983658/204552905-c669754d-4911-4779-ab61-3412a72020c6.png)

<br>

- update `terraform.tfvars`

<br>

```

ami = "ami-0b0af3577fe5e3532"

keypair = "devops"

# Ensure to change this to your acccount number
account_no = "1234567890"

master-username = "libby"

master-password = "devopspbl"

```

<br>

![tfvars_3d](https://user-images.githubusercontent.com/92983658/204554558-5f18673f-66ac-43af-8388-15443788755b.png)

<br>

### Project 17 Additional tasks: 
Access project 17 additional task file <a href="https://github.com/earchibong/terraform-pbl/blob/master/summary.md">here</a>

<br>

Access project 17 repository files <a href="https://github.com/earchibong/terraform-pbl/tree/master">here</a>

<br>


