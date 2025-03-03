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

argocd checks the repo every 3 minutes to make the desired changes, aka reconciliation loop - using timeout
need to change the config map value and restart the argocd-repo-server deployment

![image](https://github.com/user-attachments/assets/3704b5b8-426e-4fa4-b478-c9b9557a5787)

![image](https://github.com/user-attachments/assets/6a336c40-5778-4bf8-9a9c-bec87b0b59db)

sync strategies

![image](https://github.com/user-attachments/assets/7e8fbf83-f912-4161-abcf-1a8b1a505625)

Enable self heal and auto pruning
```
argocd app set health-check-app  --sync-policy=auto --self-heal --auto-prune
```

### Add another Cluster to the argocd repo

```
argocd repo add https://3000-port-tcybnbdg2jycxf74.labs.kodekloud.com/bob/gitops-argocd.git
```
```
argocd app create helm-random-shapes \
--repo https://3000-port-tcybnbdg2jycxf74.labs.kodekloud.com/bob/gitops-argocd.git \
--path ./helm-chart \
--helm-set color.circle=pink \
--helm-set color.square=red \
--helm-set service.type=NodePort \
--dest-namespace default \
--dest-server https://kubernetes.default.svc
```
```
argocd app sync helm-random-shapes
```

```
argocd app get helm-random-shapes
```
### first add cluster to the kubeconfig file

![image](https://github.com/user-attachments/assets/b7656894-c4ec-44e1-9950-32e717e2d9c5)

#### then add cluster to the argocd

```
argocd cluster list

argocd cluster add cluster2
```

Either create using the declarative approach or use the yaml file to create 

```
argocd app create health-check-app \
--repo https://3000-port-tcybnbdg2jycxf74.labs.kodekloud.com/bob/gitops-argocd.git \
--path ./health-check \
--helm-set color.circle=pink \
--helm-set color.square=red \
--helm-set service.type=NodePort \
--dest-namespace default \
--dest-server https://kubernetes.default.svc
```
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: health-check-app
spec:
  destination:
    name: ''
    namespace: health-check
    server: 'https://cluster2-controlplane:6443'
  source:
    path: ./health-check
    repoURL: >-
      https://3000-port-tcybnbdg2jycxf74.labs.kodekloud.com/bob/gitops-argocd.git
    targetRevision: HEAD
  project: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```

```
argocd app sync health-check-app
```
```
kubectl apply -f https://3000-port-tcybnbdg2jycxf74.labs.kodekloud.com/bob/gitops-argocd/raw/branch/master/declarative/multi-app/app-of-apps.yml -n argocd

kubectl get Application -n argocd
```






