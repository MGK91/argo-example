
# 📦 ArgoCD Application Example with Directory Structure and Use Case

This document provides:

- Real-world use case for ArgoCD
- ArgoCD Application YAML
- Project directory structure for GitOps deployment
- Usage with Helm charts and Kubernetes manifests

---

## 🎯 Use Case: GitOps-Driven Deployment of a Sample App

Imagine you have a web application (e.g., `myapp`) that consists of:

- A Kubernetes Deployment
- A Service (ClusterIP)
- A ConfigMap

You want ArgoCD to:

- Watch a Git repository
- Automatically apply the Kubernetes resources when changes are pushed
- Optionally use a Helm chart for templated deployments

---

## 📁 Directory Structure for Git Repository

```
myapp-gitops/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── helm/
│   └── myapp/              # If using Helm chart
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           └── service.yaml
└── app-of-apps/
    └── myapp-argocd-app.yaml
```

---

## 📄 ArgoCD Application YAML (app-of-apps style)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-gitops.git
    targetRevision: HEAD
    path: base
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

---

## 🧾 Kubernetes Deployment (base/deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

## 🧾 Service (base/service.yaml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

---

## 🧾 ConfigMap (base/configmap.yaml)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  ENV: production
```

---

## ✅ How It Works

1. ArgoCD syncs the app from the Git repo.
2. It creates all the resources in the `dev` namespace.
3. If `automated sync` is enabled, any Git change is applied immediately.
4. ArgoCD dashboard shows app health, sync status, history, and logs.

---

## 🧠 Pro Tip

You can convert this into a Helm-based app by changing the `spec.source` like so:

```yaml
source:
  repoURL: https://github.com/myorg/myapp-gitops.git
  targetRevision: HEAD
  chart: myapp
  helm:
    valueFiles:
      - values.yaml
```

---

## 📌 Summary

- ✅ GitOps simplifies environment consistency
- ✅ ArgoCD supports raw manifests, Helm, and Kustomize
- ✅ Automates sync and rollback for Kubernetes deployments
