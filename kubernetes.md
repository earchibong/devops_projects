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

## Labs
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md#part-one-install-client-tools"> Installing The Client Tools</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md#part-two-set-up-aws-cloud-resources-for-kubernetes-cluster">Set Up AWS Cloud Resources</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md#part-three-set-up-compute-resources">Provision Compute Resources</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md#part-four-prepare-self-signed-certificate-authority--general-tls-certificates">Provisioning a CA and Generating TLS Certificates</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md#part-five-generating-kubernetes-configuration-files-for-authentication">Generating Kubernetes Configuration Files For Authentication</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md#part-six-generating-the-data-encryption-config-and-key">Generating The Data Encryption Config And Key</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md#part-six-generating-the-data-encryption-config-and-key">Bootstrapping The ETCD Cluster</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md#part-eight-bootstrapping-the-kubernetes-control-plane">Bootstrapping The Kubernetes Control Pane</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md#part-nine-bootstrapping-the-kubernetes-worker-nodes">Bootstrapping the Kubernetes Worker Nodes</a>
- <a href="https://github.com/earchibong/devops_training/edit/main/kubernetes.md#cleaning-up">Cleaning Up</a>

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
- in the new directory, create a custom VPC, store the ID as a variable and tag it so it is named `k8s-cluster`:
```

VPC_ID=$(aws ec2 create-vpc \
--cidr-block 172.31.0.0/16 \
--tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=k8s-cluster}]' \
--output text --query 'Vpc.VpcId'
)

```
<br>

<img width="799" alt="customer_vpc" src="https://user-images.githubusercontent.com/92983658/220583395-7f5b0dec-51f9-485b-935a-eb3dadd26d35.png">

<br>

- store the name ID as a variable:

```

NAME=k8s-cluster
  
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
NAME=k8s-cluster

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
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=k8s-cluster}]' \
  --output text --query 'Subnet.SubnetId')
  
```

*note: The  `172.31.0.0/24` IP address range can host up to 254 compute instances.*

<br>

<img width="1195" alt="subnet" src="https://user-images.githubusercontent.com/92983658/219372126-04018165-e5ed-47e9-95bf-d27f6e101fc8.png">

<br>

#### 5. Internet Gateway

- Create the Internet Gateway and attach it to the VPC:
```

INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=k8s-cluster}]' \
  --output text --query 'InternetGateway.InternetGatewayId')

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
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=k8s-cluster}]' \
  --output text --query 'RouteTable.RouteTableId')
  
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
  --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=k8s-cluster}]' \
  --output text --query 'GroupId')

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

- List the created security group rules:

```

aws ec2 describe-security-group-rules \
  --filters "Name=group-id,Values=${SECURITY_GROUP_ID}" \
  --query 'sort_by(SecurityGroupRules, &CidrIpv4)[].{a_Protocol:IpProtocol,b_FromPort:FromPort,c_ToPort:ToPort,d_Cidr:CidrIpv4}' \
  --output table

```
<br>

<img width="1385" alt="output_tcp" src="https://user-images.githubusercontent.com/92983658/219683137-78b9668d-3533-4055-9f26-19661cbb993d.png">

<br>


#### 2. Network Load Balancer
- Create a network Load balancer
```

LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer \
--name ${NAME} \
--subnets ${SUBNET_ID} \
--scheme internet-facing \
--type network \
--output text --query 'LoadBalancers[].LoadBalancerArn')

```

<br>

<img width="1196" alt="loadbalancer" src="https://user-images.githubusercontent.com/92983658/219378407-d48963ad-4971-4c7d-8cfc-f3bd5494e6d2.png">

<br>

#### 3. Target Groups
- Create a target group: (For now it will be unhealthy because there are no real targets yet.)
```

TARGET_GROUP_ARN=$(aws elbv2 create-target-group \
  --name ${NAME} \
  --protocol TCP \
  --port 6443 \
  --vpc-id ${VPC_ID} \
  --target-type ip \
  --output text --query 'TargetGroups[].TargetGroupArn')
  
```

<br>

<img width="1194" alt="target_1a" src="https://user-images.githubusercontent.com/92983658/219384408-8d29e828-384a-486a-8b29-9edf5534cf72.png">

<br>

- Register targets: (Just like above, no real targets. You will just put the IP addresses so that, when the nodes become available, they will be used as targets.)

```

aws elbv2 register-targets \
  --target-group-arn ${TARGET_GROUP_ARN} \
  --targets Id=172.31.0.1{0,1,2}


```

<br>

<img width="1197" alt="targets" src="https://user-images.githubusercontent.com/92983658/219385680-cf1ea339-fbad-415f-bd8a-1ebc3f60a759.png">

<br>

- Create a listener to listen for requests and forward to the target nodes on TCP port 6443
```

aws elbv2 create-listener \
--load-balancer-arn ${LOAD_BALANCER_ARN} \
--protocol TCP \
--port 6443 \
--default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN} \
--output text --query 'Listeners[].ListenerArn'

```

<br>

<img width="1380" alt="listener" src="https://user-images.githubusercontent.com/92983658/219385514-f6cc78ed-645d-478a-ab3a-e1af30e42d7a.png">

<br>

<img width="1196" alt="listener_1b" src="https://user-images.githubusercontent.com/92983658/219386158-d8824c76-0660-40da-9a62-3e2f02327d13.png">

<br>

#### 4. K8s Public Address
- Get the Kubernetes Public address
```

KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers \
--load-balancer-arns ${LOAD_BALANCER_ARN} \
--output text --query 'LoadBalancers[].DNSName')

