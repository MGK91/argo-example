
# 🛠️ ArgoCD with Helm Chart Example (Full Walkthrough)

This example demonstrates how to deploy a Helm chart-managed app using ArgoCD in a GitOps workflow.

---

## 🎯 Use Case

You have a web application (`myapp`) managed using a **Helm chart**. You want ArgoCD to:

- Watch a Git repo containing the Helm chart
- Automatically sync and deploy the app to a Kubernetes cluster
- Allow dynamic values management via `values.yaml`

---

## 📁 Git Repo Structure for Helm-based App

```
myapp-gitops/
└── helm/
    └── myapp/
        ├── Chart.yaml
        ├── values.yaml
        └── templates/
            ├── deployment.yaml
            └── service.yaml
```

---

## 📘 1. `Chart.yaml`

```yaml
apiVersion: v2
name: myapp
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0"
```

---

## 📘 2. `values.yaml`

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

config:
  ENV: production
```

---

## 📘 3. `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "myapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "myapp.name" . }}
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 80
        env:
        - name: ENV
          value: {{ .Values.config.ENV | quote }}
```

---

## 📘 4. `templates/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ include "myapp.name" . }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 80
```

---

## 📄 ArgoCD Application YAML (Helm-based)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-helm
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-gitops.git
    targetRevision: HEAD
    path: helm/myapp
    helm:
      valueFiles:
        - values.yaml
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

## ✅ How This Works

- ArgoCD pulls the Helm chart from the Git repo
- Renders templates using `values.yaml`
- Applies the generated Kubernetes manifests
- Watches for Git changes and automatically syncs

---

## 🧠 Benefits of Using Helm with ArgoCD

| Feature           | Benefit                                         |
|-------------------|--------------------------------------------------|
| Templates         | Dynamic and reusable configurations             |
| Value overrides   | Easily switch env configs (dev/stage/prod)      |
| GitOps control    | Full audit history of changes                   |
| ArgoCD dashboard  | See rendered manifests, diffs, rollback support |

---

## 📌 Summary

- ArgoCD + Helm = GitOps + templated, dynamic deployments
- Use this structure for teams managing multiple environments (e.g., values-dev.yaml, values-prod.yaml)
- Easily integrate with App-of-Apps pattern

