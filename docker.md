# MIGRATION TO THE CLOUD WITH CONTAINERIZATION - DOCKER

In this project, the frontend and the backend(MySQL) of tooling application is built and containerized using DOCKER of which its image 
is pushed to Docker registry. And further in the project, the php-todo application is also built into a container and pushed into the 
AWS Elastic Container Registry using a CI/CD tool known as Jenkins and Docker Compose is also implemented.

## Part One: Install Docker and prepare for migration to the Cloud

Docker allocates not the whole guest OS for an application, but only isolates a minimal part of it – this isolated container has all that the application needs and at the same time is lighter, faster, and can be shipped as a Docker image to multiple physical or virtual environments, as long as this environment can run Docker engine. This approach also solves the environment incompatibility issue. If the application is shipped as a container, it has its own environment isolated from the rest of the world, and it will always work the same way on any server that has Docker engine.

- find out how to install `docker` <a href="https://docs.docker.com/desktop/install/mac-install/">here</a>

<br>

## Part Two: Migrate the Tooling Web Application from a VM-based solution into a containerized one.
### Create MySQL Container For Tooling App Backend
Assembly of the application will start from the Database layer – a pre-built MySQL database container will be used. It will be configured and made sure to receive requests from the PHP application.

<br>

#### Step 1: Pull MySQL Docker Image from Docker Hub Registry
- in terminal, type: `docker pull mysql/mysql-server:latest`

<br>

<img width="772" alt="mysql_docker_pull" src="https://user-images.githubusercontent.com/92983658/215411202-723fa862-9053-4996-ba6d-c19724a6a838.png">

<br>

#### Step 2: Deploy the MySQL Container to your Docker Engine

- Once you have the image, deploy a new MySQL container with: `docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest`

*Replace <container_name> with the name of your choice. If you do not provide a name, Docker will generate a random one
The -d option:  instructs Docker to run the container as a service in the background
Replace <my-secret-pw> with your chosen password
In the command above, we used the latest version tag. This tag may differ according to the image you downloaded*

<br>
  
<img width="780" alt="deploy_msql" src="https://user-images.githubusercontent.com/92983658/215412587-e5333625-5a54-49b8-9dd2-80d06a731d3b.png">

<br>

- check to see if the MySQL container is running: `docker ps -a`

*You should see the newly created container listed in the output. It includes container details, one being the status of this virtual environment. The status changes from health: starting to healthy, once the setup is complete.*


<br>
  
<img width="1303" alt="mysql_confirm" src="https://user-images.githubusercontent.com/92983658/215413358-b96681a8-4cbf-4a11-b6e9-e03c5f4cb78f.png">

<br>
  
## Part Three: Connect To Mysql Docker Container
You can either connect directly to the container running the MySQL server or use a second container as a MySQL client.

### Connecting directly to the container running the MySQL server: - Approach one: 

```

docker exec -it mysql bash

or

docker exec -it mysql mysql -uroot -p
  
```
  
<br>
  
### Approach two: Connecting to the MySQL server from a second container running the MySQL client utility
- remove the previous mysql docker container and verify it is deleted
```
  
docker ps -a
docker stop <container name>
docker rm <container name> or <container ID> 04a34f46fb98
docker ps -a

```
  
<br>
  
<img width="1336" alt="remove_msql_docker" src="https://user-images.githubusercontent.com/92983658/215417795-d5a0c8bf-cbc7-4f95-83b4-05b88852e9c1.png">

<br>

#### create a network
creating a custom network is not neccessary in most cases because docker will create a default network. But there are use cases where this is nevessary...For example, if there is a requirement to control the `cidr range` of the containers running the entire application stack. This will be an ideal situation to create a network and specify the `--subnet`

For clarity’s sake, we will create a network with a subnet dedicated for our project and use it for both MySQL and the application so that they can connect.
  

- `docker network create --subnet=172.18.0.0/24 tooling_app_network`

<br>
  
