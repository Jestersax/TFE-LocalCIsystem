# TFE-LocalCIsystem


Here, I'm about to explain how to install and configure the local system for continous integration of code that I designed as part of my  Bachelor Thesis (Bachelor in Computer Science and Engineering)

This system includes Gitea as code host, Jenkins as the automation server, and SonarQube for analyzing the code included in Gitea, generating quality metrics so you can get info about the reliability, availability, usability and security of the software you would be developing.

# Table of contents
[1. Previous steps](https://github.com/Jestersax/TFE-LocalCIsystem#previous-steps)

[2. Getting Started](https://github.com/Jestersax/TFE-LocalCIsystem#getting-started)

...[2.1. Minikube Cluster](https://github.com/Jestersax/TFE-LocalCIsystem#minikube-cluster)

[3. Deploying tools](https://github.com/Jestersax/TFE-LocalCIsystem#deploying-tools)

...[3.1. Gitea](https://github.com/Jestersax/TFE-LocalCIsystem#gitea)

...[3.2. Jenkins](https://github.com/Jestersax/TFE-LocalCIsystem#jenkins)

...[3.3. SonarQube](https://github.com/Jestersax/TFE-LocalCIsystem#sonarqube)

[4. Integrating tools - guide](https://github.com/Jestersax/TFE-LocalCIsystem#integration-tools--guide)


## Previous Steps

First of all, you need to check if you have to check if virtualization is supported on your computer. The way you can check it is different depending on your operating system. So you can see how to do it on this [documentation](https://kubernetes.io/docs/tasks/tools/install-minikube/) 

Once you know if it's supported, you get to download minikube so you can start your local kubernetes cluster.
To download minikube, install kubectl, helm, and a VM if you don't already have one, check the documentation mentioned before.


## GETTING STARTED

### MINIKUBE CLUSTER

Local cluster for development and testing. Start the VM with the command:
```
minikube start --cpus=4 --memory=8192
```
We can also set the amount of disk size we want the cluster to get with the flag "--disk-size=" in MB. 

The amount of memory we set for this cluster is enough to make de CI system work propertly, with no service interruption.


## DEPLOYING TOOLS

Here I'll explain how to get the helm charts needed, and how to change them so the tool fits in the CI system designed for this thesis, so you know how to modify them to make them fit in your own continous integration system.

If you are not intrested in learning about that, you can just get all the helm charts [HERE](https://github.com/Jestersax/TFE-LocalCIsystem/charts)

### GITEA

GitHub-like SMC tool. Our application code will be located in a repository we'll create in Gitea.

To get Gitea's helm chart, we need to add the helm repository to our helm repos. Using the next command:
```
helm repo add k8s-land https://charts.k8s.land
```
Next, load the values.yaml into your local so you can edit:
```
helm show values k8s-land/gitea > gitea-values.yaml
```
**Service type change!**: Change the Service type to NodePort in the helm chart so it can be accessed from your browser.
```
service:
  http:
    serviceType: NodePort
    port: 3000
```
Ready to deploy gitea in our cluster with Helm:
```
helm install gitea k8s-land/gitea -f gitea-values.yaml
```

### JENKINS

Our orchestration server. 

Getting Jenkins helm chart on your computer. There's no need to add a new repo to our helm repo list. Use the next command:
```
helm show values stable/jenkins > jenkins-values.yaml
```

**Service type change!**: Change the Service type to NodePort in the helm chart so it can be accessed from your browser, like you did with gitea.
```
service:
  type: NodePort
```
Ready to deploy jenkins in our cluster:
```
helm install jenkins stable/jenkins -f jenkins-values.yaml
```
It'll show a command to get your admin password in Jenkins. Execute that command and write down your password for later.

### SONARQUBE

Static code analysis platform.

To get SonarQube's helm chart, we need to add the helm repository to our helm repos. Using the next command:
```
helm repo add oteemocharts https://oteemo.github.io/charts
```
Next, load the values.yaml into your local so you can edit:
```
helm show values oteemocharts/sonarqube > sonarqube-values.yaml
```
**Service type change!**: Change the Service type to NodePort in the helm chart so it can be accessed from your browser.
```
service:
  http:
    serviceType: NodePort
```
**Important**: Make sure PostgreSQL is enabled, so you deploy the database chart as you deploy sonarqube's chart.
```
postgresql:
enabled: true
```

Ready to deploy gitea in our cluster with Helm:
```
helm install sonarqube oteemocharts/sonarqube -f sonarqube-values.yaml
```

## INTEGRATING TOOLS GUIDE

### GITEA - JENKINS INTEGRATION


First, we'll need to access gitea service and jenkins server through our browser. To list the services deployed in your cluster, you use the command:
```
kubectl get services
```

To access the services, you can do it using their IP address within the cluster or using the next command:
```
minikube service [servicenameinourcluster]
```

In this case:
```
minikube service gitea-gitea-http
```
```
minikube service jenkins
```

Once you access them through the browser, you need to create a user in gitea so you can commit changes to your code, create repositories, etc.





