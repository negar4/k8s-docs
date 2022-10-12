# [Kubernetes Overview](https://kubernetes.io/docs/home/)

## Containers

<img src="https://www.developer-tech.com/wp-content/uploads/sites/3/2021/10/frank-mckenna-tjX_sniNzgQ-unsplash-scaled.jpg" alt="containers" width="400"/>

Linux containers are technologies that allow you to package and isolate applications with their entire runtime environment-all of the files necessary to run. This makes it easy to move the contained application between the environments(dev, staging/test, prod, etc) while retaining full functionality. Containers help reduce conflicts between your development and operations teams by separating areas of responsibility.

## What is container orchestration?

Container orchestration automates the deployment, management, scaling, and networking of containers. The companies that need to deploy and manage hundreds or thousands of containers and hosts can benefit from container orchestration.

Container orchestration automates and manages tasks such as:

1. Provisioning and deployment
2. Configuration and scheduling
3. Container availability
4. Load balancing and traffic routing
5. Scaling and removing containers

## What is Kubernetes?

<img src="https://kubernetes.io/images/kubernetes-horizontal-color.png" alt="kube-logo" width="400"/>

**Kubernetes** (also known as k8s or “Kube”) is an open-source container orchestration platform that automates many of the manual processes involved in deploying, managing, and scaling containerized applications.

In other words, you can cluster together groups of hosts running Linux containers, and Kubernetes helps you easily and efficiently.

## Why you need Kubernetes?

Containers are a good and easy way to bundle and run your applications. In a production environment, you need to manage the containers that run the applications and ensure that there is no downtime. For example, if a container goes down, another container needs to start.
You have to ssh to the server, then launch the container, again ssh to another server launch another one. maintain if any container fails… then again check and launch a new container.

Wouldn’t it be easier if this behavior was handled by a system?
That’s how Kubernetes comes to the rescue! Kubernetes provides us an interface to run distributed systems smoothly. It takes care of scaling and failover for your application, provides deployment patterns, and more.

Kubernetes provides you with:

* **Service discovery and load balancing**: Kubernetes can expose a container using the DNS name or using their own IP address. If traffic to a container is high, Kubernetes is able to load balance and distribute the network traffic so that the deployment is stable.
* **Storage orchestration**: Kubernetes allows you to automatically mount a storage system of your choice, such as local storage, public cloud providers, and more.
* **Automated rollouts and rollbacks**: You can describe the desired state for your deployed containers using Kubernetes, and it can change the actual state to the desired state at a controlled rate. For example, you can automate Kubernetes to create new containers for your deployment, remove existing containers, and adopt all their resources to the new container.
* **Automatic bin packing**: You provide Kubernetes with a cluster of nodes that it can use to run containerized tasks. You tell Kubernetes how much CPU and memory (RAM) each container needs. Kubernetes can fit containers onto your nodes to make the best use of your resources.
* **Self-healing**: Kubernetes restarts containers that fail, replaces containers, kills containers that don’t respond to your user-defined health check, and doesn’t advertise them to clients until they are ready to serve.
* **Secret and configuration management**: Kubernetes lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. You can deploy and update secrets and application configuration without rebuilding your container images, and without exposing secrets in your stack configuration.

## Kubernetes Components

When you deploy Kubernetes, you get a cluster.

A Kubernetes cluster consists of a set of worker machines, called nodes, that run containerized applications. Every cluster has at least one worker node.

The worker node(s) host the Pods that are the components of the application workload. The control plane manages the worker nodes and the Pods in the cluster. In production environments, the control plane usually runs across multiple computers and a cluster usually runs multiple nodes, providing fault-tolerance and high availability.

![The components of a Kubernetes cluster](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)The components of a Kubernetes cluster

Control Plane Components:

| Name                     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| kube-apiserver           | The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.The main implementation of a Kubernetes API server is kube-apiserver. kube-apiserver is designed to scale horizontally—that is, it scales by deploying more instances. You can run several instances of kube-apiserver and balance traffic between those instances.                                                                                                                          |
| etcd                     | Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data. If your Kubernetes cluster uses etcd as its backing store, make sure you have a back up plan for those data.                                                                                                                                                                                                                                                                                                                                         |
| kube-scheduler           | Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on. Factors taken into account for scheduling decisions include: individual and collective resource requirements, hardware/software/policy constraints, affinityand anti-affinity specifications, data locality, inter-workload interference, and deadlines.                                                                                                                                                                           |
| kube-controller-manager  | Control plane component that runs controller processes. Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process                                                                                                                                                                                                                                                                                                                                                   |
| cloud-controller-manager | A Kubernetes control plane component that embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster. The cloud-controller-manager only runs controllers that are specific to your cloud provider. If you are running Kubernetes on your own premises, or in a learning environment inside your own PC, the cluster does not have a cloud controller manager. |

Node Components:

| Name              | Description                                                                                                                                                                                                                           |
| ----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| kubelet           | An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.                                                                                                                                   |
| kube-proxy        | kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.                                                                                                            |
| Container runtime | The container runtime is the software that is responsible for running containers. Kubernetes supports container runtimes such as containerd, CRI-O, and any other implementation of the Kubernetes CRI (Container Runtime Interface). |

If you want to learn more on kubernetes, just access [the official documentation page](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/).
