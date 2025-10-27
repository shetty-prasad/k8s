
🔍 Common Reasons for Pods in Pending State
Here are the most frequent causes, based on latest insights:
1. Insufficient Node Resources
- Cluster nodes may not have enough CPU, memory, or ephemeral storage.
- Especially common with large pods or tight resource requests.
2. Node Selector / Affinity Mismatch
- Pod may have nodeSelector, affinity, or antiAffinity rules that no node satisfies.
- Example: nodeSelector: disktype: ssd — but no node has that label.
3. Taints and Tolerations
- Nodes may be tainted (e.g., NoSchedule), and pod lacks matching toleration.
- Result: Pod is blocked from scheduling.
4. PersistentVolumeClaim Issues
- Pod requests a volume via PVC, but no matching PV is available or bound.
- Pod waits until volume is provisioned.
5. Pod Quotas or Limits
- Namespace may have resource quotas that block new pods.
- Example: pods: 10 quota reached.
6. Image Pull Failures
- If image registry is unreachable or image name is incorrect, pod may stay pending.
- Check kubectl describe pod for ImagePullBackOff or ErrImagePull.
7. Cluster Autoscaler Delay
- In autoscaling clusters, new nodes may take time to provision.
- Pod remains pending until node is ready.
8. Scheduler Constraints
- Custom schedulers or misconfigured default scheduler may delay pod assignment.

🧪 How to Diagnose Pending Pods
Run:
kubectl describe pod <pod-name>


Look for:
- Events section: shows scheduling failures, resource issues, taints, etc.
- Conditions: like PodScheduled=False with reason.

🧠 Pro Tip
To list all pending pods:
kubectl get pods --field-selector=status.phase=Pending

### Network Policies

Imagine a simple application with three main components:

A web server delivering the front end to users. \
An API server managing back-end requests. \
A database server storing application data.

The data flow in this setup is as follows:

A user sends an HTTP request to the web server on port 80. \
The web server forwards the request to the API server on port 5000. \
The API server queries the database server on port 3306 and sends the response back to the user.

Now you want to restrict the access to db server and only the api server should talk to the db server . In such cases, **network policies** let you ensure that only the API server can access the database server.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              name: api-pod
      ports:
        - protocol: TCP
          port: 3306
```

### Deployment Strategies

1. **Recreate Deployment Strategy**
What it is: Terminate all old instances, then create new ones.

2. **Rolling Update (Default Kubernetes Strategy)**
What it is: Gradually replace old instances with new ones.
```
# Rolling Update Strategy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-app
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1      # Max pods that can be unavailable
      maxSurge: 2           # Max pods above desired replica count
  selector:
    matchLabels:
      app: rolling-app
  template:
    metadata:
      labels:
        app: rolling-app
    spec:
      containers:
      - name: app
        image: nginx:1.21
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

**maxUnavailable**
Matlab: Deployment ke time kitne pods unavailable (down) ho sakte hain maximum.

**maxSurge**
Matlab: Desired replica count se kitne extra pods create kar sakte hain maximum.

Deployment Process:

```
# Update the image
kubectl set image deployment/rolling-app app=nginx:1.22

# Watch the rollout
kubectl rollout status deployment/rolling-app

# Check rollout history
kubectl rollout history deployment/rolling-app
```

Use Cases:

- Production applications requiring zero downtime
- Stateless applications
- When you have sufficient resources for overlapping instances

3. **Blue-Green Deployment**
What it is: Maintain two identical environments, switch traffic between them.

```
# Blue Environment (Current Production)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
  labels:
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:v1.0
        ports:
        - containerPort: 8080

---
# Green Environment (New Version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
  labels:
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: app
        image: myapp:v2.0
        ports:
        - containerPort: 8080

---
# Service (Switch between blue/green)
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Switch to 'green' for deployment
  ports:
  - port: 80
    targetPort: 8080
```

```
# Service (Switch between blue/green)
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Switch to 'green' for deployment
  ports:
  - port: 80
    targetPort: 8080
```

Use Cases:

- Critical production applications
- When you need instant rollback capability
- Applications requiring extensive testing before traffic switch
- When you have sufficient infrastructure resources

