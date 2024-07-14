---
title: "Installer Addon for NetApp Trident using kubectl - Update"
date:  2021-01-22
draft: false
categories: howto
tags: ["NetApp","Docker","Kubernetes","Trident","DevOps"]
banner: /assets/images/cobtent/autotridentupdate.png
layout: post
author : "Fabian Born"
---
### Trident Installer Addon
Three month ago I published my first version of an automated NetApp Trident installer for Kubernetes cluster. 
In the last days I had some time to continue development on my installer. Time to change:  my first version based on a bash script which runs in extra docker container. For running the script it was required to mount the kubeconfig file into the docker container.

Now the architecture of the installer changed to a single manifest which includes all required parts. The new installer is using following "features": 

  - containerized tridentctl
  - Deploying of multiple backends and storage classes
  - Using a service account for deploying trident

I shared the Dockerfile as well as a detailed [description/documentation]( https://github.com/fabian-born/trident-installer-addon "Trident Installer Addon") on my GitHub repo. 

Enjoy testing 😉


#### Feedback
Try this method and give me a feedback. If you encounter any problems, please open a [github issue]( https://github.com/fabian-born/trident-installer-addon/issues  "open a issue").
 