```

<br>

<img width="1015" alt="k8s_public_add" src="https://user-images.githubusercontent.com/92983658/219387303-a159a665-fa0e-4e3d-ba3d-be9bd889a742.png">

<br>

## PART THREE: Set Up Compute Resources

### Step One: Create Compute Resources
#### 1. AMI
- Get an image to create EC2 instances:
```

IMAGE_ID=$(aws ec2 describe-images --owners 099720109477 \
  --filters \
  'Name=root-device-type,Values=ebs' \
  'Name=architecture,Values=x86_64' \
  'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-*' \
  | jq -r '.Images|sort_by(.Name)[-1]|.ImageId')
  
  echo ${IMAGE_ID}

```

<br>

<img width="995" alt="ami" src="https://user-images.githubusercontent.com/92983658/219389056-eefdd744-16f1-4be6-864e-2be14dca7d1c.png">

<br>

#### 2. SSH Key-Pair
- Create SSH Key-Pair
```

mkdir -p ssh

aws ec2 create-key-pair \
  --key-name k8s-cluster \
  --output text --query 'KeyMaterial' \
  > ssh/k8s-cluster.id_rsa
  
chmod 600 ssh/k8s-cluster.id_rsa

```

<br>

<img width="1024" alt="keypar" src="https://user-images.githubusercontent.com/92983658/219389092-cc21abae-de51-47ce-b6cb-9def19d340bc.png">

<br>

#### 3. EC2 Instances for Controle Plane (Master Nodes)
- Create 3 Master nodes: *(Note – Using t2.micro instead of t2.small as t2.micro is covered by AWS free tier)*
```

for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name k8s-cluster \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.1${i} \
    --user-data "name=master-${i}" \
    --subnet-id ${SUBNET_ID} \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=master-${i}}]" \
    --output text --query 'Instances[].InstanceId')

  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
done

```

<br>

<img width="1011" alt="master_nodes" src="https://user-images.githubusercontent.com/92983658/219390094-295140b3-e001-4307-bca5-d94c65c99bbe.png">

<br>

<img width="1196" alt="instances_1a" src="https://user-images.githubusercontent.com/92983658/219390378-029b2df9-e6d7-46f0-a172-99e91641f44a.png">

<br>

#### 4. EC2 Instances for Worker Nodes
- Create 3 worker nodes:
```

for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name k8s-cluster \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.2${i} \
    --user-data "name=worker-${i}|pod-cidr=172.20.${i}.0/24" \
    --subnet-id ${SUBNET_ID} \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=worker-${i}}]" \
    --output text --query 'Instances[].InstanceId')

  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
done

```

<br>

<img width="1331" alt="worker_nodes" src="https://user-images.githubusercontent.com/92983658/219391298-5ab19002-f2ac-44cf-bacb-9cdfb9e74488.png">

<br>

<img width="1198" alt="worker_nodes" src="https://user-images.githubusercontent.com/92983658/219391497-93401320-03d6-4c82-8edc-4d1a02ca3d39.png">

<br>

verify instances
```

aws ec2 describe-instances \
  --filters Name=vpc-id,Values=${VPC_ID} \
  --query 'sort_by(Reservations[].Instances[],&PrivateIpAddress)[].{d_INTERNAL_IP:PrivateIpAddress,e_EXTERNAL_IP:PublicIpAddress,a_NAME:Tags[?Key==`Name`].Value | [0],b_ZONE:Placement.AvailabilityZone,c_MACHINE_TYPE:InstanceType,f_STATUS:State.Name}' \
  --output table
  
```

<br>

<img width="1387" alt="verify_instances" src="https://user-images.githubusercontent.com/92983658/219687996-3f8224df-178b-4b62-a802-fa0d54a00722.png">

<br>



## PART FOUR: Prepare Self-Signed Certificate Authority & General TLS Certificates
### Step One: Prepare The Self-Signed Certificate Authority And Generate TLS Certificates
The following components running on the Master node will require TLS certificates.

- kube-controller-manager
- kube-scheduler
- etcd
- kube-apiserver

The following components running on the Worker nodes will require TLS certificates.

- kubelet
- kube-proxy

Therefore, a `PKI Infrastructure` will have to be provisioned using `cfssl` which will have a Certificate Authority. The `CA` will then generate certificates for all the individual components.

#### 1. Self-Signed Root Certificate Authority (CA)
provision a CA that will be used to sign additional TLS certificates.

- Create a directory and cd into it: `mkdir ca-authority && cd ca-authority`
- Generate the CA configuration file, Root Certificate, and Private key:

```
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "Nigeria",
      "L": "Abuja",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "FCT"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}

```

<br>

<img width="1272" alt="certificate" src="https://user-images.githubusercontent.com/92983658/219393861-c1deddef-7452-44c4-b6b9-5f81a5bd9abf.png">

<br>

- List the directory to see the created files: `ls -ltr`

<br>

<img width="1149" alt="CA_files" src="https://user-images.githubusercontent.com/92983658/219394200-96f8a048-ac5a-485b-813c-c782dd026865.png">

note: The 3 important files here are:

- ca.pem: The Root Certificate
- ca-key.pem: The Private Key
- ca.csr: The Certificate Signing Request

<br>

### Step Two: Generating TLS Certificates For Client and Server

Client/Server certificates will need to be provisioned for all the components. It is a MUST to have encrypted communication within the cluster. Therefore, the server here are the master nodes running the api-server component. While the client is every other component that needs to communicate with the api-server.

Now we have a certificate for the Root CA, we can then begin to request more certificates which the different Kubernetes components, i.e. clients and server, will use to have encrypted communication.

the clients here refer to every other component that will communicate with the api-server. These are:

- kube-controller-manager
- kube-scheduler
- etcd
- kubelet
- kube-proxy
- Kubernetes Admin User

#### 1. Kubernetes API-Server Certificate and Private Key
The certificate for the Api-server must have `IP addresses`, `DNS names`, and a `Load Balancer address` included.

The `kubernetes-master` static IP address will be included in the list of subject alternative names for the Kubernetes API Server certificate. This will ensure the certificate can be validated by remote clients.

- Generate the Certificate Signing Request (CSR), Private Key and the Certificate for the Kubernetes Master Nodes.

```

