# Website Deployment Automation With Continuous Integration

**Task: Configure a job to automatically deploy source code changes from Git to NFS server.**

<br>

![jenins_architecture](https://user-images.githubusercontent.com/92983658/185145169-c767dad0-02e8-4033-b09f-d573b279049b.png)

<br>

<br>

## Pre-requisites
- NFS server
- Database
- Webservers

<br>

<br>

## Project Steps
- <a href="https://github.com/earchibong/devops_projects/blob/main/CI.md#install-and-configure-jenkins-server">Install And Configure Jenkins Server</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/CI.md#configure-jenkins-to-retrieve-source-codes-from-github-using-webhooks">Configure Jenkins To Retrieve Source Codes From Github Using Webhooks</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/CI.md#configure-jenkins-to-copy-file-to-nfs-via-ssh">Configure Jenkins To copy File To NFS Via SSH</a>

<br>

<br>

## Install And Configure Jenkins Server
### Step One: install Jenkins Server
- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
- Install JDK:
```

sudo apt update
sudo apt install default-jdk-headless

```

<br>

<br>

- install Jenkins
```

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

```

<br>

<br>

- confirm Jenkins status: `sudo systemctl status jenkins`

<br>

<br>

![jenkins](https://user-images.githubusercontent.com/92983658/185379910-3c24b4e5-2ca0-4e31-91f3-f49f4ba37887.png)

<br>

<br>

- open `TCP port 8080` by creating an inbound rule in EC2 security group

<br>

<br>

![security_jenkins](https://user-images.githubusercontent.com/92983658/185380649-d47dc6c4-cdd6-48ff-9236-1288a0e26385.png)

<br>

<br>

- ### Perform Jenkins Initial Setup
- From the browser access: `http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`
  - retrieve defaul admin password from server: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
  - install `suggested plugins`
  - create `admin user` once plugin installation is complete
  - get jenkins server address

<br>

<br>

![jenkins_server](https://user-images.githubusercontent.com/92983658/185386534-6cc7297b-73f4-444c-abe7-2fce0c8c98f9.png)

<br>

<br>

![jenkins_ready](https://user-images.githubusercontent.com/92983658/185386683-27e1c008-9b35-449c-8566-b27ce7a20296.png)

<br>

<br>

### Configure Jenkins To Retrieve Source Codes From Github Using Webhooks

- enable `webhooks` in github repository settings: `github repository -> settings -> webhooks`

<br>

<br>

![webhook](https://user-images.githubusercontent.com/92983658/185939093-8560742f-6162-4c5a-8b44-91eba9bcdd2b.png)

<br>

<br>

- on Jenkins Console, click "New Item" and create a "Freestyle Project" called "tooling-github"
    - connect to github repository: get the repository URL

  <br>

  <br>
  
![https](https://user-images.githubusercontent.com/92983658/185390017-96390495-f6f4-47fe-bd73-3494d06cf277.png)

<br>

<br>

 - in jenkins, under the `general`tab, in the `source code management` section
 - input `github url`
 - under `credentials`, click `add` and select `username and password option`
 - input github username and password -> save
 - under `credentials` selct new added user credentials
 - save configutation
 - check configuration: click `build now` button
 - if configuration is correct, then `build` will be successful and will appear under `#1` at the bottom left of the dashboard
   
 <br>

 <br>
 
 ![build_now](https://user-images.githubusercontent.com/92983658/185391629-826a2cba-39be-49d9-9e30-f0ae65450f2d.png)

<br>

<br>

    - open `build` and check in `console output` to confirm successful run
 
 <br>

 <br>
 
![console_output](https://user-images.githubusercontent.com/92983658/185391965-d8175c4a-ee6a-4728-92c6-805f748999e6.png)

<br>

<br>

- add the following configurations to the project:
    - go back to `project` dashboard
    - click `configure`
    - under the `build triggers` tab:
     - Configure triggering the job from GitHub webhook: click `github hook trigger for GITScm polling`
    - Under the `post-build` tab: 
     - Select "Archive the artifacts" to archive all the files:
     - use `**` under `files to archive`
     - save configuration 
 
 - in github repository, make changes to any file to test configurations in Jenkins.
    - update the `README.md` and push the changes to the master branch
    - head over to `jenkins` and the changes should have updated automatically.

<br>

<br>

 ![changes_file](https://user-images.githubusercontent.com/92983658/185940025-d03740d7-56f9-45b3-bfbc-070c5ae7ada4.png)

<br>

![console_output_1](https://user-images.githubusercontent.com/92983658/185940041-15fa6605-4c01-4ad9-9a99-a524867601d2.png)

 <br>

 <br>
 
 
## Configure Jenkins To copy File To NFS Via SSH

- Install "Publish Over SSH" plugin:
  - From main dashboard, select "Manage Jenkins" and choose "Manage Plugins" menu item.
  - On "Available" tab search for "Publish Over SSH" plugin and `install without restart`

- Configure the job/project to copy artifacts over to NFS server
  - On main dashboard select `Manage Jenkins` and choose `Configure System` menu item.
  - Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:
   - Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty):
<br>

   ```
   Get the private key with the following steps:
    - on terminal, navigate to where EC2-private key is stored on system
    - access private key: cat "EC2 private key.pem"
   
   ```

  <br>

  <br>
  
  - on jenkins `SSH plugin configuration section` click `add ssh server`:
   - Name: can be any arbitrary name
   - Hostname: can be `private IP address of your NFS server`
   - Username: `ec2-user` (since NFS server is based on EC2 with RHEL 8)
   - Remote directory – `/mnt/apps` since our Web Servers use it as a mointing point to retrieve files from the NFS server
   
  - Test the configuration and make sure the connection returns Success.

<br>

<br>

![private_key](https://user-images.githubusercontent.com/92983658/185879366-b84ac88f-f6c9-4f9a-a133-8f11ea9f6416.png)

<br>

<br>

![ssh_config](https://user-images.githubusercontent.com/92983658/185879391-3e71a4a5-30e5-4efd-8e27-62cc1f34cb6e.png)

<br>

<br>

- head over to the project configuration -> add another "Post-build Action" -> `send build artifacts over SSH`
    - source files: `**`
    - remote directory: `/mnt/apps`
 
- ensure that file ownsership on NFS server is `nobody` : `sudo chown -R nobody:nobody /mnt/apps`
- change file permissions on NFS server: `sudo chmod -R 777 /mnt/apps`

- head over to github and edit the `README.md` file
- Webhook should trigger a new job in Jenkins and in the "Console Output" of the job you will find something like this:
```

SSH: Transferred 25 file(s)
Finished: SUCCESS

```

<br>

<br>

![console](https://user-images.githubusercontent.com/92983658/186126836-df0b96d9-cd5d-4157-89d3-77af7714b433.png)

<br>

<br>

- make sure that the files in /mnt/apps have been udated:
 - connect via SSH/Putty to your NFS server
 - check README.MD file : `cat /mnt/apps/README.md`
 - If you see the changes you had previously made in your GitHub – the job works as expected.
 
 
<br>

<br>
