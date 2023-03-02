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

eksctl create cluster --managed=false \
--name K8s-cluster \
--tags Key=Name,Value=K8s-cluster
--nodegroup-name k8s-cluster-nodes \
--node-type t2.micro \
--nodes int 2

```

*- note: for the above command to authenticate on AWS you will needs to add your AWS secret key and password on the path using `aws configure`...find out more <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md">here</a>*

*- the above command is to create a cluster with 2 worker nodes. The process can tak up to 20 mins*

<br>

<img width="983" alt="confirm_eksctl" src="https://user-images.githubusercontent.com/92983658/222435685-4daa5b07-5038-4adf-b23a-2f8a26c60562.png">

<br>


## Deploy A Pod
Deploy a basic Nginx container to run inside a Pod.

### Create a Pod yaml manifest on a master node
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

kubectl apply -f nginx-pod.yaml --kubeconfig admin.kubeconfig

```

- output:
```
pod/nginx-pod created

```

<br>

<img width="1041" alt="manifest" src="https://user-images.githubusercontent.com/92983658/221602012-ea713379-b1a4-4e9e-a33e-05f482789487.png">

<br>

 - Get an output of the pods running in the cluster
 ```
kubectl get pods --kubeconfig admin.kubeconfig

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

<img width="759" alt="get_pods" src="https://user-images.githubusercontent.com/92983658/221603769-bbe75c7f-b983-4ed3-819d-16155cc24165.png">

<br>

- To see other fields introduced by kubernetes after you have deployed the resource, simply run below command, and examine the output. You will see other fields that kubernetes updates from time to time to represent the state of the resource within the cluster. -o simply means the output format.

```
kubectl get pod nginx-pod -o yaml --kubeconfig admin.kubeconfig

or

kubectl describe pod nginx-pod --kubeconfig admin.kubeconfig

```

<br>

## Accessing The Application From The Browser
A service is an object that accepts requests on behalf of the Pods and forwards it to the Pod’s IP address. Run the command below, you will be able to see the Pod’s IP address.

```

kubectl get pod nginx-pod  -o wide --kubeconfig admin.kubeconfig

```

<br>

<img width="1078" alt="service_kubectl" src="https://user-images.githubusercontent.com/92983658/221848138-dffcc459-4c0c-4be4-9a4d-bcd111faaf54.png">

<br>


### Access the Pod through its IP address from within the K8s cluster.
- access an image that already has curl software installed
```

kubectl run curl --image=dareyregistry/curl -i --tty --kubeconfig admin.kubeconfig

```

<br>

- curl and point to the IP address of the Nginx Pod (Use the IP address of your own Pod)
```

curl -v ipaddress:80

```

<br>



