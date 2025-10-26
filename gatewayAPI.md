
#### Limitations of Ingress

* Ingress only supports HTTP-based rules such as host and path matching.
* However, it does not natively support other routing protocols like TCP, UDP, or advanced features such as traffic splitting, header manipulation, authentication, or rate limiting.
* To implement these capabilities, controller-specific annotations are used.
```
  # ingress-with-annotations.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

#### Introducing the Gateway API

The Gateway API was created as an official Kubernetes project to overcome the limitations of Ingress. \
It supports both layer 4 (transport) and layer 7 (application) routing, representing the next generation of load balancing and service mesh APIs. \
By decoupling responsibilities, the Gateway API introduces three distinct objects:

* **Gateway Class**: Cluster-wide resource jo ek controller type (implementation) ko represent karta hai; operators yeh define karte hain.
* **Gateway**: Infrastructure instance (controller-managed) jo listeners (ports/protocols) expose karta hai; Gateway ko operator manage karta hai.
* **HTTPRoute (and other route types)**: Application-facing route objects jo Gateway pe bind ho kar specific hosts, paths, headers, traffic-splitting rules, retries, timeouts, aur backend Service targets define karte hain; app teams banao aur manage karte hain.
 
**Gateway API Configuration Example**
The following example illustrates how to define a Gateway Class, a Gateway, and an HTTPRoute using the Gateway API:
```
# gateway-class.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: example-class
spec:
  controllerName: example.com/gateway-controller
```

```
# gateway.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
spec:
  gatewayClassName: example-class
  listeners:
    - name: http
      protocol: HTTP
      port: 80
```

```
# http-route.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-httproute
spec:
  parentRefs:
    - name: example-gateway
  hostnames:
    - "www.example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /login
      backendRefs:
        - name: example-svc
          port: 8080
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/084f98eb-5a50-4f1a-ae9b-af3d4fd126ac" />
