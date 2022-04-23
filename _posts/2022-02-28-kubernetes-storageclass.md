---
layout: single
title:  "Kubernetes Storage Class"
date:   2022-02-28 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"

toc_sticky: true
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
show_date: true
header:
  teaser: /assets/images/sc-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes StorageClass


### StorageClass

In the previous example, we used a storageClass named `manual`, but that does not seem to be present. 

```shell
pradeep@learnk8s$ kubectl get sc
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  12d
```

There is a storage class by name standard, and is the default storage class.

```shell
pradeep@learnk8s$ kubectl describe sc
Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"labels":{"addonmanager.kubernetes.io/mode":"EnsureExists"},"name":"standard"},"provisioner":"k8s.io/minikube-hostpath"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           k8s.io/minikube-hostpath
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```
:memo: Changing the storage class name to `standard` also did not help solve the Forbidden issue with the `nginx`.



Strangely, inside the Pod, we can access the `index.html` file with `cat` command, but `curl` is not working.

```shell
pradeep@learnk8s$ kubectl exec -it my-pv-pod -- /bin/bash
root@my-pv-pod:/# cat /usr/share/nginx/html/indext.html
Hello from Kubernetes storage
```

```shell
pradeep@learnk8s$ kubectl explain sc
KIND:     StorageClass
VERSION:  storage.k8s.io/v1

DESCRIPTION:
     StorageClass describes the parameters for a class of storage for which
     PersistentVolumes can be dynamically provisioned.

     StorageClasses are non-namespaced; the name of the storage class according
     to etcd is in ObjectMeta.Name.

FIELDS:
   allowVolumeExpansion	<boolean>
     AllowVolumeExpansion shows whether the storage class allow volume expand

   allowedTopologies	<[]Object>
     Restrict the node topologies where volumes can be dynamically provisioned.
     Each volume plugin defines its own supported topology specifications. An
     empty TopologySelectorTerm list means there is no topology restriction.
     This field is only honored by servers that enable the VolumeScheduling
     feature.

   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   mountOptions	<[]string>
     Dynamically provisioned PersistentVolumes of this storage class are created
     with these mountOptions, e.g. ["ro", "soft"]. Not validated - mount of the
     PVs will simply fail if one is invalid.

   parameters	<map[string]string>
     Parameters holds the parameters for the provisioner that should create
     volumes of this storage class.

   provisioner	<string> -required-
     Provisioner indicates the type of the provisioner.

   reclaimPolicy	<string>
     Dynamically provisioned PersistentVolumes of this storage class are created
     with this reclaimPolicy. Defaults to Delete.

   volumeBindingMode	<string>
     VolumeBindingMode indicates how PersistentVolumeClaims should be
     provisioned and bound. When unset, VolumeBindingImmediate is used. This
     field is only honored by servers that enable the VolumeScheduling feature.
```



### Static Binding 

Here is the PersistentVolume definition. If we omit the storageClassName, PVC is net getting bound. As you see, there is a default storageClass named `standard` and PVC is using it by default.

```shell
pradeep@learnk8s$ cat pv.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv
spec:
  storageClassName: standard
  capacity:
    storage: 512m
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /data/config
```

Create the PV from the YAML file.
```shell
pradeep@learnk8s$ kubectl create -f pv.yaml
persistentvolume/pv created
```

Verify the available PVs. The newly created PV should be in `Available` state. Currently, there are no Claims for this volume.

```shell
pradeep@learnk8s$ kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv     512m       RWX            Retain           Available           standard                8s
```
Describe the PV for addtional details.

```shell
pradeep@learnk8s$ kubectl describe pv pv
Name:            pv
Labels:          <none>
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Available
Claim:
Reclaim Policy:  Retain
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        512m
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /data/config
    HostPathType:
Events:            <none>
```
Now, create a PVC with same `accessModes: ReadWriteMany`.

```yaml
pradeep@learnk8s$ cat pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 256m
```
Create PVC from the YAML file.

