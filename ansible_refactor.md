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


