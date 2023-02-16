# Setting Up A Kubernetes Cluster (Manually)

In this project, a Kubernetes cluster is manually setup from the scratch without any automated helpers in order to better understand each aspect of spinning up a Kubernetes cluster as each components are manually installed from scratch. 

To successfully implement "K8s From-Ground-Up", the following and even more will be done by a K8s administrator:

- Install and configure master (also known as control plane) components and worker nodes (or just nodes).
- Apply security settings across the entire cluster (i.e., encrypting the data in transit, and at rest)
- In transit encryption means encrypting communications over the network using HTTPS
- At rest encryption means encrypting the data stored on a disk
- Plan the capacity for the backend data store etcd
- Configure network plugins for the containers to communicate
- Manage periodical upgrade of the cluster
- Configure observability and auditing

**Note**:The following setup of Kubernetes is just for learning purpose only, and not to be considered for production. This is because setting up a K8s cluster for production use has a lot more moving parts, especially when it comes to planning the nodes, and securing the cluster. 

The purpose of "K8s From-Ground-Up" is to get a better understanding of the different components as shown in the architecture diagram. 

**Tools to be used and expected result of the Project 20**
- VM: AWS EC2
- OS: Ubuntu 20.04 lts+
- Docker Engine
- kubectl console utility
- cfssl and cfssljson utilities
- Kubernetes cluster

**3 EC2 Instances will be created, and in the end, the following parts of the cluster will be properly configured:**

- One Kubernetes Master
- Two Kubernetes Worker Nodes
- Configured SSL/TLS certificates for Kubernetes components to communicate securely
- Configured Node Network
- Configured Pod Network

<br>

## PART ONE: Install Client Tools

### Install and configure AWS CLI
`AWS CLI` is a unified tool to manage your AWS services

