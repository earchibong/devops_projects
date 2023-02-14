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
  
- Run the container: `docker run --network tooling_app_network -p 8080:8080 -e"PORT=8085" -it php-todo:0.0.1 ` 
  *note: do not exit the terminal whilst the container is running*  

<br>
  

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
  
## Part Three: Build ANd Push Docker Image Using Jenkins
  
### Amazon Web Services setup
  
- Create a private repository in `AWS Elastic Container Registry`
You need to set up an image repository for each image that you publish. Give the repository the same name you want the image to have.

You will see your repository under `Amazon ECR`, then `Repositories`. Make a note of the zone it's in, in the `URI field`.
  
<br>

<img width="1194" alt="ecr_1a" src="https://user-images.githubusercontent.com/92983658/216297822-05b4e6e4-0e66-4d17-b4a3-6692a2aa1f3f.png">

<br>

<img width="1195" alt="ecr_1b" src="https://user-images.githubusercontent.com/92983658/216297879-b84f377e-b53f-49c2-a7d7-a0c2a99f5299.png">

<br>
  
- Create an IAM role with `AmazonEC2ContainerRegistryFullAccess` policy and attach it with `ec2 instance`
  
<br>
  
<img width="1190" alt="iam_1a" src="https://user-images.githubusercontent.com/92983658/216974469-6d093897-322d-440b-94ba-638867367c37.png">

<br>
  
<img width="1194" alt="iam_1b" src="https://user-images.githubusercontent.com/92983658/216974503-13a8cfc6-39d7-478f-b03a-35fd030fbcb8.png">

<br>
  
 

### Jenkins Setup
  
#### set up a jenkins server
  
- launch EC2 instance, ensure `port 8080` is enabled and install Jenkins server. (learn how to do that <a href="https://github.com/earchibong/devops_training/blob/main/CI.md">here</a>)

<br>
  
<img width="1223" alt="jenkins" src="https://user-images.githubusercontent.com/92983658/216305857-c7eef134-57aa-4772-899c-3d5ea6ac8654.png">

<br>
  
- in the Jenkins instance, modify the IAM role for the instance: select `Actions > Security > Modify IAM Role` 
  
<br>
  
<img width="1195" alt="iam_1d" src="https://user-images.githubusercontent.com/92983658/217239247-9182aa4b-0f6e-4d9a-a46c-c3331c4af7c9.png">

<br>
  
<img width="1198" alt="iam_1e" src="https://user-images.githubusercontent.com/92983658/217239384-70c54070-95ac-4020-a2f4-64d5c2b09a32.png">

<br>

- install docker : find out more <a href="https://docs.docker.com/engine/install/ubuntu/">here</a>

<br>
  
<img width="1224" alt="docker_ubuntu" src="https://user-images.githubusercontent.com/92983658/216352152-ea512284-14be-498b-bec1-18fc5189699e.png">

<br>
  
- install docker plugins to build and push the Docker image to  Docker Hub` and `ECR  : `CloudBees Docker Build and Publish`, `Amazon ECR`, `Docker pipeline` and `blue ocean`

<br>
  
<img width="1195" alt="docker_plugins" src="https://user-images.githubusercontent.com/92983658/216308789-fba245d8-645f-486d-b759-d5975523397d.png">
  
<br>
  
<img width="1225" alt="docker_ubuntu_1b" src="https://user-images.githubusercontent.com/92983658/216353385-64364980-7c3f-428b-a51f-1f27c420f2e6.png">

<br>
  
- Configure Jenkins pipeline: Go to the `Jenkins Dashboard` and `create new job`
  - Enter the name of the job and select the type of job you wish to run on Jenkins. 
  - Select the `Multi-branch Pipeline` option to create a Jenkins pipeline that executes a series of steps from multiple branches.
  
<br>
  
<img width="1191" alt="multi-branch" src="https://user-images.githubusercontent.com/92983658/217520433-62202b94-93c5-4977-b65e-a5bc5f4d9704.png">

<br>
  
- For branch sources, enter your `git credentials` (you can use username and password) and `git repository`...then validate
  
<br>
  
<img width="1196" alt="multi-branch-sources" src="https://user-images.githubusercontent.com/92983658/217522087-49183420-1f8b-43f9-8afc-511254a8f1f0.png">

<br>
  
- **build configuration**: `Jenkinsfile`  
- **properties**: `Docker Registry URL` is `< your AWS ECR URI >` 
- then select your `ECR region`
  
<br>
  
<img width="1190" alt="multi-properties" src="https://user-images.githubusercontent.com/92983658/217523311-dc920ed4-c835-411c-8706-39cd6ede7781.png">

<br>
  


- Create multiple branches  `php-todo` github repo - `develop` and `feature`:

```
  
git checkout -b develop
git push --set-upstream origin develop
git checkout -b feature
git push --set-upstream origin feature

  
```
  
<br>
  
<img width="983" alt="feature" src="https://user-images.githubusercontent.com/92983658/217525686-1ce4571a-b2c0-4614-aea6-58de9d40d132.png">

<br>
  
