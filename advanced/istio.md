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