- Configure AWS CLI to access all AWS services used:
  - for this, create a user with programmatic access keys configured in AWS Identity and Access Management (IAM)..(i'm using one i created for project 19)
  - Generate access keys and store them in a safe place.

<br>

<img width="1194" alt="IAM" src="https://user-images.githubusercontent.com/92983658/219353590-a820e9aa-9ead-4427-9486-ffa6c8ad8019.png">

<br>

- On your local workstation download and install the <a href="https://aws.amazon.com/cli/">latest version of AWS CLI</a>
- configure `AWS CLI`: `aws configure`
```
# fill in the following details when prompted
AWS Access Key ID [None]: <your IAM access key>
AWS Secret Access Key [None]: < your secret key>
Default region name [None]: < your region >
Default output format [None]: json

```

- Test `AWS CLI` by running: `aws ec2 describe-vpcs`

<br>

<img width="1180" alt="aws_cli_test" src="https://user-images.githubusercontent.com/92983658/219356486-3bc92728-942b-41b2-a780-8df172538484.png">

<br>

### Install Kubectl (on Mac OS)

Kubectl is a command line utility will be the main control tool to manage the K8s cluster. 
*note: access the kubectl <a href="https://kubernetes.io/docs/reference/kubectl/cheatsheet/">Cheat Sheet</a> with the most used `kubectl` commands.

- Download the binary
- Make it executable
- Move to the Bin directory 

```

curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/darwin/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

```

<br>

<img width="1277" alt="kubectl" src="https://user-images.githubusercontent.com/92983658/219358637-36bb4174-d918-45c3-9305-0232845d4852.png">

<br>

### Install CFSSL and CFSSLJSON (on Mac OS)
`cfssl` is an open source toolkit for everything TLS/SSL from Cloudflare
`cfssljson` isa program, which takes the JSON output from the cfssl and writes certificates, keys, CSRs, and bundles to disk.

```

curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssl
curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssljson
chmod +x cfssl cfssljson
sudo mv cfssl cfssljson /usr/local/bin/

```
*note: if you have problems using the binaries directly, use <a href="https://formulae.brew.sh/formula/cfssl"> Homebrew Package Manager</a>

<br>

<img width="1151" alt="cfssl" src="https://user-images.githubusercontent.com/92983658/219361571-fde9cc7c-d42c-435a-bb2a-5523d10eb5a5.png">

<br>

## PART TWO: Set Up AWS Cloud Resources For Kubernetes Cluster

### Step One: Configure Network Infrastructure

#### 1. Virtual Private Cloud – VPC
- Create a directory named `k8s-cluster`
- in the new directory, create a VPC and store the ID as a variable:
```

VPC_ID=$(aws ec2 create-vpc \
--cidr-block 172.31.0.0/16 \
--output text --query 'Vpc.VpcId'
)

```
<br>

<img width="857" alt="k8_vpc" src="https://user-images.githubusercontent.com/92983658/219364710-5015fdd8-3b64-455f-9b6e-d5b94d114fb0.png">

<br>

- Tag the VPC so that it is named:

```

NAME=k8s-cluster-from-ground-up

aws ec2 create-tags \
  --resources ${VPC_ID} \
  --tags Key=Name,Value=${NAME}
  
```

<br>

#### 2. Domain Name System – DNS
- Enable DNS support for VPC:

```

aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-support '{"Value": true}'

```

<br>

- Enable DNS support for hostnames:
```

aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-hostnames '{"Value": true}'

```

<br>

<img width="1197" alt="vpc_dns_confirm" src="https://user-images.githubusercontent.com/92983658/219366622-b792d676-da52-4201-8c2c-d152d1eee1b4.png">

<br>

#### 3. AWS Region

```
AWS_REGION=eu-west-2

```

<br>

#### 4. Dynamic Host Configuration Protocol – DHCP

This is a network management protocol used on Internet Protocol networks for automatically assigning IP addresses and other communication parameters to devices connected to the network using a client–server architecture.

AWS automatically creates and associates a DHCP options set for an Amazon VPC upon creation and sets two options: `domain-name-servers` (defaults to AmazonProvidedDNS) and `domain-name` (defaults to the domain name for your set region). 

`AmazonProvidedDNS` is an Amazon Domain Name System (DNS) server, and this option enables DNS for instances to communicate using DNS names.

By default EC2 instances have fully qualified names like `ip-172-50-197-106.eu-central-1.compute.internal.` But, you can set your own configuration using an example below.

```
DHCP_OPTION_SET_ID=$(aws ec2 create-dhcp-options \
  --dhcp-configuration \
    "Key=domain-name,Values=$AWS_REGION.compute.internal" \
    "Key=domain-name-servers,Values=AmazonProvidedDNS" \
  --output text --query 'DhcpOptions.DhcpOptionsId')
  
```
<br>

- Tag the `DHCP` Option set

```

aws ec2 create-tags \
  --resources ${DHCP_OPTION_SET_ID} \
  --tags Key=Name,Value=${NAME}
  
```

<br>

<img width="1194" alt="dhcp" src="https://user-images.githubusercontent.com/92983658/219370603-24b7eb05-b3bd-4640-a076-24ad1798fb9e.png">

<br>

- Associate the DHCP Option set with the VPC:

```

aws ec2 associate-dhcp-options \
  --dhcp-options-id ${DHCP_OPTION_SET_ID} \
  --vpc-id ${VPC_ID}
  
```

<br>

<img width="1194" alt="vpc_dhcp" src="https://user-images.githubusercontent.com/92983658/219371244-e697538c-1388-4c4a-9312-69af33e92fab.png">

<br>

#### 5. Subnets

- Create the Subnet and tags:

```

SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 172.31.0.0/24 \
  --output text --query 'Subnet.SubnetId')
  
```

<br>

```

aws ec2 create-tags \
  --resources ${SUBNET_ID} \
  --tags Key=Name,Value=${NAME}
  
```

<br>

<img width="1195" alt="subnet" src="https://user-images.githubusercontent.com/92983658/219372126-04018165-e5ed-47e9-95bf-d27f6e101fc8.png">

<br>

#### 5. Internet Gateway

- Create the Internet Gateway and attach it to the VPC:
```

INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
  --output text --query 'InternetGateway.InternetGatewayId')
  
aws ec2 create-tags \
  --resources ${INTERNET_GATEWAY_ID} \
  --tags Key=Name,Value=${NAME}
 
```

<br>

```

aws ec2 attach-internet-gateway \
  --internet-gateway-id ${INTERNET_GATEWAY_ID} \
  --vpc-id ${VPC_ID}

```

<br>

<img width="1197" alt="internet_gateways" src="https://user-images.githubusercontent.com/92983658/219372826-750cd367-929b-4b5b-b95d-76892b436bd4.png">

<br>

#### 6. Route Tables

- Create route tables, associate the route table to subnet, and create a route to allow external traffic to the Internet through the Internet Gateway:

```

ROUTE_TABLE_ID=$(aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --output text --query 'RouteTable.RouteTableId')
  
aws ec2 create-tags \
  --resources ${ROUTE_TABLE_ID} \
  --tags Key=Name,Value=${NAME}
  
aws ec2 associate-route-table \
  --route-table-id ${ROUTE_TABLE_ID} \
  --subnet-id ${SUBNET_ID}
  
aws ec2 create-route \
  --route-table-id ${ROUTE_TABLE_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id ${INTERNET_GATEWAY_ID}

```

<br>

<img width="1063" alt="route_table" src="https://user-images.githubusercontent.com/92983658/219373619-c1894cdb-e238-4c1f-bec0-b0bd41652842.png">

<br>

<img width="1195" alt="route_confirm" src="https://user-images.githubusercontent.com/92983658/219374057-e97d5b6c-807b-495b-89c4-19b20b18b990.png">

<br>

### Step Two: Security Groups

#### 1. Configure Security Groups


```

# Create the security group and store its ID in a variable
SECURITY_GROUP_ID=$(aws ec2 create-security-group \
  --group-name ${NAME} \
  --description "Kubernetes cluster security group" \
  --vpc-id ${VPC_ID} \
  --output text --query 'GroupId')

# Create the NAME tag for the security group
aws ec2 create-tags \
  --resources ${SECURITY_GROUP_ID} \
  --tags Key=Name,Value=${NAME}

# Create Inbound traffic for all communication within the subnet to connect on ports used by the master node(s)
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=2379,ToPort=2380,IpRanges='[{CidrIp=172.31.0.0/24}]'

# # Create Inbound traffic for all communication within the subnet to connect on ports used by the worker nodes
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=30000,ToPort=32767,IpRanges='[{CidrIp=172.31.0.0/24}]'

# Create inbound traffic to allow connections to the Kubernetes API Server listening on port 6443
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 6443 \
  --cidr 0.0.0.0/0

# Create Inbound traffic for SSH from anywhere (Do not do this in production. Limit access ONLY to IPs or CIDR that MUST connect)
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Create ICMP ingress for all types
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol icmp \
  --port -1 \
  --cidr 0.0.0.0/0
  
 ```
 
 <br>

<img width="1383" alt="sg_1a" src="https://user-images.githubusercontent.com/92983658/219376549-c4cae68b-30d0-45e0-874f-4606caa79c8c.png">

<br>

<img width="1384" alt="sg_1b" src="https://user-images.githubusercontent.com/92983658/219376629-f808673c-9a60-4c2f-bee5-c5cd9f09ed1f.png">

<br>

<img width="1383" alt="sg_1c" src="https://user-images.githubusercontent.com/92983658/219376681-c4800e19-68ef-469a-a6eb-19b7556feb8e.png">

<br>

<img width="1379" alt="sg_1d" src="https://user-images.githubusercontent.com/92983658/219376711-63af722b-fe14-4850-9646-6388c0810b9e.png">

<br>

<img width="1383" alt="sg_1e" src="https://user-images.githubusercontent.com/92983658/219376760-af9ced9a-a0ab-4f89-a530-e732fe1e4510.png">

<br>

<img width="1198" alt="AWS_sg_detail" src="https://user-images.githubusercontent.com/92983658/219376831-bfc9c63e-b585-41fe-bede-3382f50262ae.png">

<br>
