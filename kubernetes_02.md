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

**Hybrid CI/CD by combining different tools such as: Gitlab CICD, Jenkins. And also concepts around GitOps using Weaveworks Flux.

## Labs
- <a href="https://github.com/earchibong/devops_training/edit/main/kubernetes_02.md#create-a-kubernetes-cluster-on-aws-eks">Create a kubernetes clusters eksctl</a>
- <a href="https://github.com/earchibong/devops_training/new/main#deploying-a-random-pod">Deploy A Pod</a>
- <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes_02.md#accessing-the-application-from-the-browser">Accessing The Application From The Browser</a>

<br>

## Create A Kubernetes Cluster ON AWS EKS
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
```

sudo cat <<EOF | sudo tee ./nginx-service.yaml
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
 EOF
 

 ```
 
 <br>
 
 

- access the app using `kubectl's port-forward` functionality.
```

kubectl  port-forward svc/nginx-service 8089:80

```

*note: `kubectl's port-forward` functionality is being used because no public ip address*








