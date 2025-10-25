### Pods

A Pod is the smallest deployable unit in Kubernetes. Think of it as a wrapper around one or more containers that share:
- Networking: Same IP, hostname, and network namespace.
- Storage: Can share volumes mounted into each container.
- Lifecycle: Containers in a Pod are managed together—if the Pod dies, they all go down together.

```
kubectl run redis --image=redis --dry-run=client -o yaml > redis-definition.yaml
kubectl create -f redis-definition.yaml
```

```
kubectl edit pod redis
kubectl apply -f redis-definition.yaml
```

```
kubectl run nginx --image nginx

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```

### ReplicaSet

A ReplicaSet is a modern alternative to the replication controller, using an updated API version and some improvements.
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
```

Scale directly from the command line:
```
kubectl scale --replicas=6 -f replicaset-definition.yml
```
```
kubectl scale rs new-replica-set --replicas=5
```

#### Labels and Selectors
 labels and selectors are the backbone of resource organization and dynamic grouping. They let you tag resources with meaningful metadata and then query or associate them based on that metadata.
 - Purpose: Identify, categorize, and group resources.
Selectors let you query or associate resources based on their labels. 

kubectl get pods -l 'environment in (production, qa)' \
kubectl get pods -l '!tier'

### Deployment

A Deployment defines the desired state of your application and lets Kubernetes maintain it automatically.
- Creates and manages ReplicaSets, which in turn manage Pods.
- Supports rolling updates, rollbacks, and scaling.
- Ideal for stateless workloads like web servers or APIs.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
```
```
kubectl create -f deployment-definition.yml
```

```
kubectl get deployments
kubectl get replicasets
kubectl get pods
kubectl get all
```
```
kubectl create deployment http-frontend --image=<your-httpd-image> --replicas=3
```

