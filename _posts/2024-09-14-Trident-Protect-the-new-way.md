---
title: "Trident Protect"
date:  2024-09-19
draft: true
categories: howto
tags: ["NetApp","Astra","Kubernetes","Trident","DevOps","Backup"]
banner: /assets/images/content/neptun1.png
layout: post
toc: false
author: "Fabian Born"
---

### Prepare your environment

#### Install / Update Trident
Trident Protect requires Trident 24.06.1 or higher. So the first stepp is to install/update Trident
- install/update Trident


#### Download and install protectctl

```bash
docker login cr.astra.netapp.io -u ACCOUNT_ID –p API_TOKEN
docker pull cr.astra.netapp.io/trident-protectctl:24.10.0-preview.1
docker run --rm -v $(pwd):/pctl-tmp cr.astra.netapp.io/trident-protectctl:24.10.0-preview.1 cp protectctl /pctl-tmp

mv protectctl /usr/local/bin
```

You can also run the container with no commands to get these instructions:
```bash
docker run --rm cr.astra.netapp.io/trident-protectctl:24.10.0-preview.1
```

The default platform is linux/amd64, but two more are available:
- cr.astra.netapp.io/trident-protectctl:24.10.0-preview.1-linuxarm64
- cr.astra.netapp.io/trident-protectctl:24.10.0-preview.1-darwinarm64

### important containers:
docker login cr.astra.netapp.io -u d13e74bf-3ced-488d-b31a-8ed307395e93 -p hwTgNdnLXQEa8mD5h_YxOSuhBNakxQfA-npBgHU6Dsg=   
cr.astra.netapp.io/trident-protectctl:24.10.0-preview.1

#### Create backup target
+ protectctl create appvault ontap-s3 ontap-s3-vault --bucket astrabackup --endpoint ontap.s3.digital-twin.labs --secret s3-creds --dry-run

or

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: s3-creds
  namespace: trident-protect
type: Opaque
stringData:
  accessKeyID: K24FD352IIADNEGI467X
  secretAccessKey: dQyabV4daie_ZQiD664hPC82_3ExHj2Bgy2SAcjB
---
apiVersion: protect.trident.netapp.io/v1
kind: AppVault
metadata:
  name: ontap-s3-appvault
  namespace: trident-protect
spec:
  providerType: ontap-s3
  providerConfig:
    s3:
      endpoint: ontap.s3.digital-twin.labs
      bucketName: astrabackup
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




##### 
+ restore backup to new namespace
protectctl create backuprestore restore1 demoapp-wordpress:demoapp-wordpress-restore --appvault ontap-s3-appvault --backup ondemand-bkp1

+ restore backup to existing namespace
