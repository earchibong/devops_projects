# Deploying Applications Into Kubenetes Cluster
## Introduction
Following from <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md">project 21</a>, this project 
demonstrates how containerised applications are deployed as pods in Kubernetes and how to access the application from 
the browser.

Within this project we are going to learn and see in action following:

**Deployment of software applications using YAML manifest files with following K8s objects:**
- Pods
- ReplicaSets
- Deployments
- StatefulSets
- Services (ClusterIP, NodeIP, Loadbalancer)
- Configmaps
- Volumes
- PersistentVolumes
- PersistentVolumeClaims


**Difference between stateful and stateless applications**
- Deploy MySQL as a StatefulSet and explain why

**Limitations of using manifests directly to deploy on K8s**
- Working with Helm templates, its components and the most important parts – semantic versioning
- Converting all the .yaml templates into a helm chart

**Deploying more tools with Helm charts on AWS Elastic Kubernetes Service (EKS)**
- Jenkins
- -MySQL
- -Ingress Controllers (Nginx)
- Cert-Manager
- Ingress for Jenkins
- Ingress for the actual application

**Deploy Monitoring Tools**
- Prometheus
- Grafana

**Hybrid CI/CD by combining different tools such as: Gitlab CICD, Jenkins. And also concepts around GitOps using Weaveworks Flux**

## Labs
- <a href="https://github.com/earchibong/devops_training/edit/main/kubernetes_02.md#create-a-kubernetes-cluster-on-aws-eks">Create a kubernetes clusters eksctl</a>
- <a href="https://github.com/earchibong/devops_training/new/main#deploying-a-random-pod">Deploy A Pod</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes_02.md#accessing-the-application-from-the-browser">Accessing The Application From The Browser</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes_02.md#create-a-replica-set">Create A Replica Set</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes_02.md#access-kubernetes-services-with-aws-load-balancer">Accessing Kubernetes Service With AWS Loadbalancer</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes_02.md#cleaning-up">Cleaning Up</a>

<br>

## Create A Kubernetes Cluster On AWS EKS
find out more about AWS EKS <a href="https://www.youtube.com/watch?v=p6xDCz00TxU">here</a>

- install `eksctl` on mac
```

brew tap weaveworks/tap
brew install weaveworks/tap/eksctleksctl version


```

<br>

<img width="537" alt="eks_version" src="https://user-images.githubusercontent.com/92983658/222418943-ab968a06-31cc-4e91-931f-871d7c850af1.png">

<br>

- create cluster using eksctl
```

eksctl create cluster \
--name PBL22-cluster \
--tags Key=Name,Value=PBL22-cluster \
--ssh-access --ssh-public-key=devops --region=eu-west-2 \
--nodegroup-name PBL22-nodes \
--node-type t2.micro \
--nodes 2 \

```

*- note: for the above command to authenticate on AWS you will needs to add your AWS secret key and password on the path using `aws configure`...find out more <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md">here</a>*

*- the above command is to create a cluster with 2 worker nodes. The process can tak up to 20 mins*

*for ssh-access use your existing ssh keypair set up for previous projects and state the region on AWS where it is stored*

<br>

<img width="1061" alt="verify_eksctl" src="https://user-images.githubusercontent.com/92983658/222449145-a7f54747-ea0a-4324-9ff2-4405c8735355.png">

<br>

<img width="1193" alt="cluster" src="https://user-images.githubusercontent.com/92983658/222450893-340e2d1a-8af7-4eff-b735-03e87b7d9013.png">

<br>

<img width="1199" alt="nodes" src="https://user-images.githubusercontent.com/92983658/222450927-3fa327eb-8472-412a-af46-41d5fbc5f34c.png">

<br>


## Deploy A Pod
Deploy a basic Nginx container to run inside a Pod.

### Create a Pod yaml manifest on eks control pane

- on terminal, run the following:
```

sudo cat <<EOF | sudo tee ./nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - image: nginx
    name: nginx-pod
    ports:
    - containerPort: 80
      protocol: TCP  
EOF

```

<br>

- Apply the manifest with the help of `kubectl`
```

kubectl apply -f nginx-pod.yaml

```

- output:
```
pod/nginx-pod created

```

<br>

<img width="1013" alt="manifest_2" src="https://user-images.githubusercontent.com/92983658/222701677-6925bec1-4885-48ba-a7ba-5343f6dd6dcb.png">

<br>

 - Get an output of the pods running in the cluster
 ```
kubectl get pods

```

- output
```

NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          19m

```

- If the Pods were not ready for any reason, for example if there are no worker nodes, you will see something like the below output.

```

NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   0/1     Pending   0          111s

```

<br>

<img width="1216" alt="get_pods_1b" src="https://user-images.githubusercontent.com/92983658/222701933-68d9cb09-6168-4392-94a6-79d6f63f8aaa.png">

<br>

- To see other fields introduced by kubernetes after you have deployed the resource, simply run below command, and examine the output. You will see other fields that kubernetes updates from time to time to represent the state of the resource within the cluster. -o simply means the output format.

```
kubectl get pod nginx-pod -o yaml --kubeconfig admin.kubeconfig

or

kubectl describe pod nginx-pod --kubeconfig admin.kubeconfig

```

<br>

## Accessing The Application From The Browser
A service is an object that accepts requests on behalf of the Pods and forwards it to the Pod’s IP address. 
- Run the command below, to get the Pod’s IP address.

```

kubectl get pod nginx-pod  -o wide

```

<br>

<img width="1386" alt="nginx_pod_ip" src="https://user-images.githubusercontent.com/92983658/222702896-0c85a203-328f-4cfe-a11f-cb135886a2e4.png">


<br>


### Access the Pod through its IP address from within the K8s cluster.
- access an image that already has curl software installed
```

kubectl run curl --image=dareyregistry/curl -i --tty

```

<br>

- curl and point to the IP address of the Nginx Pod (Use the IP address of your own Pod)
```

curl -v <pod ip address>:80

```

<br>

<img width="1206" alt="pod" src="https://user-images.githubusercontent.com/92983658/222703268-56d00c60-6bc0-465c-8a28-e584a2b5e6fe.png">

<br>

*- If the use case for your solution is required for internal use ONLY, without public Internet requirement. Then, this should be OK. but, in most cases it is not, so a Kubernetes service will be required*
*- A service is An object that abstracts the underlining IP addresses of Pods. A service can serve as a load balancer, and a reverse proxy which basically takes the request using a human readable DNS name, resolves to a Pod IP that is running and forwards the request to it. This way, you do not need to use an IP address. Rather, you can simply refer to the service name directly.*

### create a service to access the Nginx Pod
- Create a Service `yaml` manifest file:

```

sudo cat <<EOF | sudo tee ./nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-pod 
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF

```

<br>


- Create a `nginx-service` resource by applying the manifest
```

kubectl apply -f nginx-service.yaml

```

<br>

<img width="1005" alt="nginx_service_manifest" src="https://user-images.githubusercontent.com/92983658/222706198-63518bbb-251d-4362-89b1-30ebbb19dd78.png">

<br>

- verfiy service
```

kubectl get service

```

<br>

<img width="1038" alt="verfiy_service" src="https://user-images.githubusercontent.com/92983658/222706922-069c6de5-88ca-4aee-a444-f26afba9141e.png">

<br>

*The TYPE column in the output shows that there are different service types.*
*- ClusterIP*
*- NodePort*
*- LoadBalancer *
*- Headless Service *

*Since type was not specified, the default type is ClusterIP in this case *

<br>

- Update the Pod manifest with the below and apply the manifest:

<br>

```

sudo nano nginx-pod.yaml

```

<br>

```

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-pod  
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
      protocol: TCP
 

 ```
 
 <br>
 
 ```
 kubectl apply -f nginx-pod.yaml
 
```

<br>

<img width="1236" alt="pod_configured" src="https://user-images.githubusercontent.com/92983658/222714878-472f0fbd-5224-421b-95ee-1fa05f317937.png">

<br>

- access the app using `kubectl's port-forward` functionality.
```

kubectl  port-forward svc/nginx-service 8089:80

```

*note: `kubectl's port-forward` functionality is being used because no public ip address...tunnelling traffic through the machine's port number to the port number of the nginx-service*

- output:
```

kubectl  port-forward svc/nginx-service 8089:80
Forwarding from 127.0.0.1:8089 -> 80
Forwarding from [::1]:8089 -> 80

```

<br>

<img width="1202" alt="port_forward" src="https://user-images.githubusercontent.com/92983658/222715030-c603bbc1-d76c-4879-9a7b-c7063330bd0f.png">

<br>

- verfiy in web browser: `localhost:8089`

<br>

<img width="1200" alt="localhost8089" src="https://user-images.githubusercontent.com/92983658/222715553-6b2f4a8a-7a57-416b-a567-8103281bf4cd.png">

<br>

