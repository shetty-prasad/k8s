
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


