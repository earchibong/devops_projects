# How to deploy and configure UAT Web Servers using Ansible imports and roles.

<br>

![project12_architecture](https://user-images.githubusercontent.com/92983658/188095299-218cd955-2dda-4436-aac2-083e095f90a3.png)

<br>

## Code Refactoring:

### Jenkins Job Enhancement

- Go to the `Jenkins-Ansible` server and create a new directory called `ansible-config-artifact`:
`sudo mkdir /home/ubuntu/ansible-config-artifact`

- Change permissions to this directory, so Jenkins could save files there: `sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact`
- In `Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab`
  - search for `Copy Artifact` and `install without restarting Jenkins`

<br>

![copy_artifacts](https://user-images.githubusercontent.com/92983658/188102085-8a39e8e7-2bde-40cb-b85f-71e839761669.png)

<br>

- Create a new Freestyle project and name it `save_artifacts`.
- Configure `save artifacts` to be triggered by completion of the existing `ansible` project.
  - click `configure` under the the project dashboard
  - under `general` tab, click `discard old builds`
   - under `max # of builds to keep` : `2`
  - under `source code management` tab: click none
  - under `build triggers`: click `build after other projects are built`
   -  enter `ansible` for `projects to watch`
   -  `trigger only if build is stable`    

<br>

![general_configure](https://user-images.githubusercontent.com/92983658/188104450-09dd09ff-906b-4f45-be79-e4f31aef535d.png)

<br>

![source_build_configure](https://user-images.githubusercontent.com/92983658/188104929-0da2a52d-ab4d-40ef-8701-2fa6dfcdeb8b.png)

<br>

- create a `Build` step and choose `Copy artifacts from other project` :
  - specify `ansible` as a source project
  - `**` for artifacts to copy
  - `/home/ubuntu/ansible-config-artifact` as a target directory.

<br>

![build_step](https://user-images.githubusercontent.com/92983658/188105979-2dd9a0d7-d78a-48bd-979e-c3a657398605.png)

<br>

- Test set up by making some change in `README.MD` file inside `ansible-config-mgt` repository (right inside master branch).
*If both Jenkins jobs have completed one after another – you shall see your files inside `/home/ubuntu/ansible-config-artifact`
directory and it will be updated with every commit to your master branch.*

<br>

![jenkins_confim](https://user-images.githubusercontent.com/92983658/188107069-8708ea42-38e4-4e25-8776-d44d7218627c.png)

<br>

![ansible_dir_confirm](https://user-images.githubusercontent.com/92983658/188107099-b3f2f81d-4bf7-42fd-a191-ed3a2299abce.png)

<br>

### Refactor Ansible code by importing other playbooks into site.yml

- pull down the latest code from master (main) branch of `ansible-config-mgt`, and create a new branch named `refactor`.
```

cd ansible-config-mgt   
git pull
git checkout -b refactor

```

- In the `playbooks` folder, create a new file and name it `site.yml`:
```

cd playbooks
touch site.yml

```
*This file will now be considered as an entry point into the entire infrastructure configuration. 
Other playbooks will be included here as a reference. 
In other words, site.yml will become a parent to all other playbooks that will be developed.*

- Create a new folder in root of the repository and name it `static-assignments`:
```

cd ..
mkdir static-assignments

```
- Move `common.yml` file into the newly created `static-assignments` folder:
```

cd ansible-config-mgt
mv ./playbooks/common.yml static-assignments

```

- Inside site.yml file, import common.yml playbook: `nano ./playbooks/site.yml`
```

---
- hosts: all
- import_playbook: ../static-assignments/common.yml

```

<br>

![import_site_yml](https://user-images.githubusercontent.com/92983658/188113640-1fbe7b16-1456-436b-9ee3-4c367be4e373.png)

<br>

- **Run `ansible-playbook` command against the `dev environment`:**
  - create another playbook under `static-assignments` and name it `common-del.yml`: `touch ./static-assignments/common-del.yml` 
  - configure deletion of wireshark utility in `common-del.yml` : `nano ./static-assignments/common-del.yml`
```

---
- name: update web, nfs servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB & DB server
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
      
 ```
 
 <br>
 
 ![common_del](https://user-images.githubusercontent.com/92983658/188121949-a0f8ae5a-741f-48b7-b03d-288ba0ae5005.png)

<br>

- update `site.yml`: `nano ./playbooks/site.yml`
```

---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml

```

<br>

![site_yml_update](https://user-images.githubusercontent.com/92983658/188116512-f677a363-b290-4f5d-9907-ee506e3c12ec.png)

<br>

- run playbook against `dev` servers:
```

cd /home/ubuntu/ansible-config-mgt/

ansible-playbook -i inventory/dev.yml playbooks/site.yml

```

<br>

![ansible_delete_wireshark](https://user-images.githubusercontent.com/92983658/188122455-966770b3-9d46-4383-92e2-75afa5c8a53c.png)

<br>

### Configure UAT Webservers with a role `webserver`

- Launch 2 fresh EC2 instances using RHEL 8 image, 
- instances will used them as `uat servers` named : `Web1-UAT` and `Web2-UAT`.
- In `Jenkins-Ansible` server, create a role:
  -  create a directory called `roles/`, relative to the playbook file or in `/etc/ansible/ directory`
```

cd ansible-config-mgt
mkdir roles
cd roles
ansible-galaxy init webserver
cd webserver
rm -R tests
rm -R vars
rm -R files

```

- Update the inventory `ansible-config-mgt/inventory/uat.yml` file with IP addresses of the 2 `UAT Web servers`:
  - `cd ../..`
  - `nano ./inventory/uat.yml`

```

[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

```

<br>

![uat](https://user-images.githubusercontent.com/92983658/188127172-3e6a1e88-5b7a-4346-b9bc-3b2fcafcaed9.png)

<br>

*NOTE: Ensure you are using ssh-agent to ssh into the Jenkins-Ansible instance*

- In `/etc/ansible/ansible.cfg` file:
  -  uncomment `roles_path` string :  `sudo vi /etc/ansible/ansible.cfg`
  -  provide a full path to your roles directory `roles_path` = `/home/ubuntu/ansible-config-mgt/roles`

<br>

![roles_path](https://user-images.githubusercontent.com/92983658/188128924-c5692ec3-79ed-4253-b58e-4b380b603a19.png)

<br>

- in `tasks` directory, and within the main.yml file, start writing configuration tasks to do the following:
  - Install and configure `Apache` (httpd service)
  - Clone Tooling website from GitHub `https://github.com/<your-name>/tooling.git`
  - Ensure the tooling website code is deployed to `/var/www/html` on each of 2 UAT Web servers.
  - Make sure `httpd` service is started

```

cd roles/webserver/tasks
nano main.yml

```

<br>

```

---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
    
 ```
 
 <br>
 
 ![tasks](https://user-images.githubusercontent.com/92983658/188130571-008d46ea-cc35-4581-bdef-aa3770c02189.png)

<br>

### Reference `Webserver` Role

- In the `static-assignments` folder, create a new assignment for `uat-webservers` namd  `uat-webservers.yml`:
```

cd ../../..
cd static-assignments
touch uat-webservers.yml
nano uat-webservers.yml

```

<br>

```

---
- hosts: uat-webservers
  roles:
     - webserver

```

<br>

![uat-webser_yml](https://user-images.githubusercontent.com/92983658/188133008-c90425f5-7fc0-460c-90a1-025f1e3fcc6e.png)

<br>

- update `site.yml` -  refer `uat-webservers.yml` role inside `site.yml` : `nano ./playbooks/site.yml`
```

---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml


```

<br>

![site_yml_update](https://user-images.githubusercontent.com/92983658/188136239-ab217bf5-e65f-4756-b586-8fd75e562bd1.png)

<br>

### Commit & Test

- Commit your changes, create a Pull Request and merge them to master branch
*make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.*

```

git status
git add
git commit -m "update message here"

```

- merge pull request on `github`
- Head back to terminal, checkout from the feature branch into the master, and pull down the latest changes:

```

git checkout main
git pull

```

<br>

![jenkins_cofirm](https://user-images.githubusercontent.com/92983658/188148708-25f2341f-6e29-4a87-9363-586015f50824.png)

<br>

![var_confirm](https://user-images.githubusercontent.com/92983658/188149685-a9777bfa-e2f3-471b-8e72-7a9d128e5431.png)

<br>

- run the playbook against `uat inventory` :
```
cd ansible-config-mgt
ansible-playbook -i ./inventory/uat.yml ./playbooks/site.yml
