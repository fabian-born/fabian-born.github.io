---
title: Astra Control
subtitle: Add new Kubernetes Cluster
date: 2022-04-03
draft: false
tag: NetApp Docker Kubernetes Trident DevOps Azure GCP AWS AstraControl
categorie: howto
banner: /assets/images/content/nadevops-autobackup.png
layout: post
---

## Automated enabling backup


In the first post, we wrote about the requirements for an optimized DevOps environment. In the second, we provisioned the Kubernetes cluster with NetApp storage.

As soon as applications run in the Kubernetes cluster and generate data, there is no getting around the topic of backup. Astra Control is used as the backup solution.

Astra Control is NetApp’s solution for Application-Aware Backup of Kubernetes apps. Astra Control backs up application metadata in addition to PVCs. Thus, entire applications can be restored. For this purpose, well-known NetApp technologies such as SnapShots are used. Snapshots are stored on S3 storage. If applications require special handling before and after SnapShots are created, special execution hooks can be created.

However, before you can automatically add the Kubernetes cluster to Astra Control Center, you must first make preparations on the Astra Control Service:

* Generate API token

This short video shows how to generate the API Key:

[![Generate API TOKEN](https://img.youtube.com/vi/SBiXwD3Bb9Y/0.jpg)](https://youtu.be/SBiXwD3Bb9Y) 

Now that the API key has been generated, let’s look at the tools that can be used to add a Kubernetes cluster to Astra Control.

1. NetApp Astra Control Python SDK
2. Ansible Playbook

The basis for both tools is the access to the Rest API. To perform actions it is necessary to get UUIDs for individual objects. To succeed, you need to perform the following steps.

1. Get Cloud Provider Id
2. Get Kubernetes Cluster Id
3. Get Default StorageClass Id
4. Enable management of Kubernetes Cluster


## NetApp Astra Control Python SDK

NetApp provides the Astra Control Python SDK on GitHub. This can be downloaded via

```bash
git clone https://github.com/NetApp/netapp-astra-toolkits.git
```
To get it running, some Python programs have to be installed. Detailed installation instruction is available on the GitHub repository.

Based on the four steps above, the use of the toolkit is as follows:

List of available cloud provider:
```bash
./toolkit.py -oyaml list clouds     
```
```yaml
    48cc550d-3b69-49d9-9515-0b1f8b20f35e:
    - Azure
    - Azure
    bc96b574-cb68-4113-9693-6438020cc27b:
    - GCP
    - GCP

```
List of available clusters and search for the IDs of the previously created Kubernetes cluster:

```bash
./toolkit.py -oyaml list clusters                                                                                                  
```
Output:
```yaml
    71aab527-0912-447e-a2cd-54ece8eb3305:
    - fabianb-holy-mink
    - aks
    - unmanaged
    - 48cc550d-3b69-49d9-9515-0b1f8b20f35e
```
List the available storage classes:

```bash
./toolkit.py -oyaml list storageclasses
```
Output:
```yaml
48cc550d-3b69-49d9-9515-0b1f8b20f35e:
  08ca2a26-b2bd-499a-a031-23522d77d40b: {}
  258aa38c-353c-4c94-b421-c3e7a630b70f: {}
  71aab527-0912-447e-a2cd-54ece8eb3305:
    472f6719-3fd8-400d-8423-55a4f62215a8: azurefile
    589c626a-1b58-4785-b445-ba777a151f7c: managed-csi-premium
    5e7c0e49-378c-49a7-83a9-8d2cd4856753: managed
    6b55adbf-74ea-4f59-8929-b82f14ea56c4: default
    ab215262-438f-4bfc-864e-1a8649795cde: azurefile-premium
    aef790c4-2050-49e9-9d0b-f31604e9d166: azurefile-csi-premium
    ba6d5a64-a321-4fd7-9842-9adce829229a: netapp-anf-perf-standard
    c78e3e2f-184c-45aa-bfd6-dcacf5b7750c: managed-csi
    e8e0163f-3b42-4589-9b22-aeabbf7fab03: azurefile-csi
    ff184761-6b2f-48df-9716-8f0c9f1b50fd: managed-premium
  9e67112f-e14a-4a3e-b6b2-13bb0f9816ad: {}
bc96b574-cb68-4113-9693-6438020cc27b: {}
```

Now that all the IDs are there, set the cluster to managed:

```bash
./toolkit.py manage cluster 71aab527-0912-447e-a2cd-54ece8eb3305 ba6d5a64-a321-4fd7-9842-9adce829229a
```

[![Add Cluster to ACS with Toolkit](https://img.youtube.com/vi/8JZcwOYJEgM/0.jpg)](https://youtu.be/8JZcwOYJEgM) 

Alternatively, Ansible Playbook can be used, which is available in the GitHub repository build-demo-environment.

## The Ansible playbook

For the execution of the Ansible playbook, the following two parameters must also be adjusted in the variable file:
```yaml
clustername: „clustername“
storageclassname: „default-storage-name“
cloudprovidername: „Azure / GCP“
```
Now you can run the playbook:

```bash
ansible-playbook add-cluster-to-acs.yaml
```
[![Add Cluster to ACS with Toolkit](https://img.youtube.com/vi/mJf5SBCN4eo/0.jpg)](https://youtu.be/mJf5SBCN4eo)

After the Kubernetes cluster has been successfully added to Astra Control, you can manage your applications. How this works has already been shown in this [Video](https://youtu.be/UcUUqcjWzFg) (which is currently only available in German - sorry for that).



### Resources
[GitHub Repository for this Post](https://github.com/fabian-born/build-demo-environment)

[Blog Post: Build your DevOps Environment!](https://project.epicshit.io/blog/2022/03/21/buildyourenv/)

[Video: Backup and Restore](https://youtu.be/UcUUqcjWzFg)

[NetApp Astra Control Python Toolkit](https://github.com/NetApp/netapp-astra-toolkits.git)