<img width="831" alt="network" src="https://user-images.githubusercontent.com/92983658/215420865-5ce2e284-52a5-4ff8-aae2-901883db449b.png">

<br>

#### Run the MySQL Server container using the created network.
- create an environment variable to store the root password: `export MYSQL_PW=<your password>`
- verify the environment variable is created: `echo $MYSQL_PW`
- pull the image and run the container: `docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest`
  
Flags used

- `-d` runs the container in detached mode
- `--network` connects a container to a network
- `-h` specifies a hostname
- If the image is not found locally, it will be downloaded from the registry.
  
<br>
  
<img width="837" alt="mysql_run" src="https://user-images.githubusercontent.com/92983658/215422639-ce733718-7360-49f6-9117-c7bf8c7d8ba2.png">

<br>
  
- Verify the container is running: `docker ps -a`

<br>
  
<img width="1357" alt="server_verify" src="https://user-images.githubusercontent.com/92983658/215423020-a385c73d-814d-4ca8-8287-5096605d7b79.png">

<br>
  
- Because it's not a good practice to connect to MySQL server remotely using the root user, create an SQL script that will create a user we can use to connect remotely.
  - Create a file and name it `create_user.sql` : `touch create_user.sql`
  - and add the following code in the file: `CREATE USER 'libby'@'%' IDENTIFIED BY 'devopspbl'; GRANT ALL PRIVILEGES ON * . * TO 'libby'@'%'; `
  
  <br>
  
  <img width="1378" alt="create_user" src="https://user-images.githubusercontent.com/92983658/215765520-b68cfc44-dea2-4103-aca8-a06bbdddfae4.png">

  <br>
  
  - use the following script to create new user: `docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql`

 *note: If you see a warning like below, it is acceptable to ignore:`mysql: [Warning] Using a password on the command line interface can be insecure.`
                                                                                                        
<br>
  
<img width="1037" alt="mysql_script" src="https://user-images.githubusercontent.com/92983658/215428375-78ef2219-c3e6-4680-9f67-283cc68b1828.png">

<br>
  
### Connect to the MySQL server from a second container running the MySQL client utility
  
The good thing about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the MySQL server.
  
- Run the MySQL Client Container: `docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u <enter username>  -p `
*note: to stay connected to the server, do not exit the terminal*
  
Flags used:

- `--name` gives the container a name
- `-it` runs in interactive mode and Allocate a pseudo-TTY
- `--rm` automatically removes the container when it exits
- `--network` connects a container to a network
- `-h a MySQL` flag specifying the MySQL server Container hostname
- `-u user` created from the SQL script
- `username` admin username-for-user-created-from-the-SQL-script-create_user.sql
- `-p` password specified for the user created from the SQL script
  
<br>  
                                                                                                        
<img width="1203" alt="client_server" src="https://user-images.githubusercontent.com/92983658/215768560-e4f0006f-0745-4887-9872-40d8f303a66f.png">

<br>

## Part Four: Prepare database schema
A database schema needs to be created so that the Tooling application can connect to it.

- on a new terminal, clone the Tooling-app repository: `git clone https://github.com/darey-devops/tooling.git`
  
<br>
  
<img width="1266" alt="git_clone" src="https://user-images.githubusercontent.com/92983658/215440923-3af19232-00ee-4556-a84a-5790e401bb41.png">

<br>
  
- On the terminal in `tooling` directory, export the location of the SQL file: `export tooling_db_schema=*/tooling_db_schema.sql`
- Verify that the path is exported: `echo $tooling_db_schema`  
*You can find the tooling_db_schema.sql in the tooling/html/tooling_db_schema.sql folder of cloned repo*
 
<br>
  
<img width="1004" alt="schema_1b" src="https://user-images.githubusercontent.com/92983658/215449638-30ce8044-64ad-42e5-9339-b633d799ea63.png">

<br>
  
- Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.: `docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema`

<br>
  
<img width="1001" alt="schema_1c" src="https://user-images.githubusercontent.com/92983658/215449948-2b1f9f2d-2586-4f4c-a5b8-e376ede2caab.png">

