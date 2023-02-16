# Setting Up A Kubernetes Cluster (Manually)

In this project, a Kubernetes cluster is manually setup from the scratch without any automated helpers in order to better understand each aspect of spinning up a Kubernetes cluster as each components are manually installed from scratch. 

To successfully implement "K8s From-Ground-Up", the following and even more will be done by a K8s administrator:

- Install and configure master (also known as control plane) components and worker nodes (or just nodes).
- Apply security settings across the entire cluster (i.e., encrypting the data in transit, and at rest)
- In transit encryption means encrypting communications over the network using HTTPS
- At rest encryption means encrypting the data stored on a disk
- Plan the capacity for the backend data store etcd
- Configure network plugins for the containers to communicate
- Manage periodical upgrade of the cluster
- Configure observability and auditing

**Note**:The following setup of Kubernetes is just for learning purpose only, and not to be considered for production. This is because setting up a K8s cluster for production use has a lot more moving parts, especially when it comes to planning the nodes, and securing the cluster. 

The purpose of "K8s From-Ground-Up" is to get a better understanding of the different components as shown in the architecture diagram. 

**Tools to be used and expected result of the Project 20**
- VM: AWS EC2
- OS: Ubuntu 20.04 lts+
- Docker Engine
- kubectl console utility
- cfssl and cfssljson utilities
- Kubernetes cluster

**3 EC2 Instances will be crreated, and in the end, the following parts of the cluster will be properly configured:**

- One Kubernetes Master
- Two Kubernetes Worker Nodes
- Configured SSL/TLS certificates for Kubernetes components to communicate securely
- Configured Node Network
- Configured Pod Network

<br>

## PART ONE: Install Client Tools

### Install and configure AWS CLI
`AWS CLI` is a unified tool to manage your AWS services

- Configure AWS CLI to access all AWS services used:
  - for this, create a user with programmatic access keys configured in AWS Identity and Access Management (IAM)..(i'm using one i created for project 19)
  - Generate access keys and store them in a safe place.

<br>

<img width="1194" alt="IAM" src="https://user-images.githubusercontent.com/92983658/219353590-a820e9aa-9ead-4427-9486-ffa6c8ad8019.png">

<br>

- On your local workstation download and install the <a href-"https://aws.amazon.com/cli/">latest version of AWS CLI</a>
- configure `AWS CLI`: `aws configure`
```
# fill in the following details when prompted
AWS Access Key ID [None]: <your IAM access key>
AWS Secret Access Key [None]: < your secret key>
Default region name [None]: < your region >
Default output format [None]: json

```

- Test `AWS CLI` by running: `aws ec2 describe-vpcs`

<br>

<img width="1180" alt="aws_cli_test" src="https://user-images.githubusercontent.com/92983658/219356486-3bc92728-942b-41b2-a780-8df172538484.png">

<br>



