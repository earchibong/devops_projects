# Deploy and Manage Business Application in Kubernetes – Helm | Kustomize | GitOps
There are multiple choices available when it comes to deploying applications into kubernetes.

**1. Write YAML files and deploy with kubectl:**
This is the easiest method where you write YAML for deployments, services, ingress. etc., and then deploy with `kubectl apply -f <YAML-FILE>`. 
This is usually the default way when getting started with kubernetes OR during development and exploration. 
But it is not sufficient or reliable when it comes to managing the infrastructure in production because it is hard to keep track of multiple yaml files. 
Imagine a project where there are tens or hundreds of microservices/applications that need to be managed across multiple environments. 
With this type of approach, you will end up in a `PEBKAC Situation`

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

**3.Kustomize Overlays:**
To overcome the challenges of Helm identified above, using a tool that is already embeded as part of kubectl; makes more sense for most use cases.
The downside to this is that it does not manage the lifecycle of aplications like Helm is able to do. Therefore, other methods will need to be used
alongside Kustomize for this.

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

 
