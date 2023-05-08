# Deploy and Manage Business Application in Kubernetes – Helm | Kustomize | GitOps
There are multiple choices available when it comes to deploying applications into kubernetes.

**1. Write YAML files and deploy with kubectl:**
This is the easiest method where you write YAML for deployments, services, ingress. etc., and then deploy with `kubectl apply -f <YAML-FILE>`. 
This is usually the default way when getting started with kubernetes OR during development and exploration. 
But it is not sufficient or reliable when it comes to managing the infrastructure in production because it is hard to keep track of multiple yaml files. 
Imagine a project where there are tens or hundreds of microservices/applications that need to be managed across multiple environments. 
With this type of approach, you will end up in a `PEBKAC Situation`

<br>

**2.Use a templating engine like HELM:**
The value proposition of Helm is to install applications, manage the lifecycle of those applications across multiple environments, and customise 
applications through templating. Without going deeper into its obvious benefits, it also has its downsides too. for example:
- Helm only adds value when you install community components. Tools you can easily find on `artifacthub.io` Otherwise you need to write yaml anyway.
- Leads to too much logic in the templates (This is not good for inexperienced kubernetes users, and a problem to hiring managers)
- Learning another DevOps tool. Always be careful about introducing yet another tool into your team/project. Similar to number 3 above, 
it slows down the project.
- Everyone’s Kubernetes cluster is different. Your needs are different. You might need to make a change to the way the chart is deployed... like changing 
a small piece of configuration in the templates folder. But, like a remote control, you’ve only got a limited set of controls which are exposed to you 
(the values that are exposed in the values.yaml file). If you want to make any deeper changes to the Helm chart, you’ll need to fork it and change it 
yourself. (This is a bit like taking a remote control apart, and adding a new button!).

<br>

**3.Kustomize Overlays:**
To overcome the challenges of Helm identified above, using a tool that is already embeded as part of kubectl; makes more sense for most use cases.
The downside to this is that it does not manage the lifecycle of aplications like Helm is able to do. Therefore, other methods will need to be used
alongside Kustomize for this.

<br>

## Labs
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#how-kustomize-works">How Kustomize Works</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#create-base-layer-files">Create base Layer Files</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#create-overlays-files-dev-environment">Create Overlay Files: Dev Environment</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#patching-configuration-with-kustomize">Patching Configuration With Kustomize</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#create-overlays-files-sit-environment">Create Overlay Files: Sit Environment</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#create-overlays-files-prod-environment">Create Overlay Files: Prod Environment</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#integrate-the-tooling-app-aith-amazon-aurora-for-dev-sit-and-prod-environments">Integrate Tooling App With AWS Aurora For Sit, Dev and Prod Environments</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#configure-terraform-to-deploy-an-aurora-instance-integrate-vault-with-kubernetes">Configure Vault To Deploy Aurora Instance: Integrate Vault With Kubernetes</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#configure-terraform-to-deploy-an-aurora-instance-initialize-the-vault-cluster">Configure Vault To Deploy Aurora Instance: Initialize Vault Cluster</a>
- <a href="https://github.com/earchibong/devops_projects/blob/main/kubernetes_07.md#dynamically-inject-secrets-into-the-tooling-app-container">Dynamically Inject Secrets Into Tooling App Container</a>

<br>

## How Kustomize Works
Kustomize relies on the following system of configuration management layering to achieve reusability:

- **Base Layer** – Specifies the most common resources
- **Patch Layers** – Specifies use case specific resources

Using a deployment scenario involving 3 different environments: `dev, sit, and prod`. 
In this example we’ll use `service`, `deployment`, and `namespace` resources. 

For the dev environment, there won't be any specific changes as it will use the same configuration from the base setting. 
In sit and prod environments, the replica settings will be different.

- Using the tooling app for this example, create a folder structure as below.
```

└── tooling-app-kustomize
    ├── base
    │   ├── deployment.yaml
    │   ├── kustomization.yaml
    │   └── service.yaml
    └── overlays
        ├── dev
        │   ├── kustomization.yaml
        │   └── namespace.yaml
        ├── prod
        │   ├── deployment.yaml
        │   ├── kustomization.yaml
        │   └── namespace.yaml
        └── sit
            ├── deployment.yaml
            ├── kustomization.yaml
            └── namespace.yaml
            
            
 ```
 
 <br>
 
## Create Base Layer Files

- add the following to the kubenetes deployment file: `tooling-app-kustomize/base/deployment.yaml`
```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tooling-deployment
  labels:
    app: tooling
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tooling
  template:
    metadata:
      labels:
        app: tooling
    spec:
      containers:
      - name: tooling
        image: dareyregistry/tooling-app:1.0.2
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
            
```

<br>

<img width="1107" alt="image" src="https://user-images.githubusercontent.com/92983658/236859134-8d33eaf0-7a25-4fbc-8745-aca19a0ddb03.png">

<br>

- add the following to the kubenetes service file: `tooling-app-kustomize/base/service.yaml`

