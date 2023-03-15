
# Provision An EKS Cluster With Terraform
This project focuses on EKS, and how to get it up and running using Terraform. the following will be covered:

- use Terraform to create a Kubernetes EKS cluster and dynamically add scalable worker nodes
- deploy multiple applications using HELM
- experience more kubernetes objects and how to use them with Helm. Such as Dynamic provisioning of volumes to make pods stateful
- improve on CI/CD skills with Jenkins

## Labs
- <a href="https://github.com/earchibong/devops_training/edit/main/kubenetes_04.md#building-eks-with-terraform">Building EKS With Terraform</a>
- 

## Building EKS with Terraform

- on laptop, create a new directory named `eks`

```
mkdir eks

```

- Use AWS CLI to create an S3 bucket
```
aws s3api create-bucket \
    --bucket pbl24 \
    --region eu-west-2 \
    --create-bucket-configuration LocationConstraint=eu-west-2
    
```

<br>

<img width="1184" alt="s3_bucket" src="https://user-images.githubusercontent.com/92983658/225295147-b839856c-c58c-42da-8627-4160d80c6aca.png">

<br>

- Create a file – `backend.tf` ensure the backend is configured for remote state in S3
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

terraform {
  backend "s3" {
    bucket         = "pbl24"
    key            = "global/s3/terraform.tfstate"
    region         = "eu-west-2"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

```

<br>

<img width="785" alt="backend" src="https://user-images.githubusercontent.com/92983658/225299661-d3bc94a8-d7dd-45dc-af76-a3e57049ff5b.png">

<br>

- Create a file – `network.tf`
 - provision Elastic IP for Nat Gateway, VPC, Private and public subnets.
 - Create VPC using the official AWS module

```

# reserve Elastic IP to be used in the NAT gateway
resource "aws_eip" "nat_gw_elastic_ip" {
  vpc = true

  tags = {
    Name            = "${var.cluster_name}-nat-eip"
    iac_environment = var.iac_environment_tag
  }
}

module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "${var.name_prefix}-vpc"
  cidr = var.main_network_block
  azs  = data.aws_availability_zones.available_azs.names

  private_subnets = [
    # this loop will create a one-line list as ["10.0.0.0/20", "10.0.16.0/20", "10.0.32.0/20", ...]
    # with a length depending on how many Zones are available
    for zone_id in data.aws_availability_zones.available_azs.zone_ids :
    cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) - 1)
  ]

  public_subnets = [
    for zone_id in data.aws_availability_zones.available_azs.zone_ids :
    cidrsubnet(var.main_network_block, var.subnet_prefix_extension, tonumber(substr(zone_id, length(zone_id) - 1, 1)) + var.zone_offset - 1)
  ]
  enable_nat_gateway     = true
  single_nat_gateway     = true
  one_nat_gateway_per_az = false
  enable_dns_hostnames   = true
  reuse_nat_ips          = true
  external_nat_ip_ids    = [aws_eip.nat_gw_elastic_ip.id]

  # Add VPC/Subnet tags required by EKS
  tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    iac_environment                             = var.iac_environment_tag
  }
  public_subnet_tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/elb"                    = "1"
    iac_environment                             = var.iac_environment_tag
  }
  private_subnet_tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb"           = "1"
    iac_environment                             = var.iac_environment_tag
  }
}

```

<br>

note: The tags added to the subnets is very important. The Kubernetes Cloud Controller Manager (cloud-controller-manager) and AWS Load Balancer Controller (aws-load-balancer-controller) needs to identify the cluster’s. To do that, it querries the cluster’s subnets by using the tags as a filter.

For public and private subnets that use load balancer resources: each subnet must be tagged
```
Key: kubernetes.io/cluster/cluster-name
Value: shared
```

For private subnets that use internal load balancer resources: each subnet must be tagged
```
Key: kubernetes.io/role/internal-elb
Value: 1
```

For public subnets that use internal load balancer resources: each subnet must be tagged
```
Key: kubernetes.io/role/elb
Value: 1
```

<br>
