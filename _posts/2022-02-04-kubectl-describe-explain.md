---
layout: single
title:  "Kubectl Describe and Explain"
date:   2022-02-04 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
show_date: true
header:
  teaser: /assets/images/kubernetes.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  bio      : "My awesome biography constrained to a sentence or two goes here."
  location : "Somewhere, USA"
sidebar:
  - title: "Blog"
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubectl describe and explain


### Describe Resources

```shell
pradeep@learnk8s$ kubectl describe pods nginx
Name:         nginx
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.28
Start Time:   Fri, 04 Feb 2022 07:13:50 +0530
Labels:       new-label=awesome
              run=nginx
Annotations:  <none>
Status:       Running
IP:           10.244.1.6
IPs:
  IP:  10.244.1.6
Containers:
  nginx:
    Container ID:   docker://32bfdccc6c984f51a4c4092e4027ff7f1e53dd518938196c95d5427a44a90e40
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 04 Feb 2022 07:13:56 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9sstv (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-9sstv:
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
  Normal  Scheduled  18m   default-scheduler  Successfully assigned default/nginx to k8s-m02
  Normal  Pulling    18m   kubelet            Pulling image "nginx"
  Normal  Pulled     18m   kubelet            Successfully pulled image "nginx" in 4.864287771s
  Normal  Created    18m   kubelet            Created container nginx
  Normal  Started    18m   kubelet            Started container nginx
```

### Kubectl Explain!
```shell
pradeep@learnk8s$ kubectl explain pod.spec
KIND:     Pod
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

     PodSpec is a description of a pod.

FIELDS:
   <SNIP>
   containers	<[]Object> -required-
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.
   <SNIP>
    nodeName	<string>
     NodeName is a request to schedule this pod onto a specific node. If it is
     non-empty, the scheduler simply schedules this pod onto that node, assuming
     that it fits resource requirements.
   <SNIP>  
```

