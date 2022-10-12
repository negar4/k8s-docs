# Use an existing NFS Server as Persistent Volume in kubernetes

## Initialization

### Setup requirements

An running NFS server accesible by kubernetes cluster

### Pre-install requirements

Install `nfs-common` or `nfs-utils` (in case of Oracle Linux) within all **worker nodes**:
```bash
yum update
yum install -y nfs-utils
```

## Persistent Volume creation

1. Create a file named nfs_persistent_volume.yaml and **replace all <> values**:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs
spec:
  capacity:
    storage: <desired_capacity>
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""
  mountOptions:
    - hard
    - nfsvers=3
  nfs:
    path: <your_nfs_path>
    server: <your_nfs_IP_address>
```

2. Create the resource on the kuberenetes cluster:

```bash
kubectl create -f nfs_persistent_volume.yaml
```

## Persistent Volume Claim creation

1. Create a file named nfs_pvc.yaml:

This resource will be attached to the PV created in previous step.

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: ''
  volumeName: nfs
  volumeMode: Filesystem
```

2. Create the resource on the kuberenetes cluster:

```bash
kubectl create -f nfs_pvc.yaml
```

3. Check if PVC was attached to PV:

```bash
$ kubectl get pv,pvc
NAME                                    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   REASON  AGE
persistentvolume/nfs                    5Gi        RWX            Retain           Bound    default/nfs-pvc                         3m


NAME                                          STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/nfs-pvc                 Bound    nfs                     5Gi       RWX                           2m

```
`Now we're ready to use this PVC within in a single/multiple pods`

## Testing PV/PVC configuration

After executing the steps from previous chapters, we can use an NFS PVC in a pod.

1. Create a file named pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
    - name: busybox
      image: busybox
      volumeMounts:
        - mountPath: "/tmp"
          name: nfs-pvc
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: nfs-pvc

```

2. Create the resource on the kuberenetes cluster:

```bash
kubectl create -f pod.yaml
```

3. Check if pod is up and running:

```bash
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
busybox                             1/1     Running   0          2m17s
```

4. Execute some commands in the pod

```bash
$ kubectl exec -it busybox -- sh
/ #
/ df -h /tmp
Filesystem            Size      Used  Available Use% Mounted on
192.168.0.4:/tmp      4.5Gb     2.8Gb 1.4Gb     67%  /tmp

/ echo $(hostname)-hello123 > /tmp/hostname.txt
/ cat /tmp/hostname.txt
busybox-hello123
```

5. Delete pod and re-create it:

```bash
$ kubectl delete pod busybox
pod "busybox" deleted
$ kubectl create -f pod.yaml
pod "busybox" created
```

6. Execute some commands in the **new** pod

```bash
$ kubectl exec -it busybox -- sh
/ cat /tmp/hostname.txt
busybox-hello123
/ echo $(hostname)-hello > /tmp/hostname.txt
/ cat /tmp/hostname.txt
busybox-hello
```

As you can see, the file is in place and we're able to access and even edit it.

### Some official Kuberenetes documentation

[NFS Persistent Volume example](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod)
[Create a pod with PVC](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod)