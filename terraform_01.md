<br>

# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

This project demonstrates how the AWS infrastructure for 2 websites that was built manually in project 15 is automated with the use of Terraform.

The following outlines the steps taken:

<br>

![image](https://user-images.githubusercontent.com/92983658/203531844-0d082b08-87b1-4b95-ad5d-c921e0b61611.png)

<br>

## Pre-requisites:

- **Create an IAM user, name it `terraform` (ensure that the user has only programatic access to your AWS account) and grant this user 
 `Administrator` Access` permissions**
- **Copy the secret access key and access key ID. Save them in a notepad temporarily.**
 - search for `IAM` in AWS search console
 - click `user` on `IAM` dashboard and then `add users`
 - select `programmatic access` for AWS credential type
 
<br>

![create_user_1a](https://user-images.githubusercontent.com/92983658/203534121-91fdf847-6ff0-4d83-b25c-46a35521cb22.png)

<br>

- select `add user to group` -> `create group`
- select `adminitrator access`

<br>

![create_user_1b](https://user-images.githubusercontent.com/92983658/203535472-e4dc8957-1dc2-4e06-80fb-1c5f1c81ab29.png)

<br>

![create_user_ic](https://user-images.githubusercontent.com/92983658/203535500-8d445772-a3e3-4e0c-a8f1-fe162936a290.png)

<br>

![create_user_1d](https://user-images.githubusercontent.com/92983658/203535541-8b51d69d-86b6-40a2-862b-0b225234bc5b.png)

<br>

![create_user_1e](https://user-images.githubusercontent.com/92983658/203535596-9d0bdd18-8bf1-4fca-b98c-3aa958effc34.png)

<br>

![create_user_1f](https://user-images.githubusercontent.com/92983658/203535617-d5b1abf3-4a86-4b48-9879-3c74add84264.png)

<br>

- **Configure programmatic access from your workstation to connect to AWS using the access keys copied above and a Python 
SDK (boto3). You must have Python 3.6 or higher on your workstation.**

*If you are on Windows, use gitbash, if you are on a Mac, you can simply open a terminal.*
**For easier authentication configuration – `AWS CLI` with `aws configure` command will be used.**

- Create an S3 bucket to store Terraform state file. You can name it something like <yourname>-dev-terraform-bucket 
 (Note: S3 bucket names must be unique unique within a region partition)
 - search for `S3` in AWS conslole -> `create bucket`

<br>
 
![bucket_1a](https://user-images.githubusercontent.com/92983658/203540854-411bd131-323e-4a03-ad73-8d74b0c2a18b.png)

<br>
 
![bucket_1b](https://user-images.githubusercontent.com/92983658/203540915-6dd84aec-a8f2-48de-ae4a-6102ea9121ef.png)

 <br>
 
 ![bucket_1c](https://user-images.githubusercontent.com/92983658/203541069-89122e34-f691-4e5c-a2ae-f817b9022e31.png)

<br>
 
![cbucket_1g](https://user-images.githubusercontent.com/92983658/203542731-8825e666-4ddb-4e73-b3ee-f5c0e3f173f3.png)

<br>
 
![bucket_1e](https://user-images.githubusercontent.com/92983658/203541131-d116b60e-6b99-46bf-a5eb-8e9701686a8b.png)

<br>
 
- To install AWS SDK boto3 , it is recommended to upgrade the Python to latest version : `brew install python`
- install `pip`: `sudo easy_install pip`
- install `boto3`: `pip3 install boto3`
- install `AWS CLI`: `pip3 install awscli`
- configure credentials with `aws configure` command

<br>

![boto3_install](https://user-images.githubusercontent.com/92983658/203551877-3cebfcb7-27c3-4f87-ba8c-bc5225cb2151.png)

<br>
 
- ensure you can programmatically access AWS account by running following commands in `>python`:
```
python

import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
 
```

- *You should see your previously created S3 bucket name*
<br>



 
