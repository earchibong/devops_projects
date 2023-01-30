# MIGRATION TO THE CLOUD WITH CONTAINERIZATION - DOCKER

In this project, the frontend and the backend(MySQL) of tooling application is built and containerized using DOCKER of which its image 
is pushed to Docker registry. And further in the project, the php-todo application is also built into a container and pushed into the 
AWS Elastic Container Registry using a CI/CD tool known as Jenkins and Docker Compose is also implemented.

## Step One: Install Docker and prepare for migration to the Cloud

Docker allocates not the whole guest OS for an application, but only isolates a minimal part of it – this isolated container has all that the application needs and at the same time is lighter, faster, and can be shipped as a Docker image to multiple physical or virtual environments, as long as this environment can run Docker engine. This approach also solves the environment incompatibility issue. If the application is shipped as a container, it has its own environment isolated from the rest of the world, and it will always work the same way on any server that has Docker engine.

- find out how to install `docker` <a href="https://docs.docker.com/desktop/install/mac-install/">here</a>

<br>

## Step Two: Migrate the Tooling Web Application from a VM-based solution into a containerized one.
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

