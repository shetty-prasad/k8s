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

### Virtual Services

* Virtual Services allow you to configure routing rules, directing incoming traffic through the Ingress Gateway to the appropriate service in your service mesh.
* Creating a Virtual Service decouples your traffic routing policies from the actual service implementations, enabling granular control over how your application's requests are handled.
* Virtual Services offer flexibility by allowing you to specify hostnames, manage traffic among different service versions, and use both standard and regex URI paths.
* Once a Virtual Service is created, the Istio control plane disseminates the configuration to all Envoy sidecars in the mesh.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/9cc83d54-4c7a-45eb-b6aa-8ae685b38d3c" />

This configuration instructs Istio to forward any traffic passing through the bookinfo-gateway with the host bookinfo.app and matching the specified URL patterns to the productPage service on port 9080.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - "bookinfo.app"
  gateways:
    - bookinfo-gateway
  http:
    - match:
        - uri:
            exact: /productpage
        - uri:
            prefix: /static
        - uri:
            exact: /login
        - uri:
            exact: /logout
        - uri:
            prefix: /api/v1/products
      route:
        - destination:
            host: productpage
            port:
              number: 9080
```

**Routing Between Service Versions**
Using Virtual Services in conjunction with destination rules (which define subsets like v1 and v2), you can precisely control traffic percentages. 
Using this method A/B testing is also performed .
For example, the following Virtual Service configuration directs 99% of traffic to subset v1 and 1% to subset v2:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - route:
        - destination:
            host: reviews
            subset: v1
          weight: 99
        - destination:
            host: reviews
            subset: v2
          weight: 1
```

### Destination Rules

Destination Rules enable you to define policies that are applied after traffic is routed to a specific service, ensuring controlled distribution and effective load balancing.

Subsets used in Virtual Services are defined in Destination Rules. These rules allow you to apply specific configurations to traffic after it has been routed to a service.
Subsets represent groups of service instances identified by labels on the respective pods. The following Destination Rule illustrates how subsets for the reviews service are declared:
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews    ## this is the service name
  subsets:
    - name: v1
      labels:
        version: v1  ### This label acts as a selector to the deployment
    - name: v2
      labels:
        version: v2
```

**Customizing Load Balancing Policies**
By default, Envoy uses a round-robin load-balancing strategy. However, you can modify this behavior by specifying a traffic policy within a Destination Rule. The following example demonstrates a simple pass-through load-balancing policy:
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: PASSTHROUGH
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

If you require a different policy for a specific subset (for example, a random algorithm for subset "v2"), the global traffic policy can be overridden at the subset level. This flexibility enables you to apply a default policy across all subsets while tailoring specific configurations as necessary.

**Enabling TLS for Enhanced Security**
Destination Rules also support various security configurations such as enabling TLS at the client level.

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /myclientcert.pem
      privateKey: /client_private_key.pem
      caCertificates: /rootcacerts.pem
```

**Remember, the host field plays a crucial role in the Destination Rule. When using a short name (e.g., "reviews"), Istio interprets it relative to the rule’s namespace. To ensure that the rule correctly references the intended service, especially if it resides in a different namespace, always use the fully qualified domain name (FQDN).**

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: **reviews.default.svc.cluster.local**
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /myclientcert.pem
      privateKey: /client_private_key.pem
      caCertificates: /rootcacerts.pem
```

#### Fault Injection
Fault injection is a testing strategy designed to simulate errors and validate the resilience of your error handling mechanisms. 
Fault injection in Istio supports the simulation of two primary error types in Virtual Services:
* Delays
* Aborts

**Simulating Delay Faults**
The following example demonstrates how to inject a delay fault into a Virtual Service. In this configuration, a delay of 5 seconds is applied to 10% of the requests routed to the service.
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-service
spec:
  hosts:
  - my-service
  http:
  - fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    route:
    - destination:
        host: my-service
        subset: v1
```

**Simulating Abort Faults**
In addition to delay faults, you can configure abort faults to simulate scenarios where requests are rejected with specific error codes. Abort faults help test how your service behaves under error conditions, ensuring that fallback mechanisms and error handling policies are effective.

#### Timeouts
* Timeouts prevent a single slow service from adversely affecting the overall system.
* When a dependent service exceeds a configured waiting period, it automatically fails and returns an error, keeping the rest of the network responsive.

To prevent the system from waiting indefinitely for a response from the product page service, you can configure a timeout such that if the service takes longer than three seconds to respond, the request is automatically rejected. This is accomplished by adding a timeout option in the service configuration.

Below is the VirtualService configuration that applies a three-second timeout to the product page:
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - "bookinfo.app"
  gateways:
    - bookinfo-gateway
  http:
    - match:
        - uri:
            exact: /productpage
        - uri:
            prefix: /static
      route:
        - destination:
            host: productpage
            port:
              number: 9080
      timeout: 3s
```

#### Retries
Retries enable your service to automatically attempt to reconnect when a connection failure occurs between services. This helps mitigate transient network issues without modifying your application code.

How Retries Work
When one service fails to connect to another, Virtual Services can be set up to automatically retry the operation. The main parameters for configuration are:

* Attempts: The number of times Istio will try to route the request.
* Per Try Timeout: The timeout duration for each individual retry.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-service
spec:
  hosts:
  - my-service
  http:
  - route:
    - destination:
        host: my-service
        subset: v1
   ** retries:
      attempts: 3
      perTryTimeout: 2s**
```

#### Circuit Breaking

* In a microservices architecture, when one service fails or becomes slow, the subsequent requests are not allowed to pile up.
* Instead, the circuit breaker immediately interrupts the request flow, marking them as failed.
* This approach not only protects the application from being overwhelmed but also serves to limit the number of concurrent requests that can be sent to a particular service endpoint.

In Istio, circuit breaking is configured through Destination Rules. The following example demonstrates the configuration for the Product Page Destination Rule, which restricts the number of concurrent TCP connections to three:
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
    - name: v1
      labels:
        version: v1
**  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 3**
```

### Istio Security Architecture

Key Security Components

**Istiod: The Certification Authority**
Within Istiod, a dedicated certification authority manages keys and certificates across the Istio environment. This component:
* Validates certificates.
* Approves certificate signing requests (CSRs).

 **Envoy Proxy and Istio Agent**
 When a workload starts, the Envoy proxy requests a certificate and key from the Istio agent. This process ensures that all communications between services are securely encrypted and authenticated from the very start.

 **Configuration API Server**
 The Configuration API Server plays a crucial role by distributing authentication, authorization, and secure naming policies across the service mesh. These policies are pushed to:
* Sidecars.
* Ingress and Egress proxies.

### Authentication

Need to study  Authentication, Authorization & Certificate Management 
