---
title: "FSxN + EKS = ❤️"
date:  2024-11-17T00:00:00+02:00
draft: true
categories: ["NetApp","Docker","Kubernetes","Trident","DevOps","AWS","EKS"]
banner: "https://github.com/project-epicshit/project-epicshit.github.io/assets/36699674/6c80b9b6-c0c0-415a-9606-178ce4f9b743"
layout: post
---

## FSx for NetApp ONTAP and EKS more than a perfect match

#### Architecture
![grafik](https://blog.fabianborn.net/assets/images/content/eks-architecture.png)

#### Why EKS with FSxN
Amazon FSx for NetApp ONTAP with Amazon EKS provides a scalable and fully managed storage solution that complements containerized workloads by offering high-performance, shared file storage. Here’s why it’s a great fit:

  - Persistent Storage for Stateful Applications: FSx for NetApp ONTAP delivers a highly available, low-latency file system for Kubernetes pods that need persistent storage, ideal for databases or applications requiring data persistence across deployments. 
  - NFS and iSCSI Support: It supports Network File System (NFS) and the iSCSI block protocols, enabling seamless integration with Kubernetes volumes, making it easier for EKS to manage shared storage across different applications and services.
  - Data Management Features: FSx for NetApp ONTAP provides advanced features like snapshots, cloning, data tiering, and efficient data replication, enhancing data protection and performance for EKS workloads.
  - Cost Efficiency: With features like automatic data tiering, frequently accessed data stays on high-performance SSDs while less-accessed data moves to lower-cost storage, optimizing cost without sacrificing performance.
  - Simplified Operations: The fully managed service reduces operational overhead, allowing EKS users to focus on application development while AWS handles the complex tasks of storage management, scaling, and backup.

In essence, FSx for NetApp ONTAP with EKS enables running stateful applications with reliable, scalable, and feature-rich storage, all while maintaining ease of use and integration within the AWS ecosystem.


The combination of AWS EKS and NetApp products provides an HA solution for running Kubernetes workloads across availability zones (AZs). From a container perspective, there is always the possibility of restarting on another node in another zone. With NetApp Storage for Persistent Data there is an HA solution which managed the failover.

NetApp Trident is used to connect the storage to EKS. As a CSI driver, Trident is responsible for provisioning the PVC on FSxN and mounting it directly into the container.



#### Prerequisites
- AWS credentials that provides the necessary permissions to create the resources (VPC, Compute, etc)





## Build the perfect match
1. Creating EKS
2. Deploying  FSx for NetApp ONTAP
3. Installing and configuring Trident

### 1. Deploying EKS Cluster


### 2. Deploying FSx for NetApp ONTAP

Now that the EKS cluster has been created, the VPC and subnet IDs for FSxN are required. These are displayed on the CLI among other things:

```bash

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

Now EKS and FSxN have been deployed, here comes the Trident installation and configuration.

But, before installing Trident, some informations are needed for the configuration. All required informations are displayed in the detail view in the tab "Endpoints" of the SVM: Management IP,NFS IP or iSCSI IPs - depending on which Data Protokol will be used. **NetApp ONTAP could provide both on the same SVM.**

Trident needs access to the FSxN instance. For the access it requires credentials stored in EKS:

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

Prepare the Storage Classes for Kubernetes ```fsxn_storageclass.yaml```:
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
1. Login into EKS Cluster
2. Install Trident Provisioner operator
3. Install Trident Protect operator 
```
helm installer
```   
4. Apply configurations
```
kubectl apply -f trident_backend_fsxadmin_credentials.yaml -n trident
kubectl apply -f trident_backend_fsxn.yaml -n trident
kubectl apply -f fsxn_storageclass.yaml
```




