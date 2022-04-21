---
layout: single
title:  "Kubernetes Core Components"
date:   2022-01-28 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/calico.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubernetes Core Components


### Namespace

#### Defaults
```shell
pradeep@learnk8s$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   2m7s
kube-node-lease   Active   2m10s
kube-public       Active   2m10s
kube-system       Active   2m10s
```
As we saw at the beginning, when we setup a kubernetes cluster, all control plane and worker node compononets gets installed as pods in the system defined namespace `kube-system`.
#### Create a new namespace
```shell
pradeep@learnk8s$ kubectl create namespace prod
namespace/prod created
```
```shell
pradeep@learnk8s$ kubectl get ns
NAME              STATUS   AGE
default           Active   5m4s
prod              Active   33s
kube-node-lease   Active   5m7s
kube-public       Active   5m7s
kube-system       Active   5m7s

```
```shell
pradeep@learnk8s$ kubectl run nginx --image=nginx -n prod
pod/nginx created
```
```shell
pradeep@learnk8s$ kubectl get pods -n prod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          79s
```

### Life cycle of a Pod

1. kubectl client sends new pod request
2. Scheduler assigns new pod to worker node
3. API server instructs kubelet  to provision new Pod
4. kubelet requests that Docker create container
5. Docker retrieves nginx image
6. Docker creates container
7. Kubelet passes container details to K8s API
8. Pod configs stored in etcd
9. Network configs are created on kube-proxy

### Create a Deployment
```shell
pradeep@learnk8s$ kubectl create deployment demo --image=k8s.gcr.io/echoserver:1.4
deployment.apps/demo created
```
```shell
pradeep@learnk8s$ kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
demo   1/1     1            1           36s
```

```shell
pradeep@learnk8s$ kubectl get replicasets
NAME              DESIRED   CURRENT   READY   AGE
demo-75f9c7566f   1         1         1       31s
```
```shell
pradeep@learnk8s$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
demo-75f9c7566f-nmjw2   1/1     Running   0          26s
```

### View Cluster Events
```shell
pradeep@learnk8s$ kubectl get events
LAST SEEN   TYPE     REASON                    OBJECT                       MESSAGE
5m15s       Normal   Scheduled                 pod/demo-75f9c7566f-nmjw2    Successfully assigned default/demo-75f9c7566f-nmjw2 to k8s-m02
5m14s       Normal   Pulling                   pod/demo-75f9c7566f-nmjw2    Pulling image "k8s.gcr.io/echoserver:1.4"
4m57s       Normal   Pulled                    pod/demo-75f9c7566f-nmjw2    Successfully pulled image "k8s.gcr.io/echoserver:1.4" in 16.898815167s
4m57s       Normal   Created                   pod/demo-75f9c7566f-nmjw2    Created container echoserver
4m57s       Normal   Started                   pod/demo-75f9c7566f-nmjw2    Started container echoserver
5m15s       Normal   SuccessfulCreate          replicaset/demo-75f9c7566f   Created pod: demo-75f9c7566f-nmjw2
5m15s       Normal   ScalingReplicaSet         deployment/demo              Scaled up replica set demo-75f9c7566f to 1
14m         Normal   Starting                  node/k8s-m02                 Starting kubelet.
14m         Normal   NodeHasSufficientMemory   node/k8s-m02                 Node k8s-m02 status is now: NodeHasSufficientMemory
14m         Normal   NodeHasNoDiskPressure     node/k8s-m02                 Node k8s-m02 status is now: NodeHasNoDiskPressure
14m         Normal   NodeHasSufficientPID      node/k8s-m02                 Node k8s-m02 status is now: NodeHasSufficientPID
14m         Normal   NodeAllocatableEnforced   node/k8s-m02                 Updated Node Allocatable limit across pods
14m         Normal   Starting                  node/k8s-m02
14m         Normal   RegisteredNode            node/k8s-m02                 Node k8s-m02 event: Registered Node k8s-m02 in Controller
13m         Normal   NodeReady                 node/k8s-m02                 Node k8s-m02 status is now: NodeReady
15m         Normal   NodeHasSufficientMemory   node/k8s                     Node k8s status is now: NodeHasSufficientMemory
15m         Normal   NodeHasNoDiskPressure     node/k8s                     Node k8s status is now: NodeHasNoDiskPressure
15m         Normal   NodeHasSufficientPID      node/k8s                     Node k8s status is now: NodeHasSufficientPID
14m         Normal   Starting                  node/k8s                     Starting kubelet.
14m         Normal   NodeHasSufficientMemory   node/k8s                     Node k8s status is now: NodeHasSufficientMemory
14m         Normal   NodeHasNoDiskPressure     node/k8s                     Node k8s status is now: NodeHasNoDiskPressure
14m         Normal   NodeHasSufficientPID      node/k8s                     Node k8s status is now: NodeHasSufficientPID
14m         Normal   NodeAllocatableEnforced   node/k8s                     Updated Node Allocatable limit across pods
14m         Normal   RegisteredNode            node/k8s                     Node k8s event: Registered Node k8s in Controller
14m         Normal   Starting                  node/k8s
14m         Normal   NodeReady                 node/k8s                     Node k8s status is now: NodeReady
```

### Scaling a Deployment
```shell
pradeep@learnk8s$ kubectl scale deployment demo --replicas=3
deployment.apps/demo scaled
```
```shell
pradeep@learnk8s$ kubectl get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
demo   3/3     3            3           11m
```
```shell
pradeep@learnk8s$ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
demo-75f9c7566f   3         3         3       12m
```
```shell
pradeep@learnk8s$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
demo-75f9c7566f-fct76   1/1     Running   0          2m2s
demo-75f9c7566f-kz9kb   1/1     Running   0          2m3s
demo-75f9c7566f-nmjw2   1/1     Running   0          12m
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
demo-75f9c7566f-fct76   1/1     Running   0          2m10s   10.244.1.4   k8s-m02   <none>           <none>
demo-75f9c7566f-kz9kb   1/1     Running   0          2m11s   10.244.0.3   k8s       <none>           <none>
demo-75f9c7566f-nmjw2   1/1     Running   0          12m     10.244.1.3   k8s-m02   <none>           <none>
```

### Minikube SSH and Access Pods

```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.1.4:8080
CLIENT VALUES:
client_address=192.168.177.27
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://10.244.1.4:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=10.244.1.4:8080
user-agent=curl/7.78.0
BODY:
-no body in request-$
```
```shell
$ curl 10.244.0.3:8080
CLIENT VALUES:
client_address=10.244.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://10.244.0.3:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=10.244.0.3:8080
user-agent=curl/7.78.0
BODY:
-no body in request-$
```
```shell
$ curl 10.244.1.3:8080
CLIENT VALUES:
client_address=192.168.177.27
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://10.244.1.3:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=10.244.1.3:8080
user-agent=curl/7.78.0
BODY:
-no body in request-$
```

