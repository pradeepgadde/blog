---
layout: single
title:  "Kubernetes Persistent Volumes and Claims"
date:   2022-02-27 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/pvc-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Persistent Volumes and Claimes


### PersistentVolume

Create a directory called `/mnt/data` on the `k8s` node. Within that directory, create `index.html` file with some text.

```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ sudo mkdir /mnt/data
$ sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/indext.html"
$ cat /mnt/data/indext.html
Hello from Kubernetes storage
$ exit
logout
```



Now create a `PersistentVolume` named `my-pv-volume` of 1Gi using the `hostPath` pointing to the newly created directory. There are three types of `accessModes`, for this example use, `RWO`.

| RWO  | ReadWriteOnce |
| ---- | ------------- |
| RWX  | ReadWriteMany |
| ---- | ------------- |
| ROX  | ReadyOnlyMany |

  

```yaml
pradeep@learnk8s$ cat persistent-volume-demo.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Create the PV using this YAML file.

```shell
pradeep@learnk8s$ kubectl create -f persistent-volume-demo.yaml
persistentvolume/my-pv-volume created
```

Verify the available PVs.

```shell
pradeep@learnk8s$ kubectl get pv
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
my-pv-volume   1Gi        RWO            Retain           Available           manual                  4s
```

Describe the PersistentVolume for addtional details.

```shell
pradeep@learnk8s$ kubectl describe pv
Name:            my-pv-volume
Labels:          type=local
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    manual
Status:          Available
Claim:
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /mnt/data
    HostPathType:
Events:            <none>
```

[**:top:**](#toc)

### PersistentVolumeClaim

```yaml
pradeep@learnk8s$ cat persistent-volume-claim-demo.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 300Mi
```

```shell
pradeep@learnk8s$ kubectl create -f persistent-volume-claim-demo.yaml
persistentvolumeclaim/my-pv-claim created
```

```shell
pradeep@learnk8s$ kubectl get pvc
NAME          STATUS   VOLUME         CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pv-claim   Bound    my-pv-volume   1Gi        RWO            manual         40s
```

```shell
pradeep@learnk8s$ kubectl describe pvc
Name:          my-pv-claim
Namespace:     default
StorageClass:  manual
Status:        Bound
Volume:        my-pv-volume
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       <none>
Events:        <none>
```

```shell
pradeep@learnk8s$ kubectl get pv
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
my-pv-volume   1Gi        RWO            Retain           Bound    default/my-pv-claim   manual                  12m
```

```shell
pradeep@learnk8s$ kubectl describe pv
Name:            my-pv-volume
Labels:          type=local
Annotations:     pv.kubernetes.io/bound-by-controller: yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    manual
Status:          Bound
Claim:           default/my-pv-claim
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /mnt/data
    HostPathType:
Events:            <none>
```

The next step is to create a Pod that uses this PersistentVolumeClaim as a volume.

Here is the configuration file for the Pod:

```yaml
pradeep@learnk8s$ cat my-pv-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pv-pod
spec:
  volumes:
    - name: my-pv-storage
      persistentVolumeClaim:
        claimName: my-pv-claim
  containers:
    - name: my-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: my-pv-storage
```
```shell
pradeep@learnk8s$ kubectl create -f my-pv-pod.yaml
pod/my-pv-pod created
```
The scheduler has placed this Pod on `k8s-m02` node.
```shell
pradeep@learnk8s$ kubectl get pods my-pv-pod -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE      NOMINATED NODE   READINESS GATES
my-pv-pod   1/1     Running   0          10s   10.244.1.2   k8s-m02   <none>           <none>
```
Let us describe the pod and look at the `Volumes` section.

```shell
pradeep@learnk8s$ kubectl describe pods my-pv-pod
Name:         my-pv-pod
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Sun, 27 Feb 2022 22:35:45 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.1.2
IPs:
  IP:  10.244.1.2
Containers:
  my-pv-container:
    Container ID:   docker://9dc92c5c1037ad4d534fe6440275f62a68f521e3bd22fe47b57c06a8b3414e24
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 27 Feb 2022 22:35:49 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from my-pv-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xk6kh (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  my-pv-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  my-pv-claim
    ReadOnly:   false
  kube-api-access-xk6kh:
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
  Normal  Scheduled  19s   default-scheduler  Successfully assigned default/my-pv-pod to k8s-m02
  Normal  Pulling    18s   kubelet            Pulling image "nginx"
  Normal  Pulled     15s   kubelet            Successfully pulled image "nginx" in 2.858388422s
  Normal  Created    15s   kubelet            Created container my-pv-container
  Normal  Started    15s   kubelet            Started container my-pv-container
```
It looks like no issues, but let us verify the `nginx` application by logging into the pod and issuing the `curl localhost` command.

```shell
pradeep@learnk8s$ kubectl exec -it my-pv-pod -- /bin/sh
# curl localhost
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.21.6</center>
</body>
</html>
# exit
```

Hmm! This seems to be not working. It is forbidden. But this is expected right, we did not have the volume (hostPath) on this `k8s-m02` node. We created it on `k8s` node only.

To solve this, we can add a `nodeName` spec to the Pod definition.
Delete this pod and after adding the `nodeName` recreate the Pod.

```shell
pradeep@learnk8s$ kubectl delete pod my-pv-pod
pod "my-pv-pod" deleted
```
```shell
pradeep@learnk8s$ vi my-pv-pod.yaml
```
```yaml
pradeep@learnk8s$ cat my-pv-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pv-pod
spec:
  nodeName: k8s
  volumes:
    - name: my-pv-storage
      persistentVolumeClaim:
        claimName: my-pv-claim
  containers:
    - name: my-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: my-pv-storage

```
```shell
pradeep@learnk8s$ kubectl create -f my-pv-pod.yaml
pod/my-pv-pod created
```
:warning: There seems to be some issue with this. Even after scheduling this pod on the node `k8s`, nginx container is reporting Forbidden. There are couple of issues already reported, but could not find any resolution to this problem yet.

https://github.com/kubernetes/website/issues/9523
