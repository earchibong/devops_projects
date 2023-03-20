
# Provision An EKS Cluster With Terraform
This project focuses on EKS, and how to get it up and running using Terraform. the following will be covered:

- use Terraform to create a Kubernetes EKS cluster and dynamically add scalable worker nodes
- deploy multiple applications using HELM
- experience more kubernetes objects and how to use them with Helm. Such as Dynamic provisioning of volumes to make pods stateful
- improve on CI/CD skills with Jenkins

## Labs
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#building-eks-with-terraform">Building EKS With Terraform</a>
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#deploy-applications-with-helm">Deploy Applications With Helm</a>
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#deploy-jenkins-with-helm">Deploy Jenkins With Helm</a>
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#configuring-kubeconfig-file--kubeconfig">Configuring Kube Config File</a> 
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#deploying-devops-tools-helm-charts">Deploying DevOps Tools With Helm</a>
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#deploying-artifactory-with-helm">Deploying Artifactory With Helm</a>
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#deploying-hashicorp-vault-with-helm">Deploying Vault With Helm</a>
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#deploying-prometheus-with-helm">Deploying Premetheus With Helm</a>
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#deploying-grafana-with-helm">Deploying Grafana With Helm</a> 
- <a href="https://github.com/earchibong/devops_training/main/kubenetes_04.md#deploying-elasticsearch-elk-with-helm">Deploying Elastisearch With Helm</a> 

<br>

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

terraform {
  backend "s3" {
    bucket         = "pbl24"
    key            = "global/s3/terraform.tfstate"
    region         = "eu-west-2"
  }
}

```

<br>

<img width="782" alt="backend" src="https://user-images.githubusercontent.com/92983658/225326498-fed2a2c7-1c99-416c-b111-a110eaab605e.png">


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
  
# Enable single NAT Gateway to save some money
# WARNING: this could create a single point of failure, since we are creating a NAT Gateway in one AZ only
# feel free to change these options if you need to ensure full Availability without the need of running 'terraform apply'
# reference: https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/2.44.0#nat-gateway-scenarios

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

- Create a file – `variables.tf`
```

# create some variables
variable "cluster_name" {
type        = string
description = "EKS cluster name."
}
variable "iac_environment_tag" {
type        = string
description = "AWS tag to indicate environment name of each infrastructure object."
}
variable "name_prefix" {
type        = string
description = "Prefix to be used on each infrastructure object Name created in AWS."
}
variable "main_network_block" {
type        = string
description = "Base CIDR block to be used in our VPC."
}
variable "subnet_prefix_extension" {
type        = number
description = "CIDR block bits extension to calculate CIDR blocks of each subnetwork."
}
variable "zone_offset" {
type        = number
description = "CIDR block bits extension offset to calculate Public subnets, avoiding collisions with Private subnets."
}

```

<br>

<img width="781" alt="variables" src="https://user-images.githubusercontent.com/92983658/225303283-1664feca-e29a-41da-b691-2a1701eba3e9.png">

<br>

- Create a file : `data.tf` (This will pull the available AZs for use.)
```
# get all available AZs in our region
data "aws_availability_zones" "available_azs" {
state = "available"
}
data "aws_caller_identity" "current" {} # used for accesing Account ID and ARN

```

<br>

<img width="781" alt="data" src="https://user-images.githubusercontent.com/92983658/225303758-9a1ba2b1-b0f7-4029-91ae-320e5c1df219.png">

<br>

- Create a file – `eks.tf` and provision EKS cluster
note: Create the file only if you are not using existing Terraform code. Otherwise simply append it to the `main.tf` from your existing code
```
module "eks_cluster" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 18.0"
  cluster_name    = var.cluster_name
  cluster_version = "1.22"
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  cluster_endpoint_private_access = true
  cluster_endpoint_public_access = true

  # Self Managed Node Group(s)
  self_managed_node_group_defaults = {
    instance_type                          = var.asg_instance_types[0]
    update_launch_template_default_version = true
  }
  self_managed_node_groups = local.self_managed_node_groups

  # aws-auth configmap
  create_aws_auth_configmap = true
  manage_aws_auth_configmap = true
  aws_auth_users = concat(local.admin_user_map_users, local.developer_user_map_users)
  tags = {
    Environment = "prod"
    Terraform   = "true"
  }
}

