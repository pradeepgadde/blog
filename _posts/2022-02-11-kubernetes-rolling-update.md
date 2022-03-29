---
layout: single
title:  "Kubernetes Rolling Updates"
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
# Kubernetes Rolling Update


## Application Life Cycle Management

I ran into some issue with my cluster and had to rebuild from scratch. Here is how my new cluster looks like:
```shell
pradeep@learnk8s$ kubectl get nodes -o wide
NAME      STATUS   ROLES                  AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE              KERNEL-VERSION   CONTAINER-RUNTIME
k8s       Ready    control-plane,master   4m42s   v1.23.1   192.168.177.29   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
k8s-m02   Ready    <none>                 4m21s   v1.23.1   192.168.177.30   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
```
```shell
pradeep@learnk8s$ kubectl get pods -A
NAMESPACE     NAME                          READY   STATUS    RESTARTS   AGE
kube-system   coredns-64897985d-l9zmg       1/1     Running   0          4m34s
kube-system   etcd-k8s                      1/1     Running   1          4m45s
kube-system   kindnet-4d4xb                 1/1     Running   0          50s
kube-system   kindnet-z27gn                 1/1     Running   0          4m30s
kube-system   kube-apiserver-k8s            1/1     Running   1          4m45s
kube-system   kube-controller-manager-k8s   1/1     Running   1          4m45s
kube-system   kube-proxy-n75w8              1/1     Running   0          4m34s
kube-system   kube-proxy-v2mq5              1/1     Running   0          4m30s
kube-system   kube-scheduler-k8s            1/1     Running   1          4m45s
kube-system   storage-provisioner           1/1     Running   0          4m43s

```
Let us now create a deployment called `kodekloud` with image `webapp-color` and four replicas.
Here is the Docker image that we are going to use: https://hub.docker.com/r/kodekloud/webapp-color/tags?page=1&ordering=-last_updated. It has many versions of the same image (like v1, v2, and v3).

```shell
pradeep@learnk8s$ kubectl create deployment kodekloud --image=kodekloud/webapp-color --replicas=4
deployment.apps/kodekloud created
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE      NOMINATED NODE   READINESS GATES
kodekloud-589c9f4b47-6fbqg   1/1     Running   0          47s   10.244.1.8   k8s-m02   <none>           <none>
kodekloud-589c9f4b47-c75mz   1/1     Running   0          47s   10.244.1.7   k8s-m02   <none>           <none>
kodekloud-589c9f4b47-s54tz   1/1     Running   0          47s   10.244.0.7   k8s       <none>           <none>
kodekloud-589c9f4b47-z2sqs   1/1     Running   0          47s   10.244.0.6   k8s       <none>           <none>
```
```shell
pradeep@learnk8s$ kubectl get deployment
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
kodekloud   4/4     4            4           53s
```
Verify the current Image being used by the deployment
```shell
pradeep@learnk8s$ kubectl describe deployment kodekloud | grep Image
    Image:        kodekloud/webapp-color
```


```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.1.8:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #16a085;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-589c9f4b47-6fbqg!</h1>





</div>$ curl 10.244.1.7:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #16a085;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-589c9f4b47-c75mz!</h1>




</div>$ curl 10.244.0.7:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #2980b9;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-589c9f4b47-s54tz!</h1>




</div>$ curl 10.244.0.6:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #be2edd;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-589c9f4b47-z2sqs!</h1>




</div>$
```

### Rolling Update
Also, find out the strategy used by this deployment
```shell
pradeep@learnk8s$ kubectl describe deployment kodekloud | grep Strategy
StrategyType:           RollingUpdate
RollingUpdateStrategy:  25% max unavailable, 25% max surge
```
With `RollingUpdate` strategy, Pods are upgraded few at a time.

Now update the deployment to use a different version of the application `kodekloud/webapp-color:v2`.

For this, we need not delete or re-create the existing deployment. We can **edit** the deployment and modify the image name. Save with Esc + wq.

```shell
pradeep@learnk8s$ kubectl edit deployments kodekloud
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
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2022-02-15T07:13:38Z"
  generation: 1
  labels:
    app: kodekloud
  name: kodekloud
  namespace: default
  resourceVersion: "1592"
  uid: e5728c54-8768-4d5c-b1b0-c3e33cf62062
spec:
  progressDeadlineSeconds: 600
  replicas: 4
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: kodekloud
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: kodekloud
    spec:
      containers:
      - image: kodekloud/webapp-color:v2
        imagePullPolicy: Always
        name: webapp-color
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
-- INSERT --
```
```shell
pradeep@learnk8s$ kubectl edit deployments kodekloud
deployment.apps/kodekloud edited
```