```

apiVersion: v1
kind: Service
metadata:
  name: tooling-service
  labels:
    app: tooling
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    name: http
  type: ClusterIP
  selector:
    app: tooling
    
```

<br>

<img width="851" alt="base_service" src="https://user-images.githubusercontent.com/92983658/235634966-70a72499-8808-4401-9f6a-0605154606e3.png">

<br>

- add the following to kubernetes `kustomization` file: `tooling-app-kustomize/base/kustomization.yaml`.
The Kustomization declaritive yaml that is used to let kustomize know what resources to create and monitor in kubernetes.
Currently, the resources being monitored are `deployment` and `services`. You can simply add more to the list as you wish.



```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
  
```

<br>

<img width="870" alt="base_kustomize" src="https://user-images.githubusercontent.com/92983658/235635552-93a8f690-7752-4a7d-978c-5a9c84470b43.png">

<br>

It is assumed that we will need to deploy kubernetes resources across multiple environments, as a standard practice in most cases. Hence, to deploy resources into the Dev environment, the layout and file contents will look like the following:


<br>

## Create Overlays Files: Dev Environment
The overlays folder is where multiple environments are managed. In each environment, there is a Kustomize file that tells Kustomize where to find the base setting, and how to patch the environment using the base as the starting point.

For example: In the dev environment below, the `namespace` for `dev` is created, and the `deployment` is patched to use a replica count of “3” different from the base setting of “1”. So Kustomize will simply create all the resources in base, in addition to whatever is specified in the dev directory.

- add the following to `namespace` file: `tooling-app-kustomize/overlays/dev/namespace.yaml`
```

apiVersion: v1
kind: Namespace
metadata:
  name: dev-tooling
  
```

<br>

<img width="897" alt="dev" src="https://user-images.githubusercontent.com/92983658/235637345-c11aadde-7b95-44b4-be32-2a946c822bf3.png">

<br>

- add the following to `deployment` file: `tooling-app-kustomize/overlays/dev/deployment.yaml`
```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tooling-deployment
spec:
  replicas: 3


```

<br>

<img width="937" alt="dev_deployment" src="https://user-images.githubusercontent.com/92983658/235637871-b6e0afe7-0cb7-46c3-91d7-4404a65b225e.png">

<br>

- add the following to the dev `kustomization` file: `tooling-app-kustomize/overlays/dev/kustomization.yaml`
The Kustomization file for dev here specifies that the base configuration should be applied, and include the yaml file(s) specified in the resources section. It also indicates what namespace the configuration should be applied to.

In summary it specifies the following;

- The apiversion
- The Kind of resource (kustomization)
- The namespace where this kustomizaton will create or patch resources
- The location of the base folder, where the base configuration can be found.
- The resource(s) to be created : Such as a namespace or deployment
- A commonLabel field which ensures that kubernetes labels and selectors are automatically injected into the resources being created. such as below;

<br>

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
bases:
- ../../base

commonLabels:
  env: dev-tooling

resources:
  - namespace.yaml
  
```

<br>

<img width="936" alt="dev_kustomization" src="https://user-images.githubusercontent.com/92983658/235638384-a4d70e36-0bce-46ff-8ece-9b6add716c9e.png">

<br>

Generally, A kustomization file contains fields falling into four categories (although not all have been used in the example above):

- **resources** – what existing resources are to be customized. Example fields: resources, crds.

- **generators** – what new resources should be created. Example fields: configMapGenerator (legacy), secretGenerator (legacy), generators (v2.1).

- **transformers** – what to do to the aforementioned resources. Example fields: namePrefix, nameSuffix, images, commonLabels, etc. and the more general transformers (v2.1) field.

- **meta** – fields which may influence all or some of the above. Example fields: vars, namespace, apiVersion, kind, etc.

<br>

## Patching Configuration With Kustomize
With Kustomize,  environmentscan be patched with extra configurations that overwrites the base setting either by

- creating new resources, or
- patching existing resources.

This is all achieved through the overlays configuration.

The `overlays/dev/kustomization.yaml` file above only creates a new resource. But if we wanted to patch an existing resource, for example, increase the pod replica from the default 1 to 3 as shown in the `overlays/dev/deployment.yaml file`...the patch would look like this:

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev-tooling
bases:
- ../../base

commonLabels:
  env: dev-tooling

resources:
  - namespace.yaml

patches: #the deployment file is changed and the patch is add here
  - deployment.yaml
  
```

<br>

<img width="971" alt="patches_deployment" src="https://user-images.githubusercontent.com/92983658/235641902-6220a581-55ad-423a-ba74-86879320ad47.png">

<br>

- apply configuration to cluster:
```
kubectl apply -k overlays/dev

```

<br>

*note: `-k` flag instead of `-f` is used to make `kubectl` aware of `kustomise`

<br>

- expected output
```

namespace/dev-tooling created
service/tooling-service created
deployment.apps/tooling-deployment created

```

<br>

<img width="852" alt="dev_output" src="https://user-images.githubusercontent.com/92983658/235646879-211010bc-9288-4f72-baa9-04af93833e86.png">

<br>

