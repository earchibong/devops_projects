# Persisting Data In Kubernetes
Containers are stateless by design, which means that data does not persist in the containers. Even when the containers in 
kubernetes pods are run, they still remain stateless unless the configuration supports statefulness.

To achieve statefuleness in kubernetes, we must understand how `volumes`, `persistent volumes`, and `persistent volume claims` work.

## Labs
<a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_03.md#create-a-kubernetes-eks-cluster">Create an EKS Kubenetes Cluster</a>  
<a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_03.md#volumes">Persisting Data on Volumes</a>  
<a href="https://github.com/earchibong/devops_training/edit/main/kubenetes_03.md#static-provisioning-with-aws-elastic-block-store-volume">Static Provisioning with AWS Elastic Block Store Volume</a>  
<a href="https://github.com/earchibong/devops_training/edit/main/kubenetes_03.md#managing-volumes-dynamically-with-pv-and-pvcs">Managing volumes dynamically with PVs and PVCs</a>


## Create A Kubernetes EKS Cluster
Follow the steps in <a href="https://github.com/earchibong/devops_training/blob/main/kubernetes_02.md">Project 22</a> to achieve this.

- create cluster using eksctl
```


eksctl create cluster \
--name PBL23-cluster \
--tags Key=Name,Value=PBL23-cluster \
--ssh-access --ssh-public-key=devops --region=eu-west-2 \
--nodegroup-name PBL23-nodes \
--node-type t2.medium \
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

### Static Provisioning With AWS Elastic Block Store Volume
An **aws Elastic Block Store** volume mounts an **Amazon Web Services (AWS) EBS volume** into a pod. The contents of an EBS volume are persisted and the volume is only unmmounted when the pod crashes, or terminates. This means that an EBS volume can be pre-populated with data, and that data can be shared between pods.

- on EKS dashboard, ensure that `aws-ebs-csi-driver` add-on is enabled on cluster.

<br>

<img width="1225" alt="csi_driver" src="https://user-images.githubusercontent.com/92983658/224299782-335bb3e7-0fde-4c32-94af-ee5d83120039.png">

<br>


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

*note: Kubernetes Pod Stuck In The Pending State Due To Scheduling Failure?...fix the issue by either reducing the requests in the pod spec or by increasing the capacity of the cluster by adding more nodes or increasing the size of every node.*


<br>

<img width="1286" alt="get_pods_1a" src="https://user-images.githubusercontent.com/92983658/223104671-6caa0c04-a736-433f-8826-2cbadf6ee7dd.png">

<br>

- Exec into a pod and navigate to the nginx configuration file `/etc/nginx/conf.d`
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
kubectl get pod <pod name> -o wide

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

<img width="1270" alt="availability_zone_1a" src="https://user-images.githubusercontent.com/92983658/223118496-1cb39f27-d61a-40c5-afd4-4a7cd8622013.png">

<br>

*note: node is create in `eu-west-2b` as pointed out in the image above, so volume must be create in the same availability zone*

- on terminal, create an EBS
  - choose the size of the volume
  - ensure availability zone is the same as node above... in the is case `eu-west-2b`


```

aws ec2 create-volume --availability-zone=eu-west-2b --size=10 --volume-type=gp2

```

<br>

<img width="1057" alt="volume_2b" src="https://user-images.githubusercontent.com/92983658/224303270-b40e60f1-673a-447f-a1bd-46482f9273d1.png">

<br>

- copy the volume id and Update the deployment configuration with the volume spec and volume mount.

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
        awsElasticBlockStore:
          volumeID: "vol-03363fc4b40017a69"
          fsType: ext4
EOF

```

*note: the Elastic block sotre volume must already exist before it can be used on the manifest *

<br>

- Apply the new configuration and check the pod.

```

kubectl apply -f nginx-pod.yaml
kubectl get pods

```

<br>

