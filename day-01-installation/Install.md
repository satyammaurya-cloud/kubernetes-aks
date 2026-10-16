# Setup Kubernetes on Azure AKS using CLI

## 1. Prerequisites
- Azure subscription and sufficient permissions.
- Azure CLI (`az`) and `kubectl`.
- A local terminal, Azure VM, or Azure Cloud Shell.

## 2. Install Azure CLI
Follow: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli

Verify:
```bash
az version
```

## 3. Install kubectl
```bash
az aks install-cli
kubectl version --client
```

## 4. Login to Azure
```bash
az login
az account show --output table
```

Select a subscription if needed:
```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

For an Azure VM with a configured Managed Identity:
```bash
az login --identity
```
The identity must have sufficient permissions to create AKS resources.

## 5. Create Resource Group and AKS Cluster
Run both commands in one go using `&&`:
```bash
az group create \
  --name rg-aks-lab \
  --location centralindia \
&& az aks create \
  --resource-group rg-aks-lab \
  --name aks-lab-cluster \
  --location centralindia \
  --node-count 1 \
  --node-vm-size Standard_D2s_v5
```
- `rg-aks-lab`: Resource Group.
- `aks-lab-cluster`: AKS cluster name.
- `centralindia`: Azure region.
- `--node-count 1`: One worker node in the default system node pool.
- `Standard_D2s_v5`: Example VM size; verify regional availability and quota.

AKS creation takes several minutes and incurs charges. Azure manages the Kubernetes control plane.

## 6. Connect to AKS
```bash
az aks get-credentials \
  --resource-group rg-aks-lab \
  --name aks-lab-cluster \
  --overwrite-existing
```

Check the context and worker node:
```bash
kubectl config current-context
kubectl get nodes -o wide
kubectl get pods -A
kubectl cluster-info
```
Expected: one worker node with `Ready` status.

## 7. Deploy a Test Application
```bash
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx \
  --port=80 \
  --type=LoadBalancer

kubectl get deployments
kubectl get pods
kubectl get svc
```
Wait for the service's external IP, then open `http://EXTERNAL-IP` in your browser. A LoadBalancer service may create additional billable resources.

## 8. Troubleshoot or Access a Worker Node
List nodes:
```bash
kubectl get nodes -o wide
```
Try a node debug session:
```bash
kubectl debug node/<node-name> -it --image=mcr.microsoft.com/azurelinux/busybox:1.36
```
This requires API server connectivity, suitable Kubernetes permissions, and a compatible debug image. It is not direct SSH access.

If a DNS error mentions an old cluster endpoint, refresh credentials for the correct cluster and check `kubectl config current-context`.

## 9. Delete the AKS Lab
WARNING: This deletes all resources inside `rg-aks-lab`.
```bash
az group delete --name rg-aks-lab --yes --no-wait
az group exists --name rg-aks-lab
```
The final command returns `false` after deletion completes.

## 10. Docker Installation on Amazon Linux 2023
This is a separate local Kubernetes lab, not part of AKS.
```bash
sudo dnf install docker -y
sudo systemctl enable --now docker
sudo usermod -aG docker ec2-user
newgrp docker
docker version
```

## 11. Install Minikube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

Start a local Kubernetes cluster using Docker:
```bash
minikube start --driver=docker
minikube status
kubectl get nodes
```
Use a Linux VM with adequate CPU and RAM; 2 CPUs and 2 GB RAM is a bare minimum.

## 12. AWS EKS vs Azure AKS
| AWS EKS | Azure AKS |
|---|---|
| `aws` CLI | `az` CLI |
| `eksctl create cluster` | `az aks create` |
| EC2 worker nodes | Azure VM Scale Set worker nodes |
| IAM Role / STS | Managed Identity / Microsoft Entra ID |
| `aws eks update-kubeconfig` | `az aks get-credentials` |
| `kubectl get nodes` | `kubectl get nodes` |

## Quick Revision
- Create AKS: `az aks create`
- Connect to AKS: `az aks get-credentials`
- Check nodes: `kubectl get nodes`
- Local cluster: `minikube start`
- Delete lab: `az group delete --name rg-aks-lab --yes --no-wait`

Official guides:
- Azure CLI: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli
- AKS: https://learn.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster
- EKS: https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html
