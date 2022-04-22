---
layout: single
title:  "Kubernetes Init Containers"
date:   2022-02-16 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/pod-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Init Containers


### Init Containers

Earlier, we saw that a Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.

Init containers are exactly like regular containers, except:
Init containers always run to completion.
Each init container must complete successfully before the next one starts.

If init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.

Here is an example with one InitContainer called `wait-for-service` which is basically looking for the presence of a service called `my-service`. It does this check by performing `nslookup`. Till the kebernetes service called `my-service` is present, the name resolution will not succeed and this initContainer will not be finishing. Once the service is created, the initContainer job is done and it proceeds to the main container, in this example, it is the `ubuntu` container.

```yaml
pradeep@learnk8s$ more pod-with-init-container.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-init-container
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ['sh', '-c', 'echo The ubuntu container is running! && sleep 3600']
  initContainers:
  - name: wait-for-service
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
```
```shell
pradeep@learnk8s$ kubectl create -f pod-with-init-container.yaml
pod/pod-with-init-container created
```
Once we created the pod with initContainer, we can see the state as `Init:0/1` meaning there is one init container but it is not yet ready.

```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep init
pod-with-init-container     0/1     Init:0/1           0              7s    <none>        k8s       <none>           <none>
```
Waited for some more time, still it is in `Init:0/1` state.
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep init
pod-with-init-container     0/1     Init:0/1           0                96s   10.244.0.15   k8s       <none>           <none>
```
Let us describe this pod to get additional details.

```shell
pradeep@learnk8s$ kubectl describe pods pod-with-init-container
Name:         pod-with-init-container
Namespace:    default
Priority:     0
Node:         k8s/192.168.177.29
Start Time:   Wed, 16 Feb 2022 07:48:13 +0530
Labels:       <none>
Annotations:  <none>
Status:       Pending
IP:
IPs:          <none>
Init Containers:
  wait-for-service:
    Container ID:
    Image:         busybox:1.28
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
apiVersion: v1
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7gp7r (ro)
Containers:
  ubuntu:
    Container ID:
    Image:         ubuntu
    Image ID:
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The ubuntu container is running! && sleep 3600
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7gp7r (ro)
Conditions:
  Type              Status
  Initialized       False
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-api-access-7gp7r:
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
  Normal  Scheduled  17s   default-scheduler  Successfully assigned default/pod-with-init-container to k8s
  Normal  Pulling    14s   kubelet            Pulling image "busybox:1.28"
```
As seen above,  Init Container is in Waiting state and overall Pod status is Pending.

To make the InitiContainer successful, let us create a service called `myservice`.
Here is the sample definition of the service.

```yaml
pradeep@learnk8s$ cat init-container-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```
Create the myservice from the YAML file.
```shell
pradeep@learnk8s$ kubectl create -f init-container-service.yaml
service/myservice created
```
Once the service is created, verify if there is any change in the main Pod status. It seems there is a change.

```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep init
pod-with-init-container     0/1     PodInitializing    0                108s   10.244.0.15   k8s       <none>           <none>
```
It is not yet in running state, it might take some more time. In the meanwhile, let us inspect the logs of the InitContainer that we created `wait-for-service`.

Remember the `kubectl logs` command?! and the `-c` option to specify a container name when there are multiple containers in a pod!

```shell
pradeep@learnk8s$ kubectl logs pod-with-init-container -c wait-for-service
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'myservice.default.svc.cluster.local'
waiting for myservice
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

waiting for myservice
nslookup: can't resolve 'myservice.default.svc.cluster.local'
Server:    10.96.0.10
nslookup: can't resolve 'myservice.default.svc.cluster.local'
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

<SNIP>

nslookup: can't resolve 'myservice.default.svc.cluster.local'
waiting for myservice
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      myservice.default.svc.cluster.local
Address 1: 10.111.136.80 myservice.default.svc.cluster.local
```

A lot of similar output is omitted for brevity, but we can clearly see that this container is continuosly checking for the resolution of `myservice.default.svc.cluster.local` but nslookup: can't resolve it. 

Finally, after we created the `myservice` Service, name resolution was successful ( the last line). It (`myservice.default.svc.cluster.local`) is resolved to the IP address of 10.111.136.80.

This is nothing but the ClusterIP assigned to the `myservice`.

```shell
pradeep@learnk8s$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   19h
myservice    ClusterIP   10.111.136.80   <none>        80/TCP    9m13s
```
Going back to the main Pod, we can see it is in Running State.

```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep init
pod-with-init-container     1/1     Running            0               12m   10.244.0.15   k8s       <none>           <none>
```
If we describe it one more time, Init Container is in Terminated State with Reason as Completed. And the Ubuntu container is Running.

```shell
pradeep@learnk8s$ kubectl describe pods pod-with-init-container
Name:         pod-with-init-container
Namespace:    default
Priority:     0
Node:         k8s/192.168.177.29
Start Time:   Wed, 16 Feb 2022 07:48:13 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.0.15
IPs:
  IP:  10.244.0.15
Init Containers:
  wait-for-service:
    Container ID:  docker://8b5668914aeef2c9c43eebf3ebf7b7289eff4efa96166198fc0c6db8cdfd358b
    Image:         busybox:1.28
    Image ID:      docker-pullable://busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 16 Feb 2022 07:48:52 +0530
      Finished:     Wed, 16 Feb 2022 07:49:52 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7gp7r (ro)
Containers:
  ubuntu:
    Container ID:  docker://60caee7150e608901a2fda93fdc6758c8aa5208ec69320ed8c7982d57efde9fb
    Image:         ubuntu
    Image ID:      docker-pullable://ubuntu@sha256:669e010b58baf5beb2836b253c1fd5768333f0d1dbcb834f7c07a4dc93f474be
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The ubuntu container is running! && sleep 3600
    State:          Running
      Started:      Wed, 16 Feb 2022 07:59:33 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7gp7r (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-7gp7r:
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
  Normal  Scheduled  12m   default-scheduler  Successfully assigned default/pod-with-init-container to k8s
  Normal  Pulling    12m   kubelet            Pulling image "busybox:1.28"
  Normal  Pulled     11m   kubelet            Successfully pulled image "busybox:1.28" in 35.840326348s
  Normal  Created    11m   kubelet            Created container wait-for-service
  Normal  Started    11m   kubelet            Started container wait-for-service
  Normal  Pulling    10m   kubelet            Pulling image "ubuntu"
  Normal  Pulled     50s   kubelet            Successfully pulled image "ubuntu" in 9m39.673051136s
  Normal  Created    50s   kubelet            Created container ubuntu
  Normal  Started    50s   kubelet            Started container ubuntu
```
