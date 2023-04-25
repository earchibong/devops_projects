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
- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#deploy-jenkins-with-helm">Deploy Jenkins And Sonarqube With Helm</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/private_repositories.md#configure-tlsssl-for-url">Configure TLS/SSL For URL</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#use-an-override-values-file-to-customize-jenkins-deployment">Use An Overide Values File To Customize Jenkins Deployment</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#automate-jenkins-plugin-installation">Automate Jenkins Plugin Installation</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#option-2-packaging-plugins-as-part-of-the-jenkins-image">Custom Jenkins Image: Pre-Install Plugins</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/private_repositories.md#automating-jenkins-configuration-as-code-jcasc">Automating Jenkins Configuration As Code (JCasC)</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/private_repositories.md#automate-the-creation-of-the-tooling-applications-pipeline">Automate Creation Of Tooling Applications</a>

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
# create a config file : cluster.yaml

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

- Without any custom configuration, get the Jenkins and Sonqube Helm charts from <a href="https://artifacthub.io/">artifacthub.io</a>, and deploy using the default values.

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
```
# deploy ingress controller in `ingress-nginx` namespace
helm upgrade --install ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx --create-namespace

# confirm pods in the ingress-nginx namespace
kubectl get pods --namespace=ingress-nginx

# confirm the created load balancer in AWS 
kubectl get service -n ingress-nginx

# Check the IngressClass that identifies this ingress controller.
kubectl get ingressclass -n ingress-nginx

```

<br>

<img width="1467" alt="ingress_deploy" src="https://user-images.githubusercontent.com/92983658/231448222-adb633da-1bf7-4e23-90ce-4e639192529d.png">

<br>

```

# deploy jenkins and sonarqube ingress
# create a file ingress.yaml


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: jenkins
spec:
  ingressClassName: nginx
  rules:
  - host: "tooling.jenkins.<your domain name>"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-jenkins
            port:
              number: 8080
 
---
 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sonarqube
spec:
  ingressClassName: nginx
  rules:
  - host: "tooling.sonarqube.<your domain name>"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-sonarqube-sonarqube
            port:
              number: 9000
              
 ```
 
 <br>
 
 <img width="973" alt="ingress_file" src="https://user-images.githubusercontent.com/92983658/231450734-84c2bc6d-d44e-4738-9d91-20cd8399dba2.png">

<br>

```

# apply in `tools` namespace
kubectl apply -f ingress.yaml -n tools

# check ingressclass
kubectl get ingress.networking.k8s.io -n tools


```

<br>

<img width="1384" alt="ingress_class" src="https://user-images.githubusercontent.com/92983658/231451857-b1014456-fc9b-4599-a38b-e6884076dacf.png">

<br>

configure DNS : create Alias records.

<img width="1395" alt="record_1" src="https://user-images.githubusercontent.com/92983658/231453286-cc882a1b-3505-4cb8-891d-cd27383782fc.png">

<img width="1388" alt="record_2" src="https://user-images.githubusercontent.com/92983658/231453191-53199640-5b10-4839-8369-c848fa63a644.png">

<br>

- access the configured URL: `tooling.jenkins.<your domain>` and `tooling.sonarqube.<your domain>`

<br>

<img width="1231" alt="tooling_jenkins" src="https://user-images.githubusercontent.com/92983658/231455415-624cb2cc-c082-46aa-9e93-416f60cf5da7.png">

<br>

<img width="1230" alt="tooling_soqnarqube" src="https://user-images.githubusercontent.com/92983658/231455449-f5abf3b0-253e-449a-a1d2-a74d3cb73f3b.png">

<br>

- Ensure that you are able to logon to Jenkins and sonarqube

```

# get jenkins admin password
kubectl exec --namespace tools -it svc/my-jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo

# sonarqube details
user: admin
password: admin

```

<br>

<img width="1230" alt="jenkins_logon" src="https://user-images.githubusercontent.com/92983658/231457254-f6dce1ac-8ec6-4e90-b942-8570a6d7bb96.png">

