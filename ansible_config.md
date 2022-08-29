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

- Configure Jenkins build job to save repository content every time it is changed
  - enable webhooks in github repository settings: `ansible-config-mgt` -> settings -> webhooks 

- Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository
  - `jenkins_piblic_ip:8080`
  - sign-in to jenkins
  -   
