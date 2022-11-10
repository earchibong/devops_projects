<br>

# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

### Introduction
This project demostrates how a secure infrastructure inside AWS VPC (Virtual Private Cloud) network is built for a particular company, 
who uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. 
As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology 
from NGINX to achieve this. The infrastructure will look like following diagram:

<br>

![image](https://user-images.githubusercontent.com/92983658/199484750-f45c1807-4153-4e78-ac17-3d033579e735.png)

<br>

The following outlines the steps taken:

<br>

## Step One: Configure AWS account and organisational units

- Create an AWS Master account. (Also known as Root Account) 
- Within the Root account, create a sub-account and name it `DevOps`
  - Search for `AWS organisations` -> `create organisation` -> `AWS accounts` -> `add an AWS account`
  - Leave the IAM role name as the default *OrganizationAccountAccessRole *(this is important).

<br>

![DevOps](https://user-images.githubusercontent.com/92983658/199729743-9183cbee-04ac-42c8-99e6-6929b4edc5d3.png)

<br>

- Within the Root account, create an AWS Organization Unit (OU). Name it `Dev`
  - `AWS organisations` -> `AWS accounts` -> `root` -> `children` -> click `actions` -> `organisational unit` -> `create new`

<br>

![dev_ou](https://user-images.githubusercontent.com/92983658/199481770-ed66d92a-b915-4288-97de-bd3a8b86f091.png)

<br>

- Move the DevOps account into the Dev OU.
  - `aws organisations` -> `aws accounts` -> select `Devops` -> under `Actions` select `move`
  - set destination account to `dev`

<br>

![move_devops](https://user-images.githubusercontent.com/92983658/199483551-51f3ff0d-646d-44d0-ab98-dc433bb441ba.png)

<br>

- Login to the newly created AWS account using the new email address.
  - From the upper-right corner of the AWS Organizations console, choose the link that contains your current sign-in name and 
  then choose `Switch Role.`
  - Enter the administrator-provided `account ID` number and role name: `OrganizationAccountAccessRole`
  - For Display Name, enter the text that you want to show on the navigation bar in the upper-right corner in place of your 
  user name while you are using the role. You can optionally choose a color
  - Choose Switch Role. Now all actions that you perform are done with the permissions granted to the role that you switched to. 
  You no longer have the permissions associated with your original IAM user until you switch back
  - When you finish performing actions that require the permissions of the role, you can switch back to your normal IAM user. Choose the role name in the upper-right corner (whatever you specified as the Display Name) and then choose `Back to UserName`

<br>

![switch_role](https://user-images.githubusercontent.com/92983658/199731982-4a3630f3-c445-4880-b9d1-c8efda0e9909.png)

<br>

- Create a free domain name for your fictitious company at <a href="https://www.freenom.com/">Freenom domain registrar</a>

- Create a hosted zone in AWS, and map it to your free domain from Freenom.
  - `route 53 dashboard` -> `create hosted zone` 

<br>

![hosted_zone](https://user-images.githubusercontent.com/92983658/199962765-0d0f5dd5-b203-4fbe-9e77-a63e91335a2c.png)

<br>

![hosted_zone_2](https://user-images.githubusercontent.com/92983658/199963864-a9e9e5b4-74fc-4e2b-a1ba-86fcc75114d9.png)

<br>


Read how to do that <a href="https://github.com/earchibong/devops_training/blob/main/nginx_load_balancer.md">here</a> and <a href="https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-working-with.html">here</a> also how to <a href="https://medium.com/@kcabading/getting-a-free-domain-for-your-ec2-instance-3ac2955b0a2f"> map route 53 nameservers to freenom</a>

*NOTE : As you proceed with configuration, ensure that all resources are appropriately tagged, for example:*
*Project: <Give your project a name>*
*Environment: <dev>*
*Automated: <No> (If you create a recource using an automation tool, it would be <Yes>)*

<br>
  
## Step Two: SET UP A VIRTUAL PRIVATE NETWORK (VPC)
Always make reference to the architectural diagram and ensure that your configuration is aligned with it.

### Create a VPC
- search for VPC in AWS search bar
- Select `create VPC` in VPC console:
  - resources to create: `VPC only`
  - name_tag: `project name`
  - IPv4 CIDR block: select `IPv4 CIDR manual input`
  - IPv4 CIDR: `10.0.0.0/16`
  - IPv6 CIDR block: select `No IPv6 CIDR block`
  - tanancy: `default`

<br>

![vpc](https://user-images.githubusercontent.com/92983658/199966649-ead54dca-1c57-4c36-b661-a6eea2bd2ea6.png)

<br>
  
- Edit `DNS hostnames`: click `Your VPCs`, select a VPC, and choose `Actions` -> `Edit VPC settings` -> click `Enable DNS hostnames`

<br>

![edit_DNS_hotstname](https://user-images.githubusercontent.com/92983658/199969284-a96462d1-c0f5-4bfb-ae96-c2b1792a3475.png)

<br>
 
### Create An Internet Gateway
- select `Internet Gateway` in VPC console:
  - Select `Create Internet Gateway`
  - name: `project_15_igw`

<br>

![create_internet_gateway](https://user-images.githubusercontent.com/92983658/199969643-01611e21-5488-4944-9b63-45cbf0ace008.png)

<br>

- Attach internet gateway to VPC
  - In internet gateway console:
   - select your newly created internet gateway
   - under `actions` select `attach to VPC`
   - select `project_15_vpc`

<br>

![attach_to_vpc](https://user-images.githubusercontent.com/92983658/199970264-553ff3d6-fabe-4561-bd63-16c176991a20.png)

<br>
  
![attach_internet_gateway](https://user-images.githubusercontent.com/92983658/199970461-34af05e1-16da-43e3-881f-d7080154dd4f.png)

<br>
  

### Create 6 subnets as shown in the architecture

- subnets to create:
  - Availability zone a:
    - public subnet 1
    - private subnet 1
    - private subnet 3
  - Availability zone b:
    - public subnet 2
    - private subnet 2
    - private subnet 4
 
- select `subnets` from VPC console
  - select `create subnet`
  - VPC ID: select `project 15` VPC
  - Subnet Settings:
   - subnet name:`Public_subnet_1`
   - Availability zone: pick what is avaialbe according to diagram design and ensure the rest of the subnets match design
   - IPv4 CIDR block: `10.0.1.0/24`

<br>

![subnet_1](https://user-images.githubusercontent.com/92983658/199973267-a31541df-68cb-407f-99cd-a046a4bc4dfa.png)
![subnet_b](https://user-images.githubusercontent.com/92983658/199973274-dfcc4798-9dbc-4064-ac8b-54cb71970a07.png)

<br>

![2_](https://user-images.githubusercontent.com/92983658/199977818-4bcec5ba-d6b9-4d3e-84fb-5a9567f10000.png)

 <br>

![3_of_6](https://user-images.githubusercontent.com/92983658/199977836-0630fbde-903b-4e0d-8fb5-d601ae1a688b.png)

<br>

![4_of_6](https://user-images.githubusercontent.com/92983658/199977887-71624025-158d-4121-a006-362d271248ed.png)

<br>
  
![5_of_6](https://user-images.githubusercontent.com/92983658/199977930-be02b44e-4c20-4d11-8595-09dbe6f3984a.png)

<br>
  
![6_of_6](https://user-images.githubusercontent.com/92983658/199977966-adf87c9b-df70-4024-84aa-7757c7be9cd6.png)

<br>

![subnet_dash](https://user-images.githubusercontent.com/92983658/199979034-d803b0cc-de6e-4ccd-a071-e9073996d367.png)

<br>
  
### Create a route table and associate it with public subnets

- Select `route table` from VPC console
  - select `create route table`
  - name: give it name attached to your project
  - VPC ID: select project 15 VPC
  
<br>
  
![public_route_Table](https://user-images.githubusercontent.com/92983658/199980534-4deefd13-c5ed-4acd-a17d-4921057cc244.png)

<br>
 
- on `actions` button select `edit subnet associations`
  - select public subnets

<br>
  
![subnet_associations_1](https://user-images.githubusercontent.com/92983658/199981551-c8cbf327-9229-40b6-a54a-b4f770feb0d7.png)

<br>
  
![public_subnet_associationsb](https://user-images.githubusercontent.com/92983658/199981586-a1fbd310-23bb-4f15-9318-6635c3b096f9.png)

 <br>
  
 - Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)
  - `route table dashboard` -> select `public route table` -> click `actions` -> select `edit routes`
 
<br>
  
![edit_routes](https://user-images.githubusercontent.com/92983658/199983223-1c7eeda3-9715-4ea3-a753-73e30d9b0790.png)

<br>
  
- select `add route`
  - destination: `0.0.0.0/0`
  - target: `internet gateway`

<br>
  
![edite_route_2](https://user-images.githubusercontent.com/92983658/199983799-ffc4b1fc-84cb-4f74-ae4c-32a8c1e5a4e5.png)

<br>

 ### Create a route table and associate it with private subnets
 
- Select `route table` from VPC console
  - select `create route table`
  - name: give it name attached to your project
  - VPC ID: select project 15 VPC
  - once route table is created, associate it with private subnets

<br>
  
![private_subnet_associations](https://user-images.githubusercontent.com/92983658/199982065-f7020f78-c949-4a23-9b27-42ca54312d54.png)

<br>
  
### Create 3 Elastic IPs
- on VPC console, select `elastic ips` and then select `allocate elastic ip address`

<br>
  
![elastic_ip](https://user-images.githubusercontent.com/92983658/199986916-3f457216-6427-4dcf-8adf-bc77ca63ce25.png)

<br>
  
![elastic_ips](https://user-images.githubusercontent.com/92983658/199987797-5f1c606a-3671-452c-8eac-2867458c4ab6.png)

 <br>
  
### Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)
- select `Nat Gateways` from the VPC console and select `create NAT gateway`
  - name: give it a nemae attached to project
  - subnets: select `public subnet 1`
  - connectivity type: `public`
  - elastic ip ID: `<name of elastic ip for ALB>`

<br>
  
![nat_gateway](https://user-images.githubusercontent.com/92983658/199990180-9f80c5b3-2a7c-49cf-8d4a-aadc4408c3dc.png)

<br>
  
- Edit a route in private route table, and associate it with the Nat Gateway
  - select `private route table` and edit routes for `private subnets`

<br>
  
 ![edit_routes_private](https://user-images.githubusercontent.com/92983658/199992289-2b7d0055-1ab3-4685-80d4-7df299aaa823.png)

<br>
  
- select `add route`
  - destination: `0.0.0.0`
  - target: NAT gateway

<br>
  
 ![target_natgateway](https://user-images.githubusercontent.com/92983658/199993729-2284318f-9840-4dd1-89d1-91fb93de4ea6.png)

<br>
  
 ### Create a Security Group for:

**1. Application Load Balancer:** ALB will be available from the Internet
  
  <br>
  
  - Select`Security groups` from VPC console and click `create security group`
  - name: `external ALB`
  - VPC: select the VPC created for your project
  - Add Inbound Rule:
    - traffic type: `HTTPS`
    - Destination: `anywhere ipv4`
    - Description: `access from anywhere`
  - Add A 2nd Inbound Rule:
    - traffic: `HTTP`
    - Destination: `anywhere ipv4`
    - Description: `access from anywhere`
  - Add Outbound Rule:
    - traffic: `All traffic`
    - Destination: `anywhere ipv4`
  - Tags:
    - key: `Name`

<br>

![ALB_security_group_1](https://user-images.githubusercontent.com/92983658/200307560-830e87d0-2ca2-40d8-ba2f-41195802e2b4.png)
![ALB_security_group_b](https://user-images.githubusercontent.com/92983658/200307577-f84414fb-d80b-4c6c-a7b5-3aae88985777.png)

<br>
  
**2. Bastion Servers:** Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com

<br>
 
- Select`Security groups` from VPC console and click `create security group`
  - name: `Bastion`
  - VPC: select the VPC created for your project
  - Add Inbound Rule:
    - traffic type: `SSH`
    - Destination: `my ip`
    - Description: `access from Bastion`
  - Add Outbound Rule:
    - traffic: `All traffic`
    - Destination: `anywhere ipv4`
  - Tags:
    - key: `Name`

<br>
  
  ![bastion_a](https://user-images.githubusercontent.com/92983658/200309550-abf3863b-f612-4cb5-8e94-8038437d64ba.png)

  ![bastion_b](https://user-images.githubusercontent.com/92983658/200309549-2a835da5-dcdb-4e3b-9239-4dec04976524.png)

 <br>
  
 **3. Nginx Servers:** Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder
 
 <br>
 
 - Select`Security groups` from VPC console and click `create security group`
  - name: `Nginx-reverse-proxy`
  - VPC: select the VPC created for your project
  - Add Inbound Rule:
    - traffic type: `HTTP`
    - Destination: `custom` and `<ALB security group>`
    - Description: `http traffic from ALB`
  - Add a 2nd Inbound Rule:
    - traffic type: `HTTPS`
    - Destination: `custom` and `<ALB security group>`
    - Description: `https traffic from ALB`
   - Add a 3rd Inbound Rule:
    - traffic type: `SSH`
    - Destination: `custom` and `<Bastion Security group>`
    - Description: `SSH access from Bastion`
  - Add Outbound Rule:
    - traffic: `All traffic`
    - Destination: `anywhere ipv4`
  - Tags:
    - key: `Name`

  <br>
  
  ![nginx_a](https://user-images.githubusercontent.com/92983658/200316922-58930e58-4ee0-4861-99e2-25bbbe2ea19a.png)
![nginx_b](https://user-images.githubusercontent.com/92983658/200316944-0e970848-5ebb-4794-bffa-ae1e710d1fd5.png)
![nginx_c](https://user-images.githubusercontent.com/92983658/200316963-575a6389-7ab7-450f-99dd-20eacb58b96c.png)

 <br>
 
**4. Internal ALB**

  <br>
  
- Select`Security groups` from VPC console and click `create security group`
  - name: `Internal ALB`
  - VPC: select the VPC created for your project
  - Add Inbound Rule:
    - traffic type: `HTTP`
    - Destination: `custom` and `<nginx security group>`
    - Description: `http traffic for internal ALB`
  - Add a 2nd Inbound Rule:
    - traffic type: `HTTPS`
    - Destination: `custom` and `<nginx security group >`
    - Description: `https traffic for internal ALB`
  - Add Outbound Rule:
    - traffic: `All traffic`
    - Destination: `anywhere ipv4`
  - Tags:
    - key: `Name`

<br>
  
![internal_ALB_a](https://user-images.githubusercontent.com/92983658/200320088-62f916d6-b949-4f81-b544-509e251ee1f5.png)

![internal](https://user-images.githubusercontent.com/92983658/200320103-486bd116-5e45-4130-9d72-2800bd93882e.png)

 ![internal_ALB_c](https://user-images.githubusercontent.com/92983658/200320125-7fb4acd8-6bc4-4720-97a7-5bf85adf730e.png)

<br>

**5. Web Servers:** Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.
 
  <br>
  
- Select`Security groups` from VPC console and click `create security group`
  - name: `Webserver`
  - VPC: select the VPC created for your project
  - Add Inbound Rule:
    - traffic type: `SSH`
    - Destination: `custom` and `<bastion security group>`
    - Description: `ssh access for bastion`
  - Add a 2nd Inbound Rule:
    - traffic type: `HTTPS`
    - Destination: `custom` and `<internal ALB security group >`
    - Description: `https traffic for internal ALB`
  - Add a 3rd Inbound Rule:
    - traffic type: `HTTP`
    - Destination: `custom` and `<internal ALB security group >`
    - Description: `http traffic for internal ALB`
  - Add Outbound Rule:
    - traffic: `All traffic`
    - Destination: `anywhere ipv4`
  - Tags:
    - key: `Name`

<br>

![webserver_a](https://user-images.githubusercontent.com/92983658/200322161-30dc6782-0365-4595-a529-fc7082cdc538.png)

![webserver_b](https://user-images.githubusercontent.com/92983658/200322182-ff8fdf38-9bb9-4b89-bdda-507f56e6117f.png)

![webserver_c](https://user-images.githubusercontent.com/92983658/200322195-f692cbf8-8d0a-4e10-bc8d-c812f6699076.png)

<br>
  
**6. Data Layer:** Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

<br>
  
- Select`Security groups` from VPC console and click `create security group`
  - name: `Webserver`
  - VPC: select the VPC created for your project
  - Add Inbound Rule:
    - traffic type: `Mysql/Aurora`
    - Destination: `custom` and `<bastion security group>`
    - Description: `ssh access for bastion`
  - Add a 2nd Inbound Rule:
    - traffic type: `NFS`
    - Destination: `custom` and `<webserver security group >`
    - Description: `https traffic for webserver`
  - Add a 3rd Inbound Rule:
    - traffic type: `Mysql/Aurora`
    - Destination: `custom` and `<weserver security group >`
    - Description: ``
  - Add Outbound Rule:
    - traffic: `All traffic`
    - Destination: `anywhere ipv4`
  - Tags:
    - key: `Name`
  
 <br>
  
![data_](https://user-images.githubusercontent.com/92983658/200324004-9c1b3818-dea0-4c8f-b835-dc9bf991490c.png)

![data_b](https://user-images.githubusercontent.com/92983658/200324018-19ffd968-a702-4979-b77f-f6ddc51a95b6.png)

![data_c](https://user-images.githubusercontent.com/92983658/200324049-563f3944-bd90-4183-8234-388833a5bfe8.png)

<br>
  
## Step Three:Setup Compute Resources
The following compute resources will be to be set up inside the VPC

<br>
  
### TLS Certificates From Amazon Certificate Manager (ACM)
TLS certificates handle secured connectivity to Application Load Balancers (ALB).

<br>
  
- search for `certificate manager` in AWS search console
- Request a public wildcard certificate for the domain name you registered in Freenom
  - in `certificate manager` console, click `request certificate`
  - domain name: add domain name purchased from `freenom`
  - validation maethod: `DNS validation`
  - tags: key -> `Name`, value ->`p15_cert`

<br>

 ![request_certificate](https://user-images.githubusercontent.com/92983658/200328032-dcd679d9-b238-4df1-938d-5626ff778f5d.png)
![request_public_cert_a](https://user-images.githubusercontent.com/92983658/200328079-b3de0c05-dda6-4cfc-814b-1795baee2d39.png)
![request_public_cert_b](https://user-images.githubusercontent.com/92983658/200328125-c0e19a0a-e3fc-450f-a704-04a57672e77e.png)

<br>
 
- `view certificate` -> `create record in Route 53` -> `create records`

<br>
  
![view_certificate](https://user-images.githubusercontent.com/92983658/200329313-ccbdddd6-0d38-4cec-a115-82c90ecf047c.png)

<br>
  
![create_records](https://user-images.githubusercontent.com/92983658/200329312-057aabf9-3fb2-4cb9-8aba-0dae790bc1a3.png)

<br>

### Set Up Compute Resources for Nginx
- Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to your customers). Use EC2 instance of T2 family (e.g. t2.micro or similar)
- SSH into the server and ensure that it has the following software installed:

  - 1. python
  - 2. ntp
  - 3. net-tools
  - 4. vim
  - 5. wget
  - 6. telnet
  - 7. epel-release
  - 8. htop

<br>

```
- sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
- sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
- sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
- sudo systemctl enable chronyd
- sudo systemctl start chronyd

# configure selinux policies for webservers and nginx servers
- sudo setsebool -P httpd_can_network_connect=1
- sudo setsebool -P httpd_can_network_connect_db=1
- sudo setsebool -P httpd_execmem=1
- sudo setsebool -P httpd_use_nfs 1

# install efs-utils for mounting target on efs
- git clone https://github.com/aws/efs-utils
- cd efs-utils
- sudo yum install -y make
- sudo yum install -y rpm-build
- make rpm
- sudo yum install -y ./build/amazon-efs-utils*rpm
```

<br>
  
![nginx](https://user-images.githubusercontent.com/92983658/201080052-07dd2539-2566-47b6-b418-779a56283555.png)

<br>
  
![nginx_1](https://user-images.githubusercontent.com/92983658/201080084-8f537672-cb51-4512-bc76-e2d5d2beee53.png)

<br>
  
![nginx_2](https://user-images.githubusercontent.com/92983658/201080109-2209cb3e-9b32-4dd9-bd3c-503766991ba8.png)

<br>
  
![nginx_£](https://user-images.githubusercontent.com/92983658/201080135-5239bc5c-2b12-437f-8fe4-4c720200b219.png)

<br>
  
![nginx_4](https://user-images.githubusercontent.com/92983658/201080164-ee3dcee6-d8c2-46f7-872b-5b2678f3c094.png)

<br>
  
### Set Up Compute Resources for Bastion
- Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) with the same Availability Zone and Region as the Nginx servers
- SSH into the server and ensure that it has the following software installed:

  - 1. python
  - 2. ntp
  - 3. net-tools
  - 4. vim
  - 5. wget
  - 6. telnet
  - 7. epel-release
  - 8. htop

<br>

```
- sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
- sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
- sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y
- sudo systemctl enable chronyd
- sudo systemctl start chronyd

```

<br>

![stion](https://user-images.githubusercontent.com/92983658/201073662-3307b015-6eb4-4ba4-9be6-d368a710948f.png)

<br>
  
![bastion_1](https://user-images.githubusercontent.com/92983658/201073663-cd57cf12-e9c3-4072-bcef-d826c60c346c.png)

<br>
  
![bastion_2](https://user-images.githubusercontent.com/92983658/201073696-c9e14e95-bfce-4935-8bf4-d788fbddcc97.png)

<br>
  
 ![bastion_3](https://user-images.githubusercontent.com/92983658/201073763-c68fc28a-544f-4895-b5cb-b7f7961adc8b.png)

<br>
  
  
- Associate an Elastic IP with each of the Bastion EC2 Instances
- Create an AMI out of the EC2 instance

<br>

## Configure Application Load Balancer

<br>

### Elastic File System
Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will use the EFS service to mount filesystems on both Nginx and Webservers to store data.

<br>

**Create an EFS filesystem**

<br>

- search for `EFS` in AWS search console and click `create file system`
  - name: `project name`
  - VPC: `<project VPC>`
  - Storage class: `standard`

<br>
  
![file_system](https://user-images.githubusercontent.com/92983658/200805262-546c33fb-5c66-4b89-9612-17c69e8d6b69.png)

<br>

**Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer and Associate the Security groups created earlier for data layer.**

<br>

- click `customise` on the `create file` dashboard
- enter `tags` and click `next`
- Edit Network Access
  - Mount Targets:
    - Subnet ID: `private subnet 1` and `private subnet 2`
    - security group: `data layer`
- review and create file system

<br>
  
![mount](https://user-images.githubusercontent.com/92983658/200807206-10228848-6cac-40e7-91e7-5ecc6905e006.png)

<br>

**Create an EFS access point. (Give it a name and leave all other settings as default)**

<br>
  

- on EFS file system dashboard, select project file system and click `access points` and then `create access points`
- **Details:**
  - name: `wordpress`
  - root directory path: `/wordpress`
- **POSIX USER:**
  - user iD: `0`
  - group ID: `0`
- **Root directory creation permissions:**
  - Owner user ID : `0`
  - Owner user ID: `0`
  - POSIX permissions to apply to the root directory path: `0755`
- **Tags:**
  - key: `Name`
  - value: `wordpress_app`
   
- create a second access point named `tooling` with the same configuration as the `wordpress` access point.

<br>
  
![access_1](https://user-images.githubusercontent.com/92983658/200814014-1747dd86-c836-4d03-ad60-5510d99e2d9f.png)

 ![access_2](https://user-images.githubusercontent.com/92983658/200814037-034a7a79-cc3a-4158-be55-c8cefd59eda1.png)

 <br>
  
### Set UP RDS

<br>
  
**Pre-requisite:** Create a KMS key from `Key Management Service (KMS)` to be used to encrypt the database instance.

<br>

- search for `Key Management Service` on AWS search console and click `create a key`
  - key type: `symmetric`
  - key usage: `encrypt and decrypt`
- **Add labels:**
  - name: `p15_RDS`
  - description: `for RDS instance`
  - Tags: key ->`Name` and value -> `p15_RDS_key`
- **Define Key Administrative Positions:**
  - Key Administrators: `<select admin for yout AWS account>`
- **Define Key usage permissions:**
  - This account: `<select admin for your AWS account>`

<br>
  
![KMS](https://user-images.githubusercontent.com/92983658/200820296-ef555fd4-c6c3-4a21-8c1d-5fb219cfffe0.png)

<br>
  
To ensure that databases are highly available and also have failover support in case one availability zone fails, a multi-AZ set up of RDS MySQL database instance will be configured. In this project case, since there are only 2 AZs, there can only be a failover to one, but the same concept applies to 3 Availability Zones.

To configure RDS, follow steps below:
- **Create a subnet group and add 2 private subnets (data Layer)**

<br>

- search AWS for RDS and on the RDS console, select `subnet groups` and click `create DB subnet group`
- subnet group details:
  - name: `<any name connect to project>`
  - details: `for RDS subnets`
  - VPC: `project VPC`
- add subnet 
  - availability zones: `<select zones where subnets created at the start of the project>`
  - subnets: `according to project diagram: subnet 3 and subnet 4 with ips: 10.0.5.0/24 and 10.06.0/24`

<br>
  
![rds_a](https://user-images.githubusercontent.com/92983658/200824192-8dcebce5-9d50-4d05-9d2d-ddc65c28c9e3.png)

![rds_b](https://user-images.githubusercontent.com/92983658/200824191-a35c29a2-918b-42e6-8f2f-a591336db629.png)

![rds_c](https://user-images.githubusercontent.com/92983658/200824206-385f5d8d-2522-4095-ad05-2343ed05c74a.png)

<br>
  
- **Create an RDS Instance for `mysql 8.*.*` **
  To satisfy the architectural diagram, select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)

 <br>

- on RDS dashboard, click `create database` and configure

<br>

![database_a](https://user-images.githubusercontent.com/92983658/200830310-e5189fc0-aa00-4e52-90c3-a48d297dc7c8.png)
![database_b](https://user-images.githubusercontent.com/92983658/200830330-1e6da1b9-ef9a-4c12-ab4b-87de2fab0647.png)
![database_c](https://user-images.githubusercontent.com/92983658/200830348-bd2c90b0-f29d-4fb1-9a67-948517c42ad1.png)
![database_d](https://user-images.githubusercontent.com/92983658/200830375-658fe20e-700c-4ee5-9424-0f37daeefb9c.png)
![database_e](https://user-images.githubusercontent.com/92983658/200830392-048ed067-a617-40d6-9174-23031fb0c002.png)
![database_f](https://user-images.githubusercontent.com/92983658/200831084-22cbcc95-c0fe-4e6c-a90e-da04e7234903.png)
![database_g](https://user-images.githubusercontent.com/92983658/200831110-51613244-5f3f-4673-a6ab-81f62ec2ea71.png)
![database_h](https://user-images.githubusercontent.com/92983658/200831140-d6f0e112-1372-496d-9dc8-552b533c747d.png)
![database_i](https://user-images.githubusercontent.com/92983658/200831156-65b0f7ad-13c4-4daa-b0ba-10c0c26f7658.png)
![database_j](https://user-images.githubusercontent.com/92983658/200831180-1b501dd2-2076-4a16-b254-2ccacbdc6baf.png)
![database_k](https://user-images.githubusercontent.com/92983658/200831206-cfa41c8b-8faf-46c6-beac-8ad6f36ed3d3.png)
![database_l](https://user-images.githubusercontent.com/92983658/200832440-29992a00-5e0d-454b-9937-341d346122db.png)
![database_m](https://user-images.githubusercontent.com/92983658/200832442-66da6c8a-042b-4b41-b198-870048ee7484.png)
![database_n](https://user-images.githubusercontent.com/92983658/200832468-227346bc-0b9e-47e3-97bf-35b1dd0b018f.png)