{
cat > master-kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
   "hosts": [
   "127.0.0.1",
   "172.31.0.10",
   "172.31.0.11",
   "172.31.0.12",
   "ip-172-31-0-10",
   "ip-172-31-0-11",
   "ip-172-31-0-12",
   "ip-172-31-0-10.${AWS_REGION}.compute.internal",
   "ip-172-31-0-11.${AWS_REGION}.compute.internal",
   "ip-172-31-0-12.${AWS_REGION}.compute.internal",
   "${KUBERNETES_PUBLIC_ADDRESS}",
   "kubernetes",
   "kubernetes.default",
   "kubernetes.default.svc",
   "kubernetes.default.svc.cluster",
   "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "Nigeria",
      "L": "Abuja",
      "O": "Kubernetes",
      "OU": "Kubernetes Manual",
      "ST": "FCT"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  master-kubernetes-csr.json | cfssljson -bare master-kubernetes
}

```

<br>

- output:
```

master-kubernetes-csr.json
master-kubernetes-key.pem
master-kubernetes.csr
master-kubernetes.pem

```

<br>

<img width="1051" alt="kubernetes_api" src="https://user-images.githubusercontent.com/92983658/219398898-8472caca-d447-4608-9543-47ef359b35df.png">

<br>

#### 2. The Scheduler Client Certificate: Kube Scheduler
- Generate the `kube-scheduler` client certificate and private key:

```

{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "Nigeria",
      "L": "Abuja",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes Manual",
      "ST": "FCT"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}

```

*note: it is safe to ignore any warning messages for now*

<br>

results:
```

kube-scheduler-csr.json
kube-scheduler-key.pem
kube-scheduler.csr
kube-scheduler.pem

```

<br>

#### 3. The Kube Proxy Client Certificate
- Generate the `kube-proxy` client certificate and private key:

```

{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "Nigeria",
      "L": "Abuja",
      "O": "system:node-proxier",
      "OU": "Kubernetes Maual",
      "ST": "FCT"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}

```

<br>

output:
```
kube-proxy-csr.json
kube-proxy-key.pem
kube-proxy.csr
kube-proxy.pem

```

<br>


#### 4. The Controller Manager Client Certificate
the `kube-controller-manager` is responsible for generating and signing service account tokens which are used by pods or other resources to establish connectivity to the api-server.
Generate the `kube-controller-manager` client certificate and private key:

```

{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "Nigeria",
      "L": "Abuja",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes Manual",
      "ST": "FCT"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}

```

<br>

output:
```
kube-controller-csr.json
kube-controller-key.pem
kube-controller.csr
kube-controller.pem

```

<br>

#### 5. The Kubelet Client Certificates

Kubernetes requires that the hostname of each worker node is included in the client certificate.

Also, Kubernetes uses a special-purpose authorization mode called **Node Authorizer**, that specifically authorizes API requests made by kubelet services. In order to be authorized by the Node Authorizer, kubelets must use a credential that identifies them as being in the system:nodes group, with a username of `system:node:<nodeName>`. Notice the `"CN": "system:node:${instance_hostname}"`, in the below code.

Therefore, the certificate to be created must comply to these requirements. In the below example, there are 3 worker nodes, hence we will use bash to loop through a list of the worker nodes’ hostnames, and based on each index, the respective Certificate Signing Request (CSR), private key and client certificates will be generated.

```

for i in 0 1 2; do
  instance="worker-${i}"
  instance_hostname="ip-172-31-0-2${i}"
  cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance_hostname}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "Nigeria",
      "L": "Abuja",
      "O": "system:nodes",
      "OU": "Kubernetes Manual",
      "ST": "FCT"
    }
  ]
}
EOF

  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')

  internal_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PrivateIpAddress')

  cfssl gencert \
    -ca=ca.pem \
    -ca-key=ca-key.pem \
    -config=ca-config.json \
    -hostname=${instance_hostname},${external_ip},${internal_ip} \
    -profile=kubernetes \
    ${instance}-csr.json | cfssljson -bare ${instance}

done

```

<br>

output:
```
worker-0-key.pem
worker-0.pem
worker-1-key.pem
worker-1.pem
worker-2-key.pem
worker-2.pem

```

<br>

<img width="1456" alt="kubeket_1a" src="https://user-images.githubusercontent.com/92983658/219609369-be633998-c0b5-4571-ad94-5e2f2186646b.png">
<img width="1470" alt="kubelet_1b" src="https://user-images.githubusercontent.com/92983658/219609433-4b693336-fe6a-4878-b38f-5cb14bc6630f.png">

<br>

#### 6. The Admin Client Certificate
- Generate the `admin` client certificate and private key:


```

{
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "Nigeria",
      "L": "Abuja",
      "O": "system:masters",
      "OU": "Kubernetes Manual",
      "ST": "FCT"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
}

```
<br>

output:
```
admin-key.pem
admin.pem

```

<br>

<img width="1470" alt="kube_admin" src="https://user-images.githubusercontent.com/92983658/219610722-7ac053f7-b3af-4ec0-bf51-ff8674475004.png">

<br>

#### 6. The Service Account Key Pair
The Kubernetes Controller Manager uses a key pair to generate and sign service account tokens

Generate the `service-account` certificate and private key:

```

{

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "Nigeria",
      "L": "Abuja",
      "O": "Kubernetes",
      "OU": "Kubernetes Manual",
      "ST": "FCT"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account

}

```

<br>

output:
```
service-account-key.pem
service-account.pem

```

<br>

<img width="1289" alt="service" src="https://user-images.githubusercontent.com/92983658/219644683-facb4cd1-bace-432d-a7d4-b318f8f0c38f.png">

<br>

<img width="1024" alt="outputs" src="https://user-images.githubusercontent.com/92983658/219644800-0b398a6d-f8c3-41ec-be27-0f18ff478b37.png">

<br>

### Step Three: Distribute the Client and Server Certificates
- Copy the appropriate certificates and private keys to each worker instance:

```
  
for instance in worker-0 worker-1 worker-2; do
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/k8s-cluster.id_rsa \
    ca.pem ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done

```

<br>

<img width="1364" alt="worker_certificates" src="https://user-images.githubusercontent.com/92983658/219695833-8644c4d7-fff3-44cd-ab4b-79651110dd63.png">

<br>

- Copy the appropriate certificates and private keys to each master instance:
```

for instance in master-0 master-1 master-2; do
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/k8s-cluster.id_rsa \
    ca.pem ca-key.pem service-account-key.pem service-account.pem \
    master-kubernetes.pem master-kubernetes-key.pem ubuntu@${external_ip}:~/;
done

```

<br>

<img width="1368" alt="master_certificate" src="https://user-images.githubusercontent.com/92983658/219697389-ca3d9482-3552-456b-90a1-3053664e50b3.png">

*Note: The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet client certificates` will be used to generate client authentication configuration files later.*

<br>

## PART FIVE: Generating Kubernetes Configuration Files for Authentication
Configuring kubectl for Remote Access generating Kubernetes configuration files, also known as kubeconfigs, which enables Kubernetes clients to locate and authenticate to the Kubernetes API Servers. A client tool called `kubectl` is needed to do this

### Client Authentication Configs

#### Step One: Kubernetes Public IP Address
Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability, the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

- create a few environment variables for reuse by multiple commands.

```

KUBERNETES_API_SERVER_ADDRESS=$(aws elbv2 describe-load-balancers --load-balancer-arns ${LOAD_BALANCER_ARN} --output text --query 'LoadBalancers[].DNSName')

```

<br>

#### Step Two: The kubelet Kubernetes Configuration File
- Generate a `kubeconfig` file for each worker node:


```

for i in 0 1 2; do

instance="worker-${i}"
instance_hostname="ip-172-31-0-2${i}"

 # Set the kubernetes cluster in the kubeconfig file
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://$KUBERNETES_API_SERVER_ADDRESS:6443 \
    --kubeconfig=${instance}.kubeconfig

# Set the cluster credentials in the kubeconfig file
  kubectl config set-credentials system:node:${instance_hostname} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

# Set the context in the kubeconfig file
  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:node:${instance_hostname} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done


```
<br>

<img width="1188" alt="kubectl_config" src="https://user-images.githubusercontent.com/92983658/220082908-ff277f97-9537-4486-9705-280490f7d001.png">

<br>

- List the output: `ls *.kubeconfig`
- results:

```
worker-0.kubeconfig
worker-1.kubeconfig
worker-2.kubeconfig

```

<br>

<img width="937" alt="kubeconfig_kubelet" src="https://user-images.githubusercontent.com/92983658/220088258-2ff1423c-627b-466b-a79b-5616fd932db5.png">


<br>

#### Step Three: The kube-proxy Kubernetes Configuration File
- Generate a kubeconfig file for the `kube-proxy` service:

```

{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}

```

<br>

<img width="821" alt="kubeproxy_config" src="https://user-images.githubusercontent.com/92983658/220089166-6463d17f-d9a1-4252-8275-29c3e060bd02.png">

<br>

- results: `ls *.kubeconfig`
```
kube-proxy.kubeconfig

```

<br>

#### Step Four: The kube-controller-manager Kubernetes Configuration File
- Generate a kubeconfig file for the `kube-controller-manager` service:

```
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}

```

*note: Notice that the --server is set to use 127.0.0.1. This is because, this component runs on the API-Server so there is no point routing through the Load Balancer.*

<br>

<img width="852" alt="kubecontroller_config" src="https://user-images.githubusercontent.com/92983658/220090560-482aaa9c-fe5a-4eb5-bca7-9680b75d8279.png">

<br>

- result
```
kube-controller-manager.kubeconfig

```

<br>

#### Step Five: The kube-scheduler Kubernetes Configuration File
- Generate a kubeconfig file for the `kube-scheduler` service:

```
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}

```

<br>

<img width="757" alt="kube_scheduler_config" src="https://user-images.githubusercontent.com/92983658/220091123-6a03dcbe-b3e4-4628-823e-178bc3262198.png">

<br>

- results
```
kube-scheduler.kubeconfig

```

<br>

#### Step Six: The admin Kubernetes Configuration File
- Generate a kubeconfig file for the `admin` user:

```

{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}

```

<br>

- results
```
admin.kubeconfig

```

<br>

<img width="797" alt="kubeconfig_admin" src="https://user-images.githubusercontent.com/92983658/220091893-292e3860-7968-4cb5-8357-51293b884124.png">

<br>