4. **Canary Deployment**
What it is: Gradually shift traffic from old to new version.
```
# Stable Version (90% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      version: stable
  template:
    metadata:
      labels:
        app: myapp
        version: stable
    spec:
      containers:
      - name: app
        image: myapp:v1.0
        ports:
        - containerPort: 8080

---
# Canary Version (10% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: canary
  template:
    metadata:
      labels:
        app: myapp
        version: canary
    spec:
      containers:
      - name: app
        image: myapp:v2.0
        ports:
        - containerPort: 8080

---
# Service (Load balances across both versions)
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp  # Matches both stable and canary
  ports:
  - port: 80
    targetPort: 8080
```

**Using Istio for Advanced Canary:**

```
# VirtualService for traffic splitting
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-vs
spec:
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: myapp-service
        subset: canary
  - route:
    - destination:
        host: myapp-service
        subset: stable
      weight: 90
    - destination:
        host: myapp-service
        subset: canary
      weight: 10

---
# DestinationRule
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp-dr
spec:
  host: myapp-service
  subsets:
  - name: stable
    labels:
      version: stable
  - name: canary
    labels:
      version: canary
```

Use Cases:

- Risk mitigation for new features
- A/B testing
- Performance testing with real traffic
- Gradual feature rollouts

5. A/B Testing Deployment
What it is: Run multiple versions simultaneously for comparison. You can ask the users, which version is better .
```
# Version A
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-version-a
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: a
  template:
    metadata:
      labels:
        app: myapp
        version: a
    spec:
      containers:
      - name: app
        image: myapp:feature-a
        env:
        - name: FEATURE_FLAG
          value: "version-a"

---
# Version B
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-version-b
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: b
  template:
    metadata:
      labels:
        app: myapp
        version: b
    spec:
      containers:
      - name: app
        image: myapp:feature-b
        env:
        - name: FEATURE_FLAG
          value: "version-b"
```

**Traffic Routing with Nginx Ingress:**

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ab-testing-ingress
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "X-Version"
    nginx.ingress.kubernetes.io/canary-by-header-value: "B"
    nginx.ingress.kubernetes.io/canary-weight: "50"
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-version-b-service
            port:
              number: 80
```

6. **Shadow/Dark Launch Deployment**
What it is: Route production traffic to new version without affecting users.

```
# Main Production Service
apiVersion: v1
kind: Service
metadata:
  name: production-service
spec:
  selector:
    app: myapp
    version: production
  ports:
  - port: 80
    targetPort: 8080

---
# Shadow Service (receives copy of traffic)
apiVersion: v1
kind: Service
metadata:
  name: shadow-service
spec:
  selector:
    app: myapp
    version: shadow
  ports:
  - port: 80
    targetPort: 8080
```

Istio Configuration for Traffic Mirroring:

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: shadow-vs
spec:
  http:
  - route:
    - destination:
        host: production-service
    mirror:
      host: shadow-service
    mirrorPercentage:
      value: 100.0  # Mirror 100% of traffic
```

7. **Feature Toggle Deployment**
What it is: Deploy new code with features disabled, enable via configuration.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: feature-toggle-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v2.0
        env:
        - name: FEATURE_NEW_UI
          valueFrom:
            configMapKeyRef:
              name: feature-flags
              key: new-ui-enabled
        - name: FEATURE_PAYMENT_V2
          valueFrom:
            configMapKeyRef:
              name: feature-flags
              key: payment-v2-enabled

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: feature-flags
data:
  new-ui-enabled: "false"
  payment-v2-enabled: "true"
  experimental-feature: "false"
```

**Dynamic Feature Toggle Update:**

```
# Enable new UI feature
kubectl patch configmap feature-flags -p '{"data":{"new-ui-enabled":"true"}}'

# Restart pods to pick up new config
kubectl rollout restart deployment/feature-toggle-app
```

<img width="1399" height="396" alt="image" src="https://github.com/user-attachments/assets/6758290f-7ac9-49e6-86c2-dc03a74493f1" />


Best Practices
1. Health Checks are Critical
```
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

2. Monitoring and Observability
```
# Add monitoring labels
metadata:
  labels:
    app: myapp
    version: v2.0
    deployment-strategy: canary
```

3. Automated Rollback
```
# Set up automatic rollback on failure
kubectl annotate deployment myapp \
  deployment.kubernetes.io/revision-history-limit=10

# Rollback if deployment fails
kubectl rollout undo deployment/myapp
```

4. Pre-deployment Testing
```
# Use init containers for pre-flight checks
spec:
  initContainers:
  - name: migration
    image: myapp:v2.0
    command: ['./run-migrations.sh']
  - name: health-check
    image: curlimages/curl
    command: ['curl', '-f', 'http://dependency-service/health']
```
