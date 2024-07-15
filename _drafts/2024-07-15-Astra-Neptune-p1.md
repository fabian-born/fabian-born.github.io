---
title: "AWS announced Astra Trident as EKS Add-on"
date:  2024-02-24
draft: false
categories: announcements howto
tags: ["NetApp","EKS","Kubernetes","Trident","DevOps","AWS"]
banner: /assets/images/content/eks.png
layout: post
toc: false
---

View weeks ago AWS and NetApp announced that Astra Trident is now available as an EKS add-on.

A nice step to bring EKS and FSxN together. I will show you how you can enable and use this add-on.

The add-on must first be subscribed to before it can be used. Once everything has been completed, the Astra Trident add-on can be activated. In this short video, you can see all the necessary steps.


Now that Trident has been installed, you can continue with the backend configuration. I will share a small example – but for the complete settings take a look into the [![official documentation](https://docs.netapp.com/us-en/trident/trident-use/trident-fsx.html):

[![EKS Deployment](https://img.youtube.com/vi/cFFybX83iFE/0.jpg)](https://youtu.be/) 

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: backend-tbc-ontap-secret
type: Opaque
stringData:
  username: vsadmin
  password: netapp123
---
apiVersion: trident.netapp.io/v1
kind: TridentBackendConfig
metadata:
  name: backend-tbc-ontap-nas
spec:
  version: 1
  storageDriverName: ontap-nas
  managementLIF: your-management-ip
  backendName: tbc-ontap-nas
  exportPolicy: default
  storagePrefix: prefix
  autoExportPolicy: true
  autoExportCIDRs: ['0.0.0.0/0']
  credentials:
    name: backend-tbc-ontap-secret
```

Enjoy testing 🙂