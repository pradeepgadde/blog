---
layout: single
title:  "Kubernetes Multiple Schedulers"
date:   2022-02-11 10:55:04 +0530
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
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Multiple Schedulers


### Multiple Schedulers

Kubernetes ships with a default scheduler (`kube-scheduler`) that we discussed earlier. If the default scheduler does not suit your needs you can implement your own scheduler. Moreover, you can even run multiple schedulers simultaneously alongside the default scheduler and instruct Kubernetes what scheduler to use for each of your pods.

We can use the staticPod manifest file located at /etc/kubernetes/manifests/kube-scheduler.yaml to create our own scheduler.

First, let us take a look at the definition of the default scheduler.

```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ cat /etc/kubernetes/manifests/
etcd.yaml                     kube-apiserver.yaml           kube-controller-manager.yaml  kube-scheduler.yaml

$ sudo cat /etc/kubernetes/manifests/kube-scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    image: k8s.gcr.io/kube-scheduler:v1.23.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}
$
```

The default scheduler listens on port `10259`, so for our custom scheduler, we have to choose another port, for example `10282`.

Here is the definition of the new scheduler called 	`my-scheduler`, which is listening on the secure-port `10282`.

```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ sudo cat /etc/kubernetes/manifests/my-scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: my-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    - --secure-port=10282
    image: k8s.gcr.io/kube-scheduler:v1.23.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10282
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: my-scheduler
    resources:
      requests:
        cpu: 100m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10282
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}
$
```
Verify that both schedulers `kube-scheduler-k8s` and `my-scheduler-k8s` are in running state.
```shell
pradeep@learnk8s$ kubectl get pods  -n kube-system
NAME                          READY   STATUS    RESTARTS        AGE
coredns-64897985d-r9tzv       1/1     Running   8 (3h2m ago)    6d3h
etcd-k8s                      1/1     Running   2 (3h2m ago)    6d3h
kindnet-jpxdd                 1/1     Running   5 (9h ago)      6d3h
kindnet-p77mb                 1/1     Running   50 (3h6m ago)   6d3h
kube-apiserver-k8s            1/1     Running   3 (9h ago)      6d3h
kube-controller-manager-k8s   1/1     Running   4 (3h1m ago)    6d3h
kube-proxy-fszkr              1/1     Running   1 (2d16h ago)   6d3h
kube-proxy-m747v              1/1     Running   1               6d3h
kube-scheduler-k8s            1/1     Running   0               14m
my-scheduler-k8s              1/1     Running   0               2m14s
storage-provisioner           1/1     Running   49 (149m ago)   6d3h
```

Now, it is time to deploy a Pod that makes use of this new scheduler.
We just need to add one line for the `schedulerName` in the Pod spec.


```yaml
pradeep@learnk8s$ cat pod-my-scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-my-scheduler
  name: nginx-my-scheduler
spec:
  schedulerName: my-scheduler
  containers:
  - image: nginx
    name: nginx-my-scheduler
    resources: {}
```
```shell
pradeep@learnk8s$ kubectl create -f pod-my-scheduler.yaml
pod/nginx-my-scheduler created
```

```shell
pradeep@learnk8s$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
demo-6c54f77c95-mgz7f   1/1     Running   1          6d
demo-6c54f77c95-q679r   1/1     Running   1          5d
demo-6c54f77c95-qqzbf   1/1     Running   1          5d
demo-6c54f77c95-vjgc2   1/1     Running   1          5d
demo-6c54f77c95-wv78b   1/1     Running   1          6d
demo-ds-gtcf7           1/1     Running   0          16h
demo-ds-kkw4g           1/1     Running   0          16h
nginx-k8s-m02           1/1     Running   0          137m
nginx-manual            1/1     Running   1          4d23h
nginx-my-scheduler      0/1     Pending   0          2s
nginx-no-tolerate       1/1     Running   1          4d17h
nginx-node-selector     1/1     Running   0          2d17h
nginx-taint-demo        1/1     Running   1          4d17h
with-node-affinity      1/1     Running   0          2d17h
```

**NOTE:** There seems to be some issue with this, the Pod is in Pending state, I could not find any events. In the recent Kubernetes version 1.23, there seems to be some changes related to Scheduler definition. I will have to park this aside for some time and continue investigating.

