Kube Proxy is a lightweight process that runs on every node in the Kubernetes cluster. 
Its key function is to monitor for Service creations and configure network rules that redirect traffic to the corresponding pods. 
One common method it uses is by setting up IP tables rules.

For example, if a Service is assigned the IP 10.96.0.12, Kube Proxy configures the IP tables on each node so that any traffic directed to that IP is forwarded to the actual pod IP, such as 10.32.0.15. 
This redirection mechanism ensures that Services work transparently across the cluster, regardless of which node initiates the request.

<img width="1176" height="784" alt="image" src="https://github.com/user-attachments/assets/71b6540f-9c3f-4837-a14f-bba1b4afbf7f" />


