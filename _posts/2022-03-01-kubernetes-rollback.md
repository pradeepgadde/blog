---
layout: single
title:  "Kubernetes Deployment Rollback"
date:   2022-03-01 10:55:04 +0530
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
# Kubernetes Rollback



### Rollback

If you've decided to undo the current rollout and rollback to the previous revision, you can do so.

```shell
pradeep@learnk8s$ kubectl rollout undo deployment/nginx-deployment
deployment.apps/nginx-deployment rolled back
```
```shell
pradeep@learnk8s$ kubectl rollout status deployment nginx-deployment
deployment "nginx-deployment" successfully rolled out
```
```shell
pradeep@learnk8s$ kubectl rollout history deployment nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
2         image updated to 1.21
3         <none>
```
```shell
pradeep@learnk8s$ kubectl describe deployments.apps nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 01 Mar 2022 06:05:50 +0530
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 3
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.20
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-7b96fbf5d8 (3/3 replicas created)
Events:
  Type     Reason                 Age                From                   Message
  ----     ------                 ----               ----                   -------
  Warning  ReplicaSetCreateError  20m                deployment-controller  Failed to create new replica set "nginx-deployment-7b96fbf5d8": Unauthorized
  Normal   ScalingReplicaSet      13m                deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 1
  Normal   ScalingReplicaSet      12m                deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 2
  Normal   ScalingReplicaSet      12m                deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 2
  Normal   ScalingReplicaSet      12m                deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 1
  Normal   ScalingReplicaSet      12m                deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 3
  Normal   ScalingReplicaSet      12m                deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 0
  Normal   ScalingReplicaSet      50s                deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 1
  Normal   ScalingReplicaSet      48s                deployment-controller  Scaled down replica set nginx-deployment-5778cd94ff to 2
  Normal   ScalingReplicaSet      48s                deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 2
  Normal   ScalingReplicaSet      46s                deployment-controller  Scaled down replica set nginx-deployment-5778cd94ff to 1
  Normal   ScalingReplicaSet      45s (x2 over 20m)  deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 3
  Normal   ScalingReplicaSet      43s                deployment-controller  Scaled down replica set nginx-deployment-5778cd94ff to 0
```
Let us annotate revision 3.

```shell
pradeep@learnk8s$ kubectl annotate deployments.apps nginx-deployment kubernetes.io/change-cause="image rolledback to 1.20"
deployment.apps/nginx-deployment annotated
```

```shell
pradeep@learnk8s$ kubectl rollout history deployment nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
2         image updated to 1.21
3         image rolledback to 1.20
```

What happens if we rollback the deployment again?

```shell
pradeep@learnk8s$ kubectl rollout undo deployment/nginx-deployment
deployment.apps/nginx-deployment rolled back
```
```shell
pradeep@learnk8s$ kubectl rollout history deployment nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
3         image rolledback to 1.20
4         image updated to 1.21
```
```shell
pradeep@learnk8s$ kubectl rollout history deployment/nginx-deployment --revision=4
deployment.apps/nginx-deployment with revision #4
Pod Template:
  Labels:	app=nginx
	pod-template-hash=5778cd94ff
  Annotations:	kubernetes.io/change-cause: image updated to 1.21
  Containers:
   nginx:
    Image:	nginx:1.21
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

```shell
pradeep@learnk8s$ kubectl describe deployments.apps nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 01 Mar 2022 06:05:50 +0530
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 4
                        kubernetes.io/change-cause: image updated to 1.21
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.21
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-5778cd94ff (3/3 replicas created)
Events:
  Type     Reason                 Age                    From                   Message
  ----     ------                 ----                   ----                   -------
  Warning  ReplicaSetCreateError  31m                    deployment-controller  Failed to create new replica set "nginx-deployment-7b96fbf5d8": Unauthorized
  Normal   ScalingReplicaSet      23m                    deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 1
  Normal   ScalingReplicaSet      23m                    deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 3
  Normal   ScalingReplicaSet      23m                    deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 0
  Normal   ScalingReplicaSet      11m                    deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 1
  Normal   ScalingReplicaSet      11m                    deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 2
  Normal   ScalingReplicaSet      11m                    deployment-controller  Scaled down replica set nginx-deployment-5778cd94ff to 2
  Normal   ScalingReplicaSet      11m                    deployment-controller  Scaled down replica set nginx-deployment-5778cd94ff to 1
  Normal   ScalingReplicaSet      11m (x2 over 31m)      deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 3
  Normal   ScalingReplicaSet      11m                    deployment-controller  Scaled down replica set nginx-deployment-5778cd94ff to 0
  Normal   ScalingReplicaSet      2m27s (x2 over 24m)    deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 1
  Normal   ScalingReplicaSet      2m25s (x2 over 23m)    deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 2
  Normal   ScalingReplicaSet      2m25s (x2 over 23m)    deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 2
  Normal   ScalingReplicaSet      2m20s (x3 over 2m23s)  deployment-controller  (combined from similar events): Scaled down replica set nginx-deployment-7b96fbf5d8 to 0
