# Persisting Data In Kubernetes
Containers are stateless by design, which means that data does not persist in the containers. Even when the containers in 
kubernetes pods are run, they still remain stateless unless the configuration supports statefulness.

To achieve statefuleness in kubernetes, we must understand how `volumes`, `persistent volumes`, and `persistent volume claims` work.

## Labs
<a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_03.md#create-a-kubernetes-eks-cluster">Create an EKS Kubenetes Cluster</a>  
<a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_03.md#volumes">Persisting Data on Volumes</a>  
<a href="https://github.com/earchibong/devops_training/edit/main/kubenetes_03.md#static-provisioning-with-aws-elastic-block-store-volume">Static Provisioning with AWS Elastic Block Store Volume</a>  
<a href="https://github.com/earchibong/devops_training/edit/main/kubenetes_03.md#managing-volumes-dynamically-with-pv-and-pvcs">Managing volumes dynamically with PVs and PVCs</a>  
<a href="https://github.com/earchibong/devops_training/blob/main/kubenetes_03.md#config-map">Persisting data with Config Maps</a>


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

- on `IAM` consolse, add identity provider for eks cluster
  - on eks cluster info page, make a note of the `OpenID Connect provider URL`
  - on `iam console`, select  `identity provider` -> `add provider`
  - select `OpenID Connect` as provider type and insert `OpenID Connect provider URL` ->  get `thumb print`
  - for `audience` : `sts.amazonaws.com`

<br>

<img width="1230" alt="open_id_connect_url" src="https://user-images.githubusercontent.com/92983658/226332728-f8cb5bdc-69a8-456e-99f5-f70a1d1e5c0b.png">

<br>

<img width="1230" alt="identity_provider" src="https://user-images.githubusercontent.com/92983658/226332763-f918e114-e839-4d68-b812-cec7c96057a8.png">

<br>

<img width="1230" alt="identty_provder_2" src="https://user-images.githubusercontent.com/92983658/226332984-4708e336-c86f-44fe-b17c-1622ff178ef7.png">

<br>

- on `IAM` console, create a role to attach to the identity provider
  - on `iam` console, choose `roles` and `create role`
  - for **Trusted entity type** -> select `web identity`
  - for **identity provider** -> `<your created identity provider>`
  - for **audience** -> `sts.amazonaws.com`
  - for **permissions** -> filter and select `AmazonEBSCSIDriverPolicy`
  - name, review and create role

<br>

<img width="1228" alt="role_1a" src="https://user-images.githubusercontent.com/92983658/226334814-a4e67b3b-7f85-4a8f-96be-a69033a1f188.png">

<br>

<img width="1228" alt="role_1b" src="https://user-images.githubusercontent.com/92983658/226334832-83a6d3f6-51c5-40cc-b354-000d5d98f59d.png">

<br>

<img width="1227" alt="role_3" src="https://user-images.githubusercontent.com/92983658/226334852-74d59854-1ef5-4de7-9621-31fbb6d81bb0.png">

<br>

- open the newly created role for editing
  - under   trust relationships` tab -> select  `edit trust policy`
  - Find the line that looks similar to the following line:
  ```
  "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com"
  
  ```
  - Add a comma to the end of the previous line, and then add the following line after the previous line. 
  Replace region-code with the AWS Region that your cluster is in. Replace EXAMPLED539D4633E53DE1B71EXAMPLE with your 
  cluster's OIDC provider ID.
  ```
  "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa"
  
  ```

<br>

<img width="1230" alt="update_policy" src="https://user-images.githubusercontent.com/92983658/226336959-2afd6e99-6c0e-4f51-83a2-a3c9ade8463e.png">

<br>

- on EKS dashboard, ensure that `aws-ebs-csi-driver` add-on is enabled on cluster.
  - on cluster infor page, select `add-ons` -> `get more add-ons` -> `aws-ebs-csi-driver`
  - configure `aws-ebs-csi-driver`:
   - select driver version and newly cfreated `IAM role`
   - review and add

<br>

<img width="1225" alt="csi_driver" src="https://user-images.githubusercontent.com/92983658/224299782-335bb3e7-0fde-4c32-94af-ee5d83120039.png">

<br>

<img width="1227" alt="driver_1b" src="https://user-images.githubusercontent.com/92983658/226338286-4ba6fdd0-9aa9-4b6f-852f-544711d8e220.png">

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

<br>

## Managing Volumes Dynamically With PV and PVCs
PVs are volume plugins that have a lifecycle completely independent of any individual Pod that uses the PV. This means that even when a pod dies, the PV remains. A PV is a piece of storage in the cluster that is either provisioned by an administrator through a manifest file, or it can be dynamically created if a storage class has been pre-configured.

- exec into an available node:

```
kubectl exec <node name> -i -t -- bash

