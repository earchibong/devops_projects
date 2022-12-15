# AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 4 – TERRAFORM CLOUD
### introduction
<br>

The sole aim of this project to build the infrastructure in AWS using terraform. But, instead of running the Terraform codes in 
<a href="https://github.com/earchibong/devops_training/terraform_03.md">project 18</a> from a command line, it will be executed from **Terraform cloud** console. 

The AMI is built differently with `packer` while Ansible is used to configure the infrastructure after its been provisioned by Terraform.

The following outlines the steps:

<br>

## STEP ONE: Migrate `.tf` codes to Terraform Cloud

<br>

- 1. Create `Terraform Cloud` Account : follow <a href="https://app.terraform.io/public/signup/account">this link</a> to do so
- 2. Create an organization:
  - Select `Start from scratch`, choose a name for your organization and create it. 

<br>

![organisation](https://user-images.githubusercontent.com/92983658/207548121-befda977-0466-4707-a40a-b09cd6b0e864.png)

<br>

- 3. Configure a workspace:
  - *note: understand the difference between `version control workflow`, `CLI-driven workflow` and `API-driven workflow`. Learn more about it <a   href="https://www.youtube.com/watch?v=m3PlM4erixY&t=287s">here</a>* 
  - As the most common and recommended way to run Terraform commands triggered from our git repository, `version control workflow` will be used   for this project.
  
 <br>

  - Create a new repository in GitHub and call it `terraform-cloud`.
  - push Terraform codes developed in the previous projects to the repository.: 
    - on terminal, change Git remote url with `git remote set-url` command: 
     - `git remote set-url origin <repo url>`
     - `git push`
     
 <br>
  
  - in terraform cloud, choose `version control workflow` and you will be promped to connect the GitHub account to your workspace – follow the prompt and add your newly created repository to the workspace. 
  
<br>

![workspace_1a](https://user-images.githubusercontent.com/92983658/207563508-32f25fb7-a91b-4eff-a9fe-61c2a4233a59.png)

<br>

![wordspace_1b](https://user-images.githubusercontent.com/92983658/207563539-30328ef2-de15-4e09-9ba1-0bcf6a31b85c.png)

<br>

- 4. Configure variables:

Terraform Cloud supports two types of variables: environment variables and Terraform variables: 
   - Set two environment variables: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, set the values that you used in <a href="https://github.com/earchibong/devops_training/blob/main/terraform_01.md">Project 16</a>
 
<br>

![variables_1a](https://user-images.githubusercontent.com/92983658/207566827-515a1506-24ce-4ad5-8fcc-82e617e2fdc4.png)

<br>

![variables_1b](https://user-images.githubusercontent.com/92983658/207566862-29e4f41a-b063-40c3-bc36-75ab374783e3.png)

<br>

Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources.

<br>

## STEP TWO: Build AMI images with Packer

- install `packer`: get instructions <a href="https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli"> here</a>
- Add a new folder to workspace environment named `AMI`

<br>

![AMI](https://user-images.githubusercontent.com/92983658/207817573-fbc05c0f-4cf9-4d7d-b119-26b91c89d55a.png)

<br>

- in terminal, navigate to `AMI`
- create bastion packer file, name it `bastion.pkr.hcl` and add the following to it:

<br>
```

variable "region" {
  type    = string
  default = "<enter your region>"
}

locals {
  timestamp = regex_replace(timestamp(), "[- TZ:]", "")
}


# source blocks are generated from your builders; a source can be referenced in
# build blocks. A build block runs provisioners and post-processors on a
# source.
source "amazon-ebs" "terraform-bastion-prj-19" {
  ami_name      = "terraform-bastion-prj-19-${local.timestamp}"
  instance_type = "t2.micro"
  region        = var.region
  source_ami_filter {
    filters = {
      name                = "RHEL-8.2_HVM-20200803-x86_64-0-Hourly2-GP2"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["309956199498"]
  }
  ssh_username = "ec2-user"
  tag {
    key   = "Name"
    value = "terraform-bastion-prj-19"
  }
}

# a build block invokes sources and runs provisioning steps on them.
build {
  sources = ["source.amazon-ebs.terraform-bastion-prj-19"]

  provisioner "shell" {
    script = "bastion.sh"
  }
}

```

<br>

![bastion_packer_1a](https://user-images.githubusercontent.com/92983658/207817646-73a68cb2-c41f-4b3c-a62f-aa33f1eee113.png)

<br>

- create a new file named `bastion.sh` and add the following to it:

<br>

```

# user data for bastion

#!/bin/bash
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm 
sudo yum install -y mysql-server wget vim telnet htop git python3 net-tools zip
sudo systemctl start chronyd
sudo systemctl enable chronyd


#installing java 11
sudo yum install -y java-11-openjdk-devel
sudo echo "export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))" >> ~/.bash_profile
sudo echo "export PATH=$PATH:$JAVA_HOME/bin" >> ~/.bash_profile
sudo echo "export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar" >> ~/.bash_profile
source ~/.bash_profile

# clone the Ansible repo
git clone https://github.com/darey-devops/PBL-project-19.git


# install botocore, ansible and awscli
sudo python3 -m pip install boto
sudo python3 -m pip install boto3
sudo python3 -m pip install PyMySQL
sudo python3 -m pip install mysql-connector-python
sudo python3 -m pip install --upgrade setuptools
sudo python3 -m pip install --upgrade pip
sudo python3 -m pip install psycopg2==2.7.5 --ignore-installed
sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo unzip awscliv2.zip
sudo ./aws/install
sudo yum install ansible -y
sudo yum install -y policycoreutils-python-utils
ansible-galaxy collection install amazon.aws
ansible-galaxy collection install community.general
ansible-galaxy collection install community.mysql
ansible-galaxy collection install community.postgresql

```

<br>

![bastion_packer_1b](https://user-images.githubusercontent.com/92983658/207821116-86d1ccb0-241b-4bfa-adf6-3f2c9b1481e1.png)

<br>

- create an nginx packer file, name it `nginx.pkr.hcl` and add the following to it:

```

variable "region" {
  type    = string
  default = "eu-west-2"
}

locals { timestamp = regex_replace(timestamp(), "[- TZ:]", "") }


# source blocks are generated from your builders; a source can be referenced in
# build blocks. A build block runs provisioners and post-processors on a
# source.
source "amazon-ebs" "terraform-nginx-prj-19" {
  ami_name      = "terraform-nginx-prj-19-${local.timestamp}"
  instance_type = "t2.micro"
  region        = var.region
  source_ami_filter {
    filters = {
      name                = "RHEL-8.2_HVM-20200803-x86_64-0-Hourly2-GP2"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["309956199498"]
  }
  ssh_username = "ec2-user"
  tag {
    key   = "Name"
    value = "terraform-nginx-prj-19"
  }
}


# a build block invokes sources and runs provisioning steps on them.
build {
  sources = ["source.amazon-ebs.terraform-nginx-prj-19"]

  provisioner "shell" {
    script = "nginx.sh"
  }
}

```

<br>

![nginx_packer_1a](https://user-images.githubusercontent.com/92983658/207832863-0fe3f445-3540-43f2-a9a7-dc7b9db1cc06.png)

<br>

- create a new file named `nginx.sh` and add the following to it:

```

#!/bin/bash
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm 
sudo yum install -y mysql wget vim telnet htop git python3 net-tools 
sudo systemctl start chronyd
sudo systemctl enable chronyd

# selinux config
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db=1
sudo setsebool -P httpd_execmem=1
sudo setsebool -P httpd_use_nfs=1


# installing self signed certificate
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt \
-subj "/C=UK/ST=London/L=London/O=darey.io/OU=devops/CN=$(curl -s http://169.254.169.254/latest/meta-data/local-hostname)"

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

```

<br>

![nginx_packer_1b](https://user-images.githubusercontent.com/92983658/207833224-178f5339-2039-4e71-a33e-95e529ff2e9c.png)

<br>

- create a wordpress and tooling packer file, name it `web.pkr.hcl` and add the following to it:

```

variable "region" {
  type    = string
  default = "us-east-1"
}

locals { timestamp = regex_replace(timestamp(), "[- TZ:]", "") }


# source blocks are generated from your builders; a source can be referenced in
# build blocks. A build block runs provisioners and post-processors on a
# source.
source "amazon-ebs" "terraform-web-prj-19" {
  ami_name      = "terraform-web-prj-19-${local.timestamp}"
  instance_type = "t2.micro"
  region        = var.region
  source_ami_filter {
    filters = {
      name                = "RHEL-8.2_HVM-20200803-x86_64-0-Hourly2-GP2"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["309956199498"]
  }
  ssh_username = "ec2-user"
  tag {
    key   = "Name"
    value = "terraform-web-prj-19"
  }
}


# a build block invokes sources and runs provisioning steps on them.
build {
  sources = ["source.amazon-ebs.terraform-web-prj-19"]

  provisioner "shell" {
    script = "web.sh"
  }
}

```

<br>

![web_packer_1a](https://user-images.githubusercontent.com/92983658/207834042-0655cb13-460e-45fd-9880-2c2a64bffba3.png)

<br>

- create a new file named `web.sh` and add the following to it:

```

#!/bin/bash
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum install -y mysql wget vim telnet htop git python3 net-tools 
sudo systemctl start chronyd
sudo systemctl enable chronyd
sudo yum module reset php -y
sudo yum module enable php:remi-7.4 -y

# selinux config
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db=1
sudo setsebool -P httpd_execmem=1
sudo setsebool -P httpd_use_nfs=1

# installing efs-utils
sudo git clone https://github.com/aws/efs-utils /efs-utils/
cd /efs-utils
sudo yum install -y make
sudo yum install -y rpm-build
sudo make rpm
sudo yum install -y  ./build/amazon-efs-utils*rpm

#installing java 11
sudo yum install -y java-11-openjdk-devel
sudo echo "export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))" >> ~/.bash_profile
sudo echo "export PATH=$PATH:$JAVA_HOME/bin" >> ~/.bash_profile
sudo echo "export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar" >> ~/.bash_profile
source ~/.bash_profile


#installing self signed certificate for apache
sudo yum install -y mod_ssl

sudo openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt \
-subj "/C=UK/ST=London/L=London/O=darey.io/OU=devops/CN=$(curl -s http://169.254.169.254/latest/meta-data/local-hostname)"


sudo sed -i 's/localhost.crt/ACS.crt/g'  /etc/httpd/conf.d/ssl.conf

sudo sed -i 's/localhost.key/ACS.key/g'  /etc/httpd/conf.d/ssl.conf

```

<br>

![web_packer_1b](https://user-images.githubusercontent.com/92983658/207834455-93cad95b-765d-4564-b9f3-1787b1906052.png)

<br>

- create a Jenkins, Sonaqube and Artifactory packer file, name it `ubuntu.pkr.hcl` and add the following to it:

```

variable "region" {
  type    = string
  default = "us-east-1"
}

locals { timestamp = regex_replace(timestamp(), "[- TZ:]", "") }


# source blocks are generated from your builders; a source can be referenced in
# build blocks. A build block runs provisioners and post-processors on a
# source.
source "amazon-ebs" "terraform-ubuntu-prj-19" {
  ami_name      = "terraform-ubuntu-prj-19-${local.timestamp}"
  instance_type = "t2.micro"
  region        = var.region
  source_ami_filter {
    filters = {
      name                = "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["099720109477"]
  }
  ssh_username = "ubuntu"
  tag {
    key   = "Name"
    value = "terraform-ubuntu-prj-19"
  }
}


# a build block invokes sources and runs provisioning steps on them.
build {
  sources = ["source.amazon-ebs.terraform-ubuntu-prj-19"]

  provisioner "shell" {
    script = "ubuntu.sh"
  }
}

```

<br>

![ubuntu_packer_1a](https://user-images.githubusercontent.com/92983658/207835311-e984ca74-fcb0-4de3-93eb-02d36b615f89.png)

<br>

- create a new file named `ubuntu.sh` and add the following to it:

```

#!/bin/bash
sudo apt update

sudo apt install -y default-jre

sudo apt install -y default-jdk

sudo apt install -y  git mysql-client wget vim telnet htop python3 chrony net-tools

```

<br>

![ubuntu_packer_1b](https://user-images.githubusercontent.com/92983658/207835772-40871e4c-8221-455f-8e22-179230b75d07.png)

<br>

- Run the packer command to build AMI for Bastion server: `packer build bastion.pkr.hcl`