<img width="1209" alt="volume_1g" src="https://user-images.githubusercontent.com/92983658/223119485-545175fc-8d20-470b-b4fc-d5707f58dfbf.png">

<br>

*note: the old pod is being terminated and the new pod is being created. The new pod has a volume attached to it, and can now be used to run a container for statefuleness*

*The value provided to name in `volumeMounts` must be the same value used in the `volumes` section. It basically means mount the volume with the name provided, to the provided mountpath*

The problem with this configuration is that when we port forward the service and try to reach the endpoint, we will get a `403 error`. Mounting a volume on a filesystem that already contains data will automatically erase all the existing data To solve this issue is by using:
- Persistent Volume (PV) and Persistent Volume Claim (PVC)
- configMap


## Managing Volumes Dynamically With PV and PVCs
PVs are volume plugins that have a lifecycle completely independent of any individual Pod that uses the PV. This means that even when a pod dies, the PV remains. A PV is a piece of storage in the cluster that is either provisioned by an administrator through a manifest file, or it can be dynamically created if a storage class has been pre-configured.

- exec into an available node:

```
kubectl exec <node name> -i -t -- bash

```

- create a difrectory `mnt/PBL23` and inside that directory create an `index.html` file

<br>

<img width="923" alt="crreate_index" src="https://user-images.githubusercontent.com/92983658/224317939-6b7dc21b-0919-45c9-9edf-a9ed3b71c6e5.png">

<br>


- Verify that there is a storageClass in the cluster:
```
kubectl get storageclass

```
<br>

- Create a manifest file `pvc.yaml` for a PVC and, based on the gp2 storageClass, a PV will be dynamically created
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
   name: nginx-volume-claim
spec:
   accessModes:
   - ReadWriteOnce
   resources:
     requests:
       storage: 2Gi
   storageClassName: gp2 
 
 ```
 
 <br>
 
 - apply configuration and confirm
 ```
 
 kubectl apply -f pvc.yaml
 kubectl get pvc
 
 ```
 
 <br>
 
 <img width="1189" alt="pvc_yaml" src="https://user-images.githubusercontent.com/92983658/223133205-0ee9605e-c7ca-41c7-8571-e533102437ac.png">

<br>

*note:pvc is waiting for the first consumer to be created before binding the PV to a PV..hence the pending state*

<br>

- Check for the volume binding section:
```
 kubectl describe storageclass gp2
 
```

<br>

<img width="1385" alt="describe_storage" src="https://user-images.githubusercontent.com/92983658/223134143-686c76b5-2ca7-4270-8169-1b86e8b29891.png">

<br>

<img width="1440" alt="describe_storage_1b" src="https://user-images.githubusercontent.com/92983658/223134549-3e6e769f-3c38-4e06-84be-23f16459b46f.png">

<br>

- The Persistent Volume Claim created is in a pending state because a Persistent Volume is not created yet. Create a file name `pv.yaml` and add the following:

```


apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx-volume
spec:
  storageClassName: gp2
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/PBL23"
    
```

- apply and confirm the configuration
```
kubectl apply -f pv.yaml
kubectl get pv

```

<br>

<img width="1155" alt="get_pv" src="https://user-images.githubusercontent.com/92983658/224318791-83b107dd-df9c-44ca-b743-55529c2c25b4.png">

<br>



- Edit the `nginx-pod.yaml` to create a Persistent volume.
```

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
        - name: nginx-volume-claim
          mountPath: "/tmp/PBL23"
      volumes:
      - name: nginx-volume-claim
        persistentVolumeClaim:
          claimName: nginx-volume-claim
 
   
   
 ```
 
 *note: The '/tmp/PBL23' directory will be persisted, and any data written in there will be stored permanetly on the volume, which can be used by another Pod if the current one gets replaced.*
 
 <br>
 
- apply configuration
```

kubectl apply -f nginx-pod.yaml

```

<br>
 
- Checking the dynamically created PV
```

kubectl get pv

```

<br>

