# Deploying Applications Into Kubenetes Cluster
## Introduction
Following from <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes.md">project 21</a>, this project 
demonstrates how containerised applications are deployed as pods in Kubernetes and how to access the application from 
the browser.

- <a href="https://github.com/earchibong/devops_training/new/main#deploying-a-random-pod">Deploying A Random Pod</a>


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
- image: nginx:latest
name: nginx-pod
ports:
- containerPort: 80
  protocol: TCP
EOF

```

<br>

<img width="703" alt="nginx_yaml" src="https://user-images.githubusercontent.com/92983658/221557439-3a6c1419-e059-4317-ac58-ca4702e58c78.png">

<br>

- Apply the manifest with the help of `kubectl`
```

kubectl apply -f nginx-pod.yaml

```

<br>


- 
