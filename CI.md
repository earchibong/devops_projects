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
  - install suggested plugins`