```

Let us update the deployment one more time, finally to the latest version.
```shell
pradeep@learnk8s$ kubectl set image deployment/nginx-deployment nginx=nginx:latest
deployment.apps/nginx-deployment image updated
```
Check the rollout history, and note that the existing annotation continued to the latest revision.
```shell
pradeep@learnk8s$ kubectl rollout history deployment nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
3         image rolledback to 1.20
4         image updated to 1.21
5         image updated to 1.21
```
Change the annotation
```shell
pradeep@learnk8s$ kubectl annotate deployments.apps nginx-deployment kubernetes.io/change-cause="image updated to the latest"
deployment.apps/nginx-deployment annotated
```
```shell
pradeep@learnk8s$ kubectl rollout history deployment nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
3         image rolledback to 1.20
4         image updated to 1.21
5         image updated to the latest
```

```shell
pradeep@learnk8s$ kubectl describe deployments.apps nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 01 Mar 2022 06:05:50 +0530
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 5
                        kubernetes.io/change-cause: image updated to the latest
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-67dffbbbb (3/3 replicas created)
Events:
  Type     Reason                 Age                  From                   Message
  ----     ------                 ----                 ----                   -------
  Warning  ReplicaSetCreateError  40m                  deployment-controller  Failed to create new replica set "nginx-deployment-7b96fbf5d8": Unauthorized
  Normal   ScalingReplicaSet      32m                  deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 1
  Normal   ScalingReplicaSet      32m                  deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 3
  Normal   ScalingReplicaSet      32m                  deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 0
  Normal   ScalingReplicaSet      20m                  deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 1
  Normal   ScalingReplicaSet      20m                  deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 2
  Normal   ScalingReplicaSet      20m                  deployment-controller  Scaled down replica set nginx-deployment-5778cd94ff to 2
  Normal   ScalingReplicaSet      20m                  deployment-controller  Scaled down replica set nginx-deployment-5778cd94ff to 1
  Normal   ScalingReplicaSet      20m (x2 over 40m)    deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 3
  Normal   ScalingReplicaSet      20m                  deployment-controller  Scaled down replica set nginx-deployment-5778cd94ff to 0
  Normal   ScalingReplicaSet      11m (x2 over 33m)    deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 1
  Normal   ScalingReplicaSet      11m (x2 over 33m)    deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 2
  Normal   ScalingReplicaSet      11m (x2 over 33m)    deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 2
  Normal   ScalingReplicaSet      2m52s (x9 over 11m)  deployment-controller  (combined from similar events): Scaled down replica set nginx-deployment-5778cd94ff to 0
```
```shell
pradeep@learnk8s$ kubectl get pods
NAME                               READY   STATUS                   RESTARTS         AGE
kodekloud-8477b7849-blzrs          1/1     Running                  1                13d
kodekloud-8477b7849-m65m8          1/1     Running                  1                13d
kodekloud-8477b7849-p7psw          1/1     Running                  0                32h
kodekloud-8477b7849-vf9zs          1/1     Running                  0                32h
kodekloud-cm                       0/1     ContainerStatusUnknown   1                13d
nginx-deployment-67dffbbbb-7ttxq   1/1     Running                  0                3m35s
nginx-deployment-67dffbbbb-8dtsf   1/1     Running                  0                3m38s
nginx-deployment-67dffbbbb-sz5bb   1/1     Running                  0                3m40s
pod-with-hostpath-volume           0/1     ContainerCreating        1                8d
pod-with-init-container            1/1     Running                  45 (4m46s ago)   12d
security-context-demo-cap          1/1     Running                  15 (5m16s ago)   9d
```
```shell
pradeep@learnk8s$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
kodekloud-589c9f4b47          0         0         0       13d
kodekloud-676c6b9fcd          0         0         0       13d
kodekloud-8477b7849           4         4         4       13d
nginx-deployment-5778cd94ff   0         0         0       34m
nginx-deployment-67dffbbbb    3         3         3       3m52s
nginx-deployment-7b96fbf5d8   0         0         0       41m
```

```shell
pradeep@learnk8s$ kubectl rollout history deployment nginx-deployment --revision=5
deployment.apps/nginx-deployment with revision #5
Pod Template:
  Labels:	app=nginx
	pod-template-hash=67dffbbbb
  Annotations:	kubernetes.io/change-cause: image updated to the latest
  Containers:
   nginx:
    Image:	nginx:latest
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```