## Create Overlays Files: Sit Environment
- add the following to sit `namespace` file: `tooling-app-kustomize/overlays/sit/namespace.yaml`
```

apiVersion: v1
kind: Namespace
metadata:
  name: dev-tooling
  
```

<br>

<img width="1099" alt="sit_namespace" src="https://user-images.githubusercontent.com/92983658/235647619-789fbfa0-c320-4a75-9e53-18b060df0711.png">

<br>

- add the following to sit `deployment` file: `tooling-app-kustomize/overlays/sit/deployment.yaml`
```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tooling-deployment
spec:
  replicas: 2 # pod replica changed from from the default 1 to 2
  template: # resource limits and changed from base spec
    spec:
      containers:
      - name: tooling
        resources:
            requests:
               memory: "50Mi"
               cpu: "200m"
            limits:
               memory: "150Mi"
               cpu: "600m"
               
            
```

<br>

<img width="1036" alt="sit_deployment" src="https://user-images.githubusercontent.com/92983658/235650057-854d0bef-c6d9-45a7-b94c-b94898607079.png">

<br>

- add the following to sit `kustomization` file: `tooling-app-kustomize/overlays/sit/kustomization.yaml`

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev-tooling
bases:
- ../../base

commonLabels:
  env: dev-tooling

resources:
  - namespace.yaml

patches:
  - deployment.yaml
  
  
```

<br>

<img width="1127" alt="sit_kustomization" src="https://user-images.githubusercontent.com/92983658/235648867-1381394b-724c-490e-92e7-2ff178650cfb.png">

<br>

- output as follows:

<br>

<img width="1027" alt="sit_output" src="https://user-images.githubusercontent.com/92983658/235653113-be32634a-24f2-4827-a37d-5cffa50a6d49.png">

<br>

## Create Overlays Files: Prod Environment

- add the following to prod `namespace` file: `tooling-app-kustomize/overlays/prod/namespace.yaml`
```

apiVersion: v1
kind: Namespace
metadata:
  name: dev-tooling
  
```

<br>

<img width="1099" alt="sit_namespace" src="https://user-images.githubusercontent.com/92983658/235647619-789fbfa0-c320-4a75-9e53-18b060df0711.png">

<br>

- add the following to prod `deployment` file: `tooling-app-kustomize/overlays/prod/deployment.yaml`
```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tooling-deployment
spec:
  replicas: 4 # pod replica changed from from the default 1 to 4
  template: # resource limits and changed from base spec
    spec:
      containers:
      - name: tooling
        resources:
            requests:
               memory: "90Mi"
               cpu: "300m"
            limits:
               memory: "250Mi"
               cpu: "800m"
               
            
```

<br>

<img width="1085" alt="prod_deployment" src="https://user-images.githubusercontent.com/92983658/235654859-9b72fa2d-f5bf-463e-8088-87235d667bdb.png">

<br>

- add the following to prod `kustomization` file: `tooling-app-kustomize/overlays/prod/kustomization.yaml`

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev-tooling
bases:
- ../../base

commonLabels:
  env: dev-tooling

resources:
  - namespace.yaml

patches:
  - deployment.yaml
  
  
```

<br>

<img width="1127" alt="sit_kustomization" src="https://user-images.githubusercontent.com/92983658/235648867-1381394b-724c-490e-92e7-2ff178650cfb.png">

<br>

- deploy:

```

kubectl apply -k overlays/prod


```

<br>

- output as follows:

<br>

<img width="1025" alt="prod_output" src="https://user-images.githubusercontent.com/92983658/235655277-ff910ab5-770e-4491-96c3-1f8a9b29f0a6.png">

<br>

## Integrate The Tooling App Aith Amazon Aurora For Dev, SIT, And Prod Environments
The steps to do this are as follows:
- Configure Terraform to deploy an aurora instance
- Use the tooling.sql script to load the database schema
- Configure environment variables for database connectivity in the deployment file and patch the each environment for the appropriate values

<br>

## Configure Terraform To Deploy An Aurora Instance: Integrate Vault With Kubernetes
`helm` and `kustomize` will be used for this installation. The <a href="https://artifacthub.io/packages/helm/hashicorp/vault">Valut helm chart</a> will be used for installation. It will then be configured for <a href="https://developer.hashicorp.com/vault/docs/concepts/ha">High availability mode</a> with **integrated storage** (recommended for production-ready deployment).

<br>

- Create the folder structure as below:
```

├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── providers.tf
└── vault/
    ├── base/
    │   ├── kustomization.yaml
    │   └── namespace.yaml
    └── overlays/
        ├── dev/
        │   ├── .env
        │   ├── kustomization.yaml
        │   ├── namespace.yaml
        │   └── values.yaml
        ├── sit/
        │   ├── .env
        │   ├── kustomization.yaml
        │   ├── namespace.yaml
        │   └── values.yaml
        └── prod/
            ├── .env
            ├── kustomization.yaml
            ├── namespace.yaml
            └── values.yaml
        
 ```
 
 <br>
 
 - update `terraform/providers.tf`

```

terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "4.34.0"
    }
  }
}

provider "aws" {
  region = "eu-west-2"
}

```

