---
title: "Automated install of NetApp Trident using kubectl"
date:  2020-09-10
draft: false
categories: ["NetApp","Docker","Kubernetes","Trident","DevOps"]
banner: /assets/images/content/autotrident.png
layout: post
---

Who wasn‘t yet faced the challenge to install NetApp Trident on multiple kubernetes clusters? NetApp provides very good documentation on [ReadTheDocs]( https://netapp-trident.readthedocs.io/en/stable-v20.07/ "Netapp Trident Documentation"). But you must execute the instruction step by step. 

In addition to this documentation, I would like to introduce a way to install trident automatically in any k8s cluster via *kubectl*. 

The following yaml file can be used for this:
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: k8s-trident-installer
  labels:
    app: trident-installer
spec:
  containers:
    - name: installer
      image: fabianborn/k8s-trident-installer:latest
      ports:
        - containerPort: 19811
      volumeMounts:
      - name: kconfig
        mountPath: "/config"
        subPath: kubeconfig
      env:
        - name: DEBUG
          value: "0"
  volumes:
  - name: kconfig
    configMap:
      name: trident-install-kubeconfig
```

In preparation, the yaml file ``` subPath: kubeconfig``` must be adapted to the original kubeconfig name.

Example:
```bash
ls -la ~/.kube  
total 48
drwxr-xr-x    8 fabian  staff   256  9 Sep 20:41 .
drwxr-xr-x+  74 fabian  staff  2368  9 Sep 22:16 ..
-rw-------    1 fabian  staff  5453  9 Sep 20:41 epicshit-io-kubeconfig
```
also:
```yaml
subPath: epicshit-io-kubeconfig
```

Now the ConfigMap can be created and the yaml file can be applied to the kubernetes cluster.

```bash
kubectl create configmap trident-install-kubeconfig --from-file=$KUBECONFIG

kubectl apply -f k8s-trident-installer.yaml
```

During the installation you tail the log of the container with 
```bash 
kubectl logs --follow k8s-trident-installer installer 
```

The successfull installation will looks like: 
```
[2020-09-09 19:53:04] Moving tridentctl to /usr/local/bin
'tridentctl' -> '/usr/local/bin/tridentctl'
[2020-09-09 19:53:05] Installation finished!
[2020-09-09 19:53:05] Clean up all folder....
```

#### Feedback
Try this method and give me a feedback. If you encounter any problems, please open a [github issue]( https://github.com/fabian-born/k8s-trident-installer/issues  "open a issue").