```shell
pradeep@learnk8s$ kubectl create -f pvc.yaml
persistentvolumeclaim/pvc created
```
Verify the newly created PVC. It should be in `Bound` state. Note the `STORAGECLASS` column. It is using the default one named `standard`.

```shell
pradeep@learnk8s$ kubectl get pvc
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc    Bound    pv       512m       RWX            standard       4s
```
Describe the PVC for addtional details.

```shell
pradeep@learnk8s$ kubectl describe pvc
Name:          pvc
Namespace:     default
StorageClass:  standard
Status:        Bound
Volume:        pv
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      512m
Access Modes:  RWX
VolumeMode:    Filesystem
Used By:       <none>
Events:        <none>
```
Now, create a Pod that uses the newly defined PVC.

```yaml
pradeep@learnk8s$ cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - image: nginx
    name: app
    volumeMounts:
    - mountPath: "/data/app/config"
      name: configpvc
  volumes:
  - name: configpvc
    persistentVolumeClaim:
      claimName: pvc
  restartPolicy: Never
```
Create the Pod from the YAML file.
```shell
pradeep@learnk8s$ kubectl create -f pod.yaml
pod/app created
```
Use the `-o wide` option to check the IP and NODE details.

```shell
pradeep@learnk8s$ kubectl get pods app -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP           NODE      NOMINATED NODE   READINESS GATES
app    1/1     Running   0          18s   10.244.1.5   k8s-m02   <none>           <none>
```
Describe the Pod and look for Volumes section.

```shell
pradeep@learnk8s$ kubectl describe pods app
Name:         app
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Mon, 28 Feb 2022 09:25:46 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.1.5
IPs:
  IP:  10.244.1.5
Containers:
  app:
    Container ID:   docker://09871149e09a1da02a86a9e38ace82082def59aa38dffbf739e14f7694f0f79e
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 28 Feb 2022 09:25:50 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data/app/config from configpvc (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vfx8b (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  configpvc:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  pvc
    ReadOnly:   false
  kube-api-access-vfx8b:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  63s   default-scheduler  Successfully assigned default/app to k8s-m02
  Normal  Pulling    62s   kubelet            Pulling image "nginx"
  Normal  Pulled     59s   kubelet            Successfully pulled image "nginx" in 3.044922832s
  Normal  Created    59s   kubelet            Created container app
  Normal  Started    59s   kubelet            Started container app
```
Login to the Container and create a file in the `/data/app/config` folder, which is the container MountPath.

```shell
pradeep@learnk8s$ kubectl exec -it app -- /bin/sh
# cd /data/app/config
# ls -l
total 0
# touch hello.txt
# echo "Testing PV, PVC, and SC in K8S!" > hello.txt
# exit
```
Login to the minikube node and check if you are able to view the `hello.txt` file in the host.
On the control plane, there is no such file in the `/data/config` folder.
```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ls /data/config/
$ ls -la /data/config/
total 8
drwxr-xr-x 2 root root 4096 Feb 28 03:33 .
drwxr-xr-x 3 root root 4096 Feb 28 03:33 ..
$ exit
logout
```
On the other node, `k8s-m02`,  there is the `hello.txt` file.  This is because, the `app` pod got created in this node.

```shell
pradeep@learnk8s$ minikube ssh -n k8s-m02 -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ls /data/config
hello.txt
$ ls -la /data/config/
total 12
drwxr-xr-x 2 root root 4096 Feb 28 03:58 .
drwxr-xr-x 3 root root 4096 Feb 28 03:55 ..
-rw-r--r-- 1 root root   32 Feb 28 03:58 hello.txt
$ cat /data/config/hello.txt
Testing PV, PVC, and SC in K8S!
$ exit
logout
```
We can also view the contents of the `hello.txt` file.

Verify the PV, PVC, and SC details.

