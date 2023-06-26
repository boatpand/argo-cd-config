# Install ArgoCD on Kube Cluster

## Install follow document : https://argo-cd.readthedocs.io
```
# Create namespace for ArgoCD
kubectl create namespace argocd
# Install ArgoCD in argocd namespace
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
# Check Pod
kubectl get po -n argocd
```

## Access ArgoCD UI
```
# Find port of argocd-server
kubectl get svc -n argocd
# forward port argocd-server for external access
kubectl port-forward -n argocd svc/argocd-server 8080:443
# Check Access -> https://127.0.0.1:8080
```
## Get password of usser admin ArgoCD
### open new terminal
```
# get secret of argocd initial admin
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
# copy value of data.password attribute and decode base64
echo <data.password-value> | base64 --decode
# copy output in terminal for usse as password of user admin
```
# Configure ArgoCD
```
# example of yaml -> vi application.yaml

# apiVersion will change if new version of ArgoCD released
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default

  source:
    repoURL: <git_repo.git>
    targetRevision: HEAD
    path: <git_repo_path>
  destination: 
    server: https://kubernetes.default.svc
    namespace: <application_namespace>

  syncPolicy:
    syncOptions:
    # Create namespace if don't find matching namespace
    - CreateNamespace=true

    automated:
    # By default, mannual changes in kube cluster will not trigger automated sync
    # Set SelfHeal to true for override manaul change with git repo
      selfHeal: true
    # By default, if we reaname or delete component kube cluster will not delete the old files
    # Set prune to true for auto delete the old file that not be used
      prune: true
      
    # default sync time is 3 minutes
```
### Apply ArgoCD configuration
```
kubectl apply -f application.yaml
```
