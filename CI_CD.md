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

- update Jenkins credentials: `jenkins dashboard -> manage jenkins -> manage credentials -> global -> add credentials`
  - kind: `ssh with private key`
  - ID: `give_any_name_you_want`
  - Description: `describe the purpose`
  - username: `ec2-user` or `ubuntu` depending on what instance being used
  - private key: `cat private key.pem` -> copy and paste into jenkins

<br>

![credentials_2](https://user-images.githubusercontent.com/92983658/192094116-7776be56-7edd-4700-bf11-08319e032168.png)
![credentials1](https://user-images.githubusercontent.com/92983658/192094097-6d094c3c-34f1-4c51-a556-d2c8ff14e1d8.png)

<br>

- Create  Jenkinsfile from scratch that runs the ansible playbook

<br>

```

pipeline {
  agent any

  environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
    }

  parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
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
           ansiblePlaybook become: true, colorized: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/${inventory}', playbook: 'playbooks/site.yml'
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

