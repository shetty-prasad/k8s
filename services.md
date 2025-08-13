Kubernetes services allow different sets of Pods to interact with each other. Either within the cluster or from outside the cluster .

Different types of services :

<img width="660" height="314" alt="image" src="https://github.com/user-attachments/assets/43b54178-5c78-4b1f-8663-a1bb7390bdba" />


* **ClusterIP**: Creates a virtual IP for internal communication between services (e.g., connecting front-end to back-end servers). It is used by pods to connect internally
* **NodePort**: Maps a port on the node to a port on a Pod. It is used by pods to connect externally
* **LoadBalancer**: Provisions an external load balancer (supported in cloud environments) to distribute traffic across multiple Pods.

**NodePort Service Breakdown**
With a NodePort service, there are three key ports to consider:

* **Target Port**: The port on the Pod where the application listens (e.g., 80).

* **Port**: The virtual port on the service within the cluster.

* **NodePort**: The external port on the Kubernetes node (by default in the range 30000–32767)

**Note that if you omit targetPort, it defaults to the same value as port. Similarly, if nodePort isn’t provided, Kubernetes automatically assigns one.**

To connect the service to specific Pods, a selector is used, just as in ReplicaSets or Deployments.
```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

kubectl get services

Access the web service externally by pointing your browser or using curl with the node IP and NodePort:
curl http://192.168.1.2:30008
