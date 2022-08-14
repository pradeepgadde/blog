---
layout: single
title:  "Working with Kubernetes Resources "
date:   2022-01-28 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
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

# Working with Kubernetes Resources


### Deleting Resources
```shell
pradeep@learnk8s$ kubectl delete pod demo-75f9c7566f-fct76
pod "demo-75f9c7566f-fct76" deleted
```
```shell
pradeep@learnk8s$ kubectl get pods
NAME                    READY   STATUS              RESTARTS   AGE
demo-75f9c7566f-b45x7   0/1     ContainerCreating   0          3s
demo-75f9c7566f-kz9kb   1/1     Running             0          55m
demo-75f9c7566f-nmjw2   1/1     Running             0          65m
```
```shell
pradeep@learnk8s$ kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/demo-75f9c7566f-b45x7   1/1     Running   0          2m29s
pod/demo-75f9c7566f-kz9kb   1/1     Running   0          57m
pod/demo-75f9c7566f-nmjw2   1/1     Running   0          67m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   77m

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/demo   3/3     3            3           67m

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/demo-75f9c7566f   3         3         3       67m
```
```shell
pradeep@learnk8s$ kubectl delete deployment demo
deployment.apps "demo" deleted
```
Deleting a deployment, deletes all the associated pods, replicaset as well.
```shell
pradeep@learnk8s$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   78m
```

### Creating Resources in a Declarative Way—YAML
```yaml
pradeep@learnk8s$ kubectl run nginx --image=nginx --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
```shell
pradeep@learnk8s$ kubectl run nginx --image=nginx --dry-run=client -o yaml > demopod.yaml
```

```shell
pradeep@learnk8s$ kubectl create -f demopod.yaml
pod/nginx created
```
```shell
pradeep@learnk8s$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          50s
```

```shell
pradeep@learnk8s$ kubectl create deployment demo --image=nginx --replicas=3 --dry-run=client -o yaml > demodeploy.yaml
```
```yaml
pradeep@learnk8s$ cat demodeploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: demo
  name: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: demo
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```
```shell
pradeep@learnk8s$ kubectl create -f demodeploy.yaml
deployment.apps/demo created
```
```shell
pradeep@learnk8s$ kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/demo-6c54f77c95-6g7zq   1/1     Running   0          71s
pod/demo-6c54f77c95-sb4c9   1/1     Running   0          71s
pod/demo-6c54f77c95-w2bsw   1/1     Running   0          71s
pod/nginx                   1/1     Running   0          5m45s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   90m

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/demo   3/3     3            3           71s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/demo-6c54f77c95   3         3         3       71s
```