- Creating the Jenkinsfile for the two branches which will run docker build and push the image to AWS ECR repository

  
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
      checkout scmGit(
        branches: [[name: '*/develop'], [name: '*/feature']], 
        extensions: [], 
        userRemoteConfigs: [[credentialsId: '23ef1a81-ff88-4724-9462-8134b6d8ad86', url: 'https://github.com/earchibong/php-todo.git']]
        )
        
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

    stage('Build & Deploy For Dev Environment') {
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

    stage('Build & Deploy For Feature Environment') {
            when { 
            expression { BRANCH_NAME ==~ /feature\/[0-9]+\.[0-9]+\.[0-9]+/ }
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

*note: for `checkout stage` information...head to Jenkins, and select `pipeline syntax`: **sample step** - `checkout from version control`, **SCM** - `git`, **enter repository and git credentials**, **enter branches to build** - `develop, main and feature`...then generate the pipeline script*

<br>

<img width="857" alt="jenkinsfile" src="https://user-images.githubusercontent.com/92983658/217538770-a5d03f26-684f-46ed-8fe5-ba5732fdef78.png">

<br>

- push changes to github repository
- on terminal, log onto Jenkins instance and run the following `docker` permission: `sudo chmod 666 /var/run/docker.sock`
- on Jenkins, select your branch and build
  
<br>
  
<img width="1194" alt="feature_blue_develop" src="https://user-images.githubusercontent.com/92983658/217541124-292156c5-a6d7-4e88-b073-a9d498471d8e.png">

<br>
  
<img width="1192" alt="ocean_blue_develop" src="https://user-images.githubusercontent.com/92983658/217541202-ba1b25c7-0977-4860-b1d4-9d2a28f8bc27.png">

<br>
  
- Verify that the images pushed from the CI can be found at the registry.

<br>
  
<img width="1196" alt="ecr_verify" src="https://user-images.githubusercontent.com/92983658/217542015-2b870221-3f2a-4564-b067-8895552e8c9a.png">

<br>
  
## Part Four: Deploy With Docker Compose
  
- Install Docker Compose from <a href="https://docs.docker.com/compose/install/"><here</a>
- Create a file, name it `tooling.yaml`
- write the Docker Compose definitions with YAML syntax. The YAML file is used for defining services, networks, and volumes:
```
  
version: "3"
services:
  tooling_frontend:
    build: .
    depends_on:
      - db
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
    

  db:
    image: mysql:5.7
    platform: linux/amd64 #(add this line if you have Apple Silicon M1 chip)
    restart: always
    environment:
      MYSQL_DATABASE: toolingdb
      MYSQL_USER: libby
      MYSQL_PASSWORD: devopspbl
      MYSQL_ROOT_PASSWORD: devopspbl   
    volumes:
      - db:/var/lib/mysql
      - .datadump.sql/:/docker-entrypoint-initdb.d/datadump.sql

networks:
  default:
    name: tooling_app_network
    external: true
  
volumes:
  tooling_frontend:
  db:
  
```

** note: most of the Docker images are not compatible with the newest Apple Silicon M1 chip throwing no matching manifest for `linux/arm64/v8`
<br>
  
<img width="978" alt="tooling_yml" src="https://user-images.githubusercontent.com/92983658/217549660-c99e2cf5-e160-42ac-ac41-2ee85f687c3b.png">

<br>

- Run the command to start the containers: `docker-compose -f tooling.yaml  up -d`
- Verify that the compose is in the running status: `docker compose ls`
  
<br>
  
<img width="1443" alt="docker_compose_2" src="https://user-images.githubusercontent.com/92983658/218091091-3c107953-359c-49e5-aa91-ca5e19df2bef.png">
  
<br>
  
- connect to Mysql server from MYsql client utility: `docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h <mysql server> -u <username in tooling.yaml> -p`
  
<br>
  
<img width="1397" alt="server-client" src="https://user-images.githubusercontent.com/92983658/218097967-561256cf-d803-432b-a3da-c9fe42121c12.png">

<br>
  
## Practice Task No 2: Complete Continous Integration With A Test Stage
  
### Docker Compose Yaml File Structure

Docker Compose files work by applying multiple commands that are declared within a single `tooling.yml` configuration file.
  
```
  
version: "3"
  
services:
   tooling_frontend:
    build: .
    depends_on:
      - db
    ports:
      - "5000:80"  
    volumes:
      - tooling_frontend:/var/www/html
    

  db:
    image: mysql:5.7
    platform: linux/amd64
    restart: always
    environment:
      MYSQL_DATABASE: toolingdb
      MYSQL_USER: libby
      MYSQL_PASSWORD: devopspbl
      MYSQL_ROOT_PASSWORD: devopspbl   
    volumes:
      - db:/var/lib/mysql
      - .datadump.sql/:/docker-entrypoint-initdb.d/datadump.sql

networks:
  default:
    name: tooling_app_network
    external: true
  
volumes:
  tooling_frontend:
  db:

```
  
<br>
  
- `version 3`: This denotes that we are using version 3 of Docker Compose, and Docker will provide the appropriate features.
- `services:` This section defines all the different containers we will create. In our example, we have two services, web and database.
- `tooling_frontend`: This is the name of our app service. Docker Compose will create containers with the name we provide.
- `build`: This specifies the location of our Dockerfile, and `.` represents the directory where the `tooling.yml` file is located.
- `depends_on`: specifies the services related to the app we're building. Docker compose will start the related services first before building the app
- `ports:` This is used to map the container’s ports to the host machine. In this case, container post `5000` is mapped to host post `80`
- `volumes:` This is just like the `-v` option for mounting disks in Docker. In this example, we attach our `var/www/html` directory to the containers. This way, we won’t have to rebuild the images if changes are made.
- `image:` If we don’t have a Dockerfile and want to run a service using a pre-built image, we specify the image location using the image clause. Compose will fork a container from that image.
- `platform`: specifies the platform on which the image will be built
- `restart`: restarts all stopped services. In this case will always restart the database service
- `environment:` The clause allows us to set up an environment variable in the container. This is the same as the -e argument in Docker when running a container.
- `networks`: allows us to specify custom networks instead of just using the default netowrk. It also connects services to externally-created networks which aren’t managed by Compose... such as this instance with the `tooling_app_network`
  
### Updating Jenkinsfile With Test Stage
  
- update the `php-todo` jenkinsfile above with the following test stage:
  
```
  

  
  
  