<br>

<img width="1231" alt="sonar_logon" src="https://user-images.githubusercontent.com/92983658/231457291-9f1dc33e-7059-4fea-ac10-85d6066366d6.png">

<br>

## Configure TLS/SSL For URL
```

# deploy cert manager
# install cert manager CustomResourceDefinition (CRD) and chart

#CustomResourceDefinition 
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml

#chart
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm upgrade --install cert-manager jetstack/cert-manager -n cert-manager --create-namespace

```

<br>

<img width="1415" alt="cert_manager" src="https://user-images.githubusercontent.com/92983658/231459021-fc934aa8-57ca-4f67-80f5-b7bfce3f1e71.png">

<br>

- create an IAM policy that enables cert-manager to add records to Route53 in order to solve the DNS01 challenge

```

# in IAM dashboard, create a policy with the follwing permissions:


{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "route53:GetChange",
      "Resource": "arn:aws:route53:::change/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets",
        "route53:ListResourceRecordSets"
      ],
      "Resource": "arn:aws:route53:::hostedzone/<your hosted zone id>"
    },
    {
      "Effect": "Allow",
      "Action": "route53:ListHostedZonesByName",
      "Resource": "*"
    }
  ]
}

# once created, attach the policy to your cluster (select it from the menu)

```

<br>

<img width="1470" alt="cert_manager_policy" src="https://user-images.githubusercontent.com/92983658/231461220-50f99aa8-a6c7-4e24-a244-a38b2b1ef71b.png">

<br>

<img width="1227" alt="attach_poliocy" src="https://user-images.githubusercontent.com/92983658/231460777-cf0d3b6c-d833-4976-8230-e939ab0df8ad.png">

<br>

- alternatively, attach the role to your policy with AWS CLI as follows:

The `attach-role-policy` command can be used to attach an `IAM policy to an IAM role`. The arguments for the command are:

  - `role-name`: Name of the IAM role
  - `policy-arn`: ARN of the IAM policy you want to attach

```

aws iam attach-role-policy --role-name <your IAM role> --policy-arn <cert manager policy arn>


```

<br>

<img width="1470" alt="iam_policy_aatach" src="https://user-images.githubusercontent.com/92983658/232032392-d5def8c9-24c9-40b2-ab08-4cb3d6671153.png">

<br>


- create route53 secret crediatials for cert-manager
```

kubectl --namespace cert-manager \
create secret generic route53-credentials \
--from-literal="secret-access-key=<YOUR-AWS-SECRET-ACCESS-KEY>"

```

<br>

- create a certificate issuer

```


# create a file named cert_manager.yaml and deploy with kubectl

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  namespace: "cert-manager"
  name: "letsencrypt-prod"
spec:
  acme:
    server: "https://acme-v02.api.letsencrypt.org/directory"
    email: "infradev@oldcowboyshop.com"
    privateKeySecretRef:
      name: "letsencrypt-prod"
    solvers:
    - selector:
        dnsZones:
          - "archibong.link"
      dns01:
        route53:
          region: "eu-west-2"
          hostedZoneID: "<your route 53 hosted zone id>"
          accessKeyID: "<YOUR AWS ACCESS KEY ID>"
          secretAccessKeySecretRef:
              key: secret-access-key
              name: route53-credentials

# apply configuration
kubectl apply -f cert_manager.yaml


```

<br>

- configure ingress for TLS

```

# update ingress.yaml file


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/ingress.class: nginx
  name: jenkins
spec:
  tls:
  - hosts:
    - "tooling.jenkins.<your domain>"
    secretName: "tooling.jenkins.<your domain>"
  rules:
  - host: "tooling.jenkins.<your domain>"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-jenkins
            port:
              number: 8080

---


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/ingress.class: nginx
  name: sonarqube
spec:
  tls:
  - hosts:
    - "tooling.sonarqube.<your domain>"
    secretName: "tooling.sonarqube.<your domain>"
  rules:
  - host: "tooling.sonarqube.<your domain>"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-sonarqube-sonarqube
            port:
              number: 9000

# redeploy update:
kubectl apply -f ingress.yaml -n tools

```