<br>
  
- Update the `.env` file with connection details to the database: `nano */html/.env`
*The .env file is located in the html tooling/html/.env folder but not visible in terminal. you can use vi or nano*
  
```

MYSQL_IP=mysqlserverhost
MYSQL_USER=username
MYSQL_PASS=client-secrete-password
MYSQL_DBNAME=toolingdb
  
```

<br>
  
<img width="995" alt="env" src="https://user-images.githubusercontent.com/92983658/215451390-bd5f9b87-536a-45c3-851f-52f5c0f16e09.png">

<br>
 
## Part Five: Run The Tooling App
  
- on a new terminal, ensure you are inside the directory `tooling` that has the file Dockerfile, update docker file: `nano Dockerfile`

<br>
  
<img width="1317" alt="dockerfile" src="https://user-images.githubusercontent.com/92983658/215749655-1939e53b-ea71-456d-a0fe-4d7de7e5c762.png">

<br>
  
- Ensure you are inside the directory `tooling` that has the file Dockerfile and build your container :`docker build -t tooling:0.0.1 .`

*In the above command, we specify a parameter `-t`, so that the image can be tagged tooling"0.0.1. Also, notice the `.` at the end. This is important as that tells Docker to locate the Dockerfile in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the Dockerfile.*

<br>
  
<img width="1384" alt="build" src="https://user-images.githubusercontent.com/92983658/215771172-326636ee-0a69-4e3c-8000-5828c048a199.png">

<br> 

- Run the container: `docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1 ` 
  *note: do not exit the terminal whilst the container is running*

<br>
  
flags in the command:

- `--network` flag so that both the Tooling app and the database can easily connect on the same virtual network we created earlier.
- `-p` flag is used to map the container port with the host port. Within the container, apache is the webserver running and, by default, it listens on port 80. Confirm this with the `CMD ["start-apache"]` section of the Dockerfile. But we cannot directly use port 80 on our host machine because it is already in use. The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.
  
<br>
  
<img width="1315" alt="docker_run" src="https://user-images.githubusercontent.com/92983658/215755842-c1759d1e-5ea3-4d9b-a4b0-5bd21739749c.png">

<br>

- Test the tooling app in the browser: `http://localhost:8085`
  
<br>
  
<img width="1195" alt="tooling" src="https://user-images.githubusercontent.com/92983658/215772820-9b1b8a58-290a-4ed9-b2b7-35c5469e33d1.png">

<br>
  
## Practice Task No 1: Implement a POC to migrate the PHP-Todo app into a containerized application.

- Download `php-todo` repository: `git clone https://github.com/earchibong/php-todo.git`

## Part One:
  
**Write a `Dockerfile` for the TODO app**

- Update the `.env.sample` file with connection details to the database:

```
  
...

DB_HOST=mysqlserverhost
DB_DATABASE=toolingdb
DB_USERNAME=<username used to create mysql database>
DB_PASSWORD=<your password used to create the mysql server>
DB_CONNECTION=mysql
DB_PORT=3306
  
...
  
```

<br>
  
<img width="848" alt="env_file_1a" src="https://user-images.githubusercontent.com/92983658/216052052-4c893069-45b9-4d6d-9ea9-81b1c70ff9d2.png">

<br>
  
- in `phptodo` directory, create a new file named: `Dockerfile` and add the following:

