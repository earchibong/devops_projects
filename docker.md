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

### Approach one: 
Connecting directly to the container running the MySQL server:
```

docker exec -it mysql bash

or

docker exec -it mysql mysql -uroot -p
  
```
  
<br>
  
### Approach two:
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
  



