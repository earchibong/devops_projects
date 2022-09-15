# Ansible Dynamic Assignments (Include) And Community Roles

**Pre-requisites:**
- Complete Previous Ansible Projects <a href="https://github.com/earchibong/devops_training/README.me">here</a>

<br>

- In the GitHub repository start a new branch and call it `dynamic-assignments` : `git checkout -b dynamic-assignments`
- in `ansible-config-mgt` create a `dynamic-assignments` directory and create a file named `env-vars.yml` inside it.
- add the following to `dynamic-assignments/env-var.yml`:

<br>
 
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

- in `ansible-config-mgt` create another folder named `env-vars`
  - then create uat.yml` file  in `env-vars` that will be used to set variables
- file tree should look like this:

```

├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── uat.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
├── playbooks
    └── site.yml
└── static-assignments
    └── common.yml
    └── uat-webservers.yml
    
 ```
 
 
<br>

## Update Site.yml with Dynamic Assignment

- Update `site.yml` file:

```

---
  - hosts: all
  - name: include dynamic variables
    import_playbook: ../static-assignments/common.yml
    include: ../dynamic-assignments/env-vars.yml
    tags:
      - always
  
  - name: import webservers file
    import_playbook: ../static-assignments/uat-webservers.yml
    
  
```

<br>

![site1](https://user-images.githubusercontent.com/92983658/190400429-a190686e-6075-48ed-8a6b-7895b57009b4.png)

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


### Configure Apache
- run this command to install `posix.boolean` in `roles` directory: `ansible-galaxy collection install ansible.posix`
- in roles/apache directory, open README.md and `defaults/main.yml` files: use `README.md` to configure
- in `defaults/main.yml` add in webserver configuration: 

```

# Webservers
loadbalancer_name: "myapp1"
web1: "<your UAT webserver1 ip>"
web2: "<your UAT webserver2 ip>"

```

<br>

![main2](https://user-images.githubusercontent.com/92983658/189946446-b208a4a9-2b24-4136-a321-9e6aa0baa51d.png)


<br>

- add `loadbalancer_name: "myapp1"` to `apache_vhosts` in `defaults/main.yml`

<br>

![apache_vhosts](https://user-images.githubusercontent.com/92983658/189076784-78024f54-fd3c-4ba9-acf9-46e3f9b54328.png)

<br>

- in `apache/tasks` directory, open `setup-READHAT.yml`: add `become: true`
- add the following:

```

- name: Set httpd_can_network_connect flag on and keep it persistent across reboots
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


### Update assignment and site.yml files

- in `static-assignment` folder create new files: `db.yml` and `lb.yml`
- in `loadbalancer.yml` add the following:
```

- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
    
 ```
 <br>
 
 ![lb_yml](https://user-images.githubusercontent.com/92983658/190103726-e5ec83cd-0d23-459a-90a1-cf4b90c2c167.png)

 
<br>

- on `db.yml` add the following:
```

---
  - hosts: db
    roles:
      - mysql
    become: true
    
```

<br>

![db_yml](https://user-images.githubusercontent.com/92983658/190104497-51632e9e-02a0-4490-9071-728989887196.png)

<br>

- `site.yml` add the following:
```
    
  - name: import Loadbalancers assignment
    import_playbook: ../static-assignments/lb.yml
    when: load_balancer_is_required 
   
 ```
 
 <br>
 
 ![site2](https://user-images.githubusercontent.com/92983658/190402022-a1d79368-11a3-4038-a5bb-364ab2e7d2e0.png)


<br>

### Activate Load Balancer

- in `ansible-config-mgt` create an `env-vars` directory and within it, create a `uat.yml` file
  - in the `env-vars/uat.yml` file add the following:

```

# enable_apache_lb: true
enable_nginx_lb: true
load_balancer_is_required: true

```

*The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.*

<br>

![activate_lb](https://user-images.githubusercontent.com/92983658/190108357-062d37ad-adc7-4904-8a35-b1d6739f9fae.png)

<br>

- in `inventory/uat.yml` add the following:
```

[lb]
<UAT-WEB1 private ip> ansible_ssh_user='ec2-user' 

<UAT-WEB2 private ip> ansible_ssh_user='ec2-user'

```

<br>

- commit and push changes to git hub
```
git add .
git commit -m "message"
git push -u origin main

```

<br> 

### Test Environment

- update inventory for each environment and run ansible:
  - run playbook: `ansible-playbook -i inventory/uat.yml playbooks/site.yml`

<br>

![ansible-playbook](https://user-images.githubusercontent.com/92983658/189924552-fc2032b0-0f8a-4b96-9523-944e8a0678a1.png)

<br>





