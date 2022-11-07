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
  - name: `external ALB`
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
