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
- <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#deploy-ingress-controller-and-manage-ingress-resources">Deploy Ingress Controller & Manage Ingress Resources</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#deploy-nginx-ingress-controller">Deploy Nginx Ingress Controller</a>

<br>
  
## Deploy Jfrog Artifactory into Kubernetes

- create a config file `cluster.yaml`. ensure volume size is set to `200GB`
```

apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: PBL25
  region: eu-west-2
  version: "1.22"

managedNodeGroups:
  - name: primary
    instanceType: m5.large
    desiredCapacity: 3
    volumeSize: 200
    spot: true

```

<br>

- provision the cluster
```
eksctl create cluster -f cluster.yaml

```

*note: we're using an older version of kubernetes for this...`version 1.22` as new versions seem to create a `start probe fail` error in this exercise*

<br>


- Search for an official helm chart for Artifactory on <a href="https://artifacthub.io/">Artifact Hub</a>
- Review the Artifactory page
- Click on the install menu on the right to see the installation commands.

<br>
  
<img width="1230" alt="jfrog_artifact" src="https://user-images.githubusercontent.com/92983658/226357976-3811a4ad-359c-417f-bac4-ac00c3c76ca2.png">

<br>
  
<img width="1219" alt="artifactory_1b" src="https://user-images.githubusercontent.com/92983658/226611084-a04a43e5-e4c4-44d2-ab82-fa744c655e1f.png">

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

<img width="1386" alt="helm_repo_update" src="https://user-images.githubusercontent.com/92983658/226611464-c7b4dbe6-5058-4cac-aabb-b3751957cb73.png">

<br>

- install artifactory
```

helm upgrade --install my-artifactory jfrog/artifactory --version 107.33.12 -n tools

```

<br>

<img width="1386" alt="artifactory_install" src="https://user-images.githubusercontent.com/92983658/226611820-79ad6919-99cc-4e84-b724-cf6fb07441d1.png">

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

<img width="1257" alt="kubectl_po" src="https://user-images.githubusercontent.com/92983658/227951728-a9eef006-5718-4b06-bd9b-87f2821e87c4.png">

<br>

Each of the deployed application have their respective services.

```
kubectl get svc -n tools 

```
 
<br>

<img width="1384" alt="svc_kube" src="https://user-images.githubusercontent.com/92983658/227955620-32f3ef85-d0e3-4ab1-b40e-a7ac59308e6b.png">

<br>

**note:** Notice that, the Nginx Proxy has been configured to use the service type of LoadBalancer. Therefore, to reach Artifactory, we will need to go through the Nginx proxy’s service... Which happens to be a load balancer created in the cloud provider. 

- Run the kubectl command to retrieve the Load Balancer URL.
```
 kubectl get svc my-artifactory-artifactory-nginx -n tools

```

<br>

<img width="1387" alt="artifactory_svc" src="https://user-images.githubusercontent.com/92983658/227956250-35b74591-f02b-419f-b6ae-d7fc3d44fc9a.png">

<br>

- Copy the URL and paste in the browser. The default username is `admin` and The default password is `password`

<br>

<img width="1231" alt="jfrog_1a" src="https://user-images.githubusercontent.com/92983658/227957245-25e18a0a-9d8b-4162-93a2-478e770237c7.png">

<br>

<img width="1230" alt="jfrog_1b" src="https://user-images.githubusercontent.com/92983658/227958816-965f5bf7-4390-4692-b4bf-9b2a4b788e7e.png">

<br>

**Side note:** Helm uses the `values.yaml` file to set every single configuration that the chart has the capability to configure. THe best place to get started with an off the shelve chart from `artifacthub.io` is to get familiar with the `DEFAULT VALUES`

- click on the `DEFAULT VALUES` section on Artifact hub. Here, you can search for key and value pairs. When `nginx` is typed in the search bar, it shows all the configured options for the nginx proxy. selecting `nginx.enabled` from the list will take you directly to the configuration in the YAML file.

<br>

<img width="1229" alt="default_values" src="https://user-images.githubusercontent.com/92983658/227960914-201a5d91-2f80-4357-895e-f78239f0b46a.png">

<br>

<img width="1229" alt="nginx" src="https://user-images.githubusercontent.com/92983658/227961322-edd2cabd-7be5-4d06-8639-9a6fb07bfd00.png">

