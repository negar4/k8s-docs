# Ingress controller

## [What is an ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

An Ingress controller is a specialized load balancer for Kubernetes (and other containerized) environments. Kubernetes is the de facto standard for managing containerized applications.

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

Here is a simple example where an Ingress sends all its traffic to one Service:

![ingress](https://d33wubrfki0l68.cloudfront.net/91ace4ec5dd0260386e71960638243cf902f8206/c3c52/docs/images/ingress.svg)

### [Bare-metal considerations](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/)

In traditional *cloud* environments, where network load balancers are available on-demand, a single Kubernetes manifest suffices to provide a single point of contact to the NGINX Ingress controller to external clients and, indirectly, to any application running inside the cluster.
**Bare-metal environments lack this commodity**, requiring a slightly different setup to offer the same kind of access to external consumers.

![cloud](https://kubernetes.github.io/ingress-nginx/images/baremetal/cloud_overview.jpg)
![bm](https://kubernetes.github.io/ingress-nginx/images/baremetal/baremetal_overview.jpg)

### A pure software solution: **MetalLB**

**MetalLB** provides a network load-balancer implementation for Kubernetes clusters that do not run on a supported cloud provider, effectively allowing the usage of LoadBalancer Services within any cluster.

You can check [install guide here](2_metallb_install.md) and it's [documetation here](https://metallb.universe.tf/concepts/)

`Please note that MetalLB is requiered by ingress-nginx to create a LoadBalancer type service.`

## How to install [ingress-nginx](https://kubernetes.github.io/ingress-nginx/) with helm?

### What is [Helm](https://helm.sh/)?

[Helm](https://helm.sh/) is a Kubernetes deployment tool for automating creation, packaging, configuration, and deployment of applications and services to Kubernetes clusters.

### What is ingress-nginx?

**ingress-nginx** is an Ingress controller for Kubernetes using NGINX as a reverse proxy and load balancer.

### Installation guide

1. Install helm on your local machine

2. Add ingress-nginx helm repository:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

3. Install [ingress-nginx chart](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx) in a namespace:

```bash
helm upgrade --install ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```

This command will install the controller in the ingress-nginx namespace, creating that namespace if it doesn't already exist.

4. Check if ingress-nginx pods are running:

```bash
kubectl get pods --namespace=ingress-nginx
```

After a while, they should all be running. The following command will wait for the ingress controller pod to be up, running, and ready:

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

5. Local testing

Create a simple web server and the associated service:

```bash
kubectl create deployment demo --image=httpd --port=80
kubectl expose deployment demo
```

Then create an ingress resource. The following example uses an host that maps to localhost:

```bash
kubectl create ingress demo-localhost --class=nginx \
  --rule="demo.localdev.me/*=demo:80"
```

Now, forward a local port to the ingress controller:

```bash
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
```

At this point, if you access [thispage](http://demo.localdev.me:8080/), you should see an HTML page telling you "It works!".
