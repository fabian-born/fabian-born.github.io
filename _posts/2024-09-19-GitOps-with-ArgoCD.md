---
title: "Orchestrating Excellence"
date:  2024-10-01
draft: false
categories: blog
tags: ["NetApp","Kubernetes","Trident","DevOps","Backup"]
banner: /assets/images/content/orchexcell.jpg
layout: post
toc: false
---

To create an ArgoCD project, you need to define a project in ArgoCD that manages access and deployment for your GitOps repositories and clusters. Here's a step-by-step guide to building an ArgoCD project using YAML:

### Prerequisites:
- **ArgoCD installed and configured** on your Kubernetes cluster.
- **kubectl** configured to communicate with the cluster.

### Step 1: Define the ArgoCD Project YAML

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project  # The name of the ArgoCD project
  namespace: argocd # The namespace where ArgoCD is installed (usually 'argocd')
spec:
  description: "My project for managing apps with ArgoCD"
  # Define which source repositories are allowed
  sourceRepos:
    - https://github.com/myorg/myrepo.git
  # Define the destination clusters where ArgoCD is allowed to deploy
  destinations:
    - namespace: default
      server: https://kubernetes.default.svc
  # Optional: Define cluster resource restrictions
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
  # Optional: Define namespace resource restrictions
  namespaceResourceWhitelist:
    - group: '*'
      kind: '*'
  # Optional: Configure roles and permissions
  roles:
    - name: developer
      description: "Developer role with limited permissions"
      policies:
        - p, proj:my-project:developer, applications, get, *, allow
        - p, proj:my-project:developer, applications, create, *, allow
      groups:
        - developers
  # Optional: Define sync windows for time-based deployments
  syncWindows:
    - kind: allow
      schedule: "* * 9-17 * * 1-5"
      duration: 8h
      applications:
        - '*'
```

### Explanation:

1. **apiVersion & kind**: Defines the kind of resource. Here it’s `AppProject`.
2. **metadata**: 
   - `name`: Name of the project, e.g., `my-project`.
   - `namespace`: Namespace where ArgoCD is installed (`argocd` by default).
3. **spec**:
   - **description**: A short description of the project.
   - **sourceRepos**: The Git repository URL(s) allowed for this project.
   - **destinations**: Defines where ArgoCD is allowed to deploy. You can restrict this by specifying a namespace and cluster.
   - **clusterResourceWhitelist**: Specifies which cluster-level resources are allowed (e.g., ConfigMaps, Custom Resource Definitions).
   - **namespaceResourceWhitelist**: Specifies which namespace-level resources are allowed (e.g., Deployments, Pods).
   - **roles**: Custom roles can be defined for specific actions like `create`, `get`, etc. These roles can be assigned to groups.
   - **syncWindows**: Optional section to limit deployments to certain time windows (e.g., during business hours).

### Step 2: Apply the Project YAML

Once your YAML is ready, you can apply it to your Kubernetes cluster using `kubectl`.

```bash
kubectl apply -f argocd-project.yaml
```

### Step 3: Verify the Project in ArgoCD

You can verify that the project was successfully created by using either:

- **ArgoCD UI**: Go to the ArgoCD web UI and look for the `Projects` section.
- **kubectl**:

```bash
kubectl get appprojects -n argocd
```

### Step 4: Create an ArgoCD Application in this Project

Now that your project is set up, you can create an application that deploys resources from your Git repository to your Kubernetes cluster.

Example ArgoCD Application YAML:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: my-project # Reference to the project created above
  source:
    repoURL: https://github.com/myorg/myrepo.git
    path: manifests
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply the application using:

```bash
kubectl apply -f argocd-application.yaml
```

This will deploy the resources defined in the Git repo under the `my-project` project.

### Conclusion

By following these steps, you've successfully built and configured an ArgoCD project. You can now use this project to manage multiple applications in a GitOps style, with fine-grained control over which repositories and clusters can be accessed.