<br>

- create an `iam` policy named `cert_manager` to add records to `route 53` to resolve `DNS01` challenge. The policy document looks like this:

```

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "route53:GetChange",
      "Resource": "arn:aws:route53:::change/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets",
        "route53:ListResourceRecordSets"
      ],
      "Resource": "arn:aws:route53:::hostedzone/*"
    },
    {
      "Effect": "Allow",
      "Action": "route53:ListHostedZonesByName",
      "Resource": "*"
    }
  ]
}

```

- once policy is created, get the `arn` for the `cert_manager` policy and save it to attach to the cluster iam role in `terraform/main.tf`

<br>

- update `terraform/main.tf`

An `AWS KMS` key will be created because when a Vault server is started, it starts in a sealed state and it does not know how to decrypt data. Before any operation can be performed on the Vault, it must be unsealed. Unsealing is the process by which the Vault root key is used to decrypt the data encryption key that Vault uses to encrypt all data.

`IAM roles` for service accounts provide the ability to manage AWS credentials for the vault, similar to the way that Amazon EC2 instance profiles provide credentials to Amazon EC2 instances. Instead of using the Amazon EC2 instance’s role or creating and distributing `AWS credentials` to the vault containers through the Helm values file, we simply associate an `IAM role` with a Kubernetes service account and configure the pods to use the service account.



```

module "vault_iam_role" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  version = "5.2.0"
  role_name   = "vaultKMS"
  create_role = true
  role_policy_arns = {
    AWSKeyManagementServicePowerUser = "arn:aws:iam::aws:policy/AWSKeyManagementServicePowerUser"
    cert_manager                     = "arn:aws:iam::<your AWS account>:policy/cert_manager"
  }

  oidc_providers = {
    one = {
      provider_arn               = "arn:aws:iam::<your AWS account>:oidc-provider/oidc.eks.eu-west-2.amazonaws.com/id/<oidc provider>"
      namespace_service_accounts = ["vault:vault-kms", ]
    }
  }
  attach_custom_iam_policy = false

  tags = var.tags
}

module "vault_kms_key" {
  source  = "terraform-aws-modules/kms/aws"
  version = "1.0.2"
  description             = "external key"
  deletion_window_in_days = 7
  enable_key_rotation     = true
  is_enabled              = true
  key_usage               = "ENCRYPT_DECRYPT"
  multi_region            = false

  # Policy
  enable_default_policy = true
  key_owners            = [data.aws_iam_role.vault-kms.arn]
  key_administrators    = [data.aws_iam_role.vault-kms.arn]
  key_users             = [data.aws_iam_role.vault-kms.arn]

  # Aliases
  aliases                 = ["dev-vault-kms"]
  aliases_use_name_prefix = true

  # Grants
  grants = {}

  tags = var.tags
}


# This data source can be used to fetch information about vault IAM role.
data "aws_iam_role" "vault-kms" {
  name = "vaultKMS"
  depends_on = [
    module.vault_iam_role
  ]
}



```

<br>

*note: get more information on <a href="https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-role-for-service-accounts-eks"> `terraform iam role for service accounts`</a>. check <a href="https://registry.terraform.io/modules/terraform-aws-modules/iam/aws/latest/submodules/iam-role-for-service-accounts-eks">here</a> as well.*

<br>

*note 2: for `oidc_provider...create it as follows:*
```

#Determine whether you have an existing IAM OIDC provider for your cluster.
#Retrieve your cluster's OIDC provider ID and store it in a variable.
oidc_id=$(aws eks describe-cluster --name my-cluster --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)

#Determine whether an IAM OIDC provider with your cluster's ID is already in your account.
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

#Create an IAM OIDC identity provider for your cluster with the following command. Replace my-cluster with your own value.
eksctl utils associate-iam-oidc-provider --cluster <your-cluster name> --approve

```

<br>

- update `terraform/variables.tf`

```

variable "tags" = {
  type    = map
  default = {
    Terraform   = "true"
    Environment = "dev"
  }
}

```

<br>

On terminal, change directory to  `terraform` directory and initialize the directory containing Terraform files.

```

cd terraform
terraform init

```

<br>

<img width="1247" alt="terraform_init" src="https://user-images.githubusercontent.com/92983658/235679117-9d05a6be-03de-423f-a951-b80f64c8ff80.png">

<br>

- Run the command terraform plan to preview the changes that Terraform plans to make to infrastructure, then terraform apply to executes the actions proposed in the plan.

```

terraform plan -out tfplan

terraform apply tfplan

```

<br>

<img width="1470" alt="plan_01" src="https://user-images.githubusercontent.com/92983658/236836399-9139f4d6-b004-46a1-a464-315924fde188.png">

<br>

<img width="1467" alt="plan_02" src="https://user-images.githubusercontent.com/92983658/236836429-f110ea8d-e8bc-4851-94a2-4c01b2e41d01.png">

<br>

<img width="1386" alt="apply" src="https://user-images.githubusercontent.com/92983658/236836469-efe163b8-981b-4efa-a5bd-1de829313b69.png">

<br>
 
 - update `vault/base/namespace.yaml/`

```

