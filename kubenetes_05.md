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
- <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#deploy-artifactory-ingress">Deploy Artifactory Ingress</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#deploying-cert-manager-and-managing-tlsssl-for-ingress">Deploying Cert Manager And Managing TSL/SSL For Ingress</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#managing-certificates-in-kubenetes">Managing Certificates In Kubernetes</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#configuring-ingress-for-tls">Configuring Ingress For TLS</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_05.md#practice-tasks">Practice Tasks</a>

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
            name: my-artifactory
            port:
              number: 8082
              
```

<br>

<img width="1089" alt="artifactory_ingress_1a" src="https://user-images.githubusercontent.com/92983658/228537418-72857e1c-5d32-4bf0-9788-691b12ae750f.png">

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

Now, take note of:

- **CLASS:** The nginx controller class name nginx, 
- **HOSTS:** The hostname to be used in the browser tooling.artifactory.archibong.link, 
- **ADDRES:S** The loadbalancer address that was created by the ingress controller

<br>

#### Configure DNS

If anyone were to visit the tool, it would be very inconvenient sharing the long load balancer address. Ideally, a DNS record that is human readable is created and directs requests to the balancer. This is exactly what has been configured in the ingress object - host: `"tooling.artifactory.archibong.link"` but without a DNS record, there is no way that host address can reach the load balancer.

The `archibong.link` part of the domain is the configured HOSTED ZONE in AWS. So you will need to configure Hosted Zone in AWS console or as part of your infrastructure as code using terraform.

<br>

<img width="1230" alt="route_53_hosted_zone" src="https://user-images.githubusercontent.com/92983658/227994714-ecabc941-a976-44b0-8477-a13e6dfcd2eb.png">

<br>

#### Create Route 53 Record

Create the record to point to the ingress controller’s loadbalancer. There are 2 options. You can either use the` CNAME` or `AWS Alias`

**CNAME METHOD**

- Select the HOSTED ZONE you wish to use, and click on the create record button
- Add the subdomain `tooling.artifactory`, and select the record type `CNAME`
- Successfully created record
- Confirm that the DNS record has been properly propergated. Visit https://dnschecker.org and check the record. Ensure to select CNAME. The search should return green ticks for each of the locations on the left.

<br>

**ALIAS METHOD**
- In the create record section, type in the record name, and toggle the alias button to enable an alias. An alias is of the A DNS record type which basically routes directly to the load balancer. In the choose endpoint bar, select Alias to Application and Classic Load Balancer
- Select the region and the load balancer required. You will not need to type in the load balancer, as it will already populate.

<br>

<img width="1232" alt="alias" src="https://user-images.githubusercontent.com/92983658/227996894-a59f183e-6224-4ff7-a0c2-ddd0d01819b4.png">

<br>

#### Visiting the application from the browser

- navigagte to the application with the url `https://tooling.artifactory.<your domain>`...it should load up the artifactory application.

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

- for now, ignore the warning on chrome, click the `advanded button` and `proceed to the domain` link

<br>

<img width="1228" alt="chrome_advanced" src="https://user-images.githubusercontent.com/92983658/228539004-86448821-3e5b-4c39-a8ed-0478a67d1e73.png">

<br>

<img width="1230" alt="chrome_advanced_1b" src="https://user-images.githubusercontent.com/92983658/228539031-86e94288-0662-4050-b435-d2b3280381a1.png">

<br>

## Deploying Cert Manager and Managing TLS/SSL For Ingress
Transport Layer Security (TLS), the successor of the now-deprecated Secure Sockets Layer (SSL). The TLS protocol aims primarily to provide cryptography, including privacy (confidentiality), integrity, and authenticity through the use of certificates, between two or more communicating computer applications.

The certificates required to implement TLS must be issued by a trusted Certificate Authority (CA).

To see the list of trusted root Certification Authorities (CA) and their certificates used by Google Chrome, you need to use the Certificate Manager built inside Google Chrome as shown below:

- Open the settings section of google chrome and search for `security`

<br>

<img width="1227" alt="chrome_security" src="https://user-images.githubusercontent.com/92983658/228215942-ae6aeb84-f155-4f44-90e1-f788022f55f0.png">

<br>

- select `manage certificates` and view the installed certificates in the browser

<br>

- <img width="1225" alt="chrome_manage_certificaates" src="https://user-images.githubusercontent.com/92983658/228216521-c7b876cb-3ea3-47d4-a674-a9afce2b9461.png">

