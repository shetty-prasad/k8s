# argoCD

## Install
### Non-HA :
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.14.2/manifests/install.yaml
```

### HA :
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.14.1/manifests/ha/install.yaml
```

change service type to nodeport and add nodePort port number

Fetch the ArgoCD initial  Admin Password using 
```
kubectl -n argocd get secrets argocd-initial-admin-secret -o json | jq .data.password -r | tr -d '\n'  | base64 -d
```

# Install argocd cli
```
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.11/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```
