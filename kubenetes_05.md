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
- <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#getting-the-artifactory-url">Getting The Artifactory URL</a>
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

<br>

<img width="898" alt="artifactory_helm_1a" src="https://user-images.githubusercontent.com/92983658/226585527-1c09e039-c707-4017-ac59-29d866ac8847.png">

<br>

- install artifactory
```

helm upgrade --install my-jfrog-platform jfrog/jfrog-platform --version 10.12.0 -n tools --kubeconfig <kubeconfig file name>


```

<br>

<img width="1240" alt="artifactory_helm_1b" src="https://user-images.githubusercontent.com/92983658/226586249-148a4279-661a-4e2e-8f67-5b0c133d3a61.png">

<br>

*note: `upgrade --install` is used flag here instead of `helm install artifactory jfrog/artifactory`. This is a better practice, especially when developing CI pipelines for helm deployments. It ensures that helm does an upgrade if there is an existing installation. But if there isn’t, it does the initial install. With this strategy, the command will never fail. It will be smart enough to determine if an upgrade or fresh installation is required.*

*The helm chart version to install is very important to specify. So, the version at the time of writing may be different from what you will see from Artifact Hub. So, replace the version number to the desired. You can see all the versions by clicking on "see all" as shown in the image below.*

<br>

<img width="1230" alt="application-varsion" src="https://user-images.githubusercontent.com/92983658/226587124-7108527b-84df-4353-944f-c6902a53fccb.png">

<br>

<img width="1225" alt="application_version_2" src="https://user-images.githubusercontent.com/92983658/226587289-931033fa-cae0-4629-b912-088501624666.png">

<br>

### Getting the Artifactory URL

The artifactory helm chart comes bundled with the Artifactory software, a PostgreSQL database and an Nginx proxy which it uses to configure routes to the different capabilities of Artifactory. 
- Getting the pods after some time, you should see something like the below.
```
kubectl get po -n tools

```
<br>

<img width="722" alt="tools_po" src="https://user-images.githubusercontent.com/92983658/226588428-30d6fffd-7f87-42e3-84b6-15779ebe5b6d.png">

<br>

Each of the deployed application have their respective services.

```
kubectl get svc -n tools --kubeconfig <kubeconfig file name>

```

<br>

<img width="1076" alt="svc_tools" src="https://user-images.githubusercontent.com/92983658/226588950-f0c5994d-5a1c-40d5-bd7f-531cb33ae2f5.png">

<br>

**note:** Notice that, the Nginx Proxy has been configured to use the service type of LoadBalancer. Therefore, to reach Artifactory, we will need to go through the Nginx proxy’s service... Which happens to be a load balancer created in the cloud provider. 

- Run the kubectl command to retrieve the Load Balancer URL.
```
 kubectl get svc my-jfrog-platform-artifactory-nginx -n tools --kubeconfig kubeconfig

```

<br>

<img width="1080" alt="loadbalancer_url" src="https://user-images.githubusercontent.com/92983658/226591964-f8c13fea-d7e8-4979-b46e-6d00e68c785c.png">

<br>

- Copy the URL and paste in the browser