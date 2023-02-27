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

- <a href="https://github.com/earchibong/devops_training/new/main#deploying-a-random-pod">Deploying A Random Pod</a>

## Deploying the Tooling app using Kubernetes objects

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

<br>

<img width="759" alt="get_pods" src="https://user-images.githubusercontent.com/92983658/221603769-bbe75c7f-b983-4ed3-819d-16155cc24165.png">

<br>
