# TFE-LocalCIsystem


Here, I'm about to explain how to install and configure the local system for continous integration of code that I designed as part of my  Bachelor Thesis (Bachelor in Computer Science and Engineering)

This system includes Gitea as code host, Jenkins as the automation server, and SonarQube for analyzing the code included in Gitea, generating quality metrics so you can get info about the reliability, availability, usability and security of the software you would be developing.

# Table of contents
[1. Previous steps](https://github.com/Jestersax/TFE-LocalCIsystem#previous-steps)

[2. Getting Started](https://github.com/Jestersax/TFE-LocalCIsystem#getting-started)

... [2.1. Minikube Cluster](https://github.com/Jestersax/TFE-LocalCIsystem#minikube-cluster)

[3. Deploying tools](https://github.com/Jestersax/TFE-LocalCIsystem#deploying-tools)

... [3.1. Gitea](https://github.com/Jestersax/TFE-LocalCIsystem#gitea)

... [3.2. Jenkins](https://github.com/Jestersax/TFE-LocalCIsystem#jenkins)

... [3.3. SonarQube](https://github.com/Jestersax/TFE-LocalCIsystem#sonarqube)

[4. Integrating tools - guide](https://github.com/Jestersax/TFE-LocalCIsystem#integrating-tools-guide)

... [4.1. Gitea - Jenkins integration](https://github.com/Jestersax/TFE-LocalCIsystem#gitea---jenkins-integration)


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
With that command, you can check the IP address of each service, their access ports, and their names.

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
 
![credentials](https://github.com/Jestersax/TFE-LocalCIsystem/blob/images/guide-images/jenkins-credentials.png "Jenkins credentials")

After that, you need to download a plugin from the jenkins control panel called "Gitea server", and configure it. Just like in the image below:

![gitea plugin](https://github.com/Jestersax/TFE-LocalCIsystem/blob/images/guide-images/gitea-jenkins-servers-manage.png "Gitea server")

The server URL is the name of the gitea service in your cluster (Ex: gitea-gitea-http), followed by the name of your namespace (in this case I didn't set a name, so is 'default'); svc, por service, cluster.local, and the inner port of the service. In this case, 3000. 


Next step: Creating a code repository. The example repository would be 'tfg-repo'. Just give it the name you want.

After that, we create a multibranch pipeline:

```
Jenkins > create new job > Multibranch pipeline
```

![multibranch pipeline](https://github.com/Jestersax/TFE-LocalCIsystem/blob/images/guide-images/gitea-jenkins-branch-source-pipeline.png "Multibranch pipeline configuration")


Pointing at the gitea server you configurated in the step before, you set the name of the owner of the code's repository, and the repository you want to execute the automations on. For the build configuration, you'll need to add a 'Jenkinsfile' to your repository branch.

The jenkinsfile is where you configure the stages of a pipeline. You can get one to start with [HERE](https://github.com/Jestersax/TFE-LocalCIsystem/charts/Jenkinsfile)

When you have all this set up, all that's left to set for gitea is a Webhook, so a branch pipeline build is launched when there's an event going on in the code repository.

![Gitea webhook](https://github.com/Jestersax/TFE-LocalCIsystem/blob/images/guide-images/gitea-jenkins-webhook.png "Gitea webhook configuration")

To configure it, the URL points at the jenkins services in the cluster. On the image above it can be seen that we use a gogs webhook. Thats because when we first configure this system, the gitea webhook didn't work propertly, so you can use this kind of webhook if the gitea one is not working.


**IMPORTANT TIP!**: If the gitea webhook is not working, you need to install in jenkins a new plugin called "Gogs", so the gogs-webhook works. It doesn't need to be configured, just installed.


### SONARQUBE - JENKINS INTEGRATION

The steps decrived below are very similar to the ones listed above.

To access the sonarqube service, use the next command:
```
minikube service sonarqube-sonarqube
```

As we did with the credentials of the gitea user, so we could store them in jenkins, in SonarQube we must generate an authentication token and save it in the credentials section of the Jenkins administration.

![Sonar token](https://github.com/Jestersax/TFE-LocalCIsystem/blob/images/guide-images/sonar-jenkins-crear-token.png "SonarQube token")

We need to download a jenkins plugin called "SonarQube SonarScanner". Configuration is similar to the gitea servers plugin setup:

![Sonar server](https://github.com/Jestersax/TFE-LocalCIsystem/blob/images/guide-images/jenkins-sonar-config.png "Sonar plugin configuration")

Server URL merging the name of the sonarqube service in our cluster, namespace, svc, cluster.local, and the inner port from our service (9000). As credentials, we use the authentication token previously stored at jenkins administration.

As we add a Jenkinsfile for jenkins to recognize a code branch, we need to add a file called 'sonar-project.properties' file to the branch we want to analyze. 

````
# Metadata
sonar.projectKey=
sonar.projectName=
sonar.projectVersion=1.0

# Language
sonar.language=
````

In that file, we need to set the name and the key of the analysis project we have created in Sonar, and the programing language that we want to analyze with sonar.

You can get that sonar.properties file [HERE](https://github.com/Jestersax/TFE-LocalCIsystem/charts/sonar-project.properties) ; You'll need to set the name of the project you have created, the key to that project, and the language you would like to analyze.