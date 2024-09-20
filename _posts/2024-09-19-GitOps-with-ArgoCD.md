---
title: "Orchestrating Excellence - ArgoCD"
date:  2024-09-20
draft: false
categories: blog
tags: ["Kubernetes","Trident","DevOps","ArgoCD"]
banner: /assets/images/content/orchexcell.jpg
layout: post
toc: false
author: "Fabian Born"
---
## ArgoCD as Plattform
To create an ArgoCD project, you need to define a project in ArgoCD that manages access and deployment for your GitOps repositories and clusters. Here's a step-by-step guide to building an ArgoCD project using YAML:

### Prerequisites:
- **ArgoCD installed and configured** on your Kubernetes cluster.
- **kubectl** configured to communicate with the cluster.

### Step 1: Define the ArgoCD Project YAML

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: argocddemo # The name of the ArgoCD project
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
   - `name`: Name of the project, e.g., `argocddemo`.
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
  project: argocddemo   # Reference to the project created above
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

This will deploy the resources defined in the Git repo under the `argocddemo` project.

## Git Repo for ArgoCD applications
To implement a Git repository structure for an ArgoCD project, it’s important to organize the repository in a way that promotes scalability, modularity, and maintainability. This structure allows you to efficiently manage multiple environments (e.g., dev, staging, prod) and applications, along with reusable Kubernetes manifests (Helm charts, Kustomize, etc.).

Here’s an example for a Git repository structure for an ArgoCD project:

**High-Level Directory Structure**

```bash
├── apps/                # Contains the ArgoCD applications
│   ├── dev/             # Manifests for the 'dev' environment
│   ├── staging/         # Manifests for the 'staging' environment
│   └── prod/            # Manifests for the 'prod' environment
├── base/                # Common base manifests, reusable for all environments
│   ├── app1/
│   └── app2/
├── environments/        # Environment-specific overlays (Kustomize or Helm)
│   ├── dev/
│   ├── staging/
│   └── prod/
├── helm-charts/         # Optional: Custom Helm charts (if using Helm)
│   ├── app1-chart/
│   └── app2-chart/
└── argocd-apps/         # ArgoCD application definitions
    ├── dev-app.yaml
    ├── staging-app.yaml
    └── prod-app.yaml
```


**More on this topic in the next blog post :-)**