apiVersion: v1
kind: Namespace
metadata:
  name: vault
  
```

<br>

- update `vault/base/kustomization.yaml`

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - "namespace.yaml"
  
```

<br>

### Update Vault Dev Environment Files

- update `vault/overlays/dev/namespace.yaml`

```

apiVersion: v1
kind: Namespace
metadata:
  name: vault
  labels:
    env: dev_vault
    
```

<br>

- update `vault/overlays/dev/kustomization.yaml`:

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# this will over write the namespace all the resourses will be deployed to
namespace: vault

resources:
  - ../../base

patchesStrategicMerge:
  - namespace.yaml

# list the helm chart we want to 
helmCharts:
  - name: vault
    namespace: vault
    repo: https://helm.releases.hashicorp.com
    releaseName: vault
    version: 0.22.0
    valuesFile: values.yaml

secretGenerator:
  - name: vault-kms
    env: .env

generatorOptions:
  disableNameSuffixHash: true
  
  
```

<br>

- update `vault/overlays/dev/.env`
*get more information on `awskms` parameters <a href="https://developer.hashicorp.com/vault/docs/configuration/seal/awskms#awskms-parameters">here</a>*
<br>

```

VAULT_SEAL_TYPE=awskms
VAULT_AWSKMS_SEAL_KEY_ID=arn:aws:kms:<you aws region>:<you account id>:key/<your kms key>
```

<br>

<img width="1223" alt="kms" src="https://user-images.githubusercontent.com/92983658/236675125-dc9054ca-3f31-42e5-8847-9124ecef1999.png">


<br>


Before the content of the values file can be added, we need to install `Ingress Controller` and `Cert-Manager`. see <a href="https://github.com/earchibong/devops_projects/blob/main/private_repositories.md#jenkins-pipeline-for-business-applications"> project 26</a> for how to do this.

```

# deploy ingress controller in `ingress-nginx` namespace
helm upgrade --install ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--namespace ingress-nginx --create-namespace

# confirm the created load balancer in AWS 
kubectl get service -n ingress-nginx


```

<br>

<img width="1192" alt="ingress" src="https://user-images.githubusercontent.com/92983658/235911700-1b6615ef-5116-43f5-bc69-20828847f3cb.png">

<br>

```


# deploy cert manager
# install cert manager CustomResourceDefinition (CRD) and chart

#CustomResourceDefinition 
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml

#chart
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm upgrade --install cert-manager jetstack/cert-manager -n cert-manager --create-namespace

```

<br>

<img width="1257" alt="cert_manager" src="https://user-images.githubusercontent.com/92983658/235912691-d097c7c9-07e4-4db9-b935-96aab7875c0d.png">

<br>

- create route53 secret crediatials for cert-manager

```

kubectl --namespace cert-manager \
create secret generic route53-credentials \
--from-literal="secret-access-key=<YOUR-AWS-SECRET-ACCESS-KEY>"


```

<br>

- create a certificate cluster issuer

```

# create a file named cert_manager.yaml and deploy with kubectl

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  namespace: "cert-manager"
  name: "letsencrypt-prod"
spec:
  acme:
    server: "https://acme-v02.api.letsencrypt.org/directory"
    email: "infradev@oldcowboyshop.com"
    privateKeySecretRef:
      name: "letsencrypt-prod"
    solvers:
    - selector:
        dnsZones:
          - "archibong.link"
      dns01:
        route53:
          region: "eu-west-2"
          hostedZoneID: "<your route 53 hosted zone id>"
          accessKeyID: "<YOUR AWS ACCESS KEY ID>"
          secretAccessKeySecretRef:
              key: secret-access-key
              name: route53-credentials

# apply configuration
kubectl apply -f cert_manager.yaml

```

<br>

<img width="1036" alt="cert_manager" src="https://user-images.githubusercontent.com/92983658/236842456-fae262fa-cb79-4152-9118-f9155f6a34ef.png">

<br>

- configure vault installation from `vault/overlays/dev/values.yaml`

```