<img width="938" alt="list_kubeconfig" src="https://user-images.githubusercontent.com/92983658/220092196-57ef3e39-8b7d-4fea-aded-924eacd449e8.png">

<br>

### Distribute the Kubernetes Configuration Files
- Copy the appropriate `kubelet` and `kube-proxy` kubeconfig files to each worker instance:

```

for instance in worker-0 worker-1 worker-2; do
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/k8s-cluster.id_rsa \
    ${instance}.kubeconfig kube-proxy.kubeconfig ubuntu@${external_ip}:~/; \
done

```

<br>

<img width="1363" alt="worker_node_kube_config_dist" src="https://user-images.githubusercontent.com/92983658/220094806-d0a5d918-df52-4c31-9105-da584a48ab3b.png">

<br>

- Copy the appropriate `kube-controller-manager` and `kube-scheduler` kubeconfig files to each master instance:

```

for instance in master-0 master-1 master-2; do
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/k8s-cluster.id_rsa \
    admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ubuntu@${external_ip}:~/;
done

```

<br>

<img width="1405" alt="master_kube_config_dist" src="https://user-images.githubusercontent.com/92983658/220096500-5b0d4be9-aed3-4559-9360-ccf06e41d0dd.png">

<br>

## PART SIX: Generating the Data Encryption Config and Key

Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to encrypt cluster data at rest.

### The Encryption Key
- Generate an encryption key: `ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)`

<br>

### The Encryption Config File

- Create the `encryption-config.yaml` encryption config file:


```

cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF

```

<br>


<img width="991" alt="encryption_yaml_1a" src="https://user-images.githubusercontent.com/92983658/220100719-50963def-2b3a-4d1e-aaf4-c42a57a68b94.png">

<br>

<img width="865" alt="encryption_yaml_1b" src="https://user-images.githubusercontent.com/92983658/220701069-f2182534-a40b-4c40-a1ef-51b0b8add797.png">

<br>

#### Copy the encryption-config.yaml encryption config file to each master instance:

```

for instance in master-0 master-1 master-2; do
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/k8s-cluster.id_rsa \
    encryption-config.yaml ubuntu@${external_ip}:~/;
done

```

<br>

<img width="1371" alt="distribute_encryption_master" src="https://user-images.githubusercontent.com/92983658/220101664-67cafd26-385f-4867-a524-262ef85f26b8.png">

<br>

## PART SEVEN: Bootstrapping the etcd Cluster
The primary purpose of the `etcd` component is to store the state of the cluster. This is because Kubernetes itself is stateless. Therefore, all its stateful data will persist in `etcd`.

Since Kubernetes is a distributed system – it needs a distributed storage to keep persistent data in it. `etcd` is a highly-available key value store that fits the purpose.

The following commands must be run on each `master` instance: `master-0`, `master-1`, and `master-2`

- `tmux` can be used to run commands on multiple compute instances at the same time

#### ssh separately into each master instance
```
#master-1
master_1_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=master-0" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ../ssh/k8s-cluster.id_rsa ubuntu@${master_1_ip}

#master 2
master_2_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=master-1" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ../ssh/k8s-cluster.id_rsa ubuntu@${master_2_ip}

#master 3
master_3_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=master-2" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ../ssh/k8s-cluster.id_rsa ubuntu@${master_3_ip}

```

<br>

<img width="870" alt="ssh_master" src="https://user-images.githubusercontent.com/92983658/220931051-0482ce83-d174-4870-a3d3-535d64bc471c.png">

<br>

### Bootstrapping an etcd Cluster Member

#### Download and Install the etcd Binaries
- Download the official etcd release binaries from the etcd GitHub project:
```

wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.15/etcd-v3.4.15-linux-amd64.tar.gz"


```

<br>

- Extract and install the `etcd server` and the `etcdctl` command line utility:
```

{
  tar -xvf etcd-v3.4.15-linux-amd64.tar.gz
  sudo mv etcd-v3.4.15-linux-amd64/etcd* /usr/local/bin/
}

```

<br>

#### Configure the etcd Server

```

{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem master-kubernetes-key.pem master-kubernetes.pem /etc/etcd/
}

```

<br>

- The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance:

```

export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)

echo "${INTERNAL_IP}"

```

<br>

<img width="1456" alt="internal_ip" src="https://user-images.githubusercontent.com/92983658/220936976-9981242b-087d-401d-a000-7a8d061a151f.png">


<br>

- Each `etcd` member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
ETCD_NAME=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)

echo "${ETCD_NAME}"

```

<br>

<img width="1448" alt="etcd_name" src="https://user-images.githubusercontent.com/92983658/220937358-4eaed645-6095-470f-b804-ddee33410105.png">

<br>

- Create the `etcd.service` systemd unit file:

```

cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/master-kubernetes.pem \\
  --key-file=/etc/etcd/master-kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/master-kubernetes.pem \\
  --peer-key-file=/etc/etcd/master-kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster master-0=https://172.31.0.10:2380,master-1=https://172.31.0.11:2380,master-2=https://172.31.0.12:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```

<br>

### Start the etcd Server

```
{
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
}

```
*note: Remember to run the above commands on each controller node: master-0, master-1, and master-2.*

<br>

### Verification
- List the etcd cluster members:

```

sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/master-kubernetes.pem \
  --key=/etc/etcd/master-kubernetes-key.pem
  
```

<br>

<img width="1452" alt="verfication_1a" src="https://user-images.githubusercontent.com/92983658/220938185-2d59bf4f-dbc9-4a31-b0bb-c1fa3419b462.png">

<br>

<img width="1450" alt="etcd_status_1b" src="https://user-images.githubusercontent.com/92983658/220938542-b657e866-514f-4915-bd2d-9b40a4dfca94.png">