<br>

### Managing Certificates In Kubenetes

Similar to how Ingress Controllers are able to enable the creation of Ingress resource in the cluster, so also `cert-manager` enables the possibility to create certificate resource, and a few other resources that makes certificate management seamless.

It can issue certificates from a variety of supported sources, including Let’s Encrypt, HashiCorp Vault, and Venafi as well as private PKI. The issued certificates get stored as kubernetes secret which holds both the private key and public certificate.

In this project, `Let’s Encrypt with cert-manager` will be used. The certificates issued by Let’s Encrypt will work with most browsers because the root certificate that validates all it’s certificates is called “ISRG Root X1” which is already trusted by most browsers and servers.

Find `ISRG Root X1` in the list of certificates already installed in the browser

<br>

<img width="1223" alt="isrg_root_x1" src="https://user-images.githubusercontent.com/92983658/228217470-1d6e3fa5-45d0-4058-810f-5126fef1a6aa.png">

<br>

### Deploying Cert Manager

- Find cert-manager helm chart in Artifact Hub

<br>

<img width="1230" alt="cert_manager_1d" src="https://user-images.githubusercontent.com/92983658/228222669-63a0cda2-f8b7-46d3-be5d-5626cb657f47.png">

<br>

- install cert manager `CustomResourceDefinition` (CRD) and chart

```
#CustomResourceDefinition 
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml

#chart
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm upgrade --install cert-manager jetstack/cert-manager -n cert-manager --create-namespace
```

<br>

<img width="1179" alt="cert_manager" src="https://user-images.githubusercontent.com/92983658/229124591-431aecba-57c1-4a2a-a7fc-2c165b9118b1.png">

<br>

### Create A Certificate Issuer
A `Cluster Issuer` will be used so that it can be scoped globally.

- Update the `cert_manager.yaml` file and deploy with kubectl. 

```

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  namespace: "cert-manager"
  name: "letsencrypt-prod"
spec:
  acme:
    server: "https://acme-v02.api.letsencrypt.org/directory"
    email: "infradev@oldcowboyshop.com" # replace with your own email
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: "letsencrypt-prod"
    solvers:
    - selector:
        dnsZones:
          - "<your domain>"
      dns01:
        route53:
          region: "<your hosted zone region>"
          hostedZoneID: "<your hosted zone id>"

```

<br>

<img width="1027" alt="cert_issuer" src="https://user-images.githubusercontent.com/92983658/228602155-33bae249-316e-47c1-afd2-f4f5b7b95d64.png">

<br>

apply configuration

```

kubectl apply -f cert_manager.yaml

```

<br>

<img width="937" alt="cert_issuer_created" src="https://user-images.githubusercontent.com/92983658/228602663-72172008-563b-4e4b-9699-5ab006c8d40c.png">

<br>

### Configuring Ingress For TLS

To ensure that every created ingress also has TLS configured, the ingress manifest will have to be updated with with TLS specific configurations.

- update `artifactory_ingress.yaml` manifest with the following
```

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/ingress.class: nginx
  name: artifactory
spec:
  tls:
  - hosts:
    - "tooling.artifactory.<your domain>"
    secretName: "tooling.artifactory.<your domain>"
  rules:
  - host: "tooling.artifactory.<your domain>"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-artifactory
            port:
              number: 8082


```

<br>

<img width="936" alt="tls_ingress" src="https://user-images.githubusercontent.com/92983658/228608400-b1e19fc9-7223-47eb-af31-b8ae045b3579.png">

<br>

The most significant updates to the ingress definition is the `annotations` and `tls` sections.

<br>

- Redeploy the newly updated ingress
```

kubectl apply -f artifactory_ingress.yaml -n tools

```

<br>

- create an `IAM` policy that enables `cert-manager` to add records to `Route53` in order to solve the DNS01 challenge
  - in `IAM` dashboard, create a policy with the follwing permissions:
```

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

```

<br>

<img width="1229" alt="policy" src="https://user-images.githubusercontent.com/92983658/229139580-12acf222-954e-49d2-b7fc-7f5492d7072e.png">

<br>

- attach the policy to your cluster (select it from the menu)

<br>

<img width="1230" alt="attach_1a" src="https://user-images.githubusercontent.com/92983658/229141330-850dc5fb-b854-45ea-9942-e890757194ca.png">

<br>