```

<br>

<img width="782" alt="eks" src="https://user-images.githubusercontent.com/92983658/225305510-7e25ef0e-6c91-4eec-a104-6e27118c5387.png">

<br>

- Create a file – `locals.tf` to create local variables
```
# render Admin & Developer users list with the structure required by EKS module
locals {
  admin_user_map_users = [
    for admin_user in var.admin_users :
    {
      userarn  = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:user/${admin_user}"
      username = admin_user
      groups   = ["system:masters"]
    }
  ]
  developer_user_map_users = [
    for developer_user in var.developer_users :
    {
      userarn  = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:user/${developer_user}"
      username = developer_user
      groups   = ["${var.name_prefix}-developers"]
    }
  ]

  self_managed_node_groups = {
    worker_group1 = {
      name = "${var.cluster_name}-wg"

      min_size      = var.autoscaling_minimum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
      desired_size  = var.autoscaling_minimum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
      max_size      = var.autoscaling_maximum_size_by_az * length(data.aws_availability_zones.available_azs.zone_ids)
      instance_type = var.asg_instance_types[0].instance_type

      bootstrap_extra_args = "--kubelet-extra-args '--node-labels=node.kubernetes.io/lifecycle=spot'"

      block_device_mappings = {
        xvda = {
          device_name = "/dev/xvda"
          ebs = {
            delete_on_termination = true
            encrypted             = false
            volume_size           = 10
            volume_type           = "gp2"
          }
        }
      }

      use_mixed_instances_policy = true
      mixed_instances_policy = {
        instances_distribution = {
          spot_instance_pools = 4
        }

        override = var.asg_instance_types
      }
    }
  }
}

```

<br>

- add to `variables.tf`
```

# create some variables
variable "admin_users" {
  type        = list(string)
  description = "List of Kubernetes admins."
}
variable "developer_users" {
  type        = list(string)
  description = "List of Kubernetes developers."
}
variable "asg_instance_types" {
  description = "List of EC2 instance machine types to be used in EKS."
}
variable "autoscaling_minimum_size_by_az" {
  type        = number
  description = "Minimum number of EC2 instances to autoscale our EKS cluster on each AZ."
}
variable "autoscaling_maximum_size_by_az" {
  type        = number
  description = "Maximum number of EC2 instances to autoscale our EKS cluster on each AZ."
}

```

<br>

- Create a file – `variables.tfvars` to set values for variables.
```

cluster_name            = "tooling-app-eks"
iac_environment_tag     = "development"
name_prefix             = "libby-io-eks"
main_network_block      = "10.0.0.0/16"
subnet_prefix_extension = 4
zone_offset             = 8

# Ensure that these users already exist in AWS IAM. Another approach is that you can introduce an iam.tf file to manage 
users separately, get the data source and interpolate their ARN.
admin_users                    = ["libby", "grace"]
developer_users                = ["eddy", "abigail"]
asg_instance_types             = [ { instance_type = "t3.small" }, { instance_type = "t2.small" }, ]
autoscaling_minimum_size_by_az = 1
autoscaling_maximum_size_by_az = 10
autoscaling_average_cpu        = 30


```
<br>

<img width="782" alt="tfvars_1a" src="https://user-images.githubusercontent.com/92983658/225309792-839d32a9-be47-4639-b647-10eb47434c29.png">

<br>

- Create file – `provider.tf`
```

provider "aws" {
  region = "eu-west-2"
}

provider "random" {
}