<br>

<img width="1063" alt="ingress_tls" src="https://user-images.githubusercontent.com/92983658/231464446-e9cc27f8-dd42-4ca7-83e3-4dc4e14da93f.png">

<br>

<img width="1133" alt="ingress_redeploy" src="https://user-images.githubusercontent.com/92983658/231464929-02429731-6a80-46f2-a670-2f5cf2c002a9.png">

<br>

- run the following to confirm certificate issued
```

kubectl get certificaterequest -n tools
kubectl get order -n tools
kubectl get challenge -n tools
kubectl get certificate -n tools

```

<br>

<img width="1020" alt="confirm" src="https://user-images.githubusercontent.com/92983658/231471533-937988ef-83e6-4c52-b414-d212d202d186.png">

<br>

- head over to the browser, you should see the padlock sign without warnings of untrusted certificates.

<br>

<img width="1227" alt="jenkins_tls" src="https://user-images.githubusercontent.com/92983658/231471976-507eb090-ceb6-4d98-8498-c2a8a5dfadc7.png">

<br>

<img width="1229" alt="sonar_tls" src="https://user-images.githubusercontent.com/92983658/231471997-6a78dc63-e458-4ee6-b6e2-88c981951ab0.png">

<br>

## Use an override values file to customize Jenkins deployment
### Configure Jenkins Ingress using Helm Values

- delete the previous jenkins ingress
```

kubectl delete ingress.networking.k8s.io/jenkins -n tools

```

- Create a new file and name it `jenkins-values-overide.yaml`
- on `artifacthub.io` search for the term `controller` find the below section in the default values file. Copy and paste it exactly in the `jenkins-values-overide.yaml`
```

controller:
  componentName: "jenkins-controller"
  image: "jenkins/jenkins"
  tagLabel: jdk11
  imagePullPolicy: "Always"
  testEnabled: true

```

<br>

<img width="972" alt="overide_1a" src="https://user-images.githubusercontent.com/92983658/234256978-fe135c94-1a98-4fb6-adef-5bc1c0fc38dc.png">

<br>

<img width="1036" alt="controller_2" src="https://user-images.githubusercontent.com/92983658/231479892-4ab02da6-f974-4d58-a778-950d72e072b3.png">

<br>

- upgrade the `jenkins-values-overide.yaml` file with helm
```

helm upgrade -i my-jenkins jenkinsci/jenkins -n tools -f jenkins-values-overide.yaml

```

<br>

<img width="1382" alt="jenkins_upgraded" src="https://user-images.githubusercontent.com/92983658/231481292-896348ea-71ea-4eb1-a760-82f4a50a0af9.png">

<br>

*note: The above upgrade has been done, but without any specific changes to the existing deployment. That's because no configuration change has occured. All we now have is a shortened values file that can be easily read without too many options. With this file, you can now start making configuration updates.*

<br>

- To configure Jenkins ingress directly from the helm values, simply search for the `ingress:` section in the default values file and copy the entire section to the override values file. The default one should look similar to this:
```

ingress:
enabled: false
   # Override for the default paths that map requests to the backend
paths: []
   # - backend:
   # serviceName: ssl-redirect
   # servicePort: use-annotation
   # - backend:
   # serviceName: >-
   # {{ template "jenkins.fullname" . }}
   # # Don't use string here, use only integer value!
   # servicePort: 8080
   # For Kubernetes v1.14+, use 'networking.k8s.io/v1beta1'
   # For Kubernetes v1.19+, use 'networking.k8s.io/v1'
apiVersion: "extensions/v1beta1"
labels: {}
annotations: {}
   # kubernetes.io/ingress.class: nginx
   # kubernetes.io/tls-acme: "true"
   # For Kubernetes >= 1.18 you should specify the ingress-controller via the field ingressClassName
   # See https://kubernetes.io/blog/2020/04/02/improvements-to-the-ingress-api-in-kubernetes-1.18/#specifying-the-class-of-an-ingress
   # ingressClassName: nginx
   # Set this path to jenkinsUriPrefix above or use annotations to rewrite path
   # path: "/jenkins"
   # configures the hostname e.g. jenkins.example.com
hostName:
tls:
   # - secretName: jenkins.cluster.local
   # hosts:
   # - jenkins.cluster.local
   
```