```shell
pradeep@learnk8s$ kubectl get pv,pvc,sc
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
persistentvolume/pv   512m       RWX            Retain           Bound    default/pvc   standard                9m35s

NAME                        STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc   Bound    pv       512m       RWX            standard       8m17s

NAME                                             PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  12d
```
Describe the PV one more time, after the Pod creation, to see if there is any change.
One change that we notice is that the Claim is showing `default/pvc` which was empty earlier. Also, Staus changed to `Bound` from `Available`.

```shell
pradeep@learnk8s$ kubectl describe pv
Name:            pv
Labels:          <none>
Annotations:     pv.kubernetes.io/bound-by-controller: yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Bound
Claim:           default/pvc
Reclaim Policy:  Retain
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        512m
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /data/config
    HostPathType:
Events:            <none>
```
Similarly describe the PVC and look for the changes. The  `Used By` value has changed to `app` now, showing that it is used by this Pod.

```shell
pradeep@learnk8s$ kubectl describe pvc
Name:          pvc
Namespace:     default
StorageClass:  standard
Status:        Bound
Volume:        pv
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      512m
Access Modes:  RWX
VolumeMode:    Filesystem
Used By:       app
Events:        <none>
```
Also, look at the StorageClass description. Verify that, `IsDefaultClass:  Yes` .

```shell
pradeep@learnk8s$ kubectl describe sc
Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"labels":{"addonmanager.kubernetes.io/mode":"EnsureExists"},"name":"standard"},"provisioner":"k8s.io/minikube-hostpath"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           k8s.io/minikube-hostpath
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```
Now delete the pod.

```shell
pradeep@learnk8s$ kubectl delete pod app
pod "app" deleted
```
After deleting the pod, check if the data is persistent or not. You should still be able to view the contents of the `hello.txt` file on the `k8s-m02` node, even after deleting the Pod, confirming that the storage is persistent now.

```shell
pradeep@learnk8s$ minikube ssh -n k8s-m02 -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ cd /data/config/
$ cat hello.txt
Testing PV, PVC, and SC in K8S!
$ exit
logout
```



### Dynamic Binding

Define a new storage class named `pradeep-sc-demo` in the `kube-system` namespace  using a sample YAML file.

```yaml
pradeep@learnk8s$ cat sc.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  namespace: kube-system
  name: pradeep-sc-demo
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "false"
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
provisioner: k8s.io/minikube-hostpath
```
Create the storage class from this YAML file  and verify that it was created.
```shell
pradeep@learnk8s$ kubectl create -f sc.yaml
storageclass.storage.k8s.io/pradeep-sc-demo created
```
Now you should see two storageclasses, both with same PROVISIONER details. Also, not the `default` value next to the `standard` class, which is not present for the newly defined storage class.

```shell
pradeep@learnk8s$ kubectl get storageclass
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
pradeep-sc-demo      k8s.io/minikube-hostpath   Delete          Immediate           false                  47s
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  12d
```

Let us modify the PVC definition file to specify this `pradeep-sc-demo` StorageClass, instead of the `standard`.
First, let us delete the existing PVC.

```shell
pradeep@learnk8s$ kubectl delete pvc pvc
persistentvolumeclaim "pvc" deleted
```
Modify the PVC definition, to include the new storage class.

```yaml
pradeep@learnk8s$ cat pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc
spec:
  storageClassName: pradeep-sc-demo
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 256m
```
Create the PVC from the modified definition file.

```shell
pradeep@learnk8s$ kubectl create -f pvc.yaml
persistentvolumeclaim/pvc created
```
Verify the Status of the new PVC.
```shell
pradeep@learnk8s$ kubectl get pvc
NAME   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS      AGE
pvc    Pending                                      pradeep-sc-demo   64s
```
It is still showing `Pending`, to find out why let us describe it.

```shell
pradeep@learnk8s$ kubectl describe pvc
Name:          pvc
Namespace:     default
StorageClass:  pradeep-sc-demo
Status:        Pending
Volume:
Labels:        <none>
Annotations:   volume.beta.kubernetes.io/storage-provisioner: k8s.io/minikube-hostpath
               volume.kubernetes.io/storage-provisioner: k8s.io/minikube-hostpath
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:
Access Modes:
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age               From                         Message
  ----    ------                ----              ----                         -------
  Normal  ExternalProvisioning  2s (x8 over 76s)  persistentvolume-controller  waiting for a volume to be created, either by external provisioner "k8s.io/minikube-hostpath" or manually created by system administrator
```