server:
  enabled: "-"
  image:
    repository: "hashicorp/vault"
    tag: "1.11.3"
    # Overrides the default Image Pull Policy
    pullPolicy: IfNotPresent

  # Configure the Update Strategy Type for the StatefulSet
  updateStrategyType: "OnDelete"

  # Ingress allows ingress services to be created to allow external access
  # from Kubernetes to access Vault pods.
  ingress:
    enabled: true
    labels: {}
    annotations: 
      cert-manager.io/cluster-issuer: letsencrypt-prod
      cert-manager.io/private-key-rotation-policy: Always      

    ingressClassName: "nginx"
    pathType: Prefix

    activeService: true
    hosts:
      - host: "vault.masterclass.dev.<your domain name>"
        paths: [/]
    tls:
     - secretName: vault.masterclass.dev.<your domain name>
       hosts:
         - vault.masterclass.dev.<your domain name>

  terminationGracePeriodSeconds: 10

  # extraSecretEnvironmentVars is a list of extra environment variables to set with the stateful set.
  # These variables take value from existing Secret objects.
  extraSecretEnvironmentVars:
    - envName: VAULT_SEAL_TYPE
      secretName: vault-kms
      secretKey: VAULT_SEAL_TYPE
    - envName: VAULT_AWSKMS_SEAL_KEY_ID
      secretName: vault-kms
      secretKey: VAULT_AWSKMS_SEAL_KEY_ID

  # Enables a headless service to be used by the Vault Statefulset
  service:
    enabled: true

    # Port on which Vault server is listening
    port: 8200
    # Target port to which the service should be mapped to
    targetPort: 8200
    # Extra annotations for the service definition.
    annotations: {}

  # This configures the Vault Statefulset to create a PVC for data
  # storage when using the file or raft backend storage engines.
  dataStorage:
    enabled: true
    # Size of the PVC created
    size: 2Gi
    # Location where the PVC will be mounted.
    mountPath: "/vault/data"
    # Name of the storage class to use.  If null it will use the
    # configured default Storage Class.
    storageClass: null
    # Access Mode of the storage device being used for the PVC
    accessMode: ReadWriteOnce
    annotations: {}

  # Run Vault in "HA" mode. There are no storage requirements unless the audit log
  # persistence is required.  In HA mode Vault will configure itself to use Consul
  # for its storage backend.
  ha:
    enabled: true
    replicas: 3

    # Set the api_addr configuration for Vault HA
    # See https://www.vaultproject.io/docs/configuration#api_addr
    # If set to null, this will be set to the Pod IP Address
    apiAddr: null

    # Set the cluster_addr confuguration for Vault HA
    # See https://www.vaultproject.io/docs/configuration#cluster_addr
    # If set to null, this will be set to https://$(HOSTNAME).{{ template "vault.fullname" . }}-internal:8201
    clusterAddr: null

    # Enables Vault's integrated Raft storage.  Unlike the typical HA modes where
    # Vault's persistence is external (such as Consul), enabling Raft mode will create
    # persistent volumes for Vault to store data according to the configuration under server.dataStorage.
    # The Vault cluster will coordinate leader elections and failovers internally.
    raft:

      # Enables Raft integrated storage
      enabled: true
      # Set the Node Raft ID to the name of the pod
      setNodeId: true

      config: |
        ui = true

        listener "tcp" {
          tls_disable = 1
          address = "[::]:8200"
          cluster_address = "[::]:8201"
        }

        storage "raft" {
          path = "/vault/data"

          retry_join {
            leader_api_addr = "http://vault-0.vault-internal:8200"
          }

          retry_join {
            leader_api_addr = "http://vault-1.vault-internal:8200"
          } 

          retry_join {
            leader_api_addr = "http://vault-2.vault-internal:8200"
          }

          autopilot {
            cleanup_dead_servers = "true"
            last_contact_threshold = "200ms"
            last_contact_failure_threshold = "10m"
            max_trailing_logs = 250000
            min_quorum = 5
            server_stabilization_time = "10s"
          }
        }

        # cluster_addr = "http://vault:8200"

        service_registration "kubernetes" {}


    # config is a raw string of default configuration when using a Stateful
    # deployment. Default is to use a Consul for its HA storage backend.
    # This should be HCL.
    config: |
      ui = true

      listener "tcp" {
        tls_disable = 1
        address = "[::]:8200"
        cluster_address = "[::]:8201"
      }

      seal "awskms" {
      }

      service_registration "kubernetes" {}

      log_requests_level = "trace"

  # Definition of the serviceAccount used to run Vault.
  # These options are also used when using an external Vault server to validate
  # Kubernetes tokens.
  serviceAccount:
    # Specifies whether a service account should be created
    create: true
    # The name of the service account to use.
    # If not set and create is true, a name is generated using the fullname template
    name: "vault-kms"
    # Extra annotations for the serviceAccount definition. This can either be
    # YAML or a YAML-formatted multi-line templated string map of the
    # annotations to apply to the serviceAccount.
    annotations: 
      eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/vaultKMS ## Update role for new AWS account


# Vault UI
ui:
  # True if you want to create a Service entry for the Vault UI.
  #
  # serviceType can be used to control the type of service created. For
  # example, setting this to "LoadBalancer" will create an external load
  # balancer to access the UI.
  enabled: true
  publishNotReadyAddresses: true
  # The service should only contain selectors for active Vault pod
  activeVaultPodOnly: false
  serviceType: "ClusterIP"
  serviceNodePort: null
  externalPort: 8200
  targetPort: 8200
  
  
 ```
 
 <br>
 
 
- apply changes to cluster:

```

kubectl kustomize overlays/dev --enable-helm | kubectl apply -f -
#kubectl apply -k overlays/dev


```

<br>

<img width="1229" alt="vault_dev_deploy" src="https://user-images.githubusercontent.com/92983658/236197386-9a94379c-3a97-4808-a4a2-5066e8470773.png">

<br>

- update DNs record of host in hosted zone to `vault.masterclass.dev.<your domain name>`

<br>

<img width="1351" alt="dns_vault" src="https://user-images.githubusercontent.com/92983658/236514518-a53fd54d-7ef0-4381-b2c3-40209a9e4cfe.png">

<br>

## Configure Terraform To Deploy An Aurora Instance: Initialize the Vault Cluster
- Run the command below to execute commands in one of the pod with running status

```

