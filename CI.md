# Website Deployment Automation With Continuous Integration

Task: Configure a job to automatically deploy source code changes from Git to NFS server.

![jenins_architecture](https://user-images.githubusercontent.com/92983658/185145169-c767dad0-02e8-4033-b09f-d573b279049b.png)


## Install And Configure Jenkins Server
### Step One: install Jenkins Server
- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
- Install JDK:
```

sudo apt update
sudo apt install default-jdk-headless

```
- install Jenkins
```

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins

```
- confirm Jenkins status: `sudo systemctl status jenkins`

![jenkins](https://user-images.githubusercontent.com/92983658/185379910-3c24b4e5-2ca0-4e31-91f3-f49f4ba37887.png)


- open `TCP port 8080` by creating an inbound rule in EC2 security group

![security_jenkins](https://user-images.githubusercontent.com/92983658/185380649-d47dc6c4-cdd6-48ff-9236-1288a0e26385.png)


- ### Perform Jenkins Initial Setup
- From the browser access: `http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`
  - retrieve defaul admin password from server: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
  - install `suggested plugins`
  - create `admin user` once plugin installation is complete
  - get jenkins server address

![jenkins_server](https://user-images.githubusercontent.com/92983658/185386534-6cc7297b-73f4-444c-abe7-2fce0c8c98f9.png)

![jenkins_ready](https://user-images.githubusercontent.com/92983658/185386683-27e1c008-9b35-449c-8566-b27ce7a20296.png)


### Step Two: Configure Jenkins TO retrieve Srouce Codes From Github Using Webhooks

- enable `webhooks` in github repository settings: `github repository -> settings -> webhooks`

![webhook](https://user-images.githubusercontent.com/92983658/185389015-62743475-8eb3-4bb5-8ec4-fef03b0c365e.png)

- on Jenkins Console, click `New Item` and create a `Freestyle Project` called `tooling-github`
    - connect to github repository: get the repository URL
    
![https](https://user-images.githubusercontent.com/92983658/185390017-96390495-f6f4-47fe-bd73-3494d06cf277.png)

    - in jenkins, under the `general`tab, in te `source code management` section
    - input `github url`
    - save configutation
    - check configuration: click `build now` button
    - if configuration is correct, then `build` will be successful and will appear under `#1` at the bottom left of the dashboard
    
 ![build_now](https://user-images.githubusercontent.com/92983658/185391629-826a2cba-39be-49d9-9e30-f0ae65450f2d.png)


    - open `build` and check in `console output` to confirm successful run
![console_output](https://user-images.githubusercontent.com/92983658/185391965-d8175c4a-ee6a-4728-92c6-805f748999e6.png)

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
    - create a file `TEST.md` and push the changes to the master branch
    - head over to `jenkins` and the changes should have updated automatically.
 
 
## Configure Jenkins To copy File To NFS Via SSH

- Install "Publish Over SSH" plugin:
  - From main dashboard, select "Manage Jenkins" and choose "Manage Plugins" menu item.
  - On "Available" tab search for "Publish Over SSH" plugin and `install without restart`

- Configure the job/project to copy artifacts over to NFS server
  - On main dashboard select `Manage Jenkins` and choose `Configure System` menu item.
  - Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:
   - Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty): 
    - on terminal, navigte to where EC2-private key is stored on system
    - access private key: `cat "EC2 private key.pem"`
  
  - click `add ssh server`:
   - Name: can be any arbitrary name
   - Hostname: can be `private IP address of your NFS server`
   - Username: `ec2-user` (since NFS server is based on EC2 with RHEL 8)
   - Remote directory – `/mnt/apps` since our Web Servers use it as a mointing point to retrieve files from the NFS server
   
  - Test the configuration and make sure the connection returns Success.


![private_key](https://user-images.githubusercontent.com/92983658/185879366-b84ac88f-f6c9-4f9a-a133-8f11ea9f6416.png)

![ssh_config](https://user-images.githubusercontent.com/92983658/185879391-3e71a4a5-30e5-4efd-8e27-62cc1f34cb6e.png)
