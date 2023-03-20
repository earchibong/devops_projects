# Deploying And Packaging Applications Into Kubenetes With Helm

In <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_04.md">Project 24</a>,  we experienced helm as a tool 
used to deploy an application into Kubernetes. We installed several tools apart from Jenkins.

In this project, we will:

- deploy more DevOps tools, get familiar with some of the real world issues faced during such deployments 
and how to fix them.

- we will learn how to tweak helm values files to automate the configuration of the applications you deploy. 
- Finally, once most of the DevOps tools have been deployed, we will use them and relate with the DevOps cycle and 
how they fit into the entire ecosystem.

<br>

In this project, the requirement is to use Jfrog Artifactory as a private registry for the organisation’s Docker images and 
Helm charts. This requirement will satisfy part of the company’s corporate security policies to never download artifacts 
directly from the public into production systems. 

We will eventually have a CI pipeline that initially pulls public docker images and helm charts from the internet, 
store in artifactory and scan the artifacts for security vulnerabilities before deploying into the corporate infrastructure. 
Any found vulnerabilities will immediately trigger an action to quarantine such artifacts.

<br>

## Labs
- <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#deploy-jfrog-artifactory-into-kubernetes">Deploy Jfrog Artifactory into Kubernetes</a>
- 

<br>
  
## Deploy Jfrog Artifactory into Kubernetes

- Provision and EKS cluster with terraform
- Search for an official helm chart for Artifactory on <a href="https://artifacthub.io/">Artifact Hub</a>
- Review the Artifactory page
- Click on the install menu on the right to see the installation commands.

<br>
  
<img width="1230" alt="jfrog_artifact" src="https://user-images.githubusercontent.com/92983658/226357976-3811a4ad-359c-417f-bac4-ac00c3c76ca2.png">

<br>
  
<img width="1226" alt="jfrog_artifact_1b" src="https://user-images.githubusercontent.com/92983658/226358080-69dcf7e5-6f3b-4c79-9a8c-c079c34b6df6.png">

<br>
  
- Add the jfrog remote repository on your laptop/computer
```
helm repo add jfrog https://charts.jfrog.io

```

- Create a namespace called tools where all the tools for DevOps will be deployed.
```
kubectl create ns tools

```

- Update the helm repo index on your laptop/computer
```
helm repo update

```