<br>

## PART EIGHT: Bootstrapping the Kubernetes Control Plane

### Provision the Kubernetes Control Plane

- Create the Kubernetes configuration directory: `sudo mkdir -p /etc/kubernetes/config`


### Download and Install the Kubernetes Controller Binaries

- Download the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-apiserver" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-controller-manager" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-scheduler" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl"

```

<br>

- Install the Kubernetes binaries:

```

{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}

```

<br>

<img width="1380" alt="install_binaries" src="https://user-images.githubusercontent.com/92983658/220120738-aee5d196-b39a-4e73-831b-6e717c1067ba.png">

<br>

### Configure the Kubernetes API Server

```

{
sudo mkdir -p /var/lib/kubernetes/

sudo mv ca.pem ca-key.pem master-kubernetes-key.pem master-kubernetes.pem \
service-account-key.pem service-account.pem \
encryption-config.yaml /var/lib/kubernetes/
}

```

<br>

<img width="815" alt="configure_api" src="https://user-images.githubusercontent.com/92983658/220124176-049e9e17-c208-4155-b9fa-885051c97b57.png">

<br>

- The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```

export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)

echo $INTERNAL_IP

```

<br>

<img width="1456" alt="internal_ip_3" src="https://user-images.githubusercontent.com/92983658/220975845-ee0b558e-1c00-4d55-992d-5c6c45b3570e.png">

<br>

- Create the `kube-apiserver.service` systemd unit file:

```

cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/master-kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --etcd-servers=https://172.31.0.10:2379,https://172.31.0.11:2379,https://172.31.0.12:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/master-kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --runtime-config='api/all=true' \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-account-signing-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-account-issuer=https://${INTERNAL_IP}:6443 \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/master-kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```

<br>

### Configure the Kubernetes Controller Manager

- Move the `kube-controller-manager` kubeconfig into place: `sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/`
- Export some variables to retrieve the `vpc_cidr` – This will be required for the bind-address flag:

```

export AWS_METADATA="http://169.254.169.254/latest/meta-data"
export EC2_MAC_ADDRESS=$(curl -s $AWS_METADATA/network/interfaces/macs/ | head -n1 | tr -d '/')
export VPC_CIDR=$(curl -s $AWS_METADATA/network/interfaces/macs/$EC2_MAC_ADDRESS/vpc-ipv4-cidr-block/)
export NAME=k8s-cluster

```

<br>

<img width="1234" alt="controller_variables_export" src="https://user-images.githubusercontent.com/92983658/220125787-4e3caeae-a4d8-4f62-96a2-281add896c83.png">

<br>


- Create the `kube-controller-manager.service` systemd unit file:

```

cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --cluster-cidr=${VPC_CIDR} \\
  --cluster-name=${NAME} \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authentication-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authorization-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```

<br>

### Configure the Kubernetes Scheduler
- Move the `kube-scheduler` kubeconfig into place: `sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/`
- Create the `kube-scheduler.yaml` configuration file:

```

cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF

```

<br>


<img width="956" alt="kube_scheduler_yaml" src="https://user-images.githubusercontent.com/92983658/220126751-1f88ef4a-fbc8-4b47-a1b5-3d8e68698d7c.png">

<br>

- Create the `kube-scheduler.service` systemd unit file:

```

cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```

<br>

### Start the Controller Services

```

{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}

```

*note: Allow up to 10 seconds for the Kubernetes API Server to fully initialize.*

<br>

<img width="1383" alt="controller_services_Start" src="https://user-images.githubusercontent.com/92983658/220127685-76ad5f33-6912-4cac-813e-f4749c43364e.png">

<br>

- check status of controller services:

```

{
sudo systemctl status kube-apiserver
sudo systemctl status kube-controller-manager
sudo systemctl status kube-scheduler
}

```

*note: it can take a while for te services to fully load*

<br>

<img width="1452" alt="api_service_status" src="https://user-images.githubusercontent.com/92983658/220977868-4cb3b204-c59f-4f1e-a828-3413fd1317c9.png">

<br>

### Enable HTTP Health Checks

- get the cluster details run: `kubectl cluster-info --kubeconfig admin.kubeconfig`

<br>

<img width="1451" alt="kubeclusterinfo" src="https://user-images.githubusercontent.com/92983658/220979186-bdcdb62d-e58f-473f-8911-70daebe35ba4.png">

<br>

- get the current namespaces: `kubectl get namespaces --kubeconfig admin.kubeconfig`

<br>

<img width="1453" alt="namespaces" src="https://user-images.githubusercontent.com/92983658/220980022-a7660a68-1827-44c5-b2dd-c332cbd917dc.png">

<br>

- reach the Kubernetes API Server publicly:
```

curl --cacert /var/lib/kubernetes/ca.pem https://$INTERNAL_IP:6443/version

```

<br>

<img width="1457" alt="api_public" src="https://user-images.githubusercontent.com/92983658/220980462-cdec63a3-29dc-4f8c-9663-73c4217a053f.png">

<br>

- get the status of each component:
```
kubectl get componentstatuses --kubeconfig admin.kubeconfig

```

<br>

<img width="1460" alt="component_status" src="https://user-images.githubusercontent.com/92983658/220981805-1da48376-54f3-4356-8e6e-038aeb99c95b.png">

<br>

### RBAC for Kubelet Authorization

- configure RBAC permissions so that the api-server has necessary authorization for the kubelet. Access to the Kubelet API is required for retrieving metrics, logs, and executing commands in pods.


