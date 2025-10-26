## Kubernetes Ingress 
Ingress ek Kubernetes API object hai jo cluster ke bahar se HTTP/HTTPS traffic ko cluster ke internal Services tak route karta hai. \
Ingress khud traffic ko implement nahi karta — iske rules ko ek Ingress Controller read karke actual routing, load balancing, TLS termination, rewrites, aur additional features provide karta hai.

**Components**
- Ingress resource: YAML object jo rules, hosts, paths aur optional TLS define karta hai.
- Ingress Controller: Pod/DaemonSet jo Ingress resource ko observe karke real proxy/load‑balancer behavior provide karta hai (NGINX, Traefik, HAProxy, Contour, Istio, AWS ALB Ingress Controller, GKE/GCE Ingress, etc.).
- IngressClass: Cluster‑level object that ties an ingress to a specific controller implementation.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /         # pathType decides matching behavior
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 8080
```

Notes: pathType can be Exact, Prefix, or ImplementationSpecific; prefer Prefix or Exact for deterministic behavior.

**TLS termination** \
Ingress can terminate TLS so backend Services get plain HTTP:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  tls:
    - hosts:
        - app.example.com
      secretName: app-tls-secret   # contains tls.crt and tls.key
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-svc
                port:
                  number: 80
```

**Advanced features via Controller annotations**

Behavior like path rewrite, rate limiting, auth, sticky sessions, websocket support, client certificate validation, etc., are controller-specific and enabled via annotations or CRDs. 
Examples for NGINX Ingress Controller:
- Path rewrite to strip prefix:
  - annotation: nginx.ingress.kubernetes.io/rewrite-target: /$2
  - use regex pathType ImplementationSpecific and capture groups.
- Basic auth:
  - nginx.ingress.kubernetes.io/auth-type: basic
  - nginx.ingress.kubernetes.io/auth-secret: basic-auth-secret
```
metadata:
  name: rewrite-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: rewrite.example.com
      http:
        paths:
          - path: /service/(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: service-svc
                port:
                  number: 80
```

**IngressClass and selecting controller** \
Use IngressClass to register a controller and make selection explicit:

```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
```

**Limitations and migration path**
- Ingress API focuses on HTTP/HTTPS only; for TCP/UDP you need Service of type LoadBalancer, NodePort, or controller-specific config (ConfigMap or CRD).


