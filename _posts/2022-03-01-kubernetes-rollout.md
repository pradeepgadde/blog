---
layout: single
title:  "Kubectl Rollout"
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
# Kubectl Rollout


### Rollout 

Let us take a look at the current deployments.
```shell
pradeep@learnk8s$ kubectl get deployment
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
kodekloud   4/4     4            4           13d
```

To manage the rollout of a resource like deployments, kubernetes provides the `rollout` option, with which we can view rollout history, see the status of the rollout and even undo a previous rollout.

Here is the detailed help information.

```shell
pradeep@learnk8s$ kubectl rollout -h
Manage the rollout of a resource.

 Valid resource types include:

  *  deployments
  *  daemonsets
  *  statefulsets

Examples:
  # Rollback to the previous deployment
  kubectl rollout undo deployment/abc

  # Check the rollout status of a daemonset
  kubectl rollout status daemonset/foo

Available Commands:
  history     View rollout history
  pause       Mark the provided resource as paused
  restart     Restart a resource
  resume      Resume a paused resource
  status      Show the status of the rollout
  undo        Undo a previous rollout

Usage:
  kubectl rollout SUBCOMMAND [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```
Let us use this option to view the history of our `kodekloud` deployment.

```shell
pradeep@learnk8s$ kubectl rollout history deployment kodekloud
deployment.apps/kodekloud
REVISION  CHANGE-CAUSE
1         <none>
3         <none>
4         <none>
```

We do see that there are multiple revisions to this `kodekloud` deployment, but we do not see any `CHANGE-CAUSE`.

Even in the description, we do see only one line related to `deployment.kubernetes.io/revision`. Pay attention to the `Annotations` section.

```shell
pradeep@learnk8s$ kubectl describe deployments.apps
Name:               kodekloud
Namespace:          default
CreationTimestamp:  Tue, 15 Feb 2022 12:43:38 +0530
Labels:             app=kodekloud
Annotations:        deployment.kubernetes.io/revision: 4
Selector:           app=kodekloud
Replicas:           4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:       Recreate
MinReadySeconds:    0
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
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kodekloud-8477b7849 (4/4 replicas created)
Events:          <none>
```

Create a new deployment 
```yaml
pradeep@learnk8s$ cat nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
````
```shell
pradeep@learnk8s$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```
```shell
pradeep@learnk8s$ kubectl get deployments.apps
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
kodekloud          4/4     4            4           13d
nginx-deployment   3/3     3            3           31s
```
```shell
pradeep@learnk8s$ kubectl describe deployments.apps nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 01 Mar 2022 06:05:50 +0530
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
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
  Type     Reason                 Age   From                   Message
  ----     ------                 ----  ----                   -------
  Warning  ReplicaSetCreateError  33s   deployment-controller  Failed to create new replica set "nginx-deployment-7b96fbf5d8": Unauthorized
  Normal   ScalingReplicaSet      33s   deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 3
```


### Update Deployment (using Kubectl Set)

Earlier we have used `kubectl edit` to make changes to an existing deployment.

Here we will explore another option `kubectl set`.

```shell
pradeep@learnk8s$ kubectl set -h
Configure application resources.

 These commands help you make changes to existing application resources.

Available Commands:
  env            Update environment variables on a pod template
  image          Update the image of a pod template
  resources      Update resource requests/limits on objects with pod templates
  selector       Set the selector on a resource
  serviceaccount Update the service account of a resource
  subject        Update the user, group, or service account in a role binding or cluster role binding

Usage:
  kubectl set SUBCOMMAND [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```
Let us set the image for our `nginx-deployment` to a new version `1.21`.

```shell
pradeep@learnk8s$ kubectl set image deployment/nginx-deployment nginx=nginx:1.21
deployment.apps/nginx-deployment image updated
```
We can see the rollout status like this, using the `kubectl rollout status` command.

```shell
pradeep@learnk8s$ kubectl rollout status deployment/nginx-deployment

deployment "nginx-deployment" successfully rolled out
```
Describe the deployment 

```shell
pradeep@learnk8s$ kubectl describe deployments.apps nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 01 Mar 2022 06:05:50 +0530
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 2
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
  Type     Reason                 Age    From                   Message
  ----     ------                 ----   ----                   -------
  Warning  ReplicaSetCreateError  10m    deployment-controller  Failed to create new replica set "nginx-deployment-7b96fbf5d8": Unauthorized
  Normal   ScalingReplicaSet      10m    deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 3
  Normal   ScalingReplicaSet      3m18s  deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 1
  Normal   ScalingReplicaSet      3m12s  deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 2
  Normal   ScalingReplicaSet      3m12s  deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 2
  Normal   ScalingReplicaSet      3m10s  deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 1
  Normal   ScalingReplicaSet      3m10s  deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 3
  Normal   ScalingReplicaSet      3m3s   deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 0
```
