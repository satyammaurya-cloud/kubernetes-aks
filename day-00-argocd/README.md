## 📘 What is ArgoCD 

Argo CD is a Kubernetes-native Continuous Deployment (CD) tool that automatically deploys applications from a Git repository to a Kubernetes cluster using a pull-based approach.

👉 Short: **ArgoCD = Git-based automatic deployment tool for Kubernetes (GitOps)**

## 📘 What is GitOps ?

GitOps is a deployment methodology where Git acts as the single source of truth, and all infrastructure and application configurations are stored in Git and automatically applied to the system.

## 📘 GitOps using Argo CD 
GitOps using Argo CD is a process where Argo CD continuously monitors a Git repository, compares the desired state with the current state in Kubernetes, and automatically synchronizes the cluster to match the Git configuration.

## 📘 Benefits of Argo CD 
- **Improved Productivity:** Reduces manual deployment work
- **Faster Deployment:** Enables quick and automated releases
- **Better Collaboration:** Teams work from a single Git source
- **High Reliability:** Ensures system always matches Git state

---

##  ArgoCD Installation + Setup (Simple Steps) 🚀

### ✅ Prerequisite
- Kubernetes cluster running (Minikube / EKS / etc.)
- kubectl configured

#### 📌 Step 1: Create Namespace
```bash
kubectl create namespace argocd
```

#### 📌 Step 2: Install ArgoCD
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

#### 📌 Step 3: Verify Installation
```bash
kubectl get all -n argocd
```
> **Make sure all pods are Running** (example: argocd-server, repo-server, redis, etc.)

#### 📌 Step 4: Expose ArgoCD UI (NodePort or LoadBalancer)
> By editing the service:
```bash
kubectl edit svc argocd-server -n argocd
```
```bash
type: ClusterIP  
```
> Change service type form ClusterIP to NodePort/LoadBalancer:
```yaml
type: LoadBalancer
```


#### 📌 Step 5: Get Admin Password
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```
> Copy password (base64 encoded) and Decode the Password using below command
```bash
echo "copied_password" | base64 --decode
```

#### 📌 Step 7: Login to UI 
- Username: `admin` 
- Password: decoded password 
- URL: `NodeIP:NodePort` or `LoadBalancer URL`

#### ---------------------🎯 **ArgoCD Installation Done** ✅-------------------------

---
## 🚀 Application Deployment via ArgoCD 

### 📌 Step 8: Create App Namespace (optional)
```bash
kubectl create namespace myapp
```

### 📌 Step 9: Deployment YAML 
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: swiggy-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: swiggy-app
  template:
    metadata:
      labels:
        app: swiggy-app
    spec:
      containers:
      - name: swiggy-app
        image: nginx:latest
        ports:
        - containerPort: 3000
```       
### 📌 Step 10: Service YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: swiggy-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: swiggy-app
```    
### 📌 Step 11: ArgoCD Application YAML
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/CloudTechDevOps/Kubernetes.git
    targetRevision: HEAD
    path: day-14-argocd

  destination:
    server: https://kubernetes.default.svc
    namespace: myapp

  syncPolicy:
    syncOptions:
    - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true
```      
### 📌 Step 12: Apply Application
```bash
kubectl apply -f application.yaml
```
### 📌 Step 13: Verify in UI
```
Open ArgoCD UI
Check your app status
It will auto deploy
```
🔁 Auto Sync Behavior (Important)
- ArgoCD checks Git every 3 minutes
- Changes in Git → auto applied to cluster
#### Features:
- selfHeal: true → fixes manual changes
- prune: true → deletes removed resources
### 📌 Step 14: Test Change

👉 Example:

- Change replicas from 3 → 4 in Git
- Push code
- kubectl get pods -n myapp

✔ Pods increase automatically