<br>

<img width="1225" alt="jenkins_default_ingress" src="https://user-images.githubusercontent.com/92983658/231482644-3b2e1650-bef2-416d-a531-a3f46c166b0b.png">

<br>

- update the file with values similar to the previously deleted jenkins file. The final output should be similar to this:
```

ingress:
  enabled: true
  apiVersion: "extensions/v1beta1"
  annotations: 
    cert-manager.io/cluster-issuer: "letsencrypt-production"
    kubernetes.io/ingress.class: nginx
  hostName: tooling.jenkins.<your domain>
  tls:
  - secretName: tooling.jenkins.<your domain>
    hosts:
      - tooling.jenkins.<your domain>

```

<br>

<img width="1113" alt="ingress_override" src="https://user-images.githubusercontent.com/92983658/231492930-0fef3e5b-085f-4d9e-a541-411547db6989.png">

<br>

- upgrade deployment
```

helm upgrade -i my-jenkins jenkinsci/jenkins -n tools -f jenkins-values-overide.yaml

```

<br>

<img width="1383" alt="jenkins_o" src="https://user-images.githubusercontent.com/92983658/231492981-f7fdce4e-0077-4b1d-ae4d-9651ab1ab47a.png">

<br>

## Automate Jenkins plugin installation
There are 2 possible options to this.

- use Helm values to automate plugin installation or ...
- package the required plugins as part of the Jenkins image.

Which ever option works just fine, Its a matter of choice and unique environment setup in an organisation.

### Option 1: Using Helm values to automate plugin installation
The easiest and most straight forward approach. But it may slow down initial deployment of Jenkins since it has to download the plugins. Also, if the kubernetes workers are completely closed from the internet, downloading plugins over the internet will not work, in this case Option 2 is the way out.


```

# List of plugins to be install during Jenkins controller start
  installPlugins:
    - kubernetes:3600.v144b_cd192ca_a_
    - workflow-aggregator:581.v0c46fa_697ffd
    - git:4.11.3
    - configuration-as-code:1429.v09b_044a_c93de

  # Set to false to download the minimum required version of all dependencies.
  installLatestPlugins: true

  # Set to true to download latest dependencies of any plugin that is requested to have the latest version.
  installLatestSpecifiedPlugins: false

  # List of plugins to install in addition to those listed in controller.installPlugins
  additionalPlugins: []
  
```
<br>

In the override yaml file, you can add the `installPlugins:` and `additionalPlugins:` so that your updated override values file will look like the below.

```

controller:
  ingress:
    enabled: true
    apiVersion: "extensions/v1beta1"
    annotations: 
      cert-manager.io/cluster-issuer: "letsencrypt-production"
      kubernetes.io/ingress.class: nginx
    hostName: tooling.jenkins.sandbox.svc.darey.io
    tls:
    - secretName: tooling.jenkins.sandbox.svc.darey.io
      hosts:
        - tooling.jenkins.sandbox.svc.darey.io

  installPlugins:
    - kubernetes:3600.v144b_cd192ca_a_
    - workflow-aggregator:581.v0c46fa_697ffd
    - git:4.11.3
    - configuration-as-code:1429.v09b_044a_c93de

  additionalPlugins: []
  
  ```
  
  <br>
  
  <img width="1225" alt="install_plugins" src="https://user-images.githubusercontent.com/92983658/231494985-64c80ce4-2acc-44aa-94f3-c7f13b592abc.png">

<br>

*note: use the `additionalPlugins: []` key. the `[]` there simply means the key has a default null value and it is a list data type. So, to start adding more plugins you simply need to update that section like below.*