kubectl get pod -n vault

# check vault status:
kubectl exec -n vault vault-0 -- vault status

# Initialize Vault with one key share and one key threshold.
kubectl exec -n vault vault-0 -- vault operator init 

# copy the printout of init into a file named `vault/cluster-keys.json`


```

<br>

<img width="862" alt="pods" src="https://user-images.githubusercontent.com/92983658/236675649-6202eec6-7fbf-431e-bf8c-5e07b48fbe38.png">

<br>

<img width="876" alt="status" src="https://user-images.githubusercontent.com/92983658/236675717-939b2e0b-f431-4b61-90f0-91c33f0c0d39.png">

<br>

<img width="911" alt="vault_init" src="https://user-images.githubusercontent.com/92983658/236844992-69e606c4-bf69-40b0-a29e-62c2c7cdc8ba.png">

<br>

- check vault status again to confirm it is unsealed.

<br>

<img width="959" alt="unsealed_vault" src="https://user-images.githubusercontent.com/92983658/236677729-b0828ced-219d-4523-ac12-6e40c9ee19a5.png">

<br>

*Note: From the vault status output the Vault cluster is initialised, the seal type is `awskms`, but after initializing the vault cluster you will get the `recovery keys` (instead of unseal keys) because some of the Vault operations still require `shamir keys`. The Recovery keys generated after running vault operator init can be used to unseal the cluster when it is sealed manually or to regenerate a root token.The `awskms` key type is used for auto unseal. Using the `awskms` key type means you don’t have to manually unseal the pod if it gets recreated.*

<br>

*note 2: more information on how to unseal vault keys <a href="https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-amazon-eks">here</a>*

<br>

## Dynamically Inject Secrets Into The Tooling App Container
The objectiver here is to securely inject the tooling application database credentials from the vault cluster into the tooling application.

This method can be used to pass secrets credentials like `password``, token` and other secrets credential into an application without the application being aware of the vault cluster.

- install vault from <a href="https://developer.hashicorp.com/vault/downloads">here</a>
- Export the vault address, create token and login

```

export VAULT_ADDR="https://vault.masterclass.dev.archibong.link"
vault login (enter initial root token from the vault init)

# alternatively...create vault token with `vault create token`

```

<br>

- Enable the `kv-v2` secrets at the path app.

```

vault secrets enable -path=app kv-v2

```

<br>

<img width="828" alt="k2_v2" src="https://user-images.githubusercontent.com/92983658/236847553-2e65406e-09ca-4832-b853-c86def59f0fb.png">

<br>

- Create the tooling application database credentials at the path `app/database/config/dev`.

```

vault kv put app/database/config/dev username=db password=password host=http://database


```

<br>

<img width="1203" alt="tooling_credentials" src="https://user-images.githubusercontent.com/92983658/236847978-7a799700-491f-446f-aa21-fe3bbee87f74.png">

<br>

- Verify that the secret is defined at the path `app/database/config/dev`.

```

vault kv get app/database/config/dev

```

<br>

<img width="946" alt="secret_verify" src="https://user-images.githubusercontent.com/92983658/236848400-e4c9780c-9d8c-4599-b4cd-46f746b21fcd.png">

<br>

## Configure Kubernetes Authentication

- Enable the `kubernetes auth method` at the default path.

```

vault auth enable kubernetes

```

<br>

<img width="952" alt="kubernetes_auth" src="https://user-images.githubusercontent.com/92983658/236849048-f1bcae2d-caa6-42ee-8cb2-1ffd99b80aa4.png">

<br>

- Configure Vault to talk to Kubernetes with the /config path.
This will require the Kubernetes host address, use `kubectl cluster-info` to get the `Kubernetes host address` and `TCP port` replace it with the `kubernetes_host` from the command below.

```
# get kubernetes host address and TCP port
kubectl cluster-info

#configure vault
vault write auth/kubernetes/config \
  kubernetes_host=https://77E094ED449C0E809F03764C760A051D.gr7.eu-west-2.eks.amazonaws.com

```

<br>

<img width="1385" alt="confi" src="https://user-images.githubusercontent.com/92983658/236850116-b12826a2-e70f-4172-92f0-e217f5058b78.png">

<br>

- Create `read` policy.
In order for the tooling application to read the database credentials it needs the `read capability` to the path `app/data/database/config`, we can do this by creating a policy and attach it to Kubernetes authentication role we will create.

```

vault policy write tooling-db - <<EOF
path "app/data/database/config/*" {
  capabilities = ["read"]
}
EOF

```

<br>

<img width="987" alt="read_policy" src="https://user-images.githubusercontent.com/92983658/236850649-a6d4139e-0bc9-49e2-a8be-55b9a5f554cf.png">

<br>

- Create a kubernetes authentication role named `tooling-role`

```

