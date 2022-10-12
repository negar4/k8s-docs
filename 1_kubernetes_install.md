# Deploy Kubernetes on Oracle Linux

## Initialization

### Setup requirements

#### **Minimum**

* A server running on Oracle Linux 8.
* 2GB RAM per machine i.e Master and worker.
* Atleast 2 CPUs on control-plane (master) node
* Full network connectivity among all machines in the cluster
* Unique hostname, MAC address, and product_uuid for every node.
* TCP ports : 6443, 2379-2380 ,10250 , 10259 , and 10257 should be opened.
* Swap must be disabled.

#### **Our case**

Master, OracleLinux 8, 4 CPU, 16RAM, 100Gb Disk Space:

* dev-k8-master1.turnitvnet.eu

Workers, OracleLinux 8, 2 CPU, 16 RAM, 50Gb Disk Space:

* dev-k8-worker1.turnitvnet.eu
* dev-k8-worker2.turnitvnet.eu
* dev-k8-worker3.turnitvnet.eu

### Other pre-steps

These action should be performed before starting the install process

* Selinux: off
* Firewall: off
* Install latest OS updates
* Add VMs MAC addresses to dhcp server so the IP address won't change

## Installation steps

### On both master and worker nodes

1. Update the system packages:

```bash
sudo yum update -y
```

2. Add Kubernetes Repository

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

3. Install Kubeadm & Docker Container Engine

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io --allowerasing
sudo yum install kubeadm -y
sudo systemctl enable kubelet
sudo systemctl start kubelet
sudo systemctl enable docker
sudo systemctl start docker
```

4. Ensure that docker and kubelet are running

```bash
systemctl status docker
systemctl status kubelet
```

5. Uncommend containerd disabled_plugins config and restart the service:

[If not executed, will get some errors in further steps](https://github.com/containerd/containerd/issues/4581#issuecomment-698741693)

```bash
sed -i '/disabled_plugins/s/^/#/' /etc/containerd/config.toml
systemctl restart containerd
```

### **Steps to execute on master node only**

1. Pull required images:

```bash
sudo kubeadm config images pull
```

2. Initialize Kubernetes cluster:

```bash
sudo kubeadm init
```

The command output will look like this:

```bash
$ sudo kubeadm init
[init] Using Kubernetes version: v1.23.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8smaster kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.201.9]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8smaster localhost] and IPs [192.168.201.9 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8smaster localhost] and IPs [192.168.201.9 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 5.002282 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.23" in namespace kube-system with the configuration for the kubelets in the cluster
NOTE: The "kubelet-config-1.23" naming of the kubelet ConfigMap is deprecated. Once the UnversionedKubeletConfigMap feature gate graduates to Beta the default name will become just "kubelet-config". Kubeadm upgrade will handle this transition transparently.
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8smaster as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8smaster as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: i8gowt.xwke5bnuqn3jah0g
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.201.9:6443 --token i8gowt.xwke5bnuqn3jah0g \
--discovery-token-ca-cert-hash sha256:ca0842e57b703e161c2de0ceddbaaef50d3ac2663bbd8ecebc4fde2b6f6f5617

```

3. Please save last 2 lines from previous command:

```bash
kubeadm join 192.168.201.9:6443 --token i8gowt.xwke5bnuqn3jah0g \
--discovery-token-ca-cert-hash sha256:ca0842e57b703e161c2de0ceddbaaef50d3ac2663bbd8ecebc4fde2b6f6f5617
```

`If you run into any error while initiliazing kubeadm, run the following commands to resolve the error:`

1. Reset the kubeadm cluster then flush the iptables:

```bash
sudo kubeadm reset -f
sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
```

2. Change docker cgroup driver to systems, then restart docker service.

```bash
sudo vim etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}

