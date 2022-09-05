# Ansible Dynamic Assignments (Include) And Community Roles

**Pre-requisites:**
- Complete Previous Ansible Projects <a href="https://github.com/earchibong/devops_training/README.me">here</a>

## Update Site.yml with Dynamic Assignment

- Update `site.yml` file:

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

![site_yml](https://user-images.githubusercontent.com/92983658/188451867-3fcd5c32-3674-430e-9f86-5f3b75a1bb5c.png)


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
  - in `roles/mysql` directory, open up `defaults/main.yml
  - under `databases` and `users`
   - uncomment all lines under each of these sections 
   - change name of database to `tooling website DB name`: `tooling` 
   - change name of users: `webaccess`
   - change host to: `0.0.0.0`
   - change priv to: `'*.*:ALL,GRANT'`
 
 <br>
 
 ![mysql_config](https://user-images.githubusercontent.com/92983658/188450301-107e55de-e0e0-4d0e-9010-3d2366ac2aec.png)

<br>

- upload changes to github

<br>

```

git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature

```
<br>

- create a Pull Request and merge it to main branch on GitHub.
- on terminal `git chekout main` -> `git pull`

<br>

## Load Balancer Roles

