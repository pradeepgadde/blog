---
layout: single
title:  "Kubernetes Scheduling"
date:   2022-02-07 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/kubernetes.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Scheduling


## Scheduling 

### Manual Scheduling
Modify the Pod definition file, to includ the `nodeName` in the spec.
```shell
pradeep@learnk8s$ cat manual-sched-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-manual
spec:
  nodeName: k8s
  containers:
  - image: nginx
    name: nginx
```
```shell
pradeep@learnk8s$ kubectl create -f manual-sched-pod.yaml
pod/nginx-manual created
```
```shell
pradeep@learnk8s$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
demo-6c54f77c95-6g7zq   1/1     Running   0          20m
demo-6c54f77c95-sb4c9   1/1     Running   0          20m
demo-6c54f77c95-w2bsw   1/1     Running   0          20m
nginx                   1/1     Running   0          25m
nginx-manual            1/1     Running   0          48s
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep manual
nginx-manual            1/1     Running   0          2m9s   10.244.0.5   k8s       <none>           <none>
```

### Node Selector
You can constrain a Pod so that it can only run on particular set of Node(s). 
`nodeSelector` provides a very simple way to constrain pods to nodes with particular labels.
First, let’s label one of the nodes with `disktype=ssd`.

```shell
pradeep@learnk8s$ kubectl label nodes k8s-m02 disktype=ssd
node/k8s-m02 labeled
```
```shell
pradeep@learnk8s$ kubectl get nodes --show-labels
NAME      STATUS   ROLES                  AGE     VERSION   LABELS
k8s       Ready    control-plane,master   3d11h   v1.23.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s,kubernetes.io/os=linux,minikube.k8s.io/commit=3e64b11ed75e56e4898ea85f96b2e4af0301f43d,minikube.k8s.io/name=k8s,minikube.k8s.io/updated_at=2022_02_07T17_03_56_0700,minikube.k8s.io/version=v1.25.1,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
k8s-m02   Ready    <none>                 4m3s    v1.23.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-m02,kubernetes.io/os=linux
pradeep@learnk8s$
```
Now, let's create a pod with `nodeSelector` spec set to the label that we assigned, so that the pod gets scheduled on this node.
```yaml
pradeep@learnk8s$ cat pod-node-selector.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-node-selector
  labels:
    env: test
spec:
  containers:
  - name: nginx-node-selector
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```
Finally, verify the node on which this pod is running.
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep selector
nginx-node-selector     1/1     Running   0          17s    10.244.1.2    k8s-m02   <none>           <none>
```

As expected, this new pod is running on the node(`k8s-m02`) with the label specified by the `nodeSelector` pod spec.

If you describe this pod, you will see the configured Node-Selectors.

```shell
pradeep@learnk8s$ kubectl describe pod nginx-node-selector | grep Node-Selectors
Node-Selectors:              disktype=ssd
```

### Node Affinity
There are currently two types of node affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and `preferredDuringSchedulingIgnoredDuringExecution`.

You can think of them as "hard" and "soft" respectively, in the sense that the former specifies rules that must be met for a pod to be scheduled onto a node (similar to `nodeSelector` but using a more expressive syntax), while the latter specifies preferences that the scheduler will try to enforce but will not guarantee. 

The `IgnoredDuringExecution` part of the names means that, similar to how `nodeSelector` works, if labels on a node change at runtime such that the affinity rules on a pod are no longer met, the pod continues to run on the node. 

```yaml
pradeep@learnk8s$ cat pod-node-affinity.yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd

  containers:
  - name: with-node-affinity
    image: nginx
```
```shell
pradeep@learnk8s$ kubectl create -f pod-node-affinity.yaml
pod/with-node-affinity created
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep affinity
with-node-affinity      1/1     Running   0          9s     10.244.1.3    k8s-m02   <none>           <none>
```

### Taints and Tolerations
Taints  allow a node to repel a set of pods.

Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

```shell
pradeep@learnk8s$ kubectl describe nodes | grep Taint
Taints:             <none>
Taints:             <none>
```
```shell
pradeep@learnk8s$ kubectl taint nodes k8s key1=value1:NoSchedule

