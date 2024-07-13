---
title: "FSxN + ROSA = ❤️"
date:  2023-10-05T00:00:00+02:00
draft: true
categories: ["NetApp","Docker","Kubernetes","Trident","DevOps","AWS","RedHat","AstraControl","Openshift"]
banner: "https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/6c80b9b6-c0c0-415a-9606-178ce4f9b743"
layout: post
---

## FSx for NetApp ONTAP and RedHat Openshift on AWS a perfect match

### Solution Overview
﻿ROSA is one of the latest services offered by RedHat on Amazon Web Services (AWS). This means that it is now possible to use the same products that are familiar from the DataCenter world in the public cloud, with the advantage that they are now available as a managed service. ROSA is enhanced with Amazon FSx for NetApp ONTAP (FSxN). By using FSxN, Openshift gets more stability and redundancy across Availability Zones in the cloud.

#### Architecture
![grafik](https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/6c80b9b6-c0c0-415a-9606-178ce4f9b743)


#### Why ROSA with FSxN
The combination of RedHat and NetApp products provides an HA solution for running Kubernetes workloads across availability zones (AZs). From a container perspective, there is always the possibility of restarting on another node in another zone. With NetApp Storage for Persistent Data there is an HA solution which managed the failover.

NetApp Astra Trident is used to connect the storage to ROSA. As a CSI driver, Trident is responsible for provisioning the PVC on FSxN and mounting it directly into the container

#### Prerequisites
- AWS credentials that provides the necessary permissions to create the resources (VPC, Compute, etc)
- AWS account has been linked with RedHat (see ROSA documentation)

#### Connectivity
The most important part of this solution is that connectivity can be established between ROSA nodes and FSxN SVM. It is necessary to pay attention to VPC, subnet, routing table etc..
There are many ways to accomplish this, but for the sake of simplicity, in this guide everything is configured in the same VPC and subnets.




## Build the perfect match
1. Creating ROSA
2. Deploying  FSx for NetApp ONTAP
3. Installing and configuring Trident

### 1. Creating a  ROSA Cluster with 
There minumum of three ways to deploy a ROSA cluster. 
	- On the CLI with the command "rosa"
	- WebUI => https://console.redhat.com
	- Terraform

This guide will use the rosa CLI [^1]

```bash
rosa create cluster
```

following inputs are required - for the most questions I used the default settings!

```bash
I: Enabling interactive mode
? Cluster name: **democluster**
? Deploy cluster with Hosted Control Plane (optional): **No**
? Create cluster admin user: **Yes**
? Username: **demoadmin**
? Password: [? for help] **************
? Deploy cluster using AWS STS: **Yes**
W: In a future release STS will be the default mode.
W: --sts flag will not be necessary if you wish to use STS.
W: --non-sts/--mint-mode flag will be necessary if you do not wish to use STS.
? OpenShift version: **4.12.35**
? Configure the use of IMDSv2 for ec2 instances optional/required (optional): 
I: Using arn:aws:iam::XXXXXXXXXXXX:role/ManagedOpenShift-Installer-Role for the Installer role
I: Using arn:aws:iam::XXXXXXXXXXXX:role/ManagedOpenShift-ControlPlane-Role for the ControlPlane role
I: Using arn:aws:iam::XXXXXXXXXXXX:role/ManagedOpenShift-Worker-Role for the Worker role
I: Using arn:aws:iam::XXXXXXXXXXXX:role/ManagedOpenShift-Support-Role for the Support role
? External ID (optional): 
? Operator roles prefix: **democluster-s4i6**
? Deploy cluster using pre registered OIDC Configuration ID: **No**
W: No OIDC Configuration found; will continue with the classic flow.
? Tags (optional): 
? Multiple availability zones (optional): **Yes**
? AWS region: **us-east-1**
? PrivateLink cluster (optional): **No**
? Machine CIDR: **10.0.0.0/16**
? Service CIDR: **172.30.0.0/16**
? Pod CIDR: **10.128.0.0/14**
? Install into an existing VPC (optional): **No**
? Select availability zones (optional): **No**
? Enable Customer Managed key (optional): **No**
? Compute nodes instance type: **m5.xlarge**
? Enable autoscaling (optional): **No**
? Compute nodes: **3**
? Default machine pool labels (optional): 
? Host prefix: **23**
? Enable FIPS support (optional): **No**
? Encrypt etcd data (optional): **No**
? Disable Workload monitoring (optional): **Yes**

I: Creating cluster 'democluster'
```