```

<br>

- Run `terraform init`

<br>

<img width="817" alt="terraform_init" src="https://user-images.githubusercontent.com/92983658/225314327-66869b9c-d99b-49af-8a82-9cdd12f05473.png">

<br>


- Run `Terraform plan` - plan should have an output as below:
```
Plan: 53 to add, 0 to change, 0 to destroy.

```

if `terraform apply` is applied to this current plan it will cause some errors at some point because to connect to the cluster using the kubeconfig, Terraform needs to be able to connect and set the credentials correctly.

<br>

<img width="785" alt="plan_1a" src="https://user-images.githubusercontent.com/92983658/225326853-595b3089-8c7f-402b-8ca7-a28ebd4276d2.png">

<br>

- update `data.tf` and `provider.tf` to enable Terraform to connect and set the credentials correctly and connect cluster to kubeconfig

append to `data.tf`
```

# get EKS cluster info to configure Kubernetes and Helm providers
data "aws_eks_cluster" "cluster" {
  name = module.eks_cluster.cluster_id
}
data "aws_eks_cluster_auth" "cluster" {
  name = module.eks_cluster.cluster_id
}

```

<br>

append to `providers.tf`
```

# get EKS authentication for being able to manage k8s objects from terraform
provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
  token                  = data.aws_eks_cluster_auth.cluster.token
}

````

<br>

- Run the `init`, `plan` and `appy` again

<br>

<img width="869" alt="apply" src="https://user-images.githubusercontent.com/92983658/225334488-0d542937-f860-492b-9120-5ffaa1c0bab0.png">

<br>


- Create kubeconfig file using awscli.
```
aws eks update-kubeconfig --name <cluster_name> --region <cluster_region> --kubeconfig kubeconfig

```

<br>

<img width="1376" alt="kubeconfig" src="https://user-images.githubusercontent.com/92983658/225334994-3d01d3ea-7593-4417-a7aa-ffee25de7228.png">

<br>

## Deploy Applications With HELM
A Helm chart is a definition of the resources that are required to run an application in Kubernetes. Instead of having to think about all of the various deployments/services/volumes/configmaps/ etc that make up your application, you can use a command like:
`helm install stable/mysql` and Helm will make sure all the required resources are installed. In addition we can tweak the helm configuration by setting a single variable to a particular value and more or less resources will be deployed. For example, enabling slave for MySQL so that it can have read only replicas.

Behind the scenes, a helm chart is essentially a bunch of YAML manifests that defines all the resources required by the application. Helm takes care of creating the resources in Kubernetes (where they don’t exist) and removing old resources.

### Helm Set Up

```
wget -O helm.tar.gz https://get.helm.sh/helm-v3.8.2-linux-arm.tar.gz
tar -zxvf helm-v3.8.2-linux-amd64.tar.gz
mv linux-arm/helm /usr/local/bin/helm
helm

```

Alternatively, install Helm using other methods <a href="https://helm.sh/docs/intro/install/">here</a>

<br>

### Deploy Jenkins With Helm
- Visit <a href="https://artifacthub.io/packages/search">Artifact Hub</a> to find packaged applications as Helm Charts
- Search for Jenkins

<br>

<img width="1222" alt="helm_artifact" src="https://user-images.githubusercontent.com/92983658/225588087-a9f65341-55cb-4f4e-a4bc-a75c875b3c3e.png">

<br>

- Add the repository to helm so that it can easily be downloaded and deployed, update and install it
```
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install [RELEASE_NAME] jenkins/jenkins --kubeconfig [kubeconfig file]

```

<br>

<img width="1277" alt="heml_jenkins" src="https://user-images.githubusercontent.com/92983658/225588611-789a278b-d342-4ed3-93e9-c96a365c36b4.png">

<br>

<img width="1381" alt="helm_install" src="https://user-images.githubusercontent.com/92983658/225631395-68103ff0-b194-4b5f-bdf7-7a2c44bbc6a7.png">

<br>

*note: if the forllowing error occurs: invalid apiVersion `client.authentication.k8s.io/v1alpha1`...update `AWS CLI`*
```
#for mac os:
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /

#For Linux x86 (64-bit)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install

#For Windows
C:\> msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

```