vault write auth/kubernetes/role/tooling-role \
  ttl=6h \
  policies=tooling-db \
  bound_service_account_names=tooling-sa \
  bound_service_account_namespaces=dev
  
```

<br>

<img width="1059" alt="tooling_role" src="https://user-images.githubusercontent.com/92983658/236851017-8ee1154c-d72f-469c-9fd8-06088193e45f.png">

<br>

## Inject Secrets into the Tooling Application
In order to inject secrets into the tooling application, a service-account will need to be created with the name configured in the `kubernetes role` and attached to the `tooling application` pod. This will create a sidecar which is the Vault agent and it will do the authentication and inject the secrets into the application.

- In the `tooling-app-kustomize/overlays` directory where the Kubernetes manifest files exist, create a file `service-account.yaml` and add:

```

# tooling-app-kustomize/overlays/dev/service-account.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: tooling-sa
  
```

<br>

<img width="979" alt="service_yaml" src="https://user-images.githubusercontent.com/92983658/236853140-39813e2e-100a-4e8d-8471-d5b87254a1c4.png">

<br>

- In the `tooling-app-kustomize/overlays/dev/deployment.yaml` file, replace the content with:

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tooling-deployment
spec:
  replicas: 3
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: 'true'
        vault.hashicorp.com/role: 'tooling-role'
        vault.hashicorp.com/agent-inject-status: 'update'
        vault.hashicorp.com/agent-inject-secret-database-cred.txt: 'app/data/database/config/dev'
        vault.hashicorp.com/agent-inject-template-database-cred.txt: |
          {{- with secret "app/data/database/config/dev" -}}
          export db-username={{ .Data.data.username }}
          export db-password={{ .Data.data.password }}
          export db-host={{ .Data.data.password }}
          {{- end -}}
    spec:
      serviceAccountName: tooling-sa
      
  ````
  
  <br>
  
  <img width="1067" alt="tooling_deployment" src="https://user-images.githubusercontent.com/92983658/236853723-85fcd47d-5f55-4eba-b39c-9adb3b1eb869.png">

<br>

- Add the `service-account.yaml` file under the resources field of the `Kustomization` file in the `dev` directory. The Kustomization file should look like this:

```

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
resources:
  - ../../base
  - namespace.yaml
  - service-account.yaml

commonLabels:
  env: dev-tooling

patches:
  - deployment.yaml
  
```

<br>

<img width="1067" alt="dev_kustomisation_2a" src="https://user-images.githubusercontent.com/92983658/236854431-a08b4e3f-b1e2-4dbf-b94c-696849ccdad5.png">

<br>

- apply the configuration.

```

kubectl apply -k overlays/dev

```

<br>

<img width="985" alt="overlays_dev_config" src="https://user-images.githubusercontent.com/92983658/236855841-80236697-a40c-4c2e-85f1-94fc404f8ca9.png">

<br>

- inspect the tooling application pod

```

kubectl exec -it -n dev deployment/tooling-deployment \
  -c tooling -- cat /vault/secrets/database-cred.txt
  
```

<br>

<img width="1380" alt="inspect_tooling" src="https://user-images.githubusercontent.com/92983658/236859797-161afc54-b080-4da0-a2a0-4a2b89361686.png">

<br>

## Working with the Vault UI
Apart from `vault cli`, vault UI can also be used to do some of the configurations. To view the vault UI, copy and paste the vault address on your browser, then you will see the login page.

```

vault.masterclass.dev.<your domain>

```
  
<br>

<img width="1391" alt="vault_login" src="https://user-images.githubusercontent.com/92983658/236861036-84543e81-94c5-4548-bcc0-2ba2d1c257ad.png">

<br>

- login using the token you got after initializing the vault cluster
- Check the database KV secret created before which is at the path `app/database/config/dev`.

<br>

<img width="1389" alt="vault_3" src="https://user-images.githubusercontent.com/92983658/236862678-7667f1bb-1c23-4fa6-bc6c-07ec283302ff.png">

<br>

- Check the tooling-role kubernetes auth method role.

<br>

<img width="1389" alt="vault_0" src="https://user-images.githubusercontent.com/92983658/236863607-3566a2df-f7d7-4e7e-becd-6d894baecb1c.png">

<br>

<img width="1391" alt="vault_4" src="https://user-images.githubusercontent.com/92983658/236863356-91d0e2b8-c113-4c6f-bada-90aac0040f51.png">

<br>

<img width="1388" alt="vault_5" src="https://user-images.githubusercontent.com/92983658/236863375-f1e822ac-69b8-43f8-bf5c-80f0639efec4.png">

<br>

- Navigate to the vault policy attached to the tooling-role kubernetes auth method.

<br>

<img width="1389" alt="policy_1" src="https://user-images.githubusercontent.com/92983658/236863943-da267792-f8ed-457e-ac85-b3ee11292463.png">

<br>

<img width="1392" alt="policy_2" src="https://user-images.githubusercontent.com/92983658/236863963-8a88cf1d-5338-4bfa-b3c7-a984725bffd9.png">

<br>


