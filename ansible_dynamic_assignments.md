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

- repeat the same steps as with MSQL
- Inside `roles` directory create new Load Balancer role with `ansible-galaxy install geerlingguy.<nginx or apache>` and rename folder to `<nginx or apache>`

<br>

```
cd ansible-config-mgt
cd roles
ansible-galaxy install geerlingguy.nginx
ansible-galaxy install geerlingguy.apache
mv geerlingguy.nginx/ nginx
mv geerlingguy.apache/ apache

```

<br>

### Configure Nginx

- in nginx directory, read README.md file : `cd nginx` -> `cat README.md`
- edit roles configuration to use correct credentials for Nginx required for the `tooling website`
  - in `nginx/tasks` directory, open `main.yml`
   - under `Setup/install tasks`: delete `or ansible_os_family == 'Rocky' 
  
<br>

![rocky](https://user-images.githubusercontent.com/92983658/189299335-3a196628-4d26-4aef-8a9a-9b923545ef67.png)

<br>

  - under `vhost configuration` add the following:

```

- name: set webservers host name in /etc/hosts
  become: yes
  blockinfile: 
    path: /etc/hosts
    block: |
      {{ item.ip }} {{ item.name }}
  loop:
    - { name: web1, ip: <your UAT1 IP> }
    - { name: web2, ip: <your UAT2 ip> }
    
  ```
  
  <br>
  
  ![vhost_setup](https://user-images.githubusercontent.com/92983658/189307165-3a4cb1a1-97cd-4f7d-a0e9-be49b06cdfc2.png)
  
  <br>

  - under `nginx setup`: add `become: true`
  - name: Ensure nginx service is running as configured: add `become: true`

<br>

![nginx-setup](https://user-images.githubusercontent.com/92983658/188909397-e47bf445-a17d-40ae-8151-6099db1cb16d.png)

<br>

- in `tasks/setup-RedHat.yml`: add `become: yes`

<br>

![setup_redhat_become](https://user-images.githubusercontent.com/92983658/189301404-e3495119-7a4d-43e1-bd3b-5e3885e0e118.png)

<br>

- in `roles/nginx` directory, open up `defaults/main.yml
  - under `nginx upstreams`: uncomment all lines under each of this section 
  - under `nginx_extra_http_options`: uncomment all lines
  - under `servers`: add the following...

```

   servers:
     - "web1 weight=3"
     - "web2 weight=3"
     - "proxy_pass http://myapp1"
     
 ```

<br>

![nginx_upstreams_servers](https://user-images.githubusercontent.com/92983658/189308243-e41a1d50-d4f7-4f65-872f-99e57a6a11f6.png)

<br>

- commit and push changes to git hub
```
git add .
git commit -m "message"
git push -u origin main

```

<br>

### Configure Apache
- in roles/apache directory, open README.md and `defaults/main.yml` files: use `README.md` to configure
- in `defaults/main.yml` add in webserver configuration: REMOVE THIS!!! 

```

# Webservers
loadbalancer_name:"myapp1"
web1:"<your UAT webserver1 ip>"
web2:"<your UAT webserver2 ip>"

```

<br>

![loadbalancer_roles_config](https://user-images.githubusercontent.com/92983658/189075858-0897176a-b31a-46c0-b580-a30206dfca0c.png)

<br>

- add `loadbalancer_name:"myapp1"` to `apache_vhosts` in `defaults/main.yml`

<br>

![apache_vhosts](https://user-images.githubusercontent.com/92983658/189076784-78024f54-fd3c-4ba9-acf9-46e3f9b54328.png)

<br>

- in `apache/tasks` directory, open `setup-READHAT.yml`: add `become: true`
- add the following:

```
- name: set httpd_can_network_connect flag on and keep it persistent across reboots
  become: yes
  ansible.posix.seboolean:
      name: httpd_can_network_connect
      state: yes
      persistent: yes
      
```

<br>

![setup_redhat](https://user-images.githubusercontent.com/92983658/189077068-afc60326-0553-4d11-a702-8c2ccc4438b5.png)

<br>

### Declare Variables

- Declare a variable in `defaults/main.yml` file inside the Nginx and Apache roles. Name each variables `enable_nginx_lb` and `enable_apache_lb` respectively.
- Set both values to false: 
  - `enable_nginx_lb: false`
  -  `enable_apache_lb: false`

- Declare another variable in both roles `load_balancer_is_required` and set its value to `false` as well

<br>

![apache_lb_false](https://user-images.githubusercontent.com/92983658/189856651-09794d76-99eb-4c6e-b177-18ad8fcda13c.png)

<br>

![nginx_lb_false](https://user-images.githubusercontent.com/92983658/189856696-97b0476a-84cc-4cbe-b7c1-942d89f79738.png)

<br>

- in `ansible-config-mgt` create a `dynamic-assignments` directory and add the following:
 ```
 
 ---

 - name: collate variables from env specific file, if it exists
   hosts: all
   tasks:
      - name: looping through list of available files
        include_vars: "{{ item }}"
        with_first_found:
          - files:
              - dev.yml
              - ci.yml
              - pentest.yml
              - uat.yml
            
            paths:
              - "{{ playbook_dir }}/../env-vars"

        tags:
          - always
          
 ```
 
<br>

![dynamic-assignment](https://user-images.githubusercontent.com/92983658/189863514-2461d68b-e215-4035-8bbc-af7da73ef7e0.png)

<br>

- update `site.yml` and run playbook: `ansible-playbook -i inventory/uat.yml playbooks/site.yml`