## Create a Replica Set
- create a `rs.yaml` manifest for a ReplicaSet object:
```
touch rs.yaml && nano rs.yaml

```

- add the following to the file:
```

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      name: nginx-pod
      labels:
         app: nginx-pod
    spec:
      containers:
      - image: nginx:latest
        name: nginx-pod
        ports:
        - containerPort: 80
          protocol: TCP
 
```

<br>

<img width="1229" alt="manifest_1b" src="https://user-images.githubusercontent.com/92983658/222724063-ac6099cb-7d9c-4500-99ec-e2a88dac1aa7.png">

<br>

- apply manifest
```

kubectl apply -f rs.yaml

```

The manifest file of ReplicaSet consist of the following fields:

- **apiVersion:** This field specifies the version of kubernetes Api to which the object belongs. ReplicaSet belongs to apps/v1 apiVersion.
- **kind:** This field specify the type of object for which the manifest belongs to. Here, it is ReplicaSet.
- **metadata:** This field includes the metadata for the object. It mainly includes two fields: name and labels of the ReplicaSet.
- **spec:** This field specifies the label selector to be used to select the Pods, number of replicas of the Pod to be run and the container or list of containers which the Pod will run. In the above example, we are running 3 replicas of nginx container.

<br>

- verfiy pods
```

kubectl get pods
kubectl get rs

```

<br>

<img width="1293" alt="get_pod" src="https://user-images.githubusercontent.com/92983658/222732928-588be0dd-b61d-44ad-b32f-7bb4a996c2c2.png">

<br>

- delete a pod
```

kubectl delete pod <inser pod name>

```

- replica set immediately creates a new one

<br>

<img width="1203" alt="replica_set_1c" src="https://user-images.githubusercontent.com/92983658/222733636-423f534d-56a1-4e68-83f2-17c791856da0.png">

<br>

Two ways pods can be scaled: **Imperative and Declarative**

- scale up ReplicaSet up by specifying the desired number of replicas in an imperative command:
```

kubectl scale --replicas 5 replicaset nginx-rs

```

<br>

<img width="1152" alt="scale_replicas" src="https://user-images.githubusercontent.com/92983658/222735251-4c88d55f-2959-4dc9-ba4c-7efce02d2925.png">

<br>

- scale down ReplicaSet up by specifying the desired number of replicas in an delarative command:
open `rs.yaml` manifest, change desired number of replicas in respective section

```
spec:
  replicas: 2
  
```

- apply updated manifest
```
kubectl apply -f rs.yaml

```

<br>

<img width="1292" alt="scale_down" src="https://user-images.githubusercontent.com/92983658/222736446-ce5cdc60-42a5-4c49-9c28-66ca40a5c15e.png">

<br>

* replicaset scaled down to 2 *

<br>

## Access Kubernetes Services With AWs Load Balancer
Previously Niginx Service was accessed through `ClusterIP`, and `NodeIP`. a `load balancder` service can also be used for this purpose. This type of service does not only create a Service object in K8s, but also provisions a real external Load Balancer (e.g. Elastic Load Balancer – ELB in AWS)

- update service manifest and use the `LoadBalancer` type. Also, ensure that the selector references the Pods in the replica set.
```

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    tier: frontend
  ports:
    - protocol: TCP
      port: 80 # This is the port the Loadbalancer is listening at
      targetPort: 80 # This is the port the container is listening at
      
 ```

- apply configuration
```

kubectl apply -f nginx-service.yaml


```

- get the new configured service
```
kubectl get service nginx-service

```

<br>

<img width="1387" alt="get_service" src="https://user-images.githubusercontent.com/92983658/222740183-75e2d72c-f712-400c-bad6-a1ff034f0f42.png">

<br>

<img width="1195" alt="load_balancer_verify" src="https://user-images.githubusercontent.com/92983658/222740216-c523413b-6072-44bb-9564-5cb60aa78945.png">

<br>

<img width="1195" alt="load_balancer_1b" src="https://user-images.githubusercontent.com/92983658/222740616-dd57aa73-2b71-4b7c-89f5-2458a259eb2e.png">

<br>

<img width="1195" alt="load_tags" src="https://user-images.githubusercontent.com/92983658/222741078-df33ad8c-1394-4929-ba93-a19a075317a0.png">

<br>

- Get the output of the entire `yaml` for the service.
```
kubectl get service nginx-service -o yaml

```

- Ensure that port range `30000-32767` is opened in your inbound Security Group configuration for the loadbalancer
- copy and paste loadbalancer address to access nginx service

