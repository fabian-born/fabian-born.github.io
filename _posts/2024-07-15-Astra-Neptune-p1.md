---
title: "Astra Neptune"
date:  2024-02-24
draft: false
categories: howto
tags: ["NetApp","Astra","Kubernetes","Trident","DevOps","Backup"]
banner: /assets/images/content/astra_neptune.png
layout: post
toc: false
---
## Update Trident and activate ACP
 
The prerequisite is a current Trident (version 24.02). Depending on which version is currently on the cluster, this may need to be updated (e.g. if deployed via Helm: helm upgrade trident -n trident netapp-trident/trident-operator).
 
Then the ACP (Astra Control Provisioner) mode of Trident is activated, this enables the integration between Astra Control and Trident. If necessary, adjust the registry here (if internal registry) or create a registry secret in the Trident namespace so that it can pull the additional ACP image:

````
kubectl -n trident patch torc/trident --type=json -p='[ 
        {"op":"add", "path":"/spec/enableACP", "value": true},
        {"op":"add", "path":"/spec/acpImage","value": "cr.demo.astra.netapp.io/astra/trident-acp:24.02.0"}
    ]'
``` 
 
## Install Astra Connector
This is the component that is now running locally in the respective cluster (push vs. pull mode). This can initially also run independently of the central Astra Control instance.

 
Create CRDs and Namespaces: ```kubectl apply -f https://github.com/NetApp/astra-connector-operator/releases/download/24.02.0-202403151353/astraconnector_operator.yaml```
(creates two new namespaces, for the operator and the connector itself)
 
Create a registration secret (adapt it to your internal registration if necessary):

```kubectl create secret docker-registry regcred --docker-username= 86406506-9136-46b7-a9c3-4a7e26f17974 --docker-password= CcDtaZ-xyPjvjBMtJBdBxOWpNVG6DQjU4daIb-09RWg=  -n astra-connector --docker-server=cr.demo.astra.netapp.io````

 
Adapt the following YAML and then deploy. In principle, only the clusterName (later identifies the local cluster in the central management). If you already have it, you can also add the URL that the central instance will later have (via Ingress or Loadbalancer). 
 
```
apiVersion: astra.netapp.io/v1
kind: AstraConnector
metadata:
  name: astra-connector
  namespace: astra-connector
spec:
  astra:
    accountId: dummy
    clusterName: my-cluster
    skipTLSValidation: true #Should be set to false in production environments
    tokenRef: astra-token
  natsSyncClient:
    cloudBridgeURL: dummy.demo.netapp.com
  imageRegistry:
    name: cr.demo.astra.netapp.io
    secret: regcred
```
 
## Set up snapshots & backup 
First, you need the object store on which backups and metadata (also for snapshots) are stored. Here in the example for an Ontap-S3, for StorageGrid simply change the providerType to "storagegrid-s3". In addition, change the "name" if necessary (if several buckets are used, you can differentiate between them using this field). Also adjust the "endpoint" and "bucketName":

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: s3-creds
  namespace: astra-connector
type: Opaque
stringData:
  accessKeyID: <S3 access key>
  secretAccessKey: <S3 secret key>
---
apiVersion: astra.netapp.io/v1
kind: AppVault
metadata:
  name: my-appvault
  namespace: astra-connector
spec:
  providerType: ontap-s3
  providerConfig:
    endpoint: s3.demo.netapp.com
    bucketName: astra
    skipCertValidation: "true"
  providerCredentials:
    accessKeyID:
      valueFromSecret:
        name: s3-creds
        key: accessKeyID
    secretAccessKey:
      valueFromSecret:
        name: s3-creds
        key: secretAccessKey
``` 
 
Now you can define applications (i.e. what should be saved together), here the "standard case", an app corresponds to a namespace. Simply adjust "name" and "namespace":
 
```yaml
apiVersion: astra.netapp.io/v1
kind: Application
metadata:
  name: wordpress
  namespace: astra-connector
spec:
  includedNamespaces:
    - namespace: wordpress
```
 
Now you can take a snapshot. To do this, adjust "applicationRef" to the app name of the YAML above and, if necessary, the appVaultRef (if you have named it differently). Then check the status with ``` kubectl get snapshot -n astra-connector ```

```yaml
apiVersion: astra.netapp.io/v1
kind: Snapshot
metadata:
  name: snap1
  namespace: astra-connector
spec:
  applicationRef: wordpress
  appVaultRef: my-appvault
``` 

Very similar for a backup:

```yaml
apiVersion: astra.netapp.io/v1
kind: Backup
metadata:
  name: bkp1
  namespace: astra-connector
spec:
  applicationRef: wordpress
  appVaultRef: my-appvault
```
 
And the best way to do this is of course to set it up as a schedule:

```yaml
apiVersion: astra.netapp.io/v1
kind: Schedule
metadata:
  name: sched
  namespace: astra-connector
spec:
  applicationRef: wordpress
  appVaultRef: my-appvault
  backupRetention: "2"
  snapshotRetention: "1"
  granularity: hourly
  minute: "10"
```
 
Backup without restore would be kind of stupid... To do this, determine the backup path and use: ```kubectl -n astra-connector get backup bkp1 -o=jsonpath='{.status.appArchivePath}'```

```yaml
apiVersion: astra.netapp.io/v1
kind: BackupInplaceRestore
metadata:
  name: bkprestore-to-wordpress
  namespace: astra-connector
spec:
  appVaultRef: my-appvault
  appArchivePath: <path>
```