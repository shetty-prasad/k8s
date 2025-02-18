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
Using HELM:

```
helm repo add argo https://argoproj.github.io/argo-helm ### Add repository

helm install my-argo-cd argo/argo-cd --version 4.8.0  ### Install chart
```

To login into UI change service type to nodeport and add nodePort port number

Fetch the ArgoCD initial  Admin Password using 
```
kubectl -n argocd get secrets argocd-initial-admin-secret -o json | jq .data.password -r | tr -d '\n'  | base64 -d
```

# Install argocd cli
```
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.11/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```
create argocd app using cli 
```
argocd app create solar-system-app-2 \
--repo https://github.com/shetty-prasad/gitops-argocd.git \
--path ./solar-system \
--dest-namespace solar-system \
--dest-server https://kubernetes.default.svc
```

![image](https://github.com/user-attachments/assets/a0438bae-22d2-4a04-8508-20f9e8b3ac8f)

When u add an application, the project details are stored in secrets in the base64 decode format.

![image](https://github.com/user-attachments/assets/74f133ce-fe5c-40e7-9ddc-0f1d1c91cd12)

### argocd cli cmds :
```
argocd proj list
argocd app list
argocd app sync app1 app2 # sync multiple apps
argocd project get project-name  #get project details
argocd project get project-name -o yaml
```
