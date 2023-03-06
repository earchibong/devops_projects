# Persisting Data In Kubernetes

## Create An EKS Cluster
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


Containers are stateless by design, which means that data does not persist in the containers. Even when the containers in 
kubernetes pods are run, they still remain stateless unless the configuration supports statefulness.

To achieve statefuleness in kubernetes, we must understand how `volumes`, `persistent volumes`, and `persistent volume claims` work.

## Volumes

On-disk files in a container are ephemeral and this creates a problem with the loss of files when a container crashes. The kubelet 
restarts the container but with a clean state. A second problem occurs when sharing files between containers running together in 
a Pod. The Kubernetes volume abstraction solves both of these problems.