We can see that the PVC is waiting for a matching volume to be created.

Our existing PV is using the `standard` storageclass which is not matching with this PVC definition. So let us delete the existing PV and modify it to specify the new SC.
```shell
pradeep@learnk8s$ kubectl delete pv pv
persistentvolume "pv" deleted
```
Here is the modified PV definition.

```yaml
pradeep@learnk8s$ cat pv.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv
spec:
  storageClassName: pradeep-sc-demo
  capacity:
    storage: 512m
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /data/config
```
```shell
pradeep@learnk8s$ kubectl get pv,pvc
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS      REASON   AGE
persistentvolume/pv   512m       RWX            Retain           Bound    default/pvc   pradeep-sc-demo            15s

NAME                        STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistentvolumeclaim/pvc   Bound    pv       512m       RWX            pradeep-sc-demo   6m29s
```
Now that the storageclass is matching, both the PV and PVC are in `Bound` State.

Create the `app` pod again using the same manifest file that we used earlier.
```yaml
pradeep@learnk8s$ cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - image: nginx
    name: app
    volumeMounts:
    - mountPath: "/data/app/config"
      name: configpvc
  volumes:
  - name: configpvc
    persistentVolumeClaim:
      claimName: pvc
  restartPolicy: Never
```
```shell
pradeep@learnk8s$ kubectl create -f pod.yaml
pod/app created
```
```shell
pradeep@learnk8s$ kubectl get pods app -o wide
NAME   READY   STATUS    RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
app    1/1     Running   0          2m19s   10.244.1.6   k8s-m02   <none>           <none>
```
Shell into the Pod and create a file in the mounted directory.
```shell
pradeep@learnk8s$ kubectl exec app -it -- /bin/sh
# cd /data/app/config
# ls -l
total 4
-rw-r--r-- 1 root root 32 Feb 28 03:58 hello.txt
# touch HiAgain.txt
# echo "Hello again from K8s PV,PVC, and SC!" > HiAgain.txt
# cat HiAgain.txt
Hello again from K8s PV,PVC, and SC!
# exit
```
Becuase of persistent nature, we still see the old `hello.txt` file in this new container. 
We have created another file called `HiAgain.txt` with some sample text.

```shell
pradeep@learnk8s$ kubectl get pv,pvc,sc
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS      REASON   AGE
persistentvolume/pv   512m       RWX            Retain           Bound    default/pvc   pradeep-sc-demo            6m43s

NAME                        STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistentvolumeclaim/pvc   Bound    pv       512m       RWX            pradeep-sc-demo   12m

NAME                                             PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/pradeep-sc-demo      k8s.io/minikube-hostpath   Delete          Immediate           false                  18m
storageclass.storage.k8s.io/standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  12d
```

Let us delete the Pod again and verify the files.
```shell
pradeep@learnk8s$ kubectl delete pod app
pod "app" deleted
```
```shell
pradeep@learnk8s$ minikube ssh -n k8s-m02 -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ cd /data/config/
$ ls
HiAgain.txt  hello.txt
$ ls -la
total 16
drwxr-xr-x 2 root root 4096 Feb 28 04:50 .
drwxr-xr-x 3 root root 4096 Feb 28 03:55 ..
-rw-r--r-- 1 root root   37 Feb 28 04:50 HiAgain.txt
-rw-r--r-- 1 root root   32 Feb 28 03:58 hello.txt
$ cat HiAgain.txt
Hello again from K8s PV,PVC, and SC!
$ cat hello.txt
Testing PV, PVC, and SC in K8S!
$ exit
logout
```
This confirms that, the storage is persistent again (even after Pod deletion).
