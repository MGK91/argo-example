
# 🎯 Kubernetes Ingress + ArgoCD Setup Using Helm

This guide demonstrates how to:

1. Install NGINX Ingress Controller using Helm
2. Install ArgoCD using Helm
3. Expose ArgoCD via an Ingress resource
4. Validate everything end-to-end

---

## 🛠️ 1. Install NGINX Ingress Controller with Helm

### ➤ Add Helm repo

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### ➤ Install Ingress Controller

```bash
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace
```

### ➤ Verify Installation

```bash
kubectl get all -n ingress-nginx
```

---

## 🚀 2. Install ArgoCD with Helm

### ➤ Add Argo Helm repo

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

### ➤ Install ArgoCD

```bash
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace
```

### ➤ Verify Resources

```bash
kubectl get all -n argocd
```

---

## 🌐 3. Expose ArgoCD with Ingress Resource

### ➤ Create Ingress YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
  - host: argocd.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
```

### ➤ Apply It

```bash
kubectl apply -f argocd-ingress.yaml
```

### ➤ Update `/etc/hosts`

Add:

```
<INGRESS_CONTROLLER_EXTERNAL_IP> argocd.local
```

Find external IP:

```bash
kubectl get svc -n ingress-nginx
```

---

## 🔐 4. Access ArgoCD UI

### ➤ Get Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

### ➤ Open in Browser

Navigate to:

```
http://argocd.local
```

---

## ✅ Summary

| Component | Installed With | Accessed Via |
|-----------|----------------|--------------|
| NGINX Ingress Controller | Helm | External IP |
| ArgoCD | Helm | Ingress + Domain |
