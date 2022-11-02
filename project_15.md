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

<br>

![subaccount](https://user-images.githubusercontent.com/92983658/199480089-9e4d87cf-6ca4-406f-ae50-88fb1af25aed.png)

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