<br>

<img width="1228" alt="nginx_enabled" src="https://user-images.githubusercontent.com/92983658/227961434-2aec1e45-190a-4ba3-8285-9a336efaaff7.png">

<br>

- Search for `nginx.service` and select `nginx.service.type`...You will see the confired type of Kubernetes service for Nginx. As you can see, it is LoadBalancer by default

<br>

<img width="1224" alt="loadbalancer" src="https://user-images.githubusercontent.com/92983658/227962008-86347513-db51-41bd-99b4-8d5889c90179.png">

<br>

*To work directly with the `values.yaml` file, you can download the file locally by clicking on the download icon. Setting the service type to Load Balancer is the easiest way to get started with exposing applications running in kubernetes externally. But provissioning load balancers for each application can become very expensive over time, and more difficult to manage. Especially when tens or even hundreds of applications are deployed.*

*The best approach is to use `Kubernetes Ingress` instead. But to do that, an `Ingress Controller` will have to be deployed.
A huge benefit of using the ingress controller is a single load balancer can be used for different applications deployed. Therefore, Artifactory and any other tools can reuse the same load balancer.*

<br>

## Deploy Ingress Controller And Manage Ingress Resources

An ingress is an API object that manages external access to the services in a kubernetes cluster. It is capable to provide load balancing, SSL termination and name-based virtual hosting. In otherwords, Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

In order for the Ingress resource to work, the cluster must have an ingress controller running.

Unlike other types of controllers which run as part of the kube-controller-manager. Such as the Node Controller, Replica Controller, Deployment Controller, Job Controller, or Cloud Controller. Ingress controllers are not started automatically with the cluster.

It is possible to deploy any number of ingress controllers in the same cluster. That is the essence of an ingress class. By specifying the spec `ingressClassName` field on the ingress object, the appropriate ingress controller will be used by the ingress resource.

### Deploy Nginx Ingress Controller

- Using the Helm approach, according to the official guide, install Nginx Ingress Controller in the `ingress-nginx` namespace
```

helm upgrade --install ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx --create-namespace

```

<br>

<img width="1298" alt="nginx_ingress" src="https://user-images.githubusercontent.com/92983658/227966874-34716171-ed6f-413e-b0af-693fc7a7db25.png">

<br>

**side note:** Delete the installation after running above command. Then try to re-install it using a slightly different method you are already familiar with.

```

# delete installation:
helm delete ingress-nginx -n ingress-nginx

# add repo
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx -n ingress-nginx
helm repo update
helm upgrade --install my-nginx-ingress ingress-nginx/ingress-nginx -n ingress-nginx

```

<br>

<img width="1142" alt="helm_repo_ingress_nginx" src="https://user-images.githubusercontent.com/92983658/227969299-80cf32a0-bfff-4fa2-ba56-17b354fb6418.png">

<br>

<img width="1331" alt="my_nginx_ingress" src="https://user-images.githubusercontent.com/92983658/227971673-482624f0-0718-4841-9832-63e1590ce043.png">

<br>

- confirm pods in the `ingress-nginx` namespace:
```

kubectl get pods --namespace=ingress-nginx

  
```
  
<br>

<img width="1197" alt="ingress_pods" src="https://user-images.githubusercontent.com/92983658/227973042-59c816e0-60fc-4a14-a4c9-06a8bd203b60.png">

<br>

- confirm the created load balancer in AWS
```
kubectl get service -n ingress-nginx

```

<br>

<img width="1470" alt="ingress_loadbalancer_1d" src="https://user-images.githubusercontent.com/92983658/227985810-2a7b1a55-6337-472b-ace4-733f6df5a05b.png">

<br>

The ingress-nginx-controller service created is a type of LoadBalancer. it will be used by all applications which require external access, and is using this ingress controller.


<br>

- Check the IngressClass that identifies this ingress controller.
```

kubectl get ingressclass -n ingress-nginx

```

<br>

<img width="925" alt="ingress_class" src="https://user-images.githubusercontent.com/92983658/227987298-9756e181-df4e-4b36-9b2b-c81441805de9.png">

<br>

### Deploy Artifactory Ingress