```
  
# Tells the image to use the latest version of PHP
FROM php:7-apache
LABEL MAINTAINER Libby

ENV DB_HOST=mysqlserverhost
ENV DB_DATABASE=toolingdb
ENV DB_USERNAME=<username used to create mysql database>
ENV ENV DB_PASSWORD=<your password used to create the mysql server>
ENV DB_CONNECTION=mysql
ENV DB_PORT=3306

#install all the dependencies
RUN apt update
RUN apt install zip git nginx -y
RUN docker-php-ext-install pdo_mysql mysqli

#install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer

#define project folder
WORKDIR /var/www/html

#change uid and gid of apache to docker user uid/gid
RUN usermod -u 1000 www-data && groupmod -g 1000 www-data

#change apache setting
RUN sed -i -e "s/var\/www/app/g" /etc/apache2/apache2.conf && sed -i -e "s/html/public/g" /etc/apache2/apache2.conf
RUN a2enmod rewrite

#copy source files, run composer and set permissions
COPY . .
RUN mv /var/www/html/.env.sample /var/www/html/.env 
RUN chmod +x artisan

RUN composer install --no-interaction
RUN php artisan db:seed
RUN php artisan key:generate

ENTRYPOINT php artisan serve --host 0.0.0.0 --port 5001
  
```

<br>
  
<img width="860" alt="docker_file_2a" src="https://user-images.githubusercontent.com/92983658/216055929-f4bb4dd5-27a9-476b-8365-650923e7922d.png">
<img width="856" alt="docker_file_2b" src="https://user-images.githubusercontent.com/92983658/216055945-eb74fa65-4000-4f01-a1de-556c2704814f.png">

<br>
  
**Run both database and app on laptop Docker Engine**
  
- Connect to the exisiting MySQL server (from the first project) using a second container running the MySQL client utility: `docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u <enter username>  -p`
  
*note: if the Mysql server didn't exist, we would have had to build one first before connecting...as done above in the `tooling` project*
  
<br>
  
<img width="977" alt="mysql_client" src="https://user-images.githubusercontent.com/92983658/216038764-38875185-608e-4b56-9094-23e54f49e62a.png">

<br>
  
<img width="1261" alt="mysql_client_2" src="https://user-images.githubusercontent.com/92983658/216038813-9a021efb-ba1f-4333-bb5d-983da82ab3b9.png">

<br>

- ensure you are in the `php-todo` directory and then build docker image of the app: `docker build -t php-todo:0.0.1 .`
  
<br>
  
<img width="823" alt="php-build" src="https://user-images.githubusercontent.com/92983658/216044829-5df6357e-dc4f-40ba-9c68-8ef4e7d705fa.png">

<br>
  
- Run the container: `docker run --network tooling_app_network -p 8085:80 -it php-todo:0.0.1 ` 
  *note: do not exit the terminal whilst the container is running*  

<br>
  
<img width="813" alt="php-run" src="https://user-images.githubusercontent.com/92983658/216047865-b951ae87-4d6f-4c5a-b3a8-c6b4d8d5a5e7.png">

<br>

- in a new terminal, run `artisan migrate` command: `docker exec <your_container_name> php artisan migrate`

<br>
  
<img width="813" alt="artisan_migrate" src="https://user-images.githubusercontent.com/92983658/216058059-2b6a6afc-ba27-4808-b727-532b915f0367.png">

<br>
  
**Access the application from the browser**

- `http://localhost:8085`

<br>

<img width="1057" alt="php_app" src="https://user-images.githubusercontent.com/92983658/216281444-0f6c1744-42c8-4560-99ed-c929bc97ff08.png">

<br>
  
## Part 2 : Pushing The Docker Image To Docker Registry

- Create an account in <a href="https://hub.docker.com/">Docker Hub</a>
- Create a new Docker Hub repository
  
<br>
  
<img width="1195" alt="repo_docker_hub" src="https://user-images.githubusercontent.com/92983658/216286069-10b28490-e3aa-445c-96b7-4eb54eea245d.png">

<br>
  
- Push the docker images from your PC to the repository

<br>
  
```
  
docker login
docker tag <image id> <repository>:<tagname>
docker push <repository>:<tagname>
  
```
  
<br>
  
<img width="813" alt="docker_push" src="https://user-images.githubusercontent.com/92983658/216294554-28640306-73ff-4c41-aea2-c3ea5ee2eab7.png">

<br>
  
<img width="1195" alt="docker_push_1b" src="https://user-images.githubusercontent.com/92983658/216295062-7cd1ffba-7cd0-41fd-ad18-b425de52bc43.png">

