# Continuous integration with jenkins | ansible | artifactory | sonarqube | php

Pre-requisite: Projects 9 - 13

<br>

## SIMULATING A TYPICAL CI/CD PIPELINE FOR A PHP BASED APPLICATION
### ANSIBLE ROLES FOR CI ENVIRONMENT
#### Configuring Ansible For Jenkins Deployment

- Navigate to Jenkins URL
- Install & Open `Blue Ocean` Jenkins Plugin:
  - on jenkins dashboard click `manage jenkins` -> `manage plugins`
  - search for `blue ocean`
  - `install without restart` 
  - head back to jenkins dashboard and click `blue ocean`
- Create a new pipeline
  - select `github` to store code
  - connect jenkins to github with an access token
  - choose repository and create pipeline
  - click `admistration` to exit blue ocean

<br>

![jenkins_new_pipeline](https://user-images.githubusercontent.com/92983658/191757065-58ef4b40-18e9-463e-86e0-d8aa42685588.png)

<br>

![github_jenkins_connect_token](https://user-images.githubusercontent.com/92983658/191757175-3bd77709-f146-48cf-af05-b4a5d3b8f501.png)

<br>

![github_acess_token](https://user-images.githubusercontent.com/92983658/191757098-ea185e88-40a7-499d-a722-0e732920a489.png)

<br>

![create_pipeline](https://user-images.githubusercontent.com/92983658/191757627-aae51b41-a387-405e-8a9d-9b4c1f81a035.png)

<br>

- `Ansible-config-mgt` create a new folder `deploy`
- inside `deploy` directory, create a file named: `Jenkinsfile`
- Add the code snippet below to `jenkinsfile`

```

pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}

```

<br>

- Go back into the Ansible pipeline in Jenkins, and select `configure`
  -  in `Build Configuration` section : specify the location of the Jenkinsfile at `deploy/Jenkinsfile`
  -  Back to the pipeline again, click on `main` and select `build now`
  *This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.*

<br>

![build_configuration](https://user-images.githubusercontent.com/92983658/191765848-e873f882-263f-4d1a-b4a0-2e89c3a8b30a.png)

<br>

![build_now](https://user-images.githubusercontent.com/92983658/191765906-f91323fd-536d-466a-8939-520b2906d291.png)

<br>

- view project build in blue ocean:
  - Click on Blue Ocean
  - Select the project
  - Click on the play button against the branch

<br>

![blue_ocen_build](https://user-images.githubusercontent.com/92983658/191767116-1c19ae4e-f128-41a7-8ca1-0df2c92f4881.png)

<br>

- Create a new git branch called `feature/jenkinspipeline-stages`: `git checkout -b feature/jenkinspipeline-stages`
- add another stage called `Test` in `deploy/Jenkinsfile`
- Paste the code snippet below and push the new changes to GitHub.

```

pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}

```

<br>

![test_stage](https://user-images.githubusercontent.com/92983658/191770828-2896fd4f-f52c-440d-8866-51d5f61dba60.png)

<br>

To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

- in Jenkins, Click on the `Administration` button
- Navigate to the Ansible project and click on `Scan repository now`
- Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.

<br>

![scan_repo](https://user-images.githubusercontent.com/92983658/191772003-cca72b0f-67b6-465c-b7e7-5bb1891cfd8d.png)

<br>

![blue_ocean_test](https://user-images.githubusercontent.com/92983658/191772500-2c7a0cad-194a-44e4-b749-ffef9c417f60.png)

<br>

- Create a pull request to merge the latest code into the main branch
- Add more stages into the Jenkins file to simulate below phases. (Just add an echo command like in build and test stages)
   1. Package 
   2. Deploy 
   3. Clean up
 
 ```
 
 pipeline {
    agent any

  stages {
    stage("Initial cleanup") {
      steps {
        dir("${WORKSPACE}") {
          deleteDir()
        }
      }
    }

    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }

    stage('Package') {
      steps {
        script {
          sh 'echo "Packaging App"'
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh 'echo "Deploying To Dev"'
        }
      }
    }

    stage('Clean Up'){
      steps{
        cleanWs()
      }
    }
  }
}

```

<br>

![jenkins_stages](https://user-images.githubusercontent.com/92983658/191984787-8a0c234f-855d-4da6-8214-769d4e5cee93.png)

<br>

- Verify in Blue Ocean that all the stages are working

<br>

![ocean_blue_stages](https://user-images.githubusercontent.com/92983658/191986509-216eb766-4efe-4e3d-bf41-2cb72ef57581.png)

<br>

- After merging the PR, go back into your terminal and switch into the main branch: `git checkout main`
- Pull the latest change: `git pull`

<br>

#### Running Ansible Playbook From Jenkins

- Install Ansible on Jenkins Server
- Install Ansible plugin in Jenkins UI: 
  - `jenkins dashboard -> manage plugins -> plugin manager - > search for ansible -> install without restart`

<br>

![ansible_jenkins](https://user-images.githubusercontent.com/92983658/192092148-995b1de3-a9ce-4874-a805-f439ef08419a.png)


<br>

- launch a redhat instance named `nginx` and an ubuntu instance named `DB`
- in `deploy` folder create an ansible configuration file named `ansible.cfg`
- add the following to `ansible.cfg`
```

[defaults]
timeout = 160
callback_whitelist = profile_tasks
log_path=~/ansible.log
host_key_checking = False
gathering = smart
ansible_python_interpreter=/usr/bin/python3
allow_world_readable_tmpfiles=true


[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ControlPath=/tmp/ansible-ssh-%h-%p-%r -o ServerAliveInterval=60 -o ServerAliveCountMax=60 -o ForwardAgent=yes

```

<br>

- **update Jenkins credentials: **
- `jenkins dashboard -> manage jenkins -> manage credentials -> global -> add credentials`
  - kind: `ssh with private key`
  - ID: `give_any_name_you_want`
  - Description: `describe the purpose`
  - username: `ec2-user` or `ubuntu` depending on what platform jenkins instance is deployed on.
  - private key: `cat private key.pem` -> copy and paste into jenkins

<br>

![credentiala](https://user-images.githubusercontent.com/92983658/193022591-6b313376-1b4c-4e84-99d0-002a0d649acf.png)
![credentialb](https://user-images.githubusercontent.com/92983658/193022611-4d49461a-2a2e-407a-8b49-d28ca800815d.png)

<br>

- configure Ansible in Jenkins
  - `jenkins dashboard` -> `manage jenkins` -> `global tool configuration`
  -  under ansible: `add ansible`
    - name: `ansible`
    - path to ansible executables: `usr/bin/`
     - in terminal run: `which ansible` to determine ansible path -> paste this in jenkins
 
 <br>
 
 ![ansible_tool](https://user-images.githubusercontent.com/92983658/193037452-ba349544-3b28-4969-8239-3905c2fa6da9.png)

<br>

- **Generate pipeline script**
- `jenkins dashboard` -> `project name` -> `pipeline syntax`
  - playbooks file path in workspace: `playbooks/site.yml`
  - inventory path in workspace: `inventory/dev.yml` 
  - ssh connection credentials: select credential created
  - click `use become`
  - click `disable host SSH key check`
  - click `colorized output`
  - generate pipeline script
  - copy script and paste in `Jenkinsfile` under `run ansible playbook` stage

- Create  Jenkinsfile from scratch that runs the ansible playbook

<br>

```

pipeline {
  agent any

  environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
    }

  stages{
      stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

      stage('Checkout SCM') {
         steps{
            git branch: '<name of git branch>', url: '<git repo url>'
         }
       }

      stage('Prepare Ansible For Execution') {
        steps {
          sh 'echo ${WORKSPACE}' 
          sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'  
        }
     }

      stage('Run Ansible playbook') {
        steps {
           ansiblePlaybook become: true, colorized: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/dev.yml', playbook: 'playbooks/site.yml'
         }
      }

      stage('Clean Workspace after build'){
        steps{
          cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
        }
      }
   }

}

```

<br>

- **update inventory file*
- open `ansible-config-mgt/inventory/dev.yml`
  - update `[nginx]`: `nginx ip ansible_ssh_user='ec2-user'`
  - update `[db]`: `db ip ansible_ssh_user='ubuntu'

<br>

![dev](https://user-images.githubusercontent.com/92983658/193047770-840aa9aa-24b8-44ae-aaa0-9890160ce6cd.png)

<br>

- **update `playbooks/site.yml`**
- open `playbooks/site.yml`:
  - add the following:
```
---

 - hosts: db
 - name: database assignment
   ansible.builtin.import_playbook: ../static-assignments/database.yml

 - hosts: nginx
 - name: nginx assignment
   ansible.builtin.import_playbook: ../static-assignments/nginx.yml
   
```

<br>


- **edit roles configuration to use correct credentials for MySQL required for the tooling website**
- in roles/mysql directory, open up `defaults/main.yml
  - under databases and users
  - uncomment all lines under each of these sections
  - change name of database to tooling website DB name: `tooling`
  - change name of users: `webaccess`
  - change host to: `0.0.0.0`
  - change priv to: '*.*:ALL,GRANT'

- **edit roles configuration for nginx**
- in roles/nginx/defaults/main: leave empty

```
---
# defaults file for nginx

```

<br>

- in `nginx/tasks/main.yml`: add the following:

```

---
# tasks file for nginx
- name: install nginx on the webserver
  ansible.builtin.yum:
      name: nginx
      state: present


- name: ensure nginx is started and enabled
  ansible.builtin.service:
     name: nginx
     state: started 
     enabled: yes

- name: install PHP
  ansible.builtin.yum:
    name:
      - php 
      - php-mysqlnd
      - php-gd 
      - php-curl
    state: present
    
 ```
 
 <br>
 
 ![nginx_role](https://user-images.githubusercontent.com/92983658/193055254-79d957c4-d84e-49b7-8065-4fec9aa95316.png)

<br>

- create `static-assignments/nginx.yml` and add the following:
```

---
- hosts: nginx
  become: true
  roles:
     - nginx
     
 ```
 
 <br>
 

upload changes to github
```

git add .
git commit -m "create new jenkinsfile"
git push 

```

<br>

- Ensure that Ansible runs against the Dev environment successfully

<br>

![jenkins-ansible-1](https://user-images.githubusercontent.com/92983658/193644654-024054d5-d6ce-4d88-948c-0b27ce11aba3.png)
![jenkins-ansible-2](https://user-images.githubusercontent.com/92983658/193644670-eb2f2442-1344-4296-8410-a3591f9b383c.png)
![jenkins-ansible-3](https://user-images.githubusercontent.com/92983658/193644680-9cd00464-7bb2-4e01-9836-14aca67fddc1.png)

<br>

#### Parameterizing Jenkinsfile For Ansible Deployment

To deploy to other environments, we will need to use parameters.
- in `ansible-config-mgt/inventory` create a new folder `sit`
- Update `sit` inventory with new servers:
```

[tooling]
<SIT-Tooling-Web-Server-Private-IP-Address>

[todo]
<SIT-Todo-Web-Server-Private-IP-Address>

[nginx]
<SIT-Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<SIT-DB-Server-Private-IP-Address>

```

<br>

- Update `Jenkinsfile` to introduce parameterization
```

pipeline {
    agent any

    parameters {
      string(name: 'inventory', defaultValue: 'dev.yml',  description: 'This is the inventory file for the environment to deploy configuration')
    }
...

```

- In the Ansible execution section, remove the hardcoded inventory/dev.yml and replace with `inventory/${inventory}`

<br>

![jenkinsfile_parameters](https://user-images.githubusercontent.com/92983658/193806934-e34095a7-4910-4e99-bb4c-0d6d85b37df0.png)

<br>

- upload chanes to github
- create a Pull Request and merge it to main branch on GitHub.
- on terminal git chekout main -> `git pull`

<br>

## CI/CD PIPELINE FOR TODO APPLICATION

- deploy a new redhat instance named `artifactory` and create security group inbound rule for port `8081` and `8082`
- in `jenkins-ansible` server update Ansible with an Artifactory role
- update `ci` inventory with artifactory instance private ip

<br>

![artifactory_ci](https://user-images.githubusercontent.com/92983658/194619045-66f9ce85-f318-4c43-a028-06d44651c7ab.png)

<br>

- create `static assignment/artifactory.yml`
- update `artifactory.yml` with the following:
```

---
- hosts: artifactory
  become: true
  roles:
    - artifactory
    
```

- update `site.yml` with the following:
```

---

 - hosts: artifactory
 - name: artifactory assignment
   ansible.builtin.import_playbook: ../static-assignments/artifactory.yml
   
```

<br>

![artifactory_site](https://user-images.githubusercontent.com/92983658/194619586-9fb0de45-1d12-4085-a5b5-7a93c08c1756.png)

<br>


### Phase 1 – Prepare Jenkins
- Fork the repository below into your GitHub account: `https://github.com/darey-devops/php-todo.git`
- On Jenkins server, install PHP, its dependencies and Composer tool
```

# ubuntu:
sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}

<br>

#redhat:
- yum module reset php -y
- yum module enable php:remi-7.4 -y
- yum install -y php  php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd    php-fpm php-json
- systemctl start php-fpm
- systemctl enable php-fpm
- curl -sS https://getcomposer.org/installer | php 
- sudo mv composer.phar /usr/bin/composer
- composer --version

```

<br>

- Install Jenkins plugins
  1. Plot plugin: jenkins dashboard -> manage jenkins -> mange plugins -> search for `plot` -> install without restart
  2. Artifactory plugin: jenkins dashboard -> manage jenkins -> mange plugins -> search for `artifactory` -> install without restart

<br>

- in Jenkins, ensure ansible runs against `CI` inventory to install `artifactory`
- ensure `artifactory is installed`: `artifactory pulic ip:8081`

<br>

![artifactory](https://user-images.githubusercontent.com/92983658/195112387-47941def-4feb-4600-8887-a08ff9f45cd8.png)

<br>

- set up `artifactory`: `username: admin` and `password: password`
  - change password
  - skip the rest
- create local repository 

<br>

![artifactory_local_repo](https://user-images.githubusercontent.com/92983658/195116810-cfcdb1bf-8c28-4606-b6b7-14e6cc89384e.png)

<br>

- In Jenkins UI configure Artifactory: jenkins dashboard -> manage jenkins -> configure systems
  - Configure JFrog server ID, URL and Credentials and test connection
   - instance ID: `Artifactory Server`
   - instance URL: `Artifactory public ip:8081`
   - deployer credentials: `username: admin` and `password: new artifactory password` 
  -  run Test Connection. 
 
 <br>
 
 ![artifactory_jenkins_1](https://user-images.githubusercontent.com/92983658/195120025-c85afe44-b542-4d52-a226-39dad8b95d37.png)

![artifactory_jenkins2](https://user-images.githubusercontent.com/92983658/195120055-91a41c0b-0807-4e8f-9291-a0189718457b.png)

<br>

### Phase 2 – Integrate Artifactory repository with Jenkins
- Create a dummy Jenkinsfile in the `php-todo` repository
- ensure mysql is installed on `php-todo`: `sudo yum install mysql`
- change bind address to `0.0.0.0`: `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
- Using Blue Ocean, create a multibranch Jenkins pipeline : `jenkins dashboard -> blue ocean -> new pipeline`
  - store code: `github`
  - organisation: select your project account
  - repository: `php-todo`
   
- On the database server, create database and user:
  - go to `ansible-config-mgt/roles/msql/defaults/main.yml`
  - under `database` and `users` create a new database and user with the following:
  
```

Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';

```

<br>

![homestead_Setup](https://user-images.githubusercontent.com/92983658/195124759-4b4370da-be61-4458-9b38-903833610f6a.png)

<br>

- upload changes to gitub and run ansible in jenkins to create new database

<br>

![jenkins_homestead](https://user-images.githubusercontent.com/92983658/195127363-99b62dec-652c-4788-ad88-5ff1aa296caf.png)

<br>

- log into database instance and confirm creation of new database `homestead`

<br>

![homestead_confirm](https://user-images.githubusercontent.com/92983658/195128893-e98df7cf-91ef-4eeb-8e03-426c06e12dc4.png)

<br>

- in `php-todo` folder Update the database connectivity requirements in the file `.env.sample`:

```
DBHOST=private ip of database server
...
#add the following
DB_CONNECTION=mysql
DB_PORT=3306

```

<br>



- update jenkinsfile with pipeline configuration
```

pipeline {
    agent any

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/darey-devops/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}

```
