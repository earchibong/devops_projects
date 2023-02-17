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
--tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=k8s-cluster}]' \
--output text --query 'Vpc.VpcId'
)

```
<br>

<img width="857" alt="k8_vpc" src="https://user-images.githubusercontent.com/92983658/219364710-5015fdd8-3b64-455f-9b6e-d5b94d114fb0.png">

<br>

- Tag the VPC so that it is named:

```

NAME=k8s-cluster

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

<br>