<br>
  
## Part Three: Running Docker Build And Docker Push on Jenkins

### Amazon Web Services setup
  
- Get an access key: To create an access key, go to `Amazon Console`, then `IAM`, then `Users`, [your user], `Security credentials`, and `Create Access Key`.

Your browser will download a file containing the `Access Key ID` and the `Secret Access Key`. These values will be used in Jenkins to authenticate to Amazon.
  
- Create a private repository in `AWS Elastic Container Registry`
You need to set up an image repository for each image that you publish. Give the repository the same name you want the image to have.

You will see your repository under `Amazon ECR`, then `Repositories`. Make a note of the zone it's in, in the `URI field`.
  
<br>

<img width="1194" alt="ecr_1a" src="https://user-images.githubusercontent.com/92983658/216297822-05b4e6e4-0e66-4d17-b4a3-6692a2aa1f3f.png">

<br>

<img width="1195" alt="ecr_1b" src="https://user-images.githubusercontent.com/92983658/216297879-b84f377e-b53f-49c2-a7d7-a0c2a99f5299.png">

<br>

### Jenkins Setup
  
#### set up a jenkins server
  
- launch EC2 instance, ensure `port 8080` is enabled and install Jenkins server. (learn how to do that <a href="https://github.com/earchibong/devops_training/blob/main/CI.md">here</a>)

<br>
  
<img width="1223" alt="jenkins" src="https://user-images.githubusercontent.com/92983658/216305857-c7eef134-57aa-4772-899c-3d5ea6ac8654.png">

<br>

- install docker : find out more <a href="https://docs.docker.com/engine/install/ubuntu/">here</a>

<br>
  
<img width="1224" alt="docker_ubuntu" src="https://user-images.githubusercontent.com/92983658/216352152-ea512284-14be-498b-bec1-18fc5189699e.png">

<br>
  
- install docker plugins to run docker jobs: `CloudBees AWS credentials`, `Amazon ECR` and `Docker pipeline`

<br>
  
<img width="1195" alt="docker_plugins" src="https://user-images.githubusercontent.com/92983658/216308789-fba245d8-645f-486d-b759-d5975523397d.png">
  
<br>
  
<img width="1225" alt="docker_ubuntu_1b" src="https://user-images.githubusercontent.com/92983658/216353385-64364980-7c3f-428b-a51f-1f27c420f2e6.png">

<br>
  
- Add credentials in Jenkins: In Jenkins instance, go to `Manage Jenkins`, then `Manage Credentials`, then `Jenkins Store`, then `Global Credentials` (unrestricted), and finally `Add Credentials`.
  
Fill in the following fields, leaving everything else as default:

- `Kind` - AWS credentials
- `ID` - aws-credentials, for example
- `Access Key ID` - Access Key ID from earlier
- `Secret Access Key` - Secret Access Key from earlier

Click OK to save.

<br>
  
<img width="1183" alt="credentials" src="https://user-images.githubusercontent.com/92983658/216551232-0135ac04-4684-436c-9937-2a3fb913c4c3.png">

<br>
  
- Configure Jenkins Pipeline: Go to the `Jenkins Dashboard`, then `New Item`.
  Give your pipeline a name and select the Pipeline item, then OK.

Fill out the following fields for the pipeline, leaving everything else as default:

- `GitHub hook trigger for GITScm polling` : check the box
- `Definition` : pipeline script from SCM
- `SCM`: Git
- `Repository URL`: the URL of your forked repo and the jenkins-ecr branch
- `Credentials`:  zone of the repository
- `Branch Specifier` : */develop

  Click SAVE.
 
<br>
  
<img width="1191" alt="pipeline" src="https://user-images.githubusercontent.com/92983658/216552253-893e8b77-cf71-43cb-985e-1999a422cac2.png">

<br>

### Github Setup

- Set up a webhook so that Jenkins knows when the repository is updated: go to `Settings`, then `Webhooks`.

<br>
  