The pods should be recreated with new image. Looking at the AGE column, we can conclude that all four pods got re-created few seconds ago. If you notice carefully, the replicates is having a different Identifier now (changed from 589c9f4b47 to 8477b7849).

```shell
pradeep@learnk8s$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
kodekloud-8477b7849-4lgg2   1/1     Running   0          66s
kodekloud-8477b7849-b7grh   1/1     Running   0          44s
kodekloud-8477b7849-ndf4w   1/1     Running   0          66s
kodekloud-8477b7849-ww5qn   1/1     Running   0          47s
```
```shell
pradeep@learnk8s$ kubectl get deployments.apps
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
kodekloud   4/4     4            4           16m
```
If we look at the Events section of this deployment, we can clearly see that the deployment-controller has Scaled Up the new replica set kodekloud-8477b7849 gradually from 1 to 4 and at the same time, old replica set kodekloud-589c9f4b47 got Scaled down from 4 to 0. This is becuase of the `RollingUpdateStrategy:  25% max unavailable, 25% max surge`.
For this current strategy, only one Pod can be down during the upgrade process (25%, meaning 1 out of total 4 pods in the deployment).

```shell
pradeep@learnk8s$ kubectl describe deployments.apps kodekloud
Name:                   kodekloud
Namespace:              default
CreationTimestamp:      Tue, 15 Feb 2022 12:43:38 +0530
Labels:                 app=kodekloud
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=kodekloud
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=kodekloud
  Containers:
   webapp-color:
    Image:        kodekloud/webapp-color:v2
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
NewReplicaSet:   kodekloud-8477b7849 (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  23m    deployment-controller  Scaled up replica set kodekloud-589c9f4b47 to 4
  Normal  ScalingReplicaSet  8m22s  deployment-controller  Scaled up replica set kodekloud-8477b7849 to 1
  Normal  ScalingReplicaSet  8m22s  deployment-controller  Scaled down replica set kodekloud-589c9f4b47 to 3
  Normal  ScalingReplicaSet  8m22s  deployment-controller  Scaled up replica set kodekloud-8477b7849 to 2
  Normal  ScalingReplicaSet  8m4s   deployment-controller  Scaled down replica set kodekloud-589c9f4b47 to 2
  Normal  ScalingReplicaSet  8m3s   deployment-controller  Scaled up replica set kodekloud-8477b7849 to 3
  Normal  ScalingReplicaSet  8m     deployment-controller  Scaled down replica set kodekloud-589c9f4b47 to 1
  Normal  ScalingReplicaSet  8m     deployment-controller  Scaled up replica set kodekloud-8477b7849 to 4
  Normal  ScalingReplicaSet  7m55s  deployment-controller  Scaled down replica set kodekloud-589c9f4b47 to 0
```
Verify the Image being used by the deployment now. It says, `v2`.
```shell
pradeep@learnk8s$kubectl describe deployment kodekloud| grep Image
    Image:        kodekloud/webapp-color:v2
```
Let's collect the new IP address of the Pods and verify the application from the minikube node.

```shell
pradeep@learnk8s$ kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP            NODE      NOMINATED NODE   READINESS GATES
kodekloud-8477b7849-4lgg2   1/1     Running   0          4m2s    10.244.0.8    k8s       <none>           <none>
kodekloud-8477b7849-b7grh   1/1     Running   0          3m40s   10.244.0.9    k8s       <none>           <none>
kodekloud-8477b7849-ndf4w   1/1     Running   0          4m2s    10.244.1.9    k8s-m02   <none>           <none>
kodekloud-8477b7849-ww5qn   1/1     Running   0          3m43s   10.244.1.10   k8s-m02   <none>           <none>
```
As seen here, the application is now running the v2 version as indicated by the new headers:
```html
<h2>
    Application Version: v2
</h2>
```
```html
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.0.8:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #16a085;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-8477b7849-4lgg2!</h1>



  <h2>
    Application Version: v2
  </h2>


</div>$ curl 10.244.0.9:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #16a085;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-8477b7849-b7grh!</h1>



  <h2>
    Application Version: v2
  </h2>


</div>$ curl 10.244.1.9:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #16a085;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-8477b7849-ndf4w!</h1>



  <h2>
    Application Version: v2
  </h2>


</div>$ curl 10.244.1.10:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #16a085;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-8477b7849-ww5qn!</h1>



  <h2>
    Application Version: v2
  </h2>


</div>$ 
```
Another test is to verify the color of the web-app, it says `green`.

```shell
$ curl 10.244.0.8:8080/color
green$ curl 10.244.0.9:8080/color
green$ curl 10.244.1.9:8080/color
green$ curl 10.244.1.10:8080/color
green$
```

