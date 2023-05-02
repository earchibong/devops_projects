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

<img width="1015" alt="base_deploy" src="https://user-images.githubusercontent.com/92983658/235634361-1f8844a4-e5ee-4fe8-92b0-860a8b2f715d.png">

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
