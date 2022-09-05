# Ansible Dynamic Assignments (Include) And Community Roles

**Pre-requisites:**
- Complete Previous Ansible Projects <a href="https://github.com/earchibong/devops_training/README.me">here</a>

## Update Site.yml with Dynamic Assignment

- Update `site.yml` file:
  - ssh using `uush agent` into `Jenins-Ansible` Server
```

eval `ssh-agent -s`
ssh-add ./<path-to-private-key>
ssh -A -i "private_key.pem" ubuntu@jenkins_ansible_sever_ip
cd ansible-config-mgt
nano playbboks/site.yml
```

<br>

```

---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
  
```

<br>

![site_yml](https://user-images.githubusercontent.com/92983658/188406774-d3d6d934-e71a-4706-9cc5-46c089012081.png)

<br>

## Comunity Roles
### Download MYSQL Ansible Role
- On Jenkins-Ansible server make sure that git is installed: `which git`
- in `ansible-config-mgt` directory run the following:
```

git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature

```

<br>

- Inside `roles` directory create new MySQL role with `ansible-galaxy install geerlingguy.mysql`
- rename the folder to `mysql`: `mv geerlingguy.mysql/ mysql`

<br>

![mysql_role](https://user-images.githubusercontent.com/92983658/188410596-a85ac839-806e-4518-9ded-e2db539ff45d.png)

<br>

![mysql](https://user-images.githubusercontent.com/92983658/188410628-8272616e-819f-4180-acf1-900e5f04efe8.png)

<br>

- in mysql directory, read README.md file : `cd mysql` -> `cat README.md`
- edit roles configuration to use correct credentials for MySQL required for the `tooling website`