- create a file `artifactory-ingress.yaml`
```

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: artifactory
spec:
  ingressClassName: nginx
  rules:
  - host: "tooling.artifactory.<your domain name>"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: artifactory
            port:
              number: 8082
              
```

<br>

<img width="1341" alt="artifactory_ingress_1c" src="https://user-images.githubusercontent.com/92983658/227993192-3eb50d62-f9b7-404b-9c07-d12c405adccb.png">


<br>

- apply in `tools` namespace 

```

kubectl apply -f artifactory_ingress.yaml -n tools


```

<br>

<img width="1102" alt="artifactory_ingress_apply" src="https://user-images.githubusercontent.com/92983658/227988571-405fedda-a581-404d-955f-d37bc903aee6.png">

<br>

- check ingressclass
```

kubectl get ingress.networking.k8s.io/artifactory -n tools

```

<br>

<img width="1341" alt="artifactory_ingress_1d" src="https://user-images.githubusercontent.com/92983658/227993513-10689e07-9757-43b8-a83f-ec7d6c2aef87.png">

<br>

Now, take note of

**CLASS** – The nginx controller class name nginx
**HOSTS** – The hostname to be used in the browser tooling.artifactory.archibong.link
**ADDRESS** – The loadbalancer address that was created by the ingress controller

<br>

#### Configure DNS

If anyone were to visit the tool, it would be very inconvenient sharing the long load balancer address. Ideally, a DNS record that is human readable is created and directs requests to the balancer. This is exactly what has been configured in the ingress object - host: `"tooling.artifactory.archibong.link"` but without a DNS record, there is no way that host address can reach the load balancer.

The `archibong.link` part of the domain is the configured HOSTED ZONE in AWS. So you will need to configure Hosted Zone in AWS console or as part of your infrastructure as code using terraform.

<br>

<img width="1230" alt="route_53_hosted_zone" src="https://user-images.githubusercontent.com/92983658/227994714-ecabc941-a976-44b0-8477-a13e6dfcd2eb.png">

<br>

#### Create Route 53 Record

- create the record to point to the ingress controller’s loadbalancer. There are 2 options. You can either use the` CNAME` or `AWS Alias`

**CNAME METHOD**

- Select the HOSTED ZONE you wish to use, and click on the create record button
- Add the subdomain `tooling.artifactory`, and select the record type `CNAME`
- Successfully created record
- Confirm that the DNS record has been properly propergated. Visit https://dnschecker.org and check the record. Ensure to select CNAME. The search should return green ticks for each of the locations on the left.


**ALIAS METHOD**
- In the create record section, type in the record name, and toggle the alias button to enable an alias. An alias is of the A DNS record type which basically routes directly to the load balancer. In the choose endpoint bar, select Alias to Application and Classic Load Balancer
- Select the region and the load balancer required. You will not need to type in the load balancer, as it will already populate.

<br>

<img width="1232" alt="alias" src="https://user-images.githubusercontent.com/92983658/227996894-a59f183e-6224-4ff7-a0c2-ddd0d01819b4.png">

<br>

#### Visiting the application from the browser

- navigagte to the application with the url `tooling.artifactory.<your domain>`...it should load up the artifactory application.

<br>

<img width="1231" alt="app" src="https://user-images.githubusercontent.com/92983658/227998390-030b585e-7c5d-4034-a278-76f6aa941b75.png">

<br>

This shows that the site is indeed reachable, but insecure. This is because some browsers (chrome) do not load insecure sites by default. It is insecure because it either does not have a trusted TLS/SSL certificate, or it doesn’t have any at all.

Nginx Ingress Controller does configure a default TLS/SSL certificate. But it is not trusted because it is a self signed certificate that browsers are not aware of.

<br>

- Click on the Not Secure part of the browser.
- Select the Certificate is not valid menu
- You will see the details of the certificate. There you can confirm that yes indeed there is encryption configured for the traffic, the browser is just not cool with it.

<br>

<img width="1224" alt="certificate" src="https://user-images.githubusercontent.com/92983658/227998948-e0bd6a11-f7d0-407f-a321-76d3e0e590f6.png">

<br>

<img width="1232" alt="certificate_1b" src="https://user-images.githubusercontent.com/92983658/227999341-1a28b0b8-3ea4-4cb4-aa85-e398f0242b1a.png">

<br>

- Now try another browser. For example Internet explorer or Safari


