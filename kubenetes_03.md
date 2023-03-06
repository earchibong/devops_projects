# Persisting Data In Kubernetes
Containers are stateless by design, which means that data does not persist in the containers. Even when the containers in 
kubernetes pods are run, they still remain stateless unless the configuration supports statefulness.

To achieve statefuleness in kubernetes, we must understand how `volumes`, `persistent volumes`, and `persistent volume claims` work.

## Labs
<a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_03.md#create-a-kubernetes-eks-cluster">Create an EKS Kubenetes Cluster</a> 

<a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_03.md#volumes">Persisting Data on Volumes</a>


## Create A Kubernetes EKS Cluster
Follow the steps in <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes_02.md">Project 22</a> to achieve this.

- create cluster using eksctl
```


eksctl create cluster \
--name PBL23-cluster \
--tags Key=Name,Value=PBL23-cluster \
--ssh-access --ssh-public-key=devops --region=eu-west-2 \
--nodegroup-name PBL23-nodes \
--node-type t2.micro \
--nodes 2 \

```

<br>

<img width="1240" alt="eksctl_1a" src="https://user-images.githubusercontent.com/92983658/223100879-6b99d108-dc16-44dc-8aeb-859a79c1fe74.png">

<br>

<img width="1227" alt="eks_1a" src="https://user-images.githubusercontent.com/92983658/223100888-c06ea1ed-f0f0-4715-a38a-2144723c47ee.png">

<br>

<img width="1229" alt="eks_1b" src="https://user-images.githubusercontent.com/92983658/223100917-6ff65140-9e88-4e17-ad72-dcee98ed6309.png">

<br>

## Volumes

On-disk files in a container are ephemeral and this creates a problem with the loss of files when a container crashes. The kubelet 
restarts the container but with a clean state. A second problem occurs when sharing files between containers running together in 
a Pod. The Kubernetes volume abstraction solves both of these problems.

At its core, a volume is a directory, possibly with some data in it, which is accessible to the containers in a pod. How that directory comes to be, the medium that backs it, and the contents of it are all determined by the particular volume type used

### Persisiting Data With AWS Elastic Block Store Volume
An **aws Elastic Block Store** volume mounts an **Amazon Web Services (AWS) EBS volume** into a pod. The contents of an EBS volume are persisted and the volume is only unmmounted when the pod crashes, or terminates. This means that an EBS volume can be pre-populated with data, and that data can be shared between pods.

- update `Nginx` pod manifest
```

sudo cat <<EOF | sudo tee ./nginx-pod.yaml
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
        - containerPort: 80
EOF


```

<br>

- apply configuration
```

kubectl apply -f nginx-pod.yaml

```

<br>

<img width="1178" alt="manifest_1a" src="https://user-images.githubusercontent.com/92983658/223104322-36756a76-bca7-4caf-897c-fe745199ca4a.png">

<br>

- verify pod is running and check pod logs
```

kubectl get pods

```

<br>

<img width="1286" alt="get_pods_1a" src="https://user-images.githubusercontent.com/92983658/223104671-6caa0c04-a736-433f-8826-2cbadf6ee7dd.png">

<br>

- Exec into the pod and navigate to the nginx configuration file `/etc/nginx/conf.d`
```
kubectl exec <pod name> -i -t -- bash

```

<br>

<img width="1142" alt="exec_1a" src="https://user-images.githubusercontent.com/92983658/223105779-187ceb90-8ce9-4523-bbd2-8dbab054d1ef.png">

<br>

- Open the config files to see the default configuration.

<br>

<img width="1141" alt="default_conf" src="https://user-images.githubusercontent.com/92983658/223106298-21a93ffe-f6e3-4665-874d-ccefd6d1f802.png">

<br>

Note: restrictions when using `AWS Elastic Block Store volume`:
- The nodes on which pods are running must be AWS EC2 instances
- Those instances need to be in the same region and availability zone as the EBS volume
- EBS only supports a single EC2 instance mounting a volume.

- confirm which node is running the pod:
```
kubectl get po <pod name> -o wide

```

*note: this is important because art of the requirements is to ensure that the volume exists in the same region and availability zone as the EC2 instance running the pod*

<br>

<img width="1470" alt="node_pod" src="https://user-images.githubusercontent.com/92983658/223109803-0558b64b-85ef-47f7-8232-4314edba7f7b.png">

<br>

- describe availability zone

```

kubectl describe node <node ip>

```
<br>

<img width="1303" alt="availability_zone" src="https://user-images.githubusercontent.com/92983658/223110994-bb58fd65-06a3-46f2-b1b9-f326df14cdf3.png">

<br>

*note: node is create in `eu-west-2c` as pointed out in the image above, so volume must be create in the same availability zone*

- create a volume from the AWS console.
  - choose the size of the volume
  - ensure availability zone is the same as node above... in the is case `eu-west-2c`

<br>

<img width="1229" alt="volume_1a" src="https://user-images.githubusercontent.com/92983658/223111881-47c2dc8c-a0f4-47e4-a827-9db54d37d386.png">

<br>

- copy the volume id and Update the deployment configuration with the volume spec and volume mount.

<br>

<img width="1228" alt="volume_1b" src="https://user-images.githubusercontent.com/92983658/223112362-3b783858-7b87-4544-b9b3-b8e5b386dc4a.png">

<br>

```

sudo cat <<EOF | sudo tee ./nginx-pod.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
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
        - containerPort: 80
        volumeMounts:
        - name: nginx-volume
          mountPath: /usr/share/nginx/
      volumes:
      - name: nginx-volume
        # This AWS EBS volume must already exist.
        awsElasticBlockStore:
          volumeID: "vol-0d25f5caa98908fc1"
          fsType: ext4
EOF

```

- Apply the new configuration and check the pod.

```

kubectl apply -f nginx-pod.yaml

```

<br>

<img width="1305" alt="volume_1c" src="https://user-images.githubusercontent.com/92983658/223113792-7b4f5fb1-c0e1-46a9-baf7-704530f3a64f.png">

<br>

*note: in the first instance, the updated pod was being created, the old pod is now terminated and the updated pod is up and running. The new pod has a volume attached to it, and can now be used to run a container for statefuleness*

*The value provided to name in `volumeMounts` must be the same value used in the `volumes` section. It basically means mount the volume with the name provided, to the provided mountpath*


