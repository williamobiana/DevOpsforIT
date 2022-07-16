# Configure traefik for your kubernetes cluster

Traefik is a modern HTTP reverse proxy and load balancer that makes deploying microservices easy.

## How to configure traefik for your kubernetes cluster
- ensure prerequisites are met
- install a loadbalancer (metallb)
- install helm
- helm install traefik
- view dashboard

## Prerequisites
- active cluster (you can check out the post on how to deploy a cluster using vagrant here)

## Install a loadbalancer (metallb)
We will be installing metallb as a loadbalancer in the cluster
- go to the [installation section](https://metallb.universe.tf/installation/) of metallb website and scroll to "install by manifest"
- copy and apply the manifest in your cluster
- go to the [configuration section](https://metallb.universe.tf/configuration/) of metallb website and scroll to "layer 2 configuration"
- create a metallb.yaml file, copy and paste the IPAddressPool & L2Advertisement configuration into the metallb.yaml, **modify the IP address to match your cluster IP (eg: 10.0.0.240-10.0.0.250)**
- create the manifest

```
kubectl create -f metallb.yaml
```
- test the metallb loadbalancer is working, by deploying a simple nginx application

```
kubectl create deploy nginx --image nginx
kubectl expose deploy nginx --port 80 --type loadbalancer
```
- verify in the browser
- delete if no issues arise

```
kubectl delete svc nginx
kubectl delete deploy nginx
```

## Install helm
Instructions to [install helm](https://helm.sh/docs/intro/install/) is on the website, and my preferred is using Apt.

```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

## Helm install traefik
Using helm, install traefik

```
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
# export the traefik manifest, incase you may need to make modification to the config
helm show values traefik/traefik > /traefik.yaml
# use helm to create a namespace for the traefik resource
helm install traefik traefik/traefik --values traefik.yaml -n traefik --create-namespace
```
Verify the traefik pod is running and service is deployed on the metallb loadbalancer

```
kubectl get all -n traefik
```

## View dashboard
Copy the traefik pod name, open a seperate terminal and port-forward from your localhost to the traefik container dashboard running on port 9000

```
kubectl -n traefik port-forward <traefik pod name> 9000:9000
```
Verify on the browser with url **http://localhost:9000/dashboard**