```

additionalPlugins:
    -blueocean:1.27.3    -credentials-binding:1.24    -git-changelog:3.29    -git-client:3.6.0    -git-server:1.9    -git:4.5.1

```

<br>

<img width="1223" alt="additional_plugins" src="https://user-images.githubusercontent.com/92983658/231495542-5edd6577-25bb-4d35-b84d-11068e782c59.png">

<br>

<img width="1029" alt="install_plugins_1d" src="https://user-images.githubusercontent.com/92983658/232479125-213ab4cb-8e4a-47f2-95c2-f19cbb61a6a9.png">

<br>

- upgrade deployment
```

helm upgrade -i my-jenkins jenkinsci/jenkins -n tools -f jenkins-values-overide.yaml

```

<br>

- verify plugins have been installed
```

#  exec into the pod container and check the filesystem
kubectl exec <pod name> -n tools -it -- bash 
ls -ltr /var/jenkins_home/plugins/ | grep blueocean::1.27.3 # insert any plugin name after grep to confirm install


```

<br>





### option 2: Packaging plugins as part of the Jenkins image
Create a folder structure and empty files like below:
```
PBL26
├── Dockerfile
└── plugins.txt
      
         
```

<br>

- update each line of that `plugins.txt` file with the official Plugin ID of eah plugin, you can find the official Plugin Ids from https://plugins.jenkins.io/ .For Example: `Git client` Plugin has an official ID as : `git-client`
```

workflow-basic-steps:1010.vf7a_b_98e847c1
blueocean:1.27.3
credentials-binding:604.vb_64480b_c56ca_
git-changelog:3.29
git-client:4.2.0
git-server:99.va_0826a_b_cdfa_d
git:5.0.0
    
     
```

<br>

<img width="1055" alt="plugins_txt" src="https://user-images.githubusercontent.com/92983658/232489242-8372bcba-a4f7-4f55-8682-320ce37be559.png">

<br>

- update `Dockerfile` with the following:
```

FROM jenkins/jenkins:lts-jdk11

# if we want to install via apt...switch to root user
USER root

# update and upgrade apt package manager
RUN apt-get update && apt-get upgrade -y 

#install pre-built plugins by copying the plugin HPI file into /usr/share/jenkins/ref/plugins/
# without plugin HPI file, you can install directly using RUN jenkins-plugin-cli --plugins plugin1 plugin2 plugin3...

COPY --chown=jenkins:jenkins plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli -f /usr/share/jenkins/ref/plugins.txt


# drop back to the regular jenkins user
USER jenkins

```

*note: get more information custom jenkins images with pre-installed tools <a href="https://github.com/jenkinsci/docker#installing-more-tools">here</a> and <a href="https://github.com/jenkinsci/helm-charts/blob/main/charts/jenkins/README.md#consider-using-a-custom-image">here</a>
<br>

<img width="1298" alt="dockerfile_1b" src="https://user-images.githubusercontent.com/92983658/232490315-78222ef0-386b-4ec8-b9d6-67a1634ccf4b.png">

<br>

- Run docker commands to build, tag and push the docker image to the artifactory Docker registry created at the start of the project.
```

docker build -t jenkins:1.2.1 .

````

<br>

*Docker build:*
- *`-t` is for tagging the image.*
- *`jenkins` is the name of the image.*
- *`1.2.1` is the tag name. If you don’t add any tag, it defaults to the tag named latest.*
- *`.` means, we are referring to the Dockerfile location as the docker build context.*

<br>

<img width="1446" alt="docker_build" src="https://user-images.githubusercontent.com/92983658/232501481-17c3498a-4b91-4080-b089-c70927a7ef84.png">

<br>

```

docker login <artifactory url>
docker tag jenkins:1.2.1 <artifactory docker repo url>/jenkins:1.2.1
docker push <artifactory docker repo url>/jenkins:1.2.1

```

<br>

<img width="1364" alt="jenkins_push_custom" src="https://user-images.githubusercontent.com/92983658/232504395-71897717-59cd-45bf-8e78-0fc4fa3a6f41.png">

<br>

- create a secret to store artifactory credentials used to pull image from private registry... we will name the secret: `regcred`
```