<img width="1230" alt="attach_1b" src="https://user-images.githubusercontent.com/92983658/229141366-545b68c1-edc5-4ab7-9c82-0245ee5d50de.png">

<br>

- modify `clusterissuer` with `accesskeyID`

```

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
          - "darey.io"
      dns01:
        route53:
          region: "eu-west-2"
          hostedZoneID: "<your hosted zone id>"
          accessKeyID: "<YOUR ACCESS KEY ID>"
          
```

<br>

- run the following commands in the `tools` namespace:
```

kubectl get certificaterequest -n tools
kubectl get order -n tools
kubectl get challenge -n tools
kubectl get certificate -n tools

```

<br>

<img width="1386" alt="certificate" src="https://user-images.githubusercontent.com/92983658/229155484-4a7af184-a782-4c63-b623-202fad6db16b.png">

<br>

- head over to the browser, you should see the padlock sign without warnings of untrusted certificates.

<br>

<img width="1228" alt="padlock" src="https://user-images.githubusercontent.com/92983658/229156287-f6329184-04f7-429a-97e6-f8c11fce7f51.png">

<br>

## PRACTICE TASKS:

**1. ensure that the LoadBalancer created for artifactory is destroyed**

- run a get service kubectl 
```
kubectl get service -n tools

```

<br>

<img width="1383" alt="get_Service_1a" src="https://user-images.githubusercontent.com/92983658/229156995-3e6c2688-3d3c-4240-9a88-90081ae9f8e8.png">

<br>

notice that the loadbalancer service still exits. To delete this service take the following steps:

- update the helm values file for artifactory, and ensure that the artifactory-artifactory-nginx service uses ClusterIP

```

KUBE_EDITOR="nano" kubectl edit svc/my-artifactory-artifactory-nginx -n tools

```

<br>

<img width="1057" alt="type_change" src="https://user-images.githubusercontent.com/92983658/229159893-e02ef5fe-d631-4cfb-9ab4-ed3a79949043.png">

<br>

- confirm service type change
```

kubectl get service -n tools


```

<br>

<img width="1289" alt="type_change_2b" src="https://user-images.githubusercontent.com/92983658/229160149-b4e0dddb-b253-441c-8aa7-95b3e6089962.png">

<br>

**2. update the ingress to use artifactory-artifactory-nginx as the backend service instead of using artifactory. Remember to update the port number as well.**

-open up ingress file:`artifactory_ingress.yaml` and update the `service name` and `port number` as follows:

```

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/ingress.class: nginx
  name: artifactory
spec:
  tls:
  - hosts:
    - "tooling.artifactory.<your domain>"
    secretName: "tooling.artifactory.<your domain>"
  rules:
  - host: "tooling.artifactory.<your domain>"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-artifactory-artifactory-nginx
            port:
              number: 443
              
 ```
 
 - apply configuration
 ```
 
 kubectl apply -f artifactory_ingress.yaml -n tools
 
 ```
 
 <br>
 
 
## Configuring Artifactory
- Insert the username and password to load the Get Started page

<br>

<img width="1230" alt="get_started" src="https://user-images.githubusercontent.com/92983658/230718690-147c1c30-891d-4189-9e12-d51599ad175b.png">

<br>

- Reset the admin password

<br>

<img width="1225" alt="reset_admin" src="https://user-images.githubusercontent.com/92983658/230718752-fcc8c58b-9a49-4755-8c34-bc5615966f02.png">

<br>

- Activate the Artifactory License. To use Artifactory enterprise features a license will need to be purchased. For learning purposes, a  get a free by <a href="https://jfrog.com/platform/free-trial/">filling in this form</a> and a free license will be sent to your email.
- Skip creation of repositories for now and finish the setup.

<br>

<img width="1228" alt="license" src="https://user-images.githubusercontent.com/92983658/230719014-4996efd6-dd83-4f87-88ef-408bc4db9347.png">

<br>

<img width="1226" alt="self_hosted" src="https://user-images.githubusercontent.com/92983658/230719407-effc4d0d-cec0-4533-a8e0-8b42efbd98bd.png">

<br>

<img width="1229" alt="trial_environment" src="https://user-images.githubusercontent.com/92983658/230719468-2a098e9b-bda3-4660-983c-3b76c8258117.png">

<br>

<img width="1232" alt="repo" src="https://user-images.githubusercontent.com/92983658/230719161-795c6e3e-41f2-4f5d-954d-1e5c013e685b.png">

<br>


