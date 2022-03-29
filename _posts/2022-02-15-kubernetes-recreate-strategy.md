---
layout: single
title:  "Kubernetes Recreate Strategy"
date:   2022-02-15 10:55:04 +0530
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
# Kubernetes Recreate Strategy


### Recreate
There is another deployment strategy called `Recreate`.
Let us edit the deployment again to use another version, v3, this time and change the strategy from `RollingUpdate` to `Recreate`.
```shell
pradeep@learnk8s$ kubectl edit deployment kodekloud
deployment.apps/kodekloud edited
```
```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
  creationTimestamp: "2022-02-15T07:13:38Z"
  generation: 2
  labels:
    app: kodekloud
  name: kodekloud
  namespace: default
  resourceVersion: "2408"
  uid: e5728c54-8768-4d5c-b1b0-c3e33cf62062
spec:
  progressDeadlineSeconds: 600
  replicas: 4
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: kodekloud
  strategy:
    type: Recreate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: kodekloud
    spec:
      containers:
      - image: kodekloud/webapp-color:v3
        imagePullPolicy: Always
        name: webapp-color
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
-- INSERT --
```
```shell
pradeep@learnk8s$ kubectl get deployments.apps
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
kodekloud   4/4     4            4           33m
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE      NOMINATED NODE   READINESS GATES
kodekloud-676c6b9fcd-6bm6z   1/1     Running   0          53s   10.244.1.12   k8s-m02   <none>           <none>
kodekloud-676c6b9fcd-dggk7   1/1     Running   0          53s   10.244.0.10   k8s       <none>           <none>
kodekloud-676c6b9fcd-m2q7x   1/1     Running   0          53s   10.244.0.11   k8s       <none>           <none>
kodekloud-676c6b9fcd-p8tnl   1/1     Running   0          53s   10.244.1.11   k8s-m02   <none>           <none>
```
If we look at the latest events of the deployment, the *deployment-controller  Scaled down replica set kodekloud-8477b7849 to 0 at a time and Scaled up new replica set kodekloud-676c6b9fcd to 4*.

```shell
pradeep@learnk8s$ kubectl describe deployments.apps
Name:               kodekloud
Namespace:          default
CreationTimestamp:  Tue, 15 Feb 2022 12:43:38 +0530
Labels:             app=kodekloud
Annotations:        deployment.kubernetes.io/revision: 3
Selector:           app=kodekloud
Replicas:           4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:       Recreate
MinReadySeconds:    0
Pod Template:
  Labels:  app=kodekloud
  Containers:
   webapp-color:
    Image:        kodekloud/webapp-color:v3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kodekloud-676c6b9fcd (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  35m    deployment-controller  Scaled up replica set kodekloud-589c9f4b47 to 4
  Normal  ScalingReplicaSet  20m    deployment-controller  Scaled up replica set kodekloud-8477b7849 to 1
  Normal  ScalingReplicaSet  20m    deployment-controller  Scaled down replica set kodekloud-589c9f4b47 to 3
  Normal  ScalingReplicaSet  20m    deployment-controller  Scaled up replica set kodekloud-8477b7849 to 2
  Normal  ScalingReplicaSet  20m    deployment-controller  Scaled down replica set kodekloud-589c9f4b47 to 2
  Normal  ScalingReplicaSet  20m    deployment-controller  Scaled up replica set kodekloud-8477b7849 to 3
  Normal  ScalingReplicaSet  20m    deployment-controller  Scaled down replica set kodekloud-589c9f4b47 to 1
  Normal  ScalingReplicaSet  20m    deployment-controller  Scaled up replica set kodekloud-8477b7849 to 4
  Normal  ScalingReplicaSet  19m    deployment-controller  Scaled down replica set kodekloud-589c9f4b47 to 0
  Normal  ScalingReplicaSet  2m58s  deployment-controller  Scaled down replica set kodekloud-8477b7849 to 0
  Normal  ScalingReplicaSet  2m27s  deployment-controller  Scaled up replica set kodekloud-676c6b9fcd to 4
```
Just to verify, let us connect to one of the pods from the new version and check the output.

```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.1.12:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #e74c3c;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-676c6b9fcd-6bm6z!</h1>



  <h2>
    Application Version: v3
  </h2>


</div>$
```

From this,  it is clear that the application got upgraded to latest version.

Also, there is another test to verify the color used by the web-app. All four pods returned `red`.
```shell
$ curl 10.244.1.12:8080/color
red$ curl 10.244.1.11:8080/color
red$ curl 10.244.0.11:8080/color
red$ curl 10.244.0.10:8080/color
red$
```
