# Ansible Configuration Management

**Task**
- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create a simple Ansible playbook to automate servers configuration

## Install And Configure Ansible On EC2
- Update Name tag on your `Jenkins EC2` Instance (from project 9) to `Jenkins-Ansible`.
- In GitHub create a new repository and name it `ansible-config-mgt`.
- Install Ansible on `jenkins-ansible`
```

sudo apt update

sudo apt install ansible

```

- confirm ansible version : `ansible --version`

![ansible_version](https://user-images.githubusercontent.com/92983658/187173699-1cc70a6f-0a86-499a-9b02-369de77db140.png)

<br>

- ### Configure Jenkins build job to save repository content every time it is changed
  - Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.

![git_](https://user-images.githubusercontent.com/92983658/187188044-04d76543-c349-494f-9985-1c0091037423.png)

<br>

![jenkins_ansible](https://user-images.githubusercontent.com/92983658/187188056-c9e5ea6d-2911-4e32-9627-54f5ad29af1c.png)

<br>

  - Configure Webhook in GitHub and set webhook to trigger ansible build: `ansible-config-mgt` -> settings -> webhooks 
  - 
  <br>
  
 ![git_webhook](https://user-images.githubusercontent.com/92983658/187188260-9e17d485-f287-47cb-9b58-a8db2b4da203.png)

<br>

  - Configure build trigger to `GitHub hook trigger for GITScm polling` 
  - Configure a Post-build job to save all (**) files
  - check configuration: click build now button:
    - if configuration is correct, then build will be successful and will appear under #1 at the bottom left of the dashboard
    
  <br>  
  
  ![build_check](https://user-images.githubusercontent.com/92983658/187189829-e9e1df48-8fac-4052-898b-fd5f99065b87.png)

<br>

- Test setup by making some change in README.MD file in master / main branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder: 
  - `ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
  - `cd /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
  - `cat README.md`
  
  <br>
  
![console_output](https://user-images.githubusercontent.com/92983658/187190646-4ee622e8-b7e2-4454-9444-85cdbbba354c.png)

<br>

![ls_jenkins_build](https://user-images.githubusercontent.com/92983658/187191297-c758647b-edf2-4f31-b851-1ab1a309369e.png)

<br>

![cat_readme](https://user-images.githubusercontent.com/92983658/187192146-06ad3eb6-4afa-4e1a-8101-64e4f6a00625.png)

<br>

- ### Allocate Elastic IP to Jenkins- Ansible Server
  - In the top search bar of AWS console, enter "Elastic IP". The search bar will return several results on services and features available.
  - Click the ELASTIC IPs - EC2 feature from the list:
   - Once on the Elastic IP addresses screen, click ALLOCATE ELASTIC IP ADDRESS on the top right of the page.
   - Enter details on allocation page and allocate elastic IP
 - Under the Actions menu of elastic ip addresses click associate elastic ip address:
  - on the asocialte elastic ip address screen, assign the elastic ip to the `jenkins-ansible` server 

<br>

![elastic-ip-jenkins](https://user-images.githubusercontent.com/92983658/187193366-eb522ce6-5fab-49cf-9370-92d4f27b963e.png)

<br>

- update github webhook with ealstic ip

<br>

![elastic_ip_jenkins_webhook](https://user-images.githubusercontent.com/92983658/187193874-d4c15a20-3e9b-42c8-86c7-65f73f872764.png)

<br>

## Prepare Development Environment

- Create environment in `AWS could9`: 
  - From AWS Management Console, inside the Developer Tools choose `Cloud9`.
  - Press Create environment. Add any name for that environment then press `next`. 
  - In the configure settings, change the Environment type to Connect and run in remote server (SSH). In the user, write `ubuntu` (name you used to connect to the instance in terminal). Add the instance’s public DNS IPv4. Then press Copy Key to Clipboard. 
  - ssh into `Jenkins-Ansible`
  - in the terminal that is connected to `jenkins-ansible`, write the following commands to save the key in the machine:
```
  
  echo <Paste the Copied Key> >> ~/.ssh/authorized_keys
sudo apt-get install -y nodejs

```

  - Press Next Step in Configure Settings then Create Environment. 
 
 <br>
  
 ![ssh_environment](https://user-images.githubusercontent.com/92983658/187398193-c8e49b4b-c1d1-4edc-a9f7-8777bc2b1a4d.png)

<br>

  - Right click on “C9 install” and copy link address.
  - Press Next and leave everything ticked.
 
 <br>

  ![c9_install](https://user-images.githubusercontent.com/92983658/187400295-5d6b8319-9422-442c-a340-fbc36c551397.png)
  
  <br>

  - go to terminal and run the following commands:

```
  
  wget <Paste the copied c9 install link address>
chmod a+x c9-install.sh
sudo apt-get -y install python
sudo apt-get install build-essential
./c9-install.sh

```
  - close terminal and continue from Cloud9.

You can upload files or data from your local machine to the EC2 using Cloud9 just by drag and drop. You can also run/debug the code or use terminal below to interact with the machine / server

**Note:** *If the instance’s IP address changed (Happens when you stop an Instance then restart its running), you will just simply need to copy the new IPv4 address.*

*In Cloud9, on the top left corner, press the Cloud9 icon.*

*Click on “Go to Your Dashboard”. Press the desired environment and from the top right corner then choose Edit. Then Scroll down to the Host and just change the IPv4 address.*

find out more <a href="https://towardsdatascience.com/creating-aws-ec2-and-connecting-it-with-aws-cloud9-ide-and-aws-s3-a6313aa82ec">here</a>

- create`SSH key` for ansible: `ssh-keygen -t ed25519 -C "ansible"`


  
### Configure `cloud9` to connect to the newly created GitHub repository.

- Right-click on root folder and create a new folder under it called `repository`
- On your terminal windows, navigate to this repository folder: `cd repository`
- Check if git is installed: `git --version`
- Clone remote GitHub repository in your environment: `git clone <ansible-config-mgt repo link>`

## BEGIN ANSIBLE DEVELOPMENT

- In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
- Checkout the newly created feature branch to your local machine and start building your code and directory structure
```
git checkout -b PRJ-11

```
- Create a directory and name it `playbooks` – it will be used to store all your playbook files: `mkdir playbooks`
- Create a directory and name it `inventory` – it will be used to keep your hosts organised.: `mkdir inventory`
- Within the playbooks folder, create your first playbook, and name it common.yml
```
cd playbooks
touch common.yml

```
- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.
```

cd ..
cd inventory
touch dev.yml
touch staging.yml
touch uat.yml
touch prod.yml

```

- Update inventory/dev.yml file with this snippet of code:
 *notice, that Load Balancer and database user is ubuntu and user for RHEL-based servers is ec2-user.*
```

[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ubuntu' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'

```


![dev_yml](https://user-images.githubusercontent.com/92983658/187440923-c9b1d29a-4b2f-4e4d-813b-7477669cc509.png)


### Create a common playbook

In common.yml playbook configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure will be written

- Update your playbooks/common.yml file with following code:
```

---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
        
 ```
 
 ![common_yml](https://user-images.githubusercontent.com/92983658/187443293-6b5840c0-cea1-4c81-b34d-c5b16808d878.png)


### Commit Code To Github

- use git commands to add, commit and push your branch to GitHub.
```

git status

git add <selected files>

git commit -m "commit message"

git push origin <branch name>

```

![git_status_commit](https://user-images.githubusercontent.com/92983658/187445141-75ccddad-f527-41cd-a8f2-682d1197bcbf.png)


- Create a Pull request (PR) for branch

![compreand pull](https://user-images.githubusercontent.com/92983658/187450096-05082257-7476-4121-a286-103ee5ae0096.png)

<br>
![mergepullrequest](https://user-images.githubusercontent.com/92983658/187450122-cdc7ef2d-df20-42b0-858d-372d43df9fe6.png)

<br>
![successful_merge](https://user-images.githubusercontent.com/92983658/187450503-3004fdbf-30d0-4b09-9727-01a440045f97.png)

<br>

![updated_repo](https://user-images.githubusercontent.com/92983658/187452549-be8eef4e-3674-4cd9-98ba-3032bf8fb089.png)

<br>

- Head back on terminal, checkout from the feature branch into the master, and pull down the latest changes:
```
git checkout main
git pull

```
<br>
- Once code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on `Jenkins-Ansible server.`

![jenkins_confirm](https://user-images.githubusercontent.com/92983658/187456019-fa40193b-f2a2-4683-b888-14ba9b60d03e.png)

![var_confirm](https://user-images.githubusercontent.com/92983658/187456104-326931f2-1ea3-4d36-b7e0-0e7086796bb6.png)