After filling out all the questions you will get an overview what will be deployed now!

```
I: To create this cluster again in the future, you can run:
   rosa create cluster ...
I: To view a list of clusters and their status, run 'rosa list clusters'
I: Cluster 'democluster' has been created.
I: Once the cluster is installed you will need to add an Identity Provider before you can login into the cluster. See 'rosa create idp --help' for more information.

Name:                       democluster
ID:                         26m2gh2eplvr1m7ffaaku4mlmtkr63v6
External ID:                
Control Plane:              Customer Hosted
OpenShift Version:          
Channel Group:              stable
DNS:                        Not ready
AWS Account:                XXXXXXXXXXX
API URL:                    
Console URL:                
Region:                     us-east-1
Multi-AZ:                   true
Nodes:
 - Control plane:           3
 - Infra:                   3
 - Compute:                 3
Network:
 - Type:                    OVNKubernetes
 - Service CIDR:            172.30.0.0/16
 - Machine CIDR:            10.0.0.0/16
 - Pod CIDR:                10.128.0.0/14
 - Host Prefix:             /23
STS Role ARN:               arn:aws:iam::XXXXXXXXXXX:role/ManagedOpenShift-Installer-Role
Support Role ARN:           arn:aws:iam::XXXXXXXXXXX:role/ManagedOpenShift-Support-Role
Instance IAM Roles:
 - Control plane:           arn:aws:iam::XXXXXXXXXXX:role/ManagedOpenShift-ControlPlane-Role
 - Worker:                  arn:aws:iam::XXXXXXXXXXX:role/ManagedOpenShift-Worker-Role
Operator IAM Roles:
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-machine-api-aws-cloud-credentials
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-cloud-credential-operator-cloud-crede
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-image-registry-installer-cloud-creden
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-ingress-operator-cloud-credentials
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-cluster-csi-drivers-ebs-cloud-credent
 - arn:aws:iam::XXXXXXXXXXX:role/democluster-s4i6-openshift-cloud-network-config-controller-cloud
Managed Policies:           No
State:                      waiting (Waiting for OIDC configuration)
Private:                    No
Created:                    Oct  5 2023 08:39:36 UTC
User Workload Monitoring:   disabled
Details Page:               https://console.redhat.com/openshift/details/s/2WKxfWyuNMhPGuwp0SqdW95Q5zk
OIDC Endpoint URL:          https://rh-oidc.s3.us-east-1.amazonaws.com/26m2gh2eplvr1m7ffaaku4mlmtkr63v6 (Classic)

I: 
Run the following commands to continue the cluster creation:

	rosa create operator-roles --cluster democluster
	rosa create oidc-provider --cluster democluster

I: To determine when your cluster is Ready, run 'rosa describe cluster -c democluster'.
I: To watch your cluster installation logs, run 'rosa logs install -c democluster --watch'.

```

For creating the operator-roles and the oidc-provider I used the "auto" creation mode!

﻿![grafik](https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/9850ce16-6c30-45e5-bfcc-d4501ddbe18f)

It takes a while for the cluster to be deployed. RedHat gives a time range of 30-60 minutes!


### 2. Deploying  FSx for NetApp ONTAP

Now that the ROSA cluster has been created, the VPC and subnet IDs for FSxN are required. These are displayed on the CLI among other things:

```bash
aws ec2 describe-vpcs --region us-east-1 --filters Name=tag:red-hat-clustertype,Values=rosa
aws ec2 describe-subnets --region us-east-1 --filters Name=tag:red-hat-clustertype,Values=rosa Name=tag:Name,Values="*public*"
```
Alternatively, you can also get this information via the AWS Console. 