kubectl create secret docker-registry regcred \
  --namespace tools \
  --docker-server=<artifactory url> \
  --docker-username=<artifactory-username> \
  --docker-password=<artifactory-password>

# confirm secret in tools namespace
kubectl get secrets -n tools

```

<br>

<img width="1297" alt="secret" src="https://user-images.githubusercontent.com/92983658/233042798-bd0cd22f-d02f-4e18-9bf6-aea5270553eb.png">

<br>

- update the `image:` value in `jenkins-values-overide.yaml` file
```

# Once you built the image and pushed it to your registry you can specify it in your values file like this:

controller:
  image: "mintedcreative.jfrog.io/docker-local/jenkins"
  tag: "1.2.1"
  imagePullSecretName: regcred
  installPlugins: false
  imagePullPolicy: "Always"



#upgrade deployment
helm upgrade -i my-jenkins jenkinsci/jenkins -n tools -f jenkins-values-overide.yaml


```

<br>

<img width="1180" alt="install_plugin_false" src="https://user-images.githubusercontent.com/92983658/232511293-4d4c6a05-d321-4bbe-b5e1-4a9f55130854.png">

<br>


- verify the installation of the plugins 

```
JENKINS_HOST=username:password@myhost.com:port #username: admin and password: jenkins secret password
curl -sSL "http://$JENKINS_HOST/pluginManager/api/xml?depth=1&xpath=/*/*/shortName|/*/*/version&wrapper=plugins" | perl -pe 's/.*?<shortName>([\w-]+).*?<version>([^<]+)()(<\/\w+>)+/\1 \2\n/g'|sed 's/ /:/'

```

The above should return files relating to the plugin. If it returns empty, then the plugin has not been installed.

<br>

## Automating Jenkins Configuration As Code (JCasC)
Manually updating configs from the Jenkins user interface (UI) is not sustainable. Imagine creating a lot of folders to manage multiple projects and pipelines from the Jenkins UI and losing the Jenkins installation afterwards. You will have to manually recreate all the folders again. With Jenkins Configuration As Code (JCasC), this process can be automated and all configurations in Jenkins can now be represented as “code”

*note:Ensure that all the installed plugins are using the latest version. Visit `https://plugins.jenkins.io/`, then search for the plugin to get the latest version number.*

- To start managing Jenkins as code, search for `JCasC:` within the default yaml values file
- Copy that section out of the default and put it in the override yaml file.
```

JCasC:
defaultConfig: true
configScripts: {}
    # welcome-message: |
    # jenkins:
    # systemMessage: Welcome to our CI\CD server. This Jenkins is configured and managed 'as code'.
    # Ignored if securityRealm is defined in controller.JCasC.configScripts and
securityRealm: |-
local:
allowsSignup: false
enableCaptcha: false
users:
- id: "${chart-admin-username}"
name: "Jenkins Admin"
password: "${chart-admin-password}"
    # Ignored if authorizationStrategy is defined in controller.JCasC.configScripts
authorizationStrategy: |-
loggedInUsersCanDoAnything:
allowAnonymousRead: false

```

<br>

The `configScripts: {}` key shows that it is empty. The curly brackets `{}` indicates that it is configured to hold a dictionary type of data. This means that it can hold sub-keys with their own respective key and values. As you can see in the example below it.

```

configScripts: {}
    #  welcome-message: |
    #    jenkins:
    #      systemMessage: Welcome to our CI\CD server.  This Jenkins is configured and managed 'as code'.
    
```

<br>

To enable the section, simply remove the `{}` and uncomment the first key welcome-message. You can write whatever you want in the `systemMessage`. For example:

```

configScripts:
      welcome-message: |
        jenkins:
          systemMessage: Welcome to our CI\CD server.  This Jenkins is configured and managed strictly 'as code'. Please do not update Manually
          
```

<br>