sudo systemctl daemon-reload
sudo systemctl restart docker
```

3. Swapoff and restart and enable kubelet service.

```bash
sudo swapoff -a
sudo systemctl start kubelet 
sudo systemctl enable kubelet
```

`To start using the Cluster, run the commands below as a regular user:`

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Or as the **root** user:

```bash
mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
```

**Run the command on master node to check active nodes in the cluster:**

```bash
kubectl get nodes
```

Expected output:

```bash
$ kubectl get nodes
NAME        STATUS     ROLES                  AGE   VERSION
k8smaster   NotReady   control-plane,master   14m   v1.23.1
```

`From the output, the status is Not Ready as we have not installed the pod networks. This is expected.`

### Installing pod network using Calico Network, **should be done also only on the master node**

1. Download calico.yaml file

```bash
curl https://docs.projectcalico.org/manifests/calico.yaml -O
```

2. Apply the calico YAML file

```bash
kubectl apply -f calico.yaml
```

3. Check pods status

```bash
kubectl get pods -n kube-system
```

Expected output:

```bash
$ kubectl get pods -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-647d84984b-5rz99   1/1     Running   0          48s
calico-node-r2qmp                          1/1     Running   0          48s
coredns-64897985d-gqwn2                    1/1     Running   0          16m
coredns-64897985d-mcdsd                    1/1     Running   0          16m
etcd-k8smaster                             1/1     Running   0          16m
kube-apiserver-k8smaster                   1/1     Running   0          16m
kube-controller-manager-k8smaster          1/1     Running   0          16m
kube-proxy-zxvqs                           1/1     Running   0          16m
kube-scheduler-k8smaster                   1/1     Running   0          16m
```

4. If everything is `Running` on previous step, check nodes status:

```bash
$ kubectl get nodes
NAME        STATUS   ROLES                  AGE   VERSION
k8smaster   Ready    control-plane,master   17m   v1.23.1
```

`Master node is ready.`

### Join Worker nodes

With Master node now ready, we need to join the worker nodes.
This is done by using the token created using `kubeadm init` command, [3. from this section](#Steps-to-execute-on-master-node-only)

1. **Ensure that you have installed everything from** [this section](#On-both-master-and-worker-nodes)

2. On the **worker** nodes, run the command:

```bash
sudo kubeadm join 192.168.201.9:6443 --token i8gowt.xwke5bnuqn3jah0g \
--discovery-token-ca-cert-hash sha256:ca0842e57b703e161c2de0ceddbaaef50d3ac2663bbd8ecebc4fde2b6f6f5617 
```

Expected output:

```bash
$ sudo kubeadm join 192.168.201.9:6443 --token i8gowt.xwke5bnuqn3jah0g \
> --discovery-token-ca-cert-hash sha256:ca0842e57b703e161c2de0ceddbaaef50d3ac2663bbd8ecebc4fde2b6f6f5617 
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

2. On the **master node** enter the command below, to check if node joined thr cluster:

```bash
sudo kubectl get nodes
```

Expected output:

```bash
$ sudo kubectl get nodes
NAME         STATUS   ROLES                  AGE     VERSION
k8smaster    Ready    control-plane,master   78m     v1.23.1
k8sworker1   Ready    <none>                 5m46s   v1.23.1
```

`Worker nodes have successfully joined the master node. Perform these actions for all nodes`

3. Confirm that all pods are running by running the command:

```bash
sudo kubectl get pods --all-namespaces
```

Expected output:

```bash
sudo kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-647d84984b-5rz99   1/1     Running   0          71m
kube-system   calico-node-5pxq7                          1/1     Running   0          14m
kube-system   calico-node-92s6r                          1/1     Running   0          10m
kube-system   calico-node-r2qmp                          1/1     Running   0          71m
kube-system   coredns-64897985d-gqwn2                    1/1     Running   0          87m
kube-system   coredns-64897985d-mcdsd                    1/1     Running   0          87m
kube-system   etcd-k8smaster                             1/1     Running   0          87m
kube-system   kube-apiserver-k8smaster                   1/1     Running   0          87m
kube-system   kube-controller-manager-k8smaster          1/1     Running   0          87m
kube-system   kube-proxy-2qq52                           1/1     Running   0          10m
kube-system   kube-proxy-kw96r                           1/1     Running   0          14m
kube-system   kube-proxy-zxvqs                           1/1     Running   0          87m
kube-system   kube-scheduler-k8smaster                   1/1     Running   0          87m
```

`If a token is expired, a new one can be generated using the command below on the master node.`

```bash
kubeadm token create
```

Expected output:

```bash
$ kubeadm token create
9b3s23.2rzuu1swa9cge6dv
```