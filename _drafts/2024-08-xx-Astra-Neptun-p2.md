---
title: "Astra Neptune"
date:  2024-09-10
draft: false
categories: howto
tags: ["NetApp","Astra","Kubernetes","Trident","DevOps","Backup"]
banner: /assets/images/content/neptun2.png
layout: post
toc: false
---



### Astra Connector
#### Install Astra Connector Operator
This is the component that is now running locally in the respective cluster (push vs. pull mode). This can initially also run independently of the central Astra Control instance.

 
Create CRDs and Namespaces: ```kubectl apply -f https://github.com/NetApp/astra-connector-operator/releases/download/202407111448-main/astraconnector_operator.yaml```
(creates two new namespaces, for the operator and the connector itself)

```bash
kubectl get pods -n astra-connector-operator
NAME                                           READY   STATUS              RESTARTS   AGE
operator-controller-manager-df79977cc-zddxv    2/2     Running       0          27s
```
 

### Install Astra Connector
Create a registration secret (adapt it to your internal registration if necessary):

```kubectl create secret docker-registry regcred --docker-username= 86406506-9136-46b7-a9c3-4a7e26f17974 --docker-password= CcDtaZ-xyPjvjBMtJBdBxOWpNVG6DQjU4daIb-09RWg=  -n astra-connector --docker-server=registry````

 
Adapt the following YAML and then deploy. In principle, only the clusterName (later identifies the local cluster in the central management). If you already have it, you can also add the URL that the central instance will later have (via Ingress or Loadbalancer). 
 
```yaml
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
    cloudBridgeURL: registry
  imageRegistry:
    name: registry
    secret: regcred
```

Result should looks like:
```bash
kubectl get pods -n astra-connector -w
NAME                                        READY   STATUS    RESTARTS   AGE
nats-0                                      1/1     Running   0          71s
neptune-controller-manager-5594fbc9-qwqjx   2/2     Running   0          104s
```