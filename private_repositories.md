# Setting Up Private Repositories And Preparing CI Pipelines With Jenkins

In Artifactory, either local, remote or virtual repositories can be set up. It doesn’t matter if it is for docker, helm, or standard software binaries.

Artifactory hosts two kinds of repositories: `local and remote`. Both local and remote repositories can be aggregated under virtual repositories, in order to create controlled domains for artifacts resolution and search.

**Local Repositories:** are physical locally-managed repositories that one can deploy artifacts into. Artifacts under a local repository are directly accessible via a URL pattern such as : `https://tooling.artifactory.archibong.link/artifactory/<local-repository-name>/<artifact-path>`

**Remote Repositories:** serve as a caching proxy for a repository managed at a remote URL (including other Artifactory remote repository URLs). Artifacts are stored and updated in remote repositories according to various configuration parameters that control the caching and proxying behavior. Cached artifacts can be removed from remote repository caches, but you cannot manually deploy anything into a remote repository.

Artifacts under a remote repository are directly accessible via a URL pattern of

`https://tooling.artifactory.archibong.link/<remote-repository-name>/<artifact-path>` or
`https://tooling.artifactory.archibong.link/artifactory/<remote-repository-name>-cache/<artifact-path>`

The second URL will only serve already cached artifacts while the first one will fetch a remote artifact in the cache URL (2nd one) only if it is not already stored in the first URL.

**Virtual Repositories:** or `repository group` aggregates several repositories under a common URL. The repository is virtual in the sense that you can resolve and get artifacts from it but you cannot deploy anything to it.

By default, Artifactory uses a global virtual repository that is available at: `https://tooling.artifactory.archibong.link/artifactory/repo`

This repository contains all local and remote repositories.

## Labs

- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#create-a-local-repository-for-docker">Create A Local Repository For Docker</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#create-a-virtual-repository">Create A Virtual Repository</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#push-docker-images-to-the-repository">Push Docker Images To Artifactory Repository</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#jenkins-pipeline-for-business-applications">Jenkins Pipeline For Business Applications</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#deploy-jenkins-with-helm">Deploy Jenkins With Helm</a>

<br>


## Create A Local Repository For Docker


- get a free trial license for artifactory <a href="https://jfrog.com/start-free/#saas">here</a>

<br>

<img width="1226" alt="repos" src="https://user-images.githubusercontent.com/92983658/230719776-6530e367-9d52-4da6-87a0-e25de475cda7.png">

<br>

- Since we are exploring Local Repositories, type `Docker` in the search box and select the `Docker icon`

<br>

<img width="1221" alt="search_docker" src="https://user-images.githubusercontent.com/92983658/230719907-acd7695a-0740-46d2-9a67-e55132b21a2d.png">

<br>

<img width="1230" alt="docker_artifactory" src="https://user-images.githubusercontent.com/92983658/230719910-39e141d6-28aa-4dc0-a71a-ce3229de1f59.png">

<br>

<img width="1222" alt="docker_minted" src="https://user-images.githubusercontent.com/92983658/230719914-1d38c4db-199a-4d98-9c07-8a8f80dfffdc.png">

<br>

- create a second Local repository for `jenkins`. Use `generic` package if Jenkins is not available.

<br>

<img width="1229" alt="jenkins_local" src="https://user-images.githubusercontent.com/92983658/230721737-3a2eece7-d327-4d76-9808-3e6ddc51e13a.png">

<br>

<img width="1228" alt="local_repos" src="https://user-images.githubusercontent.com/92983658/230721743-79cbee7a-b2d6-4e3c-99ab-345db0cff7f0.png">

<br>

## Create A Virtual Repository
A virtual repository aggregates several repositories under a common URL. You can get artifacts from it but you cannot deploy anything to it.

- Select virtual Repository 
- give the virtual Repository any name you like, 
- click on the create Virtual Repository button

<br>

<img width="1231" alt="virtual" src="https://user-images.githubusercontent.com/92983658/230720599-620c13e4-8cd2-4554-a972-3f3c55acc6f1.png">

<br>

<img width="1224" alt="generic" src="https://user-images.githubusercontent.com/92983658/230720602-8afe7c0d-6c9f-4e68-847b-fd11976647fd.png">

<br>

<img width="1189" alt="generic_name_repo" src="https://user-images.githubusercontent.com/92983658/230720610-4a3780f5-cd38-4dac-a5be-d7ae539d8a5f.png">