## Using deployment Controllers
A Deployment is another layer above ReplicaSets and Pods, newer and more advanced level concept than ReplicaSets. It manages the deployment of ReplicaSets and allows for easy updating of a ReplicaSet as well as the ability to roll back to a previous version of deployment. It is declarative and can be used for rolling updates of micro-services, ensuring there is no downtime.

Officially, it is highly recommended to use Deplyments to manage replica sets rather than using replica sets directly.

- Delete the ReplicaSet
```
kubectl delete rs nginx-rs

```

- create `deployment.yaml`
```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 8
  
  
 ```
 
 ```
 
 kubectl apply -f deployment.yaml
 
 ```
 
 - Inspecting the setup:
```
kubectl get po #get the pods
kubectl get deploy #get the deployment
kubectl get rs #get replicaset

```

 <br>
 
 <img width="1382" alt="deployment_inspect" src="https://user-images.githubusercontent.com/92983658/222750987-5d8f5d48-faaf-4892-b6fc-2a09a709c10d.png">

<br>

- Exec into one of the Pod’s container to run Linux commands
```
kubectl exec <pod name> -i -t -- bash
 
```

<br>

- list all files in nginx directory

<br>

<img width="1384" alt="exec_nginx" src="https://user-images.githubusercontent.com/92983658/222752237-d4f62743-abdc-4d00-940e-51c4d5a7206f.png">

<br>

- Check the content of the default Nginx configuration file
```

cat  /etc/nginx/conf.d/default.conf 

```

<br>

<img width="1006" alt="conf" src="https://user-images.githubusercontent.com/92983658/222752720-270a2ebb-9a22-4867-9b75-d4afe92269cf.png">

<br>

Most common Kubernetes workloads covered so far

![image](https://user-images.githubusercontent.com/92983658/222753121-c69d3814-6cdc-4937-a610-cf1a8b90af92.png)


<br>

### Persisting Data For Pods
Deployments are stateless by design. Hence, any data stored inside the Pod’s container does not persist when the Pod dies.

- scale down pods to 1
```
nano rs.yaml

# then...
spec:
  replicas: 1
  
```

- apply configuration
```
kubectl apply -f rs.yaml

```

<br>

<img width="1265" alt="scale_down_2c" src="https://user-images.githubusercontent.com/92983658/222754569-feba41ef-6f45-4120-b844-20b86c79a573.png">


<br>

- Exec into the running container 
```
kubectl exec <pod name> -i -t -- bash

```

<br>

<img width="1382" alt="exec_2c" src="https://user-images.githubusercontent.com/92983658/222755373-1f0072f6-a0f6-46bf-9724-141681b6107e.png">

<br>

- install nano
```

apt update
apt install nano

```

- Update the content of the file and add the code below `/usr/share/nginx/html/index.html`

```
nano /usr/share/nginx/html/index.html

```

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to DAREY.IO!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to DAREY.IO!</h1>
<p>I love experiencing Kubernetes</p>

<p>Learning by doing is absolutely the best strategy at 
<a href="https://darey.io/">www.darey.io</a>.<br/>
for skills acquisition
<a href="https://darey.io/">www.darey.io</a>.</p>

<p><em>Thank you for learning from DAREY.IO</em></p>
</body>
</html>

```

<br>

<img width="1218" alt="nano" src="https://user-images.githubusercontent.com/92983658/222757111-1d76d7ba-d1f3-4dc4-ba2e-f667f79b1169.png">

<br>

- check browser with loadbalancer ip 

<br>

<img width="1200" alt="browser_2v" src="https://user-images.githubusercontent.com/92983658/222757504-98f2d963-c064-42b8-93ac-db0165bdb638.png">

<br>

- delete the only running Pod

```
 kubectl delete po <pod name>
 
```

<br>

<img width="1381" alt="delete_pod_3" src="https://user-images.githubusercontent.com/92983658/222758232-b777153c-ac4b-49ce-8334-4f9ccb1f95f1.png">

<br>

- Refresh the web page : You will see that the content you saved in the container is no longer there. That is because Pods do not store data when they are being recreated – that is why they are called ephemeral or stateless.
  
## Cleaning Up

- delete cluster
```
eksctl delete cluster --name PBL22-cluster

```
  
<br>

<img width="1340" alt="cluster_deleted" src="https://user-images.githubusercontent.com/92983658/222770035-090f1cf4-4b91-41c8-b6dc-feea923aec12.png">

<br>