<img width="1188" alt="webhook" src="https://user-images.githubusercontent.com/92983658/216560870-95c21c6a-4c45-4370-893f-b48b3df4e1a0.png">

<br>
  
 
- on terminal, give docker permissions: `sudo chmod 666 /var/run/docker.sock`
- log in from the command line: `aws ecr get-login-password --region <aws region> | docker login --username AWS --password-stdin <aws ecr repository url>`
  
<br>

<img width="1224" alt="login" src="https://user-images.githubusercontent.com/92983658/216358735-43c5ecc9-b644-42af-9ec4-e935d6125355.png">

<br>
  
- Create two branches in `php-todo` github repo - `develop` and `feature`: `git checkout -b develop` and `git checkout -b feature`
  
<br>
  
<img width="937" alt="branches" src="https://user-images.githubusercontent.com/92983658/216359764-df62212a-796a-455e-bb02-b218ec33582e.png">

<br>
  
- Create a Jenkinsfile for the two branches which will run `docker build` and push the image to AWS ECR repository
  
```
  
pipeline {
  agent any

      environment 
    {
        PROJECT     = 'php-todo'
        ECRURL      = '350100602815.dkr.ecr.eu-west-2.amazonaws.com/php-todo'
        DEPLOY_TO = 'develop'
    }

  stages {

    stage("Initial cleanup") {
        steps {
        dir("${WORKSPACE}") {
            deleteDir()
        }
        }
    }

    stage('Checkout')
    {
      steps {
      checkout([
        $class: 'GitSCM', 
        doGenerateSubmoduleConfigurations: false, 
        extensions: [],
        submoduleCfg: [], 
        branches: [[name: 'develop']],
        userRemoteConfigs: [[url: "https://github.com/earchibong/php-todo.git ",credentialsId:'410c5088-f780-4c70-a3e2-4f17174af72e']] 	
        ])
        
      }
        }

    stage('Build preparations')
      {
        steps
          {
              script 
                {
                    // calculate GIT lastest commit short-hash
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    shortCommitHash = gitCommitHash.take(7)
                    // calculate a sample version tag
                    VERSION = shortCommitHash
                    // set the build display name
                    currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                    IMAGE = "$PROJECT:$VERSION"
                }
            }
      }   

    stage('Build For Dev Environment') {
               when { branch pattern: "^feature.*|^bug.*|^dev", comparator: "REGEXP"}
            
        steps {
            echo 'Build Dockerfile....'
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-west-2 | sed 's|https://||')") 
                sh "docker build --network=host -t $IMAGE ."
                docker.withRegistry("https://$ECRURL"){
                docker.image("$IMAGE").push("dev-$BUILD_NUMBER")
            }
            }
        }
      }

    stage('Build For Staging Environment') {
            when {
                expression { BRANCH_NAME ==~ /(staging|develop)/ }
            }
        steps {
            echo 'Build Dockerfile....'
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-west-2 | sed 's|https://||')") 
                sh "docker build --network=host -t $IMAGE ."
                docker.withRegistry("https://$ECRURL"){
                docker.image("$IMAGE").push("dev-staging-$BUILD_NUMBER")
                }
            }
        }
    }


    stage('Build For Production Environment') {
        when { tag "release-*" }
        steps {
            echo 'Build Dockerfile....'
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-west-2 | sed 's|https://||')") 
                // sh "docker build --network=host -t $IMAGE -f deploy/docker/Dockerfile ."
                sh "docker build --network=host -t $IMAGE ."
                docker.withRegistry("https://$ECRURL"){
                docker.image("$IMAGE").push("prod-$BUILD_NUMBER")
                }
            }
        }
    }
  }

        post
    {
        always
        {
            sh "docker rmi -f $IMAGE "
        }
    }
} 
  
```
  
<br>
  
- push changes to github repository
- Create a multibranch pipeline job and linking it to the `php-todo` repository


- Write a `Jenkinsfile` that will simulate a Docker Build and a Docker Push to the registry
  
```
  
