---
layout: single
title:  "Kubernetes DaemonSets"
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
# Kubernetes Daemon Sets


### Daemon Sets
A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected.

```shell
pradeep@learnk8s$ kubectl get daemonsets.apps -A
NAMESPACE     NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kindnet      2         2         2       2            2           <none>                   5d12h
kube-system   kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   5d12h
```
As seen above, there are two daemonsets in the `kube-system` namespace. One is named `kindnet` and the other is `kube-proxy`. If you look at the `DESIRED` column, both of them have a value of `2` which is equal to the number of nodes in this (minikube) cluster.

Let's look at those pods now.
```shell
pradeep@learnk8s$ kubectl describe daemonsets.apps kindnet -n kube-system
Name:           kindnet
Selector:       app=kindnet
Node-Selector:  <none>
Labels:         app=kindnet
                k8s-app=kindnet
                tier=node
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 2
Current Number of Nodes Scheduled: 2
Number of Nodes Scheduled with Up-to-date Pods: 2
Number of Nodes Scheduled with Available Pods: 2
Number of Nodes Misscheduled: 0
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           app=kindnet
                    k8s-app=kindnet
                    tier=node
  Service Account:  kindnet
  Containers:
   kindnet-cni:
    Image:      kindest/kindnetd:v20210326-1e038dc5
    Port:       <none>
    Host Port:  <none>
    Limits:
      cpu:     100m
      memory:  50Mi
    Requests:
      cpu:     100m
      memory:  50Mi
    Environment:
      HOST_IP:      (v1:status.hostIP)
      POD_IP:       (v1:status.podIP)
      POD_SUBNET:  10.244.0.0/16
    Mounts:
      /etc/cni/net.d from cni-cfg (rw)
      /lib/modules from lib-modules (ro)
      /run/xtables.lock from xtables-lock (rw)
  Volumes:
   cni-cfg:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/cni/net.mk
    HostPathType:  DirectoryOrCreate
   xtables-lock:
    Type:          HostPath (bare host directory volume)
    Path:          /run/xtables.lock
    HostPathType:  FileOrCreate
   lib-modules:
    Type:          HostPath (bare host directory volume)
    Path:          /lib/modules
    HostPathType:
Events:            <none>
```
```shell
pradeep@learnk8s$ kubectl describe daemonsets.apps kube-proxy -n kube-system
Name:           kube-proxy
Selector:       k8s-app=kube-proxy
Node-Selector:  kubernetes.io/os=linux
Labels:         k8s-app=kube-proxy
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 2
Current Number of Nodes Scheduled: 2
Number of Nodes Scheduled with Up-to-date Pods: 2
Number of Nodes Scheduled with Available Pods: 2
Number of Nodes Misscheduled: 0
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           k8s-app=kube-proxy
  Service Account:  kube-proxy
  Containers:
   kube-proxy:
    Image:      k8s.gcr.io/kube-proxy:v1.23.1
    Port:       <none>
    Host Port:  <none>
    Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/config.conf
      --hostname-override=$(NODE_NAME)
    Environment:
      NODE_NAME:   (v1:spec.nodeName)
    Mounts:
      /lib/modules from lib-modules (ro)
      /run/xtables.lock from xtables-lock (rw)
      /var/lib/kube-proxy from kube-proxy (rw)
  Volumes:
   kube-proxy:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-proxy
    Optional:  false
   xtables-lock:
    Type:          HostPath (bare host directory volume)
    Path:          /run/xtables.lock
    HostPathType:  FileOrCreate
   lib-modules:
    Type:               HostPath (bare host directory volume)
    Path:               /lib/modules
    HostPathType:
  Priority Class Name:  system-node-critical
Events:                 <none>
```
As seen in this output, there are two pods, one pod on each node. The pod names follow the convention of `<daemonsetname>-<randomidentifier>`.
```shell
pradeep@learnk8s$ kubectl get pods -A -o wide -o wide| grep kube-proxy
kube-system   kube-proxy-fszkr              1/1     Running   1 (2d ago)     5d12h   192.168.177.28   k8s-m02   <none>           <none>
kube-system   kube-proxy-m747v              1/1     Running   1              5d12h   192.168.177.27   k8s       <none>           <none>
```
Similarly, the pods from the other daemonset—kindnet.

```shell
pradeep@learnk8s$ kubectl get pods -A -o wide -o wide| grep kindnet
kube-system   kindnet-jpxdd                 1/1     Running   1              5d12h   192.168.177.27   k8s       <none>           <none>
kube-system   kindnet-p77mb                 1/1     Running   1 (2d ago)     5d12h   192.168.177.28   k8s-m02   <none>           <none>
```
A simple way to create a DaemonSet is to first generate a YAML file for a Deployment and remove the replicas, strategy fields. Change the kind from Deployment to DaemonSet.

Here is an example (modfied from the demodeployment.yaml):

```yaml
pradeep@learnk8s$ cat demodaemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: demo-ds
  name: demo-ds
spec:
  selector:
    matchLabels:
      app: demo-ds
  template:
    metadata:
      labels:
        app: demo-ds
    spec:
      containers:
      - image: nginx
        name: nginx-ds
        resources: {}

```

```shell
pradeep@learnk8s$ kubectl create -f demodaemonset.yaml
daemonset.apps/demo-ds created
```
```shell
pradeep@learnk8s$ kubectl get ds
NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
demo-ds   2         2         0       2            0           <none>          6s
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep demo-ds
demo-ds-gtcf7           1/1     Running   0          28s    10.244.1.5    k8s-m02   <none>           <none>
demo-ds-kkw4g           1/1     Running   0          28s    10.244.0.11   k8s       <none>           <none>
```

