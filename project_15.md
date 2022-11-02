<br>

# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

### Introduction
This project demostrates how a secure infrastructure inside AWS VPC (Virtual Private Cloud) network is built for a particular company, 
who uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. 
As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology 
from NGINX to achieve this. The infrastructure will look like following diagram:

<br>

The following outlines the steps taken:

<br>

## Step One: Configure AWS account and organisational units

- Create an AWS Master account. (Also known as Root Account)
- Within the Root account, create an AWS Organization Unit (OU). Name it `Dev`
  - search for `AWS organisations` -> `create AWS organisation`
  
- Within `AWS organisations`, create a sub-account and name it `DevOps`.
  - `AWS organisations` -> `AWS accounts` -> `add an AWS account`

<br>

![subaccount](https://user-images.githubusercontent.com/92983658/199480089-9e4d87cf-6ca4-406f-ae50-88fb1af25aed.png)

<br>

Move the DevOps account into the Dev OU.
Login to the newly created AWS account using the new email address.
