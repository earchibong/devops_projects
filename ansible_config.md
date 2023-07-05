# Ansible Configuration Management

**Task**
- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create a simple Ansible playbook to automate servers configuration

architecture:

<br>

![ansible_architecture](https://user-images.githubusercontent.com/92983658/187944481-edaf388e-6c87-4c8a-ba6f-d9f77f2fc3a2.png)

<br>

## Project Steps:
- <a href="https://github.com/earchibong/devops_projects/blob/main/ansible_config.md#install-and-configure-ansible-on-ec2">Install And Configure Ansible On EC2</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/ansible_config.md#configure-jenkins-build-job-to-save-repository-content-every-time-it-is-changed">Configure Jenkins build job to save repository content every time it is changed</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/ansible_config.md#allocate-elastic-ip-to-jenkins--ansible-server">Allocate Elastic IP to Jenkins- Ansible Server</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/ansible_config.md#prepare-development-environment">Prepare Development Environment</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/ansible_config.md#set-up-ansible-inventory">Set up Ansible Inventory</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/ansible_config.md#create-a-playbook ">Create a playbook</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/ansible_config.md#run-first-ansible-test">Run Ansible Test</a>

<br>

<br>

## Install And Configure Ansible On EC2
- Update Name tag on your `Jenkins EC2` Instance (from project 9) to `Jenkins-Ansible`.
- In GitHub create a new repository and name it `ansible-config-mgt`.
- Install Ansible on `jenkins-ansible`
```

sudo apt update

sudo apt install ansible

```

- confirm ansible version: `ansible --version`

![ansible_version](https://user-images.githubusercontent.com/92983658/187173699-1cc70a6f-0a86-499a-9b02-369de77db140.png)

<br>

<br>

- ### Configure Jenkins build job to save repository content every time it is changed
  - Create a new Freestyle project `ansible` in Jenkins and point it to your ‘ansible-config-mgt’ repository.

<br>

<br>

![git_](https://user-images.githubusercontent.com/92983658/187188044-04d76543-c349-494f-9985-1c0091037423.png)

<br>

![jenkins_ansible](https://user-images.githubusercontent.com/92983658/187188056-c9e5ea6d-2911-4e32-9627-54f5ad29af1c.png)

<br>

<br>

  - Configure Webhook in GitHub and set webhook to trigger ansible build: `ansible-config-mgt` -> settings -> webhooks 
  
  <br>
  
 <br>
 
 ![ansible_jenkins_webhook](https://user-images.githubusercontent.com/92983658/187915459-1ea88589-b194-49dc-b5a2-e002185f4e55.png)


<br>

<br>

  - Configure build trigger to `GitHub hook trigger for GITScm polling` 
  - Configure a Post-build job to save all (**) files
  - check configuration: click build now button:
    - if configuration is correct, then build will be successful and will appear under #1 at the bottom left of the dashboard
    
  <br>  

  <br>
  
  ![build_check](https://user-images.githubusercontent.com/92983658/187189829-e9e1df48-8fac-4052-898b-fd5f99065b87.png)

<br>

<br>


- Test setup by making some change in README.MD file in master / main branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder: 
  - `ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
  - `cd /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
  - `cat README.md`
  
  <br>

  <br>
  
![console_output](https://user-images.githubusercontent.com/92983658/187190646-4ee622e8-b7e2-4454-9444-85cdbbba354c.png)

<br>

<br>

![ls_jenkins_build](https://user-images.githubusercontent.com/92983658/187191297-c758647b-edf2-4f31-b851-1ab1a309369e.png)

<br>

![cat_readme](https://user-images.githubusercontent.com/92983658/187192146-06ad3eb6-4afa-4e1a-8101-64e4f6a00625.png)

<br>

<br>

- ### Allocate Elastic IP to Jenkins- Ansible Server
  - In the top search bar of AWS console, enter "Elastic IP". The search bar will return several results on services and features available.
  - Click the ELASTIC IPs - EC2 feature from the list:
   - Once on the Elastic IP addresses screen, click ALLOCATE ELASTIC IP ADDRESS on the top right of the page.
   - Enter details on allocation page and allocate elastic IP
 - Under the Actions menu of elastic ip addresses click associate elastic ip address:
  - on the asocialte elastic ip address screen, assign the elastic ip to the `jenkins-ansible` server 

<br>

<br>

![elastic-ip-jenkins](https://user-images.githubusercontent.com/92983658/187193366-eb522ce6-5fab-49cf-9370-92d4f27b963e.png)

<br>

<br>

- update github webhook with ealstic ip

<br>

<br>

![elastic_ip_jenkins_webhook](https://user-images.githubusercontent.com/92983658/187193874-d4c15a20-3e9b-42c8-86c7-65f73f872764.png)

<br>

<br>

## Prepare Development Environment

- Create environment in `AWS could9`: 
  - From AWS Management Console, inside the Developer Tools choose `Cloud9`.
  - Press Create environment. Add any name for that environment then press `next`. 
  - In the configure settings, select `Create a new EC2 instance for environment (direct access)` and leave other options as selected.
  - click `next step` and review settings
  - click `create environment`
  - `Cloud9` is now ready to be used as a local machine
  
  <br>
  
  ![cloud9](https://user-images.githubusercontent.com/92983658/187868164-c2985cd9-2dfc-499e-83d7-cfb0012677ef.png)

<br>

  
### Configure `cloud9` to connect to the newly created GitHub repository.

- in bash shell check if git is installed: `which git`
- Create user config for git:
```

git config --global user.name "First Last"
git config --global user.email "somebody@somewhere.net"

```

- Create `SSH key`: `ssh-keygen -t ed25519 -C "darey"`
  -  specify the path for the key pair: copy and paste the recommended path by AWS
  -  select a passphrase
  -  press `enter` and create key pair

<br>

<br>

![keypair](https://user-images.githubusercontent.com/92983658/187872205-440ba6e5-00da-4979-97b9-49606999217a.png)

<br>

<br>

- in github at the top right corner of the page, click on the profile picture and select `settings`
- select `SSH and GPG keys`
  - under `SSH keys` click `new SSH key`
  - select any title
  - select authentication under "key type"
  - paste in public key
   - in cloud9 copy created public key: `cat ~/.ssh/id_ed25519.pub`
   - paste in github

<br>

<br>

![add_ssh_key](https://user-images.githubusercontent.com/92983658/187874384-18d2bdb3-918f-4202-baad-662373040dde.png)

<br>

<br>

- ssh into jenkins-ansible` from cloud9 environment

<br>


```

# upload privatekey to local environment in cloud9 first.

chmod 400 privatekey.pem
ssh -i "privatekey.pem" jenkins-ansible.IP


```

<br>

- Clone down your `ansible-config-mgt` repository to your Jenkins-Ansible instance: `git clone ansible-config-mgt url`

<br>

<br>

![ansible_config_clone](https://user-images.githubusercontent.com/92983658/187921544-4214f053-3184-4936-ab37-d235c07a0c99.png)


<br>

<br>

## BEGIN ANSIBLE DEVELOPMENT

- In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

<br>

<br>

![prj_11](https://user-images.githubusercontent.com/92983658/187923223-eaa2e0ce-8f05-44e1-afbc-be729205fb3e.png)

<br>

<br>

- Checkout the newly created feature branch to your local machine and start building your code and directory structure
```
cd ansible-congif-mgt
git checkout -b PRJ-11

```
- Create a directory and name it `playbooks` – it will be used to store all your playbook files: `mkdir playbooks`
- Create a directory and name it `inventory` – it will be used to keep your hosts organised.: `mkdir inventory`
- Within the playbooks folder, create your first playbook, and name it common.yml
```
cd playbooks
touch common.yml

```

<br>

<br>

- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.
```

cd ..
cd inventory
touch dev.yml
touch staging.yml
touch uat.yml
touch prod.yml

```
<br>

<br>

### Set up Ansible Inventory

- exit `Jenkins-Ansible` server
- **Set up an `SSH agent` and connect to `Jenkins-Ansible` server:**
  - on local machine: 

<br>

```

eval `ssh-agent -s`
ssh-add ./<path-to-private-key>

```

<br>

<br

- Confirm the key has been added : `ssh-add -l`
- ssh into `Jenkins-Ansible` server using ssh-agent: `ssh -A -i "private ec2 key" ubuntu@public-ip`

<br>

<br>

- Update inventory/dev.yml file with this snippet of code:
 *notice, that Load Balancer and database user is ubuntu and user for RHEL-based servers is ec2-user.*

<br>

<br>

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
<br>

<br>

![dev_yml](https://user-images.githubusercontent.com/92983658/187440923-c9b1d29a-4b2f-4e4d-813b-7477669cc509.png)

<br>

<br>

## Create a playbook

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
 
 <br>
 
 ![common_yml](https://user-images.githubusercontent.com/92983658/187443293-6b5840c0-cea1-4c81-b34d-c5b16808d878.png)


<br>

### Commit Code To Github

- use git commands to add, commit and push your branch to GitHub.
```

git status

git add <selected files>

git commit -m "commit message"

git push origin <branch name>

```

<br>

<br>

![git_status_commit](https://user-images.githubusercontent.com/92983658/187445141-75ccddad-f527-41cd-a8f2-682d1197bcbf.png)

<br>

<br>

- Create a Pull request (PR) for branch

<br>

<br>

![compreand pull](https://user-images.githubusercontent.com/92983658/187450096-05082257-7476-4121-a286-103ee5ae0096.png)

<br>

<br>

![mergepullrequest](https://user-images.githubusercontent.com/92983658/187450122-cdc7ef2d-df20-42b0-858d-372d43df9fe6.png)

<br>

<br>

![successful_merge](https://user-images.githubusercontent.com/92983658/187450503-3004fdbf-30d0-4b09-9727-01a440045f97.png)

<br>

<br>

- Head back to terminal, checkout from the feature branch into the master, and pull down the latest changes:
```
git checkout main
git pull

```
<br>

<br>

- Once code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on `Jenkins-Ansible server.`

<br>

<br>

![jenkins_confirm](https://user-images.githubusercontent.com/92983658/187456019-fa40193b-f2a2-4683-b888-14ba9b60d03e.png)

<br>

<br>

![var_confirm](https://user-images.githubusercontent.com/92983658/187456104-326931f2-1ea3-4d36-b7e0-0e7086796bb6.png)

<br>

<br>


## Run First Ansible Test

- verify if `common` playbook works:
```

cd ansible-config-mgt
ansible-playbook -i inventory/dev.yml playbooks/common.yml

```
<br>

<br>

![ansible](https://user-images.githubusercontent.com/92983658/187943833-1ea86d39-f1ea-4388-8eea-59923bc12286.png)

<br>

<br>

- go to each of the servers and check if wireshark has been installed:  `which wireshark` or `wireshark --version`

<br>

<br>

![wireshark_nfs](https://user-images.githubusercontent.com/92983658/187943890-e7066e7c-2b81-4057-b222-efaeea6fc0f3.png)

<br>

<br>







