# Configure traefik middlewares, dashboard & weighted round robin services

## Middleware
Traefik middleware act as gatekeeper before the request is sent to the service.

### Write the yaml config for traefik middlewares
you can see all the diffrent types in the  [middlewares section](https://doc.traefik.io/traefik/middlewares/overview/) on the traefik website.

**Example:** Write a middleware for redirect-scheme for HTTP to HTTPS

```
nano redirect-scheme.yaml
```
```
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: nginx-redirect-scheme
spec:
  redirectScheme:
    scheme: https
    permanent: true
    port: "443"

---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: nginx-http
  namespace: default
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`nginx.example.com`)
      kind: Rule
      middlewares:
        - name: nginx-redirect-scheme      
      services:
        - name: nginx-deploy-main
          port: 80

---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: nginx-https
  namespace: default
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`nginx.example.com`)
      kind: Rule     
      services:
        - name: nginx-deploy-main
          port: 80
  tls:
    secertName: secret-tls
```
```
kubectl create -f redirect-scheme.yaml
```

## Dashboard Services
To access the dashboard without port forwarding, we have to create a DNS name in a yaml config as an abstract

```
nano dashboard.yaml
```
```
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: dashboard
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`traefik.example.com)
      kind: Rule
      services:
        - name: api@internal
          kind: TraefikService
```
Add the DNS name to the /etc/host files

```
sudo nano /etc/hosts
# add the dns name to the traefik loadbalancer IP
```
```
kubectl create -f dashboard.yaml
```

## Weighted Round Robin
Weighted round robin is a network traffic scheduler for data flow. 
We can define which nginx application will get more netork traffic.

We will create a new resource type called TraefikService which is not part of the main traefik service, but rather an abstract.

```
nano weighted-round-robin.yaml
```
```
apiVersion: traefik.containo.us/v1alpha1
kind: TraefikService
metadata:
  name: nginx-wrr
  namespace: default
spec:
  weighted:
    services:
      - name: nginx-deploy-main
        port: 80
        weight: 1
      - name: nginx-deploy-blue
        port: 80
        weight: 3  #blue gets the most traffic
      - name: nginx-deploy-green
        port: 80
        weight: 1
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: nginx
  namespace: default
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`nginx.example.com`)
    kind: Rule
    services:
    - name: nginx-wrr
      kind: TraefikService
```
```
kubectl create -f weighted-round-robin.yaml
```