FSxN can also be configured in several ways. This. Guide now describes the way via the AWS Console. To do this, go to FSx in the Console, select "Create file system", and click on "Configure".

![grafik](https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/be16e95a-c090-4ead-a1a0-e5cd43b1682d)

Select "quick create"

Now the following fields must be filled in:
- [x] File system name: **<name>**
- [x] Deployment type: **Multi-AZ** (redundant ONTAP instance over two AZ)
- [x] SSD Capacity: **1024**
- [x] VPC: **select the VPC to be used for FSx ONTAP. In this guide, the same as the ROSA cluster.**
- [x] Storage efficiency: **enabled**

Click *"Next"*, *"Verify the following attributes before proceeding"* and then *"Create file system"*

In the background AWS creates a new FSxN instance, which takes a while.

![grafik](https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/0da9fbab-f0c7-4701-ae22-319236f2bf99)

When creating the file system, an SVM (Storage Virtual Machine) is created directly on ONTAP. The PVC will be created in this instance later.

﻿![grafik](https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/8c495740-e996-4248-b660-81404414c479)


Verification of SVM Settings:
﻿![grafik](https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/12ca3aad-050a-4fcc-bc68-2c57391028f0)

### 3. Installing and configuring Trident

Now that ROSA and FSxN have been deployed, here comes the Trident installation and configuration.

But, before installing Trident, some informations are needed for the configuration. All required informations are displayed in the detail view in the tab "Endpoints" of the SVM: Management IP,NFS IP or iSCSI IPs - depending on which Data Protokol will be used. **NetApp ONTAP could provide both on the same SVM.**

Trident needs access to the FSxN instance. For the access it requires credentials stored in OpenShift:

```
# FSxN Admin Credential
# trident_backend_fsxadmin_credentials.yaml
apiVersion: v1
kind: Secret
metadata:
  name: backend-tbc-ontap-secret
type: Opaque
stringData:
  username: fsxadmin
  password: MyFSXAdminPassword
```

In the backend file ```trident_backend_fsxn.yaml```

```
# Trident Backend
# trident_backend_fsxn.yaml
apiVersion: trident.netapp.io/v1
kind: TridentBackendConfig
metadata:
  name: backend-tbc-ontap-nas
spec:
  version: 1
  storageDriverName: ontap-nas
  managementLIF: 10.0.255.217
  backendName: tbc-ontap-nas
  exportPolicy: default
  storagePrefix: democluster_
  autoExportPolicy: true
  autoExportCIDRs: ['0.0.0.0/0']
  credentials:
    name: backend-tbc-ontap-secret
```

Prepare the Storage Classes for Openshift ```fsxn_storageclass.yaml```:
```
# fsxn_storageclass.yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-snapclass
driver: csi.trident.netapp.io
deletionPolicy: Delete
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: trident-nas
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: csi.trident.netapp.io
parameters:
  backendType: "ontap-nas"
allowVolumeExpansion: True
```

Now Trident can be installed:
1. Login into ROSA Cluster
2. Install Trident Operator
```
wget https://github.com/NetApp/trident/releases/download/v23.07.1/trident-installer-23.07.1.tar.gz
tar -xf trident-installer-23.07.1.tar.gz
cd trident-installer
kubectl create -f deploy/crds/trident.netapp.io_tridentorchestrators_crd_post1.16.yaml
kubectl apply -f deploy/namespace.yaml
kubectl apply -f deploy/bundle_<pre/post>_1_25.yaml
kubectl create -f deploy/crds/tridentorchestrator_cr.yaml
```  

3. Apply Configurations
```
oc apply -f trident_backend_fsxadmin_credentials.yaml -n trident
oc apply -f trident_backend_fsxn.yaml -n trident
oc apply -f fsxn_storageclass.yaml
```



### Links / Footnotes
[ROSA - Creating Cluster Guide](https://docs.openshift.com/rosa/rosa_install_access_delete_clusters/rosa-sts-creating-a-cluster-quickly.html)

[^1]: ROSA CLI



