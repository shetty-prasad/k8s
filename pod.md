### Pods

A Pod is the smallest deployable unit in Kubernetes. Think of it as a wrapper around one or more containers that share:
- Networking: Same IP, hostname, and network namespace.
- Storage: Can share volumes mounted into each container.
- Lifecycle: Containers in a Pod are managed together—if the Pod dies, they all go down together.

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

#### Labels and Selectors



