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

- Update the Ingress and configure TLS for the URL

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

```

# create an IAM policy that enables cert-manager to add records to Route53 in order to solve the DNS01 challenge
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


```

# create a certificate issuer
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


# apply configuration
kubectl apply -f cert_manager.yaml


```

<br>

```

# configure ingress for TLS
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



