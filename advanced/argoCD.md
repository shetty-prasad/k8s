# argoCD

## Install
### Non-HA :
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
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
* **Change type: ClusterIP to type: NodePort and under - name: https add nodePort: 32766**

```
kubectl edit svc argocd-server -n argocd
```
Fetch the ArgoCD initial  Admin Password using 

```
kubectl -n argocd get secrets argocd-initial-admin-secret -o json | jq .data.password -r | tr -d '\n'  | base64 -d
```

# Install argocd cli
```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```

<img width="1380" height="143" alt="image" src="https://github.com/user-attachments/assets/34303912-bde8-4e0e-aef9-0af0e0a8da6a" />


create argocd app using cli 
```
argocd app create solar-system-app-2 \
--repo https://github.com/shetty-prasad/gitops-argocd.git \
--path ./solar-system \
--dest-namespace solar-system \
--dest-server https://kubernetes.default.svc
```
```
argocd app sync solar-system-app-2
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

#### How often ArgoCD checks repository

argocd checks the repo every 3 minutes to make the desired changes, aka reconciliation loop - using timeout
need to change the config map value and restart the argocd-repo-server deployment

![image](https://github.com/user-attachments/assets/3704b5b8-426e-4fa4-b478-c9b9557a5787)

![image](https://github.com/user-attachments/assets/6a336c40-5778-4bf8-9a9c-bec87b0b59db)

```
# Check the current reconciliation timeout setting
kubectl -n argocd describe pod argocd-repo-server | grep -i "ARGOCD_RECONCILIATION_TIMEOUT:" -B1

# Patch the ArgoCD config map with a custom timeout of 300 seconds
kubectl -n argocd patch configmap argocd-cm --patch='{"data":{"timeout.reconciliation":"300s"}}'

# Restart the ArgoCD repo server deployment to apply the new setting
kubectl -n argocd rollout restart deploy argocd-repo-server

# Verify that the new timeout is active
kubectl -n argocd describe pod argocd-repo-server | grep -i "ARGOCD_RECONCILIATION_TIMEOUT:" -B1
```

##### Webhook events

* Without any modifications, ArgoCD typically checks for manifest changes every 3 to 5 minutes.
* However, integrating a webhook—such as one from GitHub targeting [ARGOCD_SERVER_URL]/api/webhook—can trigger instant notifications, reducing synchronization delays.

<img width="1480" height="675" alt="image" src="https://github.com/user-attachments/assets/1492f425-900e-40e1-b045-5a34ee05e469" />

**ArgoCD provides several built-in health checks, each indicating a different state of resource health:**

* Healthy: All associated resources are 100% healthy.
* Progressing: The resource is not fully healthy but may recover over time.
* Degraded: The resource has encountered a failure or cannot reach a healthy state promptly.
* Missing: The resource does not exist in the cluster.
* Suspended: The resource is paused or in a suspended state (for example, a paused deployment).
* Unknown: The health assessment has failed, resulting in an indeterminate status.

sync strategies

![image](https://github.com/user-attachments/assets/7e8fbf83-f912-4161-abcf-1a8b1a505625)

Enable self heal and auto pruning
```
argocd app set health-check-app  --sync-policy=auto --self-heal --auto-prune
```

**Write a custom health check app**

vi patch.yaml
```
  data:
    resource.customizations.health.ConfigMap: |
      hs = {}
      hs.status = "Healthy"
       if obj.data.TRIANGLE_COLOR == "white" then
          hs.status = "Degraded"
          hs.message = "Use any color other than White "
       end
      return hs
```
```
kubectl patch configmap argocd-cm -n argocd --patch-file patch.yaml
```

### ArgoCD declarative approach i.e argo manifest file

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: geocentric-model-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/sidd-harth/test-cd.git
    targetRevision: HEAD
    path: ./declarative/manifests/geocentric-model
  destination:
    server: https://kubernetes.default.svc
    namespace: geocentric-model
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
    automated:
      selfHeal: true
```

* Project Name: The project within ArgoCD.
* Source Configuration: Includes the Git repository URL, the revision (e.g., HEAD), and the path pointing to the desired Kubernetes manifests (i.e., the "geocentric-model" directory inside "declarative/manifests"). This directory contains two YAML files: one for the Deployment and one for the Service.
* Destination Configuration: Specifies the target cluster (using the in-cluster URL) and the namespace where resources will be deployed.
* Sync Policy: An optional policy that can automate the synchronization and self-healing process.

### ArgoCD app of apps i.e one app calling multiple apps 

<img width="1472" height="819" alt="image" src="https://github.com/user-attachments/assets/9c4656b1-d4bf-415a-b7f2-60160873d554" />

```
kubectl apply -f https://3000-port-tcybnbdg2jycxf74.labs.kodekloud.com/bob/gitops-argocd/raw/branch/master/declarative/multi-app/app-of-apps.yml -n argocd

kubectl get Application -n argocd
```

### Deploy apps using helm 

* Once the app is deployed by argocd using helm, it is completely managed by argocd. Even helm ls command will return nothing.
* Also the values in the deployment.yml file can be overridden using the "--helm-set" command .
  
<img width="1286" height="666" alt="image" src="https://github.com/user-attachments/assets/a35cf330-c71d-412a-8b2d-048dccbdb5eb" />


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



### multi cluster deployment

<img width="1490" height="734" alt="image" src="https://github.com/user-attachments/assets/6a184667-9991-486b-b1b9-0b4e0f93c338" />


The details about the cluster list is stored in the secrets

<img width="1473" height="457" alt="image" src="https://github.com/user-attachments/assets/3817af46-614b-4271-b986-6e2da502faba" />