<br>

- Scroll down the page to see the local repositories, select which local repository that will be part of the virtual repository. Click on the double arrows to move them.

<br>

<img width="945" alt="new_items" src="https://user-images.githubusercontent.com/92983658/230720697-24e687e9-b8af-4008-b911-ee0382ee8cc4.png">

<br>

<img width="944" alt="included_items" src="https://user-images.githubusercontent.com/92983658/230720700-13d2fac5-a366-4a69-a0c8-9ab8fc549872.png">

<br>

## Push Docker Images To the Repository
You can either pull and push docker images to the local repository for each application, or simply pull from the virtual repository

### get docker images from docker hub and push to the private registry.
- login to the artifactory repository
- enter jfrog username and password

```

docker login <artifactory url>

## example: docker login mintedcreative.jfrog.io

```

<br>

<img width="866" alt="docker_frog" src="https://user-images.githubusercontent.com/92983658/230721274-8ff31628-2154-425c-a003-d350bde5825e.png">

<br>

- A successful login will create a file here  `~/.docker/config.json`
```
cat .docker/config.json

```

<br>

<img width="1008" alt="docker_config" src="https://user-images.githubusercontent.com/92983658/230721412-08d27818-97cc-47d9-8de8-cabf7b3ce8c1.png">

<br>

- Pull the Jenkins image from Docker hub
- Tag the image so that it can pushed to Artifactory. The image below shows how to get the repository URL address. 
- Push the docker image to Docker Artifactory Repo

```

docker pull jenkins/jenkins:jdk11
docker tag jenkins/jenkins:jdk11 <artifatory docker repo url>/jenkins:jdk11
docker push <artifactory docker repo url>/jenkins:jdk11

```

<br>

<img width="936" alt="jdk" src="https://user-images.githubusercontent.com/92983658/231186302-40fc0d83-6cb9-4f82-8d40-d6a822097c51.png">

<br>

<img width="1227" alt="artifactory_jdk" src="https://user-images.githubusercontent.com/92983658/231186574-d707a4b0-71ac-437e-a404-127a51eaf1bb.png">

<br>

## Jenkins Pipeline For Business Applications

In earlier projects, pipeline for the Tooling app was based on Ansible. This time, the same application will be containerised. Since the app will be running inside a kubernetes cluster within a Pod container, then the approach to CI/CD will be different.

- The Dockerfile used to build the tooling app’s docker image will have its own CI/CD pipeline
- The helm charts used to deploy the application into kubernetes will require its own CI/CD pipeline

<br>

### Deploy Jenkins With Helm
#### Deploy without any custom configuration to the Helm Values:

- Launch an eks cluster with 3 instances. (see <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#deploy-jfrog-artifactory-into-kubernetes">project 25</a> for how to do this and the rest of this section)

```
# create a config file `cluster.yaml`

apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: PBL26
  region: eu-west-2
  version: "1.22"

managedNodeGroups:
  - name: primary
    instanceType: t2.medium
    desiredCapacity: 3
    volumeSize: 20
    spot: true

```

- provision the cluster
```

eksctl create cluster -f cluster.yaml

```

<br>

- Without any custom configuration, get the Jenkins and Sonqube Helm charts from `artifacthub.io`, and deploy using the default values.
```
# add repository to desktop
helm repo add jenkinsci https://charts.jenkins.io/
helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube

# Create a namespace called tools where all the tools for DevOps will be deployed.
kubectl create ns tools 

# Update the helm repo index
helm repo update

# install jenkins & sonarqube into the `tools` namespace
helm upgrade --install my-jenkins jenkinsci/jenkins --version 4.3.20 -n tools
helm upgrade --install my-sonarqube sonarqube/sonarqube --version 10.0.0+521 -n tools

```

<br>

<img width="1469" alt="helm_jenkins" src="https://user-images.githubusercontent.com/92983658/231441382-7146f9bb-e2cd-4532-9060-c95046a45669.png">

<br>

<img width="1303" alt="helm_sonarqube" src="https://user-images.githubusercontent.com/92983658/231441574-71a3a47e-99a7-4ff1-b451-a0263b917829.png">

<br>

- Configure DNS for jenkins and route traffic to the ingress controller load balancer
Deploy an ingress without TLS
Ensure that you are able to access the configured URL
Ensure that you are able to logon to Jenkiins.
Update the Ingress and configure TLS for the URL