<br>

- Running some commands to inspect the installation:
```
#Check the Helm deployment
helm ls --kubeconfig [kubeconfig file]

#Check the pods
kubectl get pods --kubeconfig [kubeconfig file]

#Describe the running pod
kubectl describe pod <pod name> --kubeconfig [kubeconfig file]

#check logs of running pods
kubectl logs <pod name> -c jenkins --kubeconfig [kubeconfig file]

```

<br>

<img width="1381" alt="helm_kubeconfig" src="https://user-images.githubusercontent.com/92983658/225638382-20ada17b-877a-423d-b312-da9b1c384d22.png">

<br>

### Configuring Kubeconfig File ( ~/.kube/config)
Kubectl expects to find the default kubeconfig file in the location `~/.kube/config`. But if there is another cluster using that same file, It doesn’t make sense to overwrite it. Instead, it best to to merge all the kubeconfig files together using a kubectl plugin called <a href="https://github.com/corneliusweig/konfig">konfig</a> and select whichever one is needed to be active.

- Install a package manager for kubectl called <a href="https://krew.sigs.k8s.io/docs/user-guide/setup/install/">`krew`</a> so that it will enable you to install plugins to extend the functionality of kubectl.

<br>

<img width="1318" alt="krew" src="https://user-images.githubusercontent.com/92983658/225641279-d2ef1145-364f-4b0e-b3fc-2cf7bcc47a50.png">

<br>

- Install the <a href="https://github.com/corneliusweig/konfig">`konfig plugin`</a>
```
  kubectl krew install konfig
```

- Import the kubeconfig into the default kubeconfig file. Ensure to accept the prompt to overide.
```
  sudo kubectl konfig import --save  [kubeconfig file]

```

<br>

<img width="925" alt="kubectl_krew" src="https://user-images.githubusercontent.com/92983658/225641935-19022996-17d6-43a3-a92d-8d73dcb458c2.png">

<br>

- Show all the contexts (Meaning all the clusters configured in your kubeconfig)
```
  kubectl config get-contexts
  
```

- Set the current context to use for all kubectl and helm commands
```
  kubectl config use-context [name of EKS cluster context]
  
```

<br>

<img width="1381" alt="set_context" src="https://user-images.githubusercontent.com/92983658/225643110-c4578dd0-f506-40dd-b814-004c848ed902.png">

<br>

- Test that it is working without specifying the `--kubeconfig` flag
```
  kubectl get po

```

- Display the current context. This will let you know the context in which you are using to interact with Kubernetes.
```
  kubectl config current-context

```

<br>

<img width="1002" alt="config_context" src="https://user-images.githubusercontent.com/92983658/225643676-e54de62d-76c4-4361-9864-c0afcc293f26.png">

<br>

### Access Jenkins UI With Kubectl
- acquire the Jenkins administrator's password credential 
There are some commands that were provided on the screen when Jenkins was installed with Helm. See above...Get the password to the admin user

```
kubectl exec --namespace default -it svc/pbl24-jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo

```

- Use port forwarding to access Jenkins from the UI
```
echo http://127.0.0.1:8080
kubectl --namespace default port-forward svc/pbl24-jenkins 8080:8080

```

<br>

<img width="1432" alt="port_forward_jenkins" src="https://user-images.githubusercontent.com/92983658/225646101-f874770a-428d-4c93-9090-c802841216ce.png">

<br>

- Go to the browser `localhost:8080` and authenticate with the username and password

<br>

<img width="1227" alt="jenkins_helm" src="https://user-images.githubusercontent.com/92983658/225646744-bcfa7204-49eb-4727-8f17-0d7c84bbfba1.png">


## Deploying DevOps Tools Helm Charts
This section will be setting up the following tools using helm
- Artifactory
- Hashicorp Vault
- Prometheus
- Grafana
- Elasticsearch ELK using ECK

<br>

