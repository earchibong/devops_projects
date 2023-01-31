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
  
### Connecting directly to container running Mysql server - Approach two:
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

#### Step one: create a network
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

-d runs the container in detached mode
--network connects a container to a network
-h specifies a hostname
If the image is not found locally, it will be downloaded from the registry.
  
<br>
  
<img width="837" alt="mysql_run" src="https://user-images.githubusercontent.com/92983658/215422639-ce733718-7360-49f6-9117-c7bf8c7d8ba2.png">

<br>
  
- Verify the container is running: `docker ps -a`

<br>
  
<img width="1357" alt="server_verify" src="https://user-images.githubusercontent.com/92983658/215423020-a385c73d-814d-4ca8-8287-5096605d7b79.png">

<br>
  
- create an SQL script that will create a user we can use to connect remotely.
  - Create a file and name it `create_user.sql` : `touch create_user.sql`
  - and add the following code in the file: `CREATE USER ''@'%' IDENTIFIED BY ''; GRANT ALL PRIVILEGES ON * . * TO ''@'%'; `
  
  <br>
  
  <img width="1038" alt="nano_create_sql" src="https://user-images.githubusercontent.com/92983658/215428009-f5401076-cbc4-4a2b-a832-ae377425c158.png">

  <br>
  
  - use the following script to `docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql`

 *note: If you see a warning like below, it is acceptable to ignore:`mysql: [Warning] Using a password on the command line interface can be insecure.`
                                                                                                        
<br>
  
<img width="1037" alt="mysql_script" src="https://user-images.githubusercontent.com/92983658/215428375-78ef2219-c3e6-4680-9f67-283cc68b1828.png">

<br>
  
### Connecting to the MySQL server from a second container running the MySQL client utility
  
The good thing about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the MySQL server.
  
- Run the MySQL Client Container: `docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u  -p `
  
Flags used:

- `--name` gives the container a name
- `-it` runs in interactive mode and Allocate a pseudo-TTY
- `--rm` automatically removes the container when it exits
- `--network` connects a container to a network
- `-h a MySQL` flag specifying the MySQL server Container hostname
- `-u user` created from the SQL script
- admin username-for-user-created-from-the-SQL-script-create_user.sql
- `-p` password specified for the user created from the SQL script
  
<br>  
                                                                                                        
<img width="1340" alt="2nd_container_mysql_docker" src="https://user-images.githubusercontent.com/92983658/215430580-20ea7eb8-cb6b-4c9d-a7cb-e12ac7943930.png">

<br>

## Part Four: Prepare database schema
A database schema needs to be created so that the Tooling application can connect to it.

- Clone the Tooling-app repository: `git clone https://github.com/darey-devops/tooling.git`
  
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
 
## Run The Tooling App
  
- update docker file: `nano Dockerfile`

<br>
  
<img width="1317" alt="dockerfile" src="https://user-images.githubusercontent.com/92983658/215749655-1939e53b-ea71-456d-a0fe-4d7de7e5c762.png">

<br>
  
- Ensure you are inside the directory `tooling` that has the file Dockerfile and build your container :`docker build -t tooling:0.0.1 .`

*In the above command, we specify a parameter `-t`, so that the image can be tagged tooling"0.0.1. Also, notice the `.` at the end. This is important as that tells Docker to locate the Dockerfile in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the Dockerfile.*

<br>
  
<img width="997" alt="build" src="https://user-images.githubusercontent.com/92983658/215452676-c3b9e7d7-08ef-4ea1-8e80-a0922019c56a.png">

<br> 

- Run the container: `docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1 `
  
<br>
  
<img width="1397" alt="docker_run_1a" src="https://user-images.githubusercontent.com/92983658/215457175-9bc0f026-edaf-4ebe-b888-1e915edaa938.png">

<br>



 
