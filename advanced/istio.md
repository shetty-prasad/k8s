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

 to access  Kiali UI lets create a service

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

```
istioctl dashboard kiali
```

#### What Is a Sidecar?
* Much like a sidecar on a motorcycle, which is a small passenger compartment attached for additional capacity, a sidecar container is an extra component within a Kubernetes Pod.
* While the primary container runs the core business logic of your application, the sidecar container is dedicated to handling tasks such as:

* Log shipping
* Monitoring
* File loading
* Proxying

#### What Is a Proxy?
* A proxy acts as an intermediary between a user and an application.
* Instead of embedding additional functionalities—such as TLS encryption, authentication, and request retries—directly into your application, these tasks can be offloaded to a proxy.
* This approach enables developers to concentrate on the core business logic while the proxy handles supplementary operations

<img width="801" height="357" alt="image" src="https://github.com/user-attachments/assets/34c19c56-c898-4a80-9f43-e4f93653ebc4" />

* Envoy is an open-source proxy designed specifically for modern, service-oriented architectures.
* Typically, Envoy is deployed as a sidecar container alongside your primary application containers.
* This design ensures that all inbound and outbound pod traffic is managed by Envoy, which enhances communication handling and offloads additional features from your application.

### Gateways

Istio Gateways function as load balancers at the edge of the mesh, handling both inbound and outbound traffic. When Istio is deployed on a cluster, it automatically installs both the Istio Ingress Gateway and Istio Egress Gateway.

<img width="814" height="411" alt="image" src="https://github.com/user-attachments/assets/23747dc0-6162-49a0-adcf-e72d98784a49" />

To achieve this, you first create a Gateway object that accepts HTTP traffic on port 80 for the specified hostname. To ensure that this configuration applies to the default Istio Ingress Gateway (and not any custom gateways), add a selector that targets the default controller label. Use the following configuration as a starting point:
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # uses Istio's default ingress gateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "bookinfo.app"
```

kubectl describe gateway bookinfo-gateway

