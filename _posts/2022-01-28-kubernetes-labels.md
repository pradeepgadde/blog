---
layout: single
title:  "Kubernetes Labels"
date:   2022-01-28 10:55:04 +0530
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
    image: "/assets/images/automation.png"
    image_alt: "image"
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubernetes Labels


### Show Labels
```shell
pradeep@learnk8s$ kubectl get pods --show-labels
NAME                    READY   STATUS    RESTARTS   AGE     LABELS
demo-6c54f77c95-6g7zq   1/1     Running   0          3m32s   app=demo,pod-template-hash=6c54f77c95
demo-6c54f77c95-sb4c9   1/1     Running   0          3m32s   app=demo,pod-template-hash=6c54f77c95
demo-6c54f77c95-w2bsw   1/1     Running   0          3m32s   app=demo,pod-template-hash=6c54f77c95
nginx                   1/1     Running   0          8m6s    run=nginx
```
```shell
pradeep@learnk8s$ kubectl get nodes --show-labels
NAME      STATUS   ROLES                  AGE   VERSION   LABELS
k8s       Ready    control-plane,master   92m   v1.23.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s,kubernetes.io/os=linux,minikube.k8s.io/commit=3e64b11ed75e56e4898ea85f96b2e4af0301f43d,minikube.k8s.io/name=k8s,minikube.k8s.io/updated_at=2022_02_04T05_49_32_0700,minikube.k8s.io/version=v1.25.1,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
k8s-m02   Ready    <none>                 91m   v1.23.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-m02,kubernetes.io/os=linux
```

### Apply Labels
```shell
pradeep@learnk8s$ kubectl label pods nginx new-label=awesome
pod/nginx labeled
```
```shell
pradeep@learnk8s$ kubectl get pods --show-labels
NAME                    READY   STATUS    RESTARTS   AGE     LABELS
demo-6c54f77c95-6g7zq   1/1     Running   0          8m47s   app=demo,pod-template-hash=6c54f77c95
demo-6c54f77c95-sb4c9   1/1     Running   0          8m47s   app=demo,pod-template-hash=6c54f77c95
demo-6c54f77c95-w2bsw   1/1     Running   0          8m47s   app=demo,pod-template-hash=6c54f77c95
nginx                   1/1     Running   0          13m     new-label=awesome,run=nginx
```

### Label Selector
```shell
pradeep@learnk8s$ kubectl get pods --selector=app=demo
NAME                    READY   STATUS    RESTARTS   AGE
demo-6c54f77c95-6g7zq   1/1     Running   0          10m
demo-6c54f77c95-sb4c9   1/1     Running   0          10m
demo-6c54f77c95-w2bsw   1/1     Running   0          10m
```
```shell
pradeep@learnk8s$ kubectl get pods --selector=run=nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          15m
```
```shell
pradeep@learnk8s$ kubectl get pods --selector=new-label=awesome
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          15m
```