<img width="1386" alt="jasc_msg" src="https://user-images.githubusercontent.com/92983658/232764830-ccedba6a-8bb5-4480-801c-a045b6ef514a.png">

<br>

- Upgrade Jenkins with the latest update and you should see the system message
```

helm upgrade -i my-jenkins jenkinsci/jenkins -n tools -f jenkins-values-overide.yaml

```

<br>

<img width="1398" alt="jacs_1b" src="https://user-images.githubusercontent.com/92983658/232766094-2fcfe280-be9f-4d8d-8ece-686a17ee3dec.png">

<br>

### Latest Configurations Applied To Jenkins Through JCasC And Helm

- Navigate to `Configuration as code` section in Jenkins UI. 
  - Click on `Configuration as Code`
  - view the updated code here

<br>

<img width="1399" alt="jcasc_1c" src="https://user-images.githubusercontent.com/92983658/232768617-1de93c8f-8c8f-4d80-9340-364df9ee333d.png">

<br>
  

## Automate The Creation Of The Tooling Application’s Pipeline.

- Create an access token from GitHub so that Jenkins canm use it to connect to the Github account. https://github.com/settings/tokens

<br>

<img width="1391" alt="access_token" src="https://user-images.githubusercontent.com/92983658/232772138-520ddfb1-1bc9-4679-a70f-da4d814b05a4.png">

<br>

- Using <a href"https://www.base64decode.org/">base64<a/>, encode the generated token
- Create a secret in the same namespace where Jenkins is installed. Name the key `github_token` or whatever you wish
```

kubectl --namespace tools \
create secret generic github-credentials \
--from-literal=github_token='<YOUR-BASE64-TOKEN>'

# You must use single quotes '' to escape special characters such as $, \, *, =, and ! in your strings. 
#If you don't, your shell will interpret these characters.

#inspect secret
kubectl get secret <your secret name> --output=yaml -n tools
```

<br>

<img width="1079" alt="secret" src="https://user-images.githubusercontent.com/92983658/232780621-b38fe221-5028-4283-81fc-7ee9726dc13b.png">

<br>

- In the Jenkins values file, set the values correctly so that the secret created above can be used
```
# in values-overide file, add the following in the controller section:

controller:
  # the 'name' and 'keyName' are concatenated with a '-' in between, so for example:
  # an existing secret "secret-credentials" and a key inside it named "github-password" should be used in Jcasc as ${secret-credentials-github-password}
  # 'name' and 'keyName' must be lowercase RFC 1123 label must consist of lower case alphanumeric characters or '-',
  # and must start and end with an alphanumeric character (e.g. 'my-name',  or '123-abc')
  # existingSecret existing secret "secret-credentials" and a key inside it named "github-username" should be used in Jcasc as ${github-username}
  # When using existingSecret no need to specify the keyName under additionalExistingSecrets.
  existingSecret: secret-credentials

  additionalExistingSecrets:
    - name: github-credentials
      keyName: github-token
   
 ```

<br>
 
<img width="1377" alt="secret_1f" src="https://user-images.githubusercontent.com/92983658/232783508-5c9e3049-79fb-445d-935a-f28f25ef360e.png">

<br>

- create a folder to hold your pipelines

```

JCasC:
    enabled: true
    configScripts:
      welcome-message: |
        jenkins:
          systemMessage: Welcome to our CI\CD server.  This Jenkins is configured and managed strictly 'as code'. Please do not update Manually
      pipeline: |
        jobs:
          - script: >
              folder('PBL26') {
                displayName('PBL26')
                description('Contains PBL26 Jenkins Pipelines')
              }
              
              
  ```
  
<br>

<img width="1382" alt="pipeline" src="https://user-images.githubusercontent.com/92983658/232785227-3e6c711e-6d69-46ab-b00a-adf6dc9f5f41.png">

<br>

- apply the latest changes, you should be able to see the folder created as shown below. But it doesn’t have any pipline.
```
helm upgrade -i my-jenkins jenkinsci/jenkins -n tools -f jenkins-values-overide.yaml

```