**The commands in this section will affect the entire cluster and only need to be run once from one of the master nodes.**

- Create the ClusterRole with permissions to access the Kubelet API and perform most common tasks associated with managing pods:

```

cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF

```

<br>

<img width="727" alt="cluster_role" src="https://user-images.githubusercontent.com/92983658/220983206-10181a50-bb3d-49cd-8c37-4e90d8e74508.png">

<br>

The Kubernetes API Server authenticates to the Kubelet as the kubernetes user using the client certificate as defined by the `--kubelet-client-certificate` flag.

- Create the ClusterRoleBinding to bind the kubernetes user with the role created above:
```

cat <<EOF | kubectl --kubeconfig admin.kubeconfig  apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF

```

<br>

<img width="728" alt="cluster_bind" src="https://user-images.githubusercontent.com/92983658/220983936-e9337c8d-c8fc-413d-bfb9-0c824db3f94c.png">

<br>

## PART NINE: Bootstrapping the Kubernetes Worker Nodes

The following components will be installed on each node:

- kubelet
- kube-proxy
- Containerd or Docker
- Networking plugins

<br>

#### ssh separately into each worker instance

- The commands below must be run on each worker instance: worker-0, worker-1, and worker-2. Login to each worker instance using the following AWS command.

```
#worker-1
worker_1_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=worker-0" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ../ssh/k8s-cluster.id_rsa ubuntu@${worker_1_ip}

#worker 2
worker_2_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=worker-1" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ../ssh/k8s-cluster.id_rsa ubuntu@${worker_2_ip}

#worker 3
worker_3_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=worker-2" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ../ssh/k8s-cluster.id_rsa ubuntu@${worker_3_ip}

```

<br>

<img width="1450" alt="worker_nodes" src="https://user-images.githubusercontent.com/92983658/221116452-6c827c5f-6db7-4d42-82ca-cefc127b43c8.png">

<br>

### Provisioning a Kubernetes Worker Node

- Install the OS dependencies:

```

{
  sudo apt-get update
  sudo apt-get -y install socat conntrack ipset
}

```
*The socat binary enables support for the `kubectl port-forward` command.*

<br>

### Disable Swap
By default the kubelet will fail to start if swap is enabled. It is recommended that swap be disabled to ensure Kubernetes can provide proper resource allocation and quality of service.

Verify if swap is enabled:
```
sudo swapon --show

```

If output is empty then swap is not enabled. If swap is enabled run the following command to disable swap immediately:

```
sudo swapoff -a

```

<br>

### Download and Install Worker Binaries
```

wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.21.0/crictl-v1.21.0-linux-amd64.tar.gz \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc93/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz \
  https://github.com/containerd/containerd/releases/download/v1.4.4/containerd-1.4.4-linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubelet
  
```

<br>

- Create the installation directories:
```

sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
  
```

<br>

<img width="1451" alt="installation_directories" src="https://user-images.githubusercontent.com/92983658/221118865-7409f428-9182-4816-bbac-9fe8dc53cac5.png">

<br>

- Install the worker binaries:

```

{
  mkdir containerd
  tar -xvf crictl-v1.21.0-linux-amd64.tar.gz
  tar -xvf containerd-1.4.4-linux-amd64.tar.gz -C containerd
  sudo tar -xvf cni-plugins-linux-amd64-v0.9.1.tgz -C /opt/cni/bin/
  sudo mv runc.amd64 runc
  chmod +x crictl kubectl kube-proxy kubelet runc 
  sudo mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  sudo mv containerd/bin/* /bin/
}

```

<br>

<img width="1444" alt="worker_binaries" src="https://user-images.githubusercontent.com/92983658/221119920-f8f00bbb-6a50-4205-b475-1c25cceedd44.png">

<br>

### Configure CNI Networking

- Retrieve the Pod CIDR range for the current compute instance:

```

POD_CIDR=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^pod-cidr" | cut -d"=" -f2)

echo "${POD_CIDR}"
  
```

<br>

<img width="1448" alt="pod_cidr" src="https://user-images.githubusercontent.com/92983658/221127917-45952a0c-4789-49af-8df1-0b17366f2e39.png">

<br>

- Create the bridge network configuration file:

```

cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.4.0",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF

```

<br>

<img width="1455" alt="bridge_config" src="https://user-images.githubusercontent.com/92983658/221128380-9811808e-6a74-46ba-8d87-91232d1d8a2e.png">

<br>

- Create the loopback network configuration file:
```

cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.4.0",
    "name": "lo",
    "type": "loopback"
}
EOF

```

<br>

### Configure containerd

- Create the containerd configuration file:
```
sudo mkdir -p /etc/containerd/

```

<br>

```

cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
EOF

```

<br>

<img width="731" alt="containernerd_config" src="https://user-images.githubusercontent.com/92983658/221129118-ac5a5be2-8574-4608-9e94-7b26ef41ea2d.png">

<br>

- Create the `containerd.service` systemd unit file:
```

cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF

```

<br>

### Configure the Kubelet

- set HOSTNAME in the local shell :

```
HOSTNAME=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)

echo "${HOSTNAME}"

```

<br>

<img width="1460" alt="set_hostname" src="https://user-images.githubusercontent.com/92983658/221130588-b057f64d-c279-4a22-82ff-814c148b25b1.png">

<br>

```

{
  sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
  sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
  sudo mv ca.pem /var/lib/kubernetes/
}

```

<br>

- Create the `kubelet-config.yaml` configuration file:

```

cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF

```

*The `resolvConf` configuration is used to avoid loops when using CoreDNS for service discovery on systems running systemd-resolved.*

<br>

