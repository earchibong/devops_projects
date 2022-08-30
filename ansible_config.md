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

- ### Configure Jenkins build job to save repository content every time it is changed
  - Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.

![git_](https://user-images.githubusercontent.com/92983658/187188044-04d76543-c349-494f-9985-1c0091037423.png)

![jenkins_ansible](https://user-images.githubusercontent.com/92983658/187188056-c9e5ea6d-2911-4e32-9627-54f5ad29af1c.png)


  - Configure Webhook in GitHub and set webhook to trigger ansible build: `ansible-config-mgt` -> settings -> webhooks 
  
 ![git_webhook](https://user-images.githubusercontent.com/92983658/187188260-9e17d485-f287-47cb-9b58-a8db2b4da203.png)


  - Configure build trigger to `GitHub hook trigger for GITScm polling` 
  - Configure a Post-build job to save all (**) files
  - check configuration: click build now button:
    - if configuration is correct, then build will be successful and will appear under #1 at the bottom left of the dashboard
    
  ![build_check](https://user-images.githubusercontent.com/92983658/187189829-e9e1df48-8fac-4052-898b-fd5f99065b87.png)


- Test setup by making some change in README.MD file in master / main branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder: 
  - `ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
  - `cd /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
  - `cat README.md`
  
![console_output](https://user-images.githubusercontent.com/92983658/187190646-4ee622e8-b7e2-4454-9444-85cdbbba354c.png)

![ls_jenkins_build](https://user-images.githubusercontent.com/92983658/187191297-c758647b-edf2-4f31-b851-1ab1a309369e.png)

![cat_readme](https://user-images.githubusercontent.com/92983658/187192146-06ad3eb6-4afa-4e1a-8101-64e4f6a00625.png)


- ### Allocate Elastic IP to Jenkins- Ansible Server
  - In the top search bar of AWS console, enter "Elastic IP". The search bar will return several results on services and features available.
  - Click the ELASTIC IPs - EC2 feature from the list:
   - Once on the Elastic IP addresses screen, click ALLOCATE ELASTIC IP ADDRESS on the top right of the page.
   - Enter details on allocation page and allocate elastic IP
 - Under the Actions menu of elastic ip addresses click associate elastic ip address:
  - on the asocialte elastic ip address screen, assign the elastic ip to the `jenkins-ansible` server 

![elastic-ip-jenkins](https://user-images.githubusercontent.com/92983658/187193366-eb522ce6-5fab-49cf-9370-92d4f27b963e.png)

- update github webhook with ealstic ip

![elastic_ip_jenkins_webhook](https://user-images.githubusercontent.com/92983658/187193874-d4c15a20-3e9b-42c8-86c7-65f73f872764.png)


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
  
 ![ssh_environment](https://user-images.githubusercontent.com/92983658/187398193-c8e49b4b-c1d1-4edc-a9f7-8777bc2b1a4d.png)

  - Right click on “C9 install” and copy link address.
  - Press Next and leave everything ticked.

  ![c9_install](https://user-images.githubusercontent.com/92983658/187400295-5d6b8319-9422-442c-a340-fbc36c551397.png)

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

**Note:** If the instance’s IP address changed (Happens when you stop an Instance then restart its running), you will just simply need to copy the new IPv4 address.

In Cloud9, on the top left corner, press the Cloud9 icon.

Click on “Go to Your Dashboard”. Press the desired environment and from the top right corner then choose Edit. Then Scroll down to the Host and just change the IPv4 address.

  
- Configure `cloud9` to connect to the newly created GitHub repository.
