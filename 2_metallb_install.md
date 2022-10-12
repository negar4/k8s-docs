# metallb installation on bare-metal Kubernetes clsuter

## About metallb

MetalLB hooks into your Kubernetes cluster, and provides a network load-balancer implementation. In short, it allows you to create Kubernetes services of type LoadBalancer in clusters that don’t run on a cloud provider, and thus cannot simply hook into paid products to provide load balancers.

It has two features that work together to provide this service: address allocation, and external announcement. But we will use only **address allocation** at the moment.

### Address allocation

In a Kubernetes cluster on a cloud provider, you request a load balancer, and your cloud platform assigns an IP address to you. In a bare-metal cluster, MetalLB is responsible for that allocation.

MetalLB cannot create IP addresses out of thin air, so you do have to give it pools of IP addresses that it can use. MetalLB will take care of assigning and unassigning individual addresses as services come and go, but it will only ever hand out IPs that are part of its configured pools.

How you get IP address pools for MetalLB depends on your environment. If you’re running a bare-metal cluster in a colocation facility, your hosting provider probably offers IP addresses for lease. In that case, you would lease, say, a /26 of IP space (64 addresses), and provide that range to MetalLB for cluster services.

Alternatively, your cluster might be purely private, providing services to a nearby LAN but not exposed to the internet. In that case, you could pick a range of IPs from one of the private address spaces (so-called RFC1918 addresses), and assign those to MetalLB. Such addresses are free, and work fine as long as you’re only providing cluster services to your LAN.

Or, you could do both! MetalLB lets you define as many address pools as you want, and doesn’t care what “kind” of addresses you give it.

## Install metallb using **helm**

### Pre-steps

We would need a range of IP's which will be used by metallb to provide a LoadBanacer type service, eventually those IP's should be “blocked” on DHCP side.
In our case we used `192.168.222.192/29`

Also you need to have acces to kubernetes cluster and also [helm installed on your machine](https://helm.sh/docs/intro/install/).

### [Installation process](https://metallb.universe.tf/installation/#installation-with-helm)

1. Add metallb helm repo

```bash
$ helm repo add metallb https://metallb.github.io/metallb
```

2. Create a metallb.yaml file, a configuration which should be passed to helm:

```yaml
configInline:
  address-pools:
  - addresses:
    - 192.168.222.192/29
    name: default
    protocol: layer2
```

3. Install metallb release in metallb-system namespace:

```bash
$ helm upgrade --install metallb metallb/metallb -n metallb-system -f metallb.yaml --create-namespace
```

4. Check if metallb was installed:

```bash
$ kubectl get pods -n metallb-system
NAME                                  READY   STATUS    RESTARTS   AGE
metallb-controller-6c9749d779-z9s59   1/1     Running   0          23d
metallb-speaker-9866t                 1/1     Running   0          4d3h
metallb-speaker-wnkzh                 1/1     Running   0          4d3h
metallb-speaker-xs58x                 1/1     Running   0          23d
```

5. Once you will create an service with type set to **LoadBalancer**, it's ExternalIP will have an IP from the range you provided in the config file:

```bash
$ kubectl get svc -A | grep -i load
NAMESPACE        NAME                                                      TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                        AGE
development      nginx-development-ingress-nginx-controller                LoadBalancer   10.108.86.113    192.168.222.193   80:31473/TCP,443:31189/TCP     3d20h
ds-development   nginx-ds-development-ingress-nginx-controller             LoadBalancer   10.98.198.171    192.168.222.194   80:31830/TCP,443:32070/TCP     2d19h
monitoring       nginx-monitoring-ingress-nginx-controller                 LoadBalancer   10.108.40.137    192.168.222.192   80:30603/TCP,443:31512/TCP     3d21h
testing          nginx-testing-ingress-nginx-controller                    LoadBalancer   10.102.2.43      192.168.222.195   80:31559/TCP,443:31796/TCP     2d19h
testingv2        nginx-testingv2-ingress-nginx-controller                  LoadBalancer   10.102.69.123    192.168.222.196   80:30020/TCP,443:30542/TCP     2d18h
```