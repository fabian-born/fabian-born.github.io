---
title: "Jenkins K8s NetApp CI/CD Demo - Part 1"
date:  2019-03-21T13:00:00+02:00
draft: false
categories: ["NetApp","Jenkins","Kubernetes","Trident","DevOps"]
banner: /img/jenk8sna/img_jenkins-k8s-netapp_env.png
layout: post
---
This example will show you the CI/CD integration with Jenkins, Kubernetes and NetApp on a simple Web Application.

![environment overview](/img/jenk8sna/img_jenkins-k8s-netapp_env.png)

## Pre-requisits
Before start this demo there are some requirements:
- Docker Hub Account for publishing the docker image
- a GIT repository (Github, Gitlab, Bitbucket, etc.)
- Kubernetes Cluster
- NetApp Storage System (FAS/AFF/CVO)
- Trident must be configured in K8s cluster  -- Link: [Trident Docs]
- helm charts must be configured in K8s cluster -- Link: [Helm Chart Docs]

## Preparation environment
#### Cloning Git repository
Goto my GitHub repository: [github-repo]( https://github.com/fabian-born/jenkins-k8s-netapp.git "my github repository")

```bash
cd /opt
git clone https://github.com/fabian-born/jenkins-k8s-netapp.git .
cd jenkins-k8s-netapp
```


## NodeJS and Redis Webapp
#### Implementing WebApp
The first step is to modify the app files for your environment:
* in helm/webapp/templates/webapp-redis.yaml (line 5)

```yaml
volume.beta.kubernetes.io/storage-class: <your storage class name> 
```
Now you can install the helm chart
```bash
helm install -n prod ./helm/webapp
```

Now the connection to the WebApp could be tested. For testing open the browser and add the URL of your WebApp: **http://NODE_IP:30000** or if a loadbalancer available: **http://LOADBALANCER_IP**


## Jenkins
#### Installing jenkins pod with special values
There are some changes in jenkins-helm.yaml
```yaml
Persistence:
...
  Enabled: true
... 
  StorageClass: <your storage class name>
...
```
and
```yaml
rbac:
  install: true
```

After modifying the file the jenkins container can be created:

```bash
helm install -n jenkins -f jenkins-helm.yaml stable/jenkins
```
After installing jenkins you have to find out your admin password:

1. Get your 'admin' user password by running:

```bash
printf $(kubectl get secret --namespace default jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

2. Get the Jenkins URL to visit by running these commands in the same shell:

NOTE: It may take a few minutes for the LoadBalancer IP to be available. You can watch the status of by running: **(remove the "space" charackter between the Curly Brackets!)**
```bash 
kubectl get svc --namespace default -w jenkins
export SERVICE_IP=$(kubectl get svc --namespace default jenkins --template "'{ { range (index .status.loadBalancer.ingress 0) } }{ { . } }{ { end } }'")
echo http://$SERVICE_IP:8080/login
```



If there is no Loadbalancer IP available you can use the address **http://NODE_IP:8080/login**
 
3. Login with the password from step 1 and the username: admin


--> read the next article: [Part 2](https://blog.fabianborn.net/blog/2019/04/03/jenkins-kubernetes-netapp-cicd-integration-p2/ "Part 2")

 <!---  
    Reference/Links which are included in text
 --->

[trident docs]: https://netapp-trident.readthedocs.io/en/latest/ "trident documentation"

[helm chart docs]: https://helm.sh/ "Helm docs"

[environment]: http://fabianborn.gitlab.io/img/jenk8sna/img_jenkins-k8s-netapp_env.png "environment"
