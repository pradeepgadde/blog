---
layout: single
title:  "Kubernetes Resource Limits"
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

# Kubernetes Resource Limits


### Resource Limits

Earlier, we have created a namespace called `prod`. Let's describe it to see if any resource quota applied to it.

```shell
pradeep@learnk8s$ kubectl describe namespaces prod
Name:         prod
Labels:       kubernetes.io/metadata.name=prod
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```
Let's define a quota using the example shown here:
```shell
pradeep@learnk8s$ kubectl create resourcequota -h
Create a resource quota with the specified name, hard limits, and optional scopes.

Aliases:
quota, resourcequota

Examples:
  # Create a new resource quota named my-quota
  kubectl create quota my-quota
--hard=cpu=1,memory=1G,pods=2,services=3,replicationcontrollers=2,resourcequotas=1,secrets=5,persistentvolumeclaims=10

  # Create a new resource quota named best-effort
  kubectl create quota best-effort --hard=pods=100 --scopes=BestEffort
```
```shell
pradeep@learnk8s$ kubectl create quota my-quota --hard=cpu=1,memory=1G,pods=2,services=3 -n prod
resourcequota/my-quota created
```

```shell
pradeep@learnk8s$ kubectl describe ns prod
Name:         prod
Labels:       kubernetes.io/metadata.name=prod
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:     my-quota
  Resource  Used  Hard
  --------  ---   ---
  cpu       0     1
  memory    0     1G
  pods      0     2
  services  0     3

No LimitRange resource.
```
```shell
pradeep@learnk8s$ kubectl get quota -n prod
NAME       AGE     REQUEST                                            LIMIT
my-quota   4m40s   cpu: 0/1, memory: 0/1G, pods: 0/2, services: 0/3
```

```shell
pradeep@learnk8s$ kubectl describe quota -n prod
Name:       my-quota
Namespace:  prod
Resource    Used  Hard
--------    ----  ----
cpu         0     1
memory      0     1G
pods        0     2
services    0     3
```
Let's try create a pod in this namespace and see what happens? 
```shell
pradeep@learnk8s$ kubectl run nginx-without-cpu --image=nginx -n prod
Error from server (Forbidden): pods "nginx-without-cpu" is forbidden: failed quota: my-quota: must specify cpu,memory
pradeep@learnk8s$
```
Pod creation failed with an error (Forbidden).
Let's try create a deployment in this namespace and see what happens?
```shell
pradeep@learnk8s$ kubectl create deployment test-quota --image=nginx --replicas=3 -n prod
deployment.apps/test-quota created
```
```shell
pradeep@learnk8s$ kubectl get deploy -n prod
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
test-quota   0/3     0            0           47s
```
```shell
pradeep@learnk8s$ kubectl get events -n prod
LAST SEEN   TYPE      REASON              OBJECT                             MESSAGE
104s        Warning   FailedCreate        replicaset/test-quota-666dfd598f   Error creating: pods "test-quota-666dfd598f-bgsxd" is forbidden: failed quota: my-quota: must specify cpu,memory
103s        Warning   FailedCreate        replicaset/test-quota-666dfd598f   Error creating: pods "test-quota-666dfd598f-w9p4z" is forbidden: failed quota: my-quota: must specify cpu,memory
103s        Warning   FailedCreate        replicaset/test-quota-666dfd598f   Error creating: pods "test-quota-666dfd598f-swwgq" is forbidden: failed quota: my-quota: must specify cpu,memory
103s        Warning   FailedCreate        replicaset/test-quota-666dfd598f   Error creating: pods "test-quota-666dfd598f-tjlsb" is forbidden: failed quota: my-quota: must specify cpu,memory
103s        Warning   FailedCreate        replicaset/test-quota-666dfd598f   Error creating: pods "test-quota-666dfd598f-ll2m9" is forbidden: failed quota: my-quota: must specify cpu,memory
103s        Warning   FailedCreate        replicaset/test-quota-666dfd598f   Error creating: pods "test-quota-666dfd598f-c72mn" is forbidden: failed quota: my-quota: must specify cpu,memory
103s        Warning   FailedCreate        replicaset/test-quota-666dfd598f   Error creating: pods "test-quota-666dfd598f-hqzf4" is forbidden: failed quota: my-quota: must specify cpu,memory
103s        Warning   FailedCreate        replicaset/test-quota-666dfd598f   Error creating: pods "test-quota-666dfd598f-bdh4w" is forbidden: failed quota: my-quota: must specify cpu,memory
102s        Warning   FailedCreate        replicaset/test-quota-666dfd598f   Error creating: pods "test-quota-666dfd598f-gggw2" is forbidden: failed quota: my-quota: must specify cpu,memory
21s         Warning   FailedCreate        replicaset/test-quota-666dfd598f   (combined from similar events): Error creating: pods "test-quota-666dfd598f-b5rh6" is forbidden: failed quota: my-quota: must specify cpu,memory
104s        Normal    ScalingReplicaSet   deployment/test-quota              Scaled up replica set test-quota-666dfd598f to 3
```
```yaml
pradeep@learnk8s$ cat pod-quota.yaml
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
  namespace: prod
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "800m"
      requests:
        memory: "600Mi"
        cpu: "400m"
```
```shell
pradeep@learnk8s$ kubectl create -f pod-quota.yaml
pod/quota-mem-cpu-demo created
```
```shell
pradeep@learnk8s$ kubectl get pods -n prod
NAME                 READY   STATUS    RESTARTS   AGE
quota-mem-cpu-demo   1/1     Running   0          9s
```
```shell
pradeep@learnk8s$ kubectl describe ns prod
Name:         prod
Labels:       kubernetes.io/metadata.name=prod
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:     my-quota
  Resource  Used   Hard
  --------  ---    ---
  cpu       400m   1
  memory    600Mi  1G
  pods      1      2
  services  0      3

No LimitRange resource.
```
```yaml
pradeep@learnk8s$ cat pod-quota-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo-2
  namespace: prod
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr-2
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "800m"
      requests:
        memory: "600Mi"
        cpu: "400m"
```
```shell
pradeep@learnk8s$ kubectl create -f pod-quota-2.yaml
Error from server (Forbidden): error when creating "pod-quota-2.yaml": pods "quota-mem-cpu-demo-2" is forbidden: exceeded quota: my-quota, requested: memory=600Mi, used: memory=600Mi, limited: memory=1G
```