### Deploying Artifactory With Helm

- Add Artifactory's repository to helm
- create a namespace `tools` where all the tools will be deployed
- update the repo
- install the chart

```
helm repo add jfrog https://charts.jfrog.io
kubectl create ns tools
helm repo update
# Click on the install menu on the repository page to see the installation commands.
helm upgrade --install my-jfrog-platform jfrog/jfrog-platform --version 10.12.0 -n tools

```

<br>

<img width="1470" alt="jfrog_helm" src="https://user-images.githubusercontent.com/92983658/225927674-957cec74-e010-482f-a2c9-1e2458b0e6e5.png">

<br>  


### Deploying Hashicorp Vault With Helm

- add the Hashicorp helm repository and check that you have access to the chart
- update the repo
- install the chart in `tools` namespace

```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm search repo hashicorp/vault
helm repo update
helm upgrade --install my-vault hashicorp/vault --version 0.23.0 -n tools

```

<br>

<img width="1362" alt="vault_helm" src="https://user-images.githubusercontent.com/92983658/225928943-828d8f5c-ac46-4e09-b3c7-0a99448064a7.png">

<br>


### Deploying Prometheus With Helm
```

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install my-prometheus prometheus-community/prometheus --version 19.7.2 -n tools

```

<br>

<img width="1440" alt="helm_promotheus" src="https://user-images.githubusercontent.com/92983658/225930354-ee42ebbc-801c-4cbb-ac4f-29511620e6f0.png">

<br>

- Port forward to access prometheus from the UI:
From the instructions given during installation...

```

#Get the Prometheus server URL by running these commands
export POD_NAME=$(kubectl get pods --kubeconfig kubeconfig --namespace tools -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")


kubectl --namespace tools port-forward $POD_NAME 9090 --kubeconfig kubeconfig
  

```

<br>

<img width="1385" alt="prometheus_port" src="https://user-images.githubusercontent.com/92983658/225938933-5d47da89-f01b-4d27-9b4e-50ace7c1c9b0.png">

<br>

<img width="1229" alt="prometheus_1" src="https://user-images.githubusercontent.com/92983658/225946019-e6de55bd-ab69-4e09-a09d-3d3d00ca3396.png">

<br>

### Deploying Grafana With Helm

```

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install my-grafana grafana/grafana --version 6.52.4 -n tools

```

<br>

<img width="1470" alt="grafana_helm" src="https://user-images.githubusercontent.com/92983658/225931214-3916472f-36cd-462e-b51a-0fde41ff98f1.png">

<br>

- Port forward to access grafana from the UI:
From the instructions given during installation...

```
export POD_NAME=$(kubectl get pods --kubeconfig kubeconfig --namespace tools -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=my-grafana" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace tools port-forward $POD_NAME 3000 --kubeconfig kubeconfig

```

<br>

<img width="1201" alt="grafana_3000" src="https://user-images.githubusercontent.com/92983658/225936421-9bc07358-ef1b-47db-ad8e-b4881230b6d6.png">

<br>

<img width="1228" alt="grafana_2" src="https://user-images.githubusercontent.com/92983658/225936875-cfc0d8ce-e601-4c3a-968f-91ae34162b83.png">

<br>

```

# get grafana password:

kubectl get secret --kubeconfig kubeconfig --namespace tools my-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

#username: admin

```

<br>

<img width="1231" alt="grafana_3" src="https://user-images.githubusercontent.com/92983658/225937696-b51fa09e-a046-4057-8d5c-460421645f4a.png">

<br>


### Deploying Elasticsearch ELK With Helm

```

helm repo add elastic https://helm.elastic.co
helm repo update
helm upgrade --install my-elasticsearch elastic/elasticsearch --version 8.5.1 -n tools

```

<br>

<img width="1386" alt="elastisearch_helm" src="https://user-images.githubusercontent.com/92983658/225932932-9b6e1b85-50ee-48a5-b70c-bd47f0ae6e3d.png">

<br>


