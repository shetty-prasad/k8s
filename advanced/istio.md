#  Istio

What is service mesh ?

A service mesh is an infrastructure layer that manages communication between microservices within an application. Imagine it as a dedicated layer that sits between the microservices and handles the complexities of service-to-service communication.

Here are some key aspects of a service mesh:

Traffic Management: It intelligently routes requests between services, ensuring efficient and reliable communication.

Security: It provides features like encryption and authentication to secure communication.

Observability: It offers monitoring, logging, and tracing capabilities to understand the behavior of services.

Resilience: It helps in managing retries, timeouts, and circuit-breaking to ensure services are resilient to failures.

Popular service mesh implementations include Istio, Linkerd, and Consul. They help developers focus on building business logic rather than dealing with the intricacies of microservice communication.

![image](https://github.com/user-attachments/assets/be07ce31-e4df-4a76-a689-af77d5bf7c22)

the proxy below is actually the service mesh

![image](https://github.com/user-attachments/assets/e9980cb8-644c-4370-850d-93c707a1b2a0)

### Istio Architecture

![image](https://github.com/user-attachments/assets/94653910-de4e-491e-aba7-04d959e09560)

#### Install Istio

```
curl -L https://istio.io/downloadIstio | sh -
cd istio-<version-number>
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y
```

Istio sidecars do not get created as the default namespace is not enabled for Istio injection

<img width="1062" height="255" alt="image" src="https://github.com/user-attachments/assets/9997471b-d0d8-4d70-a3ac-7de102079860" />

**Enable Istio injection**

```
kubectl label namespace default istio-injection=enabled
```

### Kiali is a GUI to visualize istio 

**Install kiali**
kubectl apply -f /root/istio-1.20.8/samples/addons

kubectl rollout status deployment/kiali -n istio-system

** to access its Kiali UI lets create a service

```
kubectl apply -f /root/kiali-svc.yml

cat kiali-svc.yml 
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kiali
    app.kubernetes.io/instance: kiali
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: kiali
    app.kubernetes.io/part-of: kiali
    app.kubernetes.io/version: v1.63.1
    helm.sh/chart: kiali-server-1.63.1
    version: v1.63.1
  name: kiali
  namespace: istio-system
spec:
  type: NodePort
  ports:
  - appProtocol: http
    name: http
    port: 20001
    protocol: TCP
    targetPort: 20001
    nodePort: 30007
  - appProtocol: http
    name: http-metrics
    port: 9090
    protocol: TCP
    targetPort: 9090
    nodePort: 30008
  selector:
    app.kubernetes.io/instance: kiali
    app.kubernetes.io/name: kiali
---
apiVersion: v1
kind: Service
metadata:
  labels:
    component: server
    app: prometheus-svc
    release: prometheus
    chart: prometheus-15.9.0
    heritage: Helm
  name: prometheus
  namespace: istio-system
spec:
  ports:
    - name: http
      port: 9090
      protocol: TCP
      targetPort: 9090
      nodePort: 30009
  selector:
    component: "server"
    app: "prometheus"
    release: "prometheus"
  type: NodePort
```