```

- create a difrectory `mnt/PBL23` and inside that directory create an `index.html` file

<br>

<img width="1382" alt="testing_persistent_volumes" src="https://user-images.githubusercontent.com/92983658/225015122-57e29b00-3832-4bb1-8375-971d830e77ed.png">

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
   volumeName: "nginx-pv-volume"
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
  name: nginx-pv-volume
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

<img width="1381" alt="bound_pv" src="https://user-images.githubusercontent.com/92983658/224329922-8af9e9b0-40a9-469f-978a-9f768cfe8b7a.png">

<br>

- Edit the `nginx-pod.yaml` to mount the `PVC`
Run a pod with an NGINX image, and specify the `PVC` created earlier in the relevant part of the pod specification

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
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: nginx-storage   
      volumes:
        - name: nginx-storage
          persistentVolumeClaim:
            claimName: nginx-volume-claim
 
   
   
 ```
 
 
 <br>
 
- apply configuration
```

kubectl apply -f nginx-pod.yaml

```

<br>
 
- Checking the dynamically created PV
Bash into the pod, install curl and run the command curl `http://localhost/`. The output should show the content of the `index.html` file created in step 1. This shows that the new pod was able to access the data in the `PV` via the `PersistentVolumeClaim`.


```

kubectl get po
kubectl exec <pod name> -i -t -- bash
apt update
apt install curl
curl http://localhost/

```

<br>

<img width="942" alt="curl_2" src="https://user-images.githubusercontent.com/92983658/224337231-d2c3e240-0ae1-4fe9-a84c-6f6b4f7666be.png">

<br>

<img width="1182" alt="pv_pvc" src="https://user-images.githubusercontent.com/92983658/224338332-b5afd057-0b65-42ca-8317-381c10e5e428.png">

<br>


### Volume Claim Template
Rather than creating separate manifest files, a volume claim template can be created where everything is defined within the deployment manifest

update `nginx-pod.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
    
---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
spec:
  selector:
    matchLabels:
      tier: frontend
  serviceName: nginx-service
  replicas: 1  
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
          mountPath: "/mnt/PBL23"
  volumeClaimTemplates:
  - metadata:
      name: nginx-volume
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
      storageClassName: standard
 
 
 ```
 
<br>

<img width="1380" alt="statefulset" src="https://user-images.githubusercontent.com/92983658/225018438-37b8fe9f-d556-4383-8b41-826969574326.png">

<br>

## Config Map
Using configMaps for persistence is not something that would be considered for data storage. Rather it is a way to manage configuration files and ensure they are not lost as a result of Pod replacement.

- Remove the volumeMounts and PVC sections of the `nginx-pod` manifest
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

- port forward the service and ensure that the "Welcome to nginx" page is visible

```

kubectl apply -f nginx-service.yaml


```

<br>

```

kubectl get pods
kubectl port-forward pods/<pod name> 8080:80

## on browser
localhost:8080

```

<br>

<img width="1128" alt="port_forward" src="https://user-images.githubusercontent.com/92983658/224994362-b2931033-e683-4e66-8299-6b70ed4cb440.png">

<br>

<img width="1228" alt="port_forward_2" src="https://user-images.githubusercontent.com/92983658/224994650-64dc4ea4-9b05-4093-8b2a-b4579a05ad19.png">

<br>

*note: kubectl port-forward syntax:*
```
#Sample command to perform port forwarding on pod:
kubectl port-forward pods/my-pod 8080:80

#Sample command to perform port forwarding on deployment:
kubectl port-forward deployment/my-deployment 8080:80

#Sample command to perform port forwarding on replicaset:
kubectl port-forward replicaset/my-replicaset 8080:80

#Sample command to perform port forwarding on service:
kubectl port-forward service/my-service 8080:80

```

<br>

- on another terminal, exec into the running container and save a copy of the `index.html` file 

```

kubectl exec -it <pod name> -- bash
cat /usr/share/nginx/html/index.html 
## Copy the output and save the file on local pc because it will needed to create a configmap.

```

<br>

<img width="1087" alt="output" src="https://user-images.githubusercontent.com/92983658/224996578-df45f872-1407-45c5-99e9-13f776d6cf32.png">

<br>

### Persisting configuration data with configMaps

- create configmap nmanifest on `configmap.yaml`
```

cat <<EOF | tee ./nginx-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: website-index-file
data:
  # file to be mounted inside a volume
  index-file: |
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
EOF

```

<br>

apply configuration
```

kubectl apply -f nginx-configmap.yaml 

```

<br>

<img width="874" alt="config_manifest" src="https://user-images.githubusercontent.com/92983658/224997535-cb928e9a-4e53-4e97-8d76-ac831f3f128a.png">

<br>

- Update the deployment file to use the configmap in the volumeMounts section

```

cat <<EOF | tee ./nginx-pod.yaml
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
          - name: config
            mountPath: /usr/share/nginx/html
            readOnly: true
      volumes:
      - name: config
        configMap:
          name: website-index-file
          items:
          - key: index-file
            path: index.html
EOF

```

- apply configuration
```
kubectl apply -f nginx-pod.yaml 

```

- the `index.html` file is no longer ephemeral because it is now using a configMap that has been mounted onto the filesystem. exec into the pod and list the `/usr/share/nginx/html` directory to confirm this.

```
kubectl exec -it <pod name> -- bash
ls -ltr  /usr/share/nginx/html

```

<br>

<img width="899" alt="configmap_2a" src="https://user-images.githubusercontent.com/92983658/224999394-d339627f-463d-448e-b2de-4734fdaa66d7.png">

<br>

*note: notice the `index.html` is now a soft link to `../data`*

Accessing the site will not change anything at this time because the same html file is being loaded through configmap.

But if any changes are made to the content of the html file through the configmap, and the pod restarted, all changes will persist.

<br>

- List the available configmaps. 
```

kubectl get configmap
or 

```

<br>

<img width="728" alt="configmap_2b" src="https://user-images.githubusercontent.com/92983658/225000020-2b76b5fe-b55a-4dc6-82dc-96a559079b5b.png">

<br>

- Update the configmap. This can be done can either through the manifest file, or the kubernetes object directly. 
```
#using kubernetes object

kubectl edit cm website-index-file 

# a vim editor, or whatever default editor the local ystem is configured to use will open
#Update the content as needed . "Only the html data section", then save the file. 
```

<br>

<img width="988" alt="config_edit_2" src="https://user-images.githubusercontent.com/92983658/225002714-7cd2d351-b557-4742-9762-78a3ed4377a6.png">

<br>

<img width="994" alt="config_edit" src="https://user-images.githubusercontent.com/92983658/225002739-c2e85d7e-93eb-4771-9a50-76264933d347.png">

<br>

- Without restarting the pod, your site should be loaded automatically.
- port forward the service
```
kubectl port-forward pods/<nginx-deployment-5dbc79879c-mldzs> 8080:80
localhost 8080

```

<br>

<img width="1225" alt="browser_2v" src="https://user-images.githubusercontent.com/92983658/225004335-c585d6a2-576b-4633-9db9-259cdb054e52.png">

<br>

- To restart the deployment for any reason, simply use the command
```
   kubectl rollout restart deploy nginx-deployment 
```

-output
```

deployment.apps/nginx-deployment restarted

```

This will terminate the running pod and spin up a new one.