node/k8s tainted
```
```shell
pradeep@learnk8s$ kubectl describe nodes | grep Taint
Taints:             key1=value1:NoSchedule
Taints:             <none>
```
```shell
pradeep@learnk8s$ kubectl taint nodes k8s-m02 key2=value2:NoSchedule

node/k8s-m02 tainted
```
```shell
pradeep@learnk8s$ kubectl describe nodes | grep Taint
Taints:             key1=value1:NoSchedule
Taints:             key2=value2:NoSchedule
```
```yaml
pradeep@learnk8s$ cat pod-toleration.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-taint-demo
  labels:
    env: test
spec:
  containers:
  - name: nginx-taint-demo
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
```
```shell
pradeep@learnk8s$ kubectl create -f pod-toleration.yaml
pod/nginx-taint-demo created
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep taint-demo
nginx-taint-demo        1/1     Running   0          9m3s    10.244.0.15   k8s       <none>           <none>
```
```yaml
pradeep@learnk8s$ cat pod-toleration-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-taint-demo-2
  labels:
    env: test
spec:
  containers:
  - name: nginx-taint-demo-2
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "key2"
    operator: "Equal"
    value: "value2"
    effect: "NoSchedule"
```
```shell
pradeep@learnk8s$ kubectl create -f pod-toleration-2.yaml
pod/nginx-taint-demo-2 created
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep taint-demo
nginx-taint-demo        1/1     Running   0          11m     10.244.0.15   k8s       <none>           <none>
nginx-taint-demo-2      1/1     Running   0          9m35s   10.244.2.5    k8s-m02   <none>    <none>
```
```yaml
pradeep@learnk8s$ cat pod-no-toleration.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-no-tolerate
  labels:
    env: test
spec:
  containers:
  - name: nginx-no-tolerate
    image: nginx
    imagePullPolicy: IfNotPresent
```
```shell
pradeep@learnk8s$ kubectl create -f pod-no-toleration.yaml
pod/nginx-no-tolerate created
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep no-tolerate
nginx-no-tolerate       0/1     Pending   0          10m     <none>        <none>    <none>          <none>
```
The Pod got created but is in `Pending` state, it is not `Running` yet. Let's find out why?!
```shell
pradeep@learnk8s$ kubectl describe pods nginx-no-tolerate
Name:         nginx-no-tolerate
Namespace:    default
Priority:     0
Node:         <none>
Labels:       env=test
Annotations:  <none>
Status:       Pending
IP:
IPs:          <none>
Containers:
  nginx-no-tolerate:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-6mz6d (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-6mz6d:
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
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  43s (x11 over 11m)  default-scheduler  0/2 nodes are available: 1 node(s) had taint {key1: value1}, that the pod didn't tolerate, 1 node(s) had taint {key2: value2}, that the pod didn't tolerate.
  
```
Look at the Reason: `FailedScheduling`, none of the nodes (0/2) are available because both have some taint that our pod couldn't tolerate!

Now, let's delete the Taint on one of the nodes and try creating the Pod again.

To untaint a node, just add a `-` at the end of the taint that you plan to remove.

```shell
pradeep@learnk8s$ kubectl taint node k8s  key1=value1:NoSchedule-
node/k8s untainted
```
```shell
pradeep@learnk8s$ kubectl describe nodes | grep Taint
Taints:             <none>
Taints:             key2=value2:NoSchedule
```
Now the `nginx-no-tolerate` pod changes to `Running` state and is scheduled on the `k8s` node which does not have any Taints at the moment.

```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep no-tolerate
nginx-no-tolerate       1/1     Running   0          16h   10.244.0.16   k8s       <none>           <none>
```