- Create the `kubelet.service` systemd unit file:

```

cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```

<br>

### Configure the Kubernetes Proxy
```
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig

```

<br>

- Create the `kube-proxy-config.yaml` configuration file:
```

cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "172.31.0.0/16"
EOF

```

<br>

<img width="1459" alt="kube_proxy_config" src="https://user-images.githubusercontent.com/92983658/221134337-91db81fe-2baa-4331-ae56-1b37a3e241e1.png">

<br>

- Create the kube-proxy.service systemd unit file:
```

cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```

<br>

<img width="1455" alt="kube_proxy_service" src="https://user-images.githubusercontent.com/92983658/221138093-aa9ee962-cc46-4516-8fc3-5c9901792838.png">

<br>

### Start the Worker Services
```

{
  sudo systemctl daemon-reload
  sudo systemctl enable containerd kubelet kube-proxy
  sudo systemctl start containerd kubelet kube-proxy
}

```

<br>

<img width="1450" alt="kube_ststus" src="https://user-images.githubusercontent.com/92983658/221141430-7071253e-1545-4878-8b31-03e1f29395a4.png">

<br>

### Verification
- The compute instances created in this tutorial will not have permission to complete this section. Run the following commands from the same machine used to create the compute instances.

```
master_1_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=master-0" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ../ssh/k8s-cluster.id_rsa ubuntu@${master_1_ip} \
kubectl get nodes --kubeconfig admin.kubeconfig -o wide

```

<br>

<img width="1299" alt="worker_verification" src="https://user-images.githubusercontent.com/92983658/221145327-d22378e6-9818-4efb-abc5-f7279961af04.png">

<br>

## Cleaning Up

### Compute Instances
```

INSTANCE_IDS=($(aws ec2 describe-instances \
      --filter "Name=tag:Name,Values=master-0,master-1,master-2,worker-0,worker-1,worker-2" "Name=instance-state-name,Values=running" \
      --output text --query 'Reservations[].Instances[].InstanceId'))

aws ec2 terminate-instances \
  --instance-ids ${INSTANCE_IDS[@]} \
  --query 'TerminatingInstances[].InstanceId' \
  --output table

aws ec2 delete-key-pair \
  --key-name k8s-cluster

aws ec2 wait instance-terminated \
  --instance-ids ${INSTANCE_IDS[@]}
  
```

### Networking
- Delete Load Balancer
```

LOAD_BALANCER_ARN=$(aws elbv2 describe-load-balancers \
  --name kubernetes-the-hard-way \
  --output text --query 'LoadBalancers[0].LoadBalancerArn')

LISTENER_ARN=$(aws elbv2 describe-listeners \
  --load-balancer-arn "${LOAD_BALANCER_ARN}" \
  --output text --query 'Listeners[0].ListenerArn')

aws elbv2 delete-listener \
  --listener-arn "${LISTENER_ARN}"

aws elbv2 delete-load-balancer \
  --load-balancer-arn "${LOAD_BALANCER_ARN}"

TARGET_GROUP_ARN=$(aws elbv2 describe-target-groups \
  --name k8s-cluster \
  --output text --query 'TargetGroups[0].TargetGroupArn')

aws elbv2 delete-target-group \
  --target-group-arn "${TARGET_GROUP_ARN}"
  
```

<br>

- Delete Security Group
```

SECURITY_GROUP_ID=$(aws ec2 describe-security-groups \
  --filters "Name=tag:Name,Values=k8s-cluster" \
  --output text --query 'SecurityGroups[0].GroupId')

aws ec2 delete-security-group \
  --group-id "${SECURITY_GROUP_ID}"
  
```

<br>

- Delete Route Table
```

ROUTE_TABLE_ASSOCIATION_ID="$(aws ec2 describe-route-tables \
  --route-table-ids "${ROUTE_TABLE_ID}" \
  --output text --query 'RouteTables[].Associations[].RouteTableAssociationId')"

aws ec2 disassociate-route-table \
  --association-id "${ROUTE_TABLE_ASSOCIATION_ID}"

ROUTE_TABLE_ID=$(aws ec2 describe-route-tables \
  --filters "Name=tag:Name,Values=k8s-cluster" \
  --output text --query 'RouteTables[0].RouteTableId')

aws ec2 delete-route-table \
  --route-table-id "${ROUTE_TABLE_ID}"
  
```

<br>

- Delete Internet Gateway
```

INTERNET_GATEWAY_ID=$(aws ec2 describe-internet-gateways \
  --filters "Name=tag:Name,Values=k8s-cluster" \
  --output text --query 'InternetGateways[0].InternetGatewayId')

aws ec2 detach-internet-gateway \
  --internet-gateway-id "${INTERNET_GATEWAY_ID}" \
  --vpc-id "${VPC_ID}"

aws ec2 delete-internet-gateway \
  --internet-gateway-id "${INTERNET_GATEWAY_ID}"
  
```

<br>

- Delete Subnet and VPC
```

SUBNET_ID=$(aws ec2 describe-subnets \
  --filters Name=tag:Name,Values=k8s-cluster \
  --output text --query 'Subnets[0].SubnetId')

aws ec2 delete-subnet \
  --subnet-id "${SUBNET_ID}"

VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=tag:Name,Values=k8s-cluster \
  --output text --query 'Vpcs[0].VpcId')

aws ec2 delete-vpc \
  --vpc-id "${VPC_ID}"
  
```
<br>

- Ensure there are no more resources left
```

aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Name,Values=k8s-cluster \
  --query 'sort_by(ResourceTagMappingList, &ResourceARN)[].ResourceARN' \
  --output table
  
```

<br>





