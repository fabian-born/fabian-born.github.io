---
title: Build your DevOps Environment

date: 2022-03-21
draft: false
tags: NetApp Docker Kubernetes Trident DevOps Azure GCP AWS AstraControl
categorie: howto
banner: /assets/images/content/nadevopsp1.png
layout: post
---



# Build your DevOps Environment!
The LinkedIn post ["NetApp in a Cloud DevOps Context"](https://www.linkedin.com/pulse/netapp-cloud-devops-context-fabian-born/) showed how an optimized Kubernetes environment can look in the cloud. This series of articles now show how to build such an environment in an automated way.

There will be four parts to this series.

   - Part 1: Planning
   - Part 2: Setting up the required infrastructure
   - Part 3: Activation of the backup
   - Part 4: Monitoring and optimization of the environment
   
The cloud providers Microsoft Azure and Google Cloud Platform are used to set up the test environments. However, this is only because NetApp's Astra Control Service does not support Amazon Web Service. As soon as it is available, this will be made up for immediately.

**Before we start, a short note:** The scripts used here can be used for testing purposes. However, I assume no liability for the use in production environments. In addition, these also **generate cloud resources, which will generate costs!**

Deploying scripts is easy. However, you need to think about several points in advance:

   Setup of the network in the cloud - IP addresses, firewall rules, etc.
   Access rights to cloud resources
   Licenses

### Preparation of the Cloud Provider Environments
To set up a test environment, you need to register the NetApp resource provider in Azure. As soon as the registration is completed, a NetApp account must be created in Azure, in which the capacity pool and volumes will later be created.

```bash
az account set --subscription <subscriptionId>
```

As soon as Azure NetApp Files has been released, a NetApp account is created in a resource group that can be used for the test environment. This resource group is then also used later on.

```bash
az netappfiles account create --resource-group <rg-name> --location <location> --account-name <anf-account-name>
```

With Google Cloud, you select the Cloud Volume Service in the Market Place and accept the EULA. Before you can provision storage, however, you have to establish communication with the managed NetApp. This can be done conveniently via the GCP Cloud Console.

```bash
## CVS "default"
gcloud --project=<gcp-project> compute addresses create netapp-addresses-sds-default --global --purpose=VPC_PEERING --prefix-length=25 --network=default --no-user-output-enable
gcloud --project=<gcp-project> services vpc-peerings connect --service=cloudvolumesgcp-sds-api-network.netapp.com --ranges=netapp-addresses-sds-default --network=default --no-user-output-enabled
gcloud --project=<gcp-project> compute networks peerings update netapp-sds-nw-customer-peer --network=default --import-custom-routes --export-custom-routes


## CVS Performance
gcloud --project=<gcp-project> compute addresses create netapp-addresses-default --global --purpose=VPC_PEERING --prefix-length=24 --network=default --no-user-output-enabled
gcloud --project=<gcp-project> services vpc-peerings connect --service=cloudvolumesgcp-api-network.netapp.com --ranges=netapp-addresses-default --network=default --no-user-output-enabled

gcloud --project=<gcp-project> compute networks peerings update netapp-cv-nw-customer-peer --network=default --import-custom-routes --export-custom-routesd
```

Now you can start provisioning storage.


### Deploying the infrastructure
Now that the basics are in place, this article looks at deploying Kubernetes platforms to cloud providers (Microsoft Azure and Google Cloud). Of course, Kubernetes clusters can be deployed manually based on virtual machines. However, in an optimized environment, the cloud provider native services should be used: i.e. Azure Kubernetes Service or Google Kubernetes Engine.

To ensure that each environment always looks the same, Terraform is used for deployment. Note: The scripts provided can be used for testing purposes. However, I, Fabian Born, do not assume any liability.

All necessary files are on GitHub and can be downloaded at any time.
```bash
git clone https://github.com/fabian-born/build-demo-environment.git
```
The repo is structured in such a way that you have a separate folder with settings options for each cloud provider:
![Folder Structure!](/buildenv-gitfolder.png "Folder Structure")

Before using the scripts, ```terraform.tfvars``` must be adapted to the cloud provider environment.

#### For Azure
```bash
az_appId  = ""
## AppPassword
az_password = ""
## TenantID
az_tenantId = ""
aks_node_number = 3
az_existingRG = "astra-demo-rg"
az_anfaccount = "astra-demo-anf"
az_anf_sl = "Standard"
az_anf_poolsize = 4
az_anf_poolname = ""
prefix = ""
```

#### and for GCP
```bash
prefix = ""

gcp-project = ""
gcp-sa = ""

region = ""
network = "default"
subnetwork = ""
ip_range_pods = ""
ip_range_services = ""
nodepoolname = "node-pool"
```


In order to start the installation, all required Terraform modules must first be downloaded (1). After that it is recommended to perform a planning run. Here you can see if it will work. Then run the script against the Cloud Provider API.

```bash
(1) terraform init
(2) terraform plan
(3) terraform apply
```

As a result, you end up with a ready-to-run Kubernetes platform on Azure or GCP.

[![Deploy it on GKE](https://img.youtube.com/vi/-UDWRjn4GUw/2.jpg)](https://youtu.be/-UDWRjn4GUw)
