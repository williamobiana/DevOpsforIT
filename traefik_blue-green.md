# Configure traefik for blue-green deployment

A blue/green deployment is a deployment strategy in which you create two separate, but identical environments

## How to configure a blue-green deployment
- make directory and write yaml for app deployment
- deploy & expose the apps on cluster_IP and verify.
- add loadbalancer IP to etc/hosts or DNS server
- write yaml files for ingressroute and create
- if tls certificate is present, reference in ingressroute

## Write the yaml files for a simple blue-green deployment
We will create 3 nginx deployments:
1. nginx-deploy-main

```
nano nginx-deploy-main.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx-deploy-main
  name: nginx-deploy-main
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-deploy-main
  template:
    metadata:
      labels:
        run: nginx-deploy-main
    spec:
      containers:
      - image: nginx
        name: nginx
```
2. nginx-deploy-blue

```
nano nginx-deploy-main.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx-deploy-blue
  name: nginx-deploy-blue
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-deploy-blue
  template:
    metadata:
      labels:
        run: nginx-deploy-blue
    spec:
      volumes:
      - name: webdata
        emptyDir: {}
      initContainers:
      - name: web-content
        image: busybox
        volumeMounts:
        - name: webdata
          mountPath: "/webdata"
        command: ["/bin/sh", "-c", 'echo "<h1>I am <font color=blue>BLUE</font></h1>" > /webdata/index.html']
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: webdata
          mountPath: "/usr/share/nginx/html"
```
3. nginx-deploy-green

```
nano nginx-deploy-main.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx-deploy-green
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-green
  template:
    metadata:
      labels:
        run: nginx-green
    spec:
      volumes:
      - name: webdata
        emptyDir: {}
      initContainers:
      - name: web-content
        image: busybox
        volumeMounts:
        - name: webdata
          mountPath: "/webdata"
        command: ["/bin/sh", "-c", 'echo "<h1>I am <font color=green>GREEN</font></h1>" > /webdata/index.html']
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: webdata
          mountPath: "/usr/share/nginx/html"
```

## Deploy the nginx application
In the directory we have created for the applications, we deploy all yaml files, and expose them.\
If we dont specity the type of service the applications should be exposed as, it will default to cluster_IP.

```
kubectl create -f nginx-deploy-main.yaml -f nginx-deploy-blue.yaml -f nginx-deploy-green.yaml
kubectl expose nginx-deploy-main --port 80
kubectl expose nginx-deploy-blue --port 80
kubectl expose nginx-deploy-green --port 80
```
Verify the deployment and exposed service

```
kubectl get all
```

## Add loadbalancer IP to etc/hosts or DNS server
If running on localhost, add the loadbalancer IP for traefik to the /etc/hosts file

```
kubectl get all -n traefik 
#copy traefik loadbalancer IP

sudo nano /etc/hosts
#paste ip and give a dns name for the deployments eg: nginx.example.com blue.nginx.example.com green.nginx.example.com
```

If running as a business, add the loadbalancer IP for traefik to the DNS record and assign a dns name.

## Write yaml files for ingressroute
We will create an ingressroute with rules on how it should route the traefik

```
nano simple-ingressroute.yaml
```
```
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
        - name: nginx-deploy-main
          port: 80
    - match: Host(`blue.nginx.example.com`)
      kind: Rule
      services:
        - name: nginx-deploy-blue
          port: 80
    - match: Host(`green.nginx.example.com`)
      kind: Rule
      services:
        - name: nginx-deploy-green
          port: 80
```
```
kubectl create -f simple-ingressroute.yaml
```

## Write yaml files for tls-ingressroute
If you have a self-signed certificate or already approved certifate from a certificate authority and dont want to use a certifcate resolver, you can follow the steps below:

1. Create a [kuberetes TLS secret](https://kubernetes.io/docs/concepts/configuration/secret/#tls-secrets) for your certifcate

```
nano secret-tls.yaml
```
```
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  # the data is abbreviated in this example
  tls.crt: |
        MIIC2DCCAcCgAwIBAgIBATANBgkqh ...
  tls.key: |
        MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ...
```
```
kubectl create -f secret-tls.yaml
```

2. Create the ingressroute
```
nano tls-ingressroute.yaml
```
```
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: nginx
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
    - match: Host(`blue.nginx.example.com`)
      kind: Rule
      services:
        - name: nginx-deploy-blue
          port: 80
    - match: Host(`green.nginx.example.com`)
      kind: Rule
      services:
        - name: nginx-deploy-green
          port: 80
  tls:
    secertName: secret-tls
```
```
kubectl create -f tls-ingressroute.yaml
```
