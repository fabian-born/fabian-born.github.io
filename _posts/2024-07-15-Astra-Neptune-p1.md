---
title: "Astra Connector"
date:  2024-02-24
draft: false
categories: howto
tags: ["NetApp","Astra","Kubernetes","Trident","DevOps","Backup"]
banner: /assets/images/content/neptun.png
layout: post
toc: false
---
### Requirements
#### Update Trident, activate ACP, install Astra Connector
 
The prerequisite is a current Trident (version 24.02). Depending on which version is currently on the cluster, this may need to be updated (e.g. if deployed via Helm: helm upgrade trident -n trident netapp-trident/trident-operator).
 
Then the ACP (Astra Control Provisioner) mode of Trident is activated, this enables the integration between Astra Control and Trident. Checkout the [ACP documentation](https://docs.netapp.com/us-en/astra-control-center/get-started/enable-acp.html) to install ACP! 

Astra Connector must be installed in the Kubernetes cluster. I will describe how exactly in a new post. Until then, have a look at the [official documentation](https://github.com/netapp/astra-connector-operator) .

### Applications 
#### Snapshots & Backup 
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
    endpoint: s3.company.org
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
#### Restore Application
Backup without restore would be kind of stupid... To do this, determine the backup path and use: 
```bash 
kubectl -n astra-connector get backup bkp1 -o=jsonpath='{.status.appArchivePath}'
```

Now you can restore your application!

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
