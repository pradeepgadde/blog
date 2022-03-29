---
layout: single
title:  "Kubernetes Environment Variables"
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
# Kubernetes Environment Variables


### Environment Variables

Now, it is time to try the other metod of passing arguments, via environment variables.
From the `kubectl run` examples, we can see that environment variables can be set with the `--env` option.

```shell
pradeep@learnk8s$ kubectl run -h
Create and run a particular image in a pod.

Examples:
  <SNIP>
  # Start a hazelcast pod and set environment variables "DNS_DOMAIN=cluster" and "POD_NAMESPACE=default" in the
container
  kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"
  <SNIP>
```

Let us redploy the same image `kodekloud/webapp-color:v3` but this time changing the color of the app with environment variable.

```shell
pradeep@learnk8s$ kubectl run kodekloud-env-color --image=kodekloud/webapp-color:v3 --env=APP_COLOR=pink
pod/kodekloud-env-color created
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep env
kodekloud-env-color         1/1     Running            0              12s     10.244.1.30   k8s-m02   <none>           <none>
```
```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.1.30:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #be2edd;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-env-color!</h1>



  <h2>
    Application Version: v3
  </h2>


</div>$ curl 10.244.1.30:8080/color
pink$
$ exit
logout
```
The curl tests confirm that the application is using the `pink` color now, which is passed with the environment variable, `APP_COLOR`.

If we look at the Pod description, `Environment` section, `APP_COLOR:  pink` is seen as expected.
```shell
pradeep@learnk8s$ kubectl describe pod kodekloud-env-color
Name:         kodekloud-env-color
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Tue, 15 Feb 2022 19:22:07 +0530
Labels:       run=kodekloud-env-color
Annotations:  <none>
Status:       Running
IP:           10.244.1.30
IPs:
  IP:  10.244.1.30
Containers:
  kodekloud-env-color:
    Container ID:   docker://376016abd3ab8f65faa794cb746831f37c3ac16c0436598b08f9365bdd6176d4
    Image:          kodekloud/webapp-color:v3
    Image ID:       docker-pullable://kodekloud/webapp-color@sha256:3ecd19b1b85db381a0b6f78272458c3c274ac2a38e878d65700393899adb3177
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 15 Feb 2022 19:22:08 +0530
    Ready:          True
    Restart Count:  0
    Environment:
      APP_COLOR:  pink
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xj9jz (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-xj9jz:
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
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m26s  default-scheduler  Successfully assigned default/kodekloud-env-color to k8s-m02
  Normal  Pulled     2m25s  kubelet            Container image "kodekloud/webapp-color:v3" already present on machine
  Normal  Created    2m25s  kubelet            Created container kodekloud-env-color
  Normal  Started    2m25s  kubelet            Started container kodekloud-env-color
```

Save the current running pod as a YAML file.
```shell
pradeep@learnk8s$ kubectl get pods kodekloud-env-color -o yaml > pod-env-variable.yaml
```
As this is a running Pod, the YAML output shows a lot of other details on Status, which is not required for now (to change the environment variable).

```yaml
pradeep@learnk8s$ cat pod-env-variable.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-02-15T13:52:07Z"
  labels:
    run: kodekloud-env-color
  name: kodekloud-env-color
  namespace: default
  resourceVersion: "15077"
  uid: bd7c01ae-553e-4f77-b589-6af6560dcc88
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: pink
    image: kodekloud/webapp-color:v3
    imagePullPolicy: IfNotPresent
    name: kodekloud-env-color
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-xj9jz
      readOnly: true
  dnsPolicy: ClusterFirst
  <SNIP>
```
Create another copy of this YAML file and modify the definition to change the color fo the App to `darkblue`.

```shell
pradeep@learnk8s$ cp pod-env-variable.yaml pod-env-variable-2.yaml
pradeep@learnk8s$ vi pod-env-variable-2.yaml
```
The modified version looks like this.
```yaml
pradeep@learnk8s$ cat pod-env-variable-2.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-02-15T13:52:07Z"
  labels:
    run: kodekloud-env-color-2
  name: kodekloud-env-color-2
  namespace: default
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: darkblue
    image: kodekloud/webapp-color:v3
    imagePullPolicy: IfNotPresent
    name: kodekloud-env-color-2
    resources: {}
```
```shell
pradeep@learnk8s$ kubectl create -f pod-env-variable-2.yaml
pod/kodekloud-env-color-2 created
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep env
kodekloud-env-color         1/1     Running            0                14m     10.244.1.30   k8s-m02   <none>           <none>
kodekloud-env-color-2       1/1     Running            0                16s     10.244.1.31   k8s-m02   <none>           <none>
```
We can verify that this new pod is using the `darkblue` color.
```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.1.31:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #130f40;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-env-color-2!</h1>



  <h2>
    Application Version: v3
  </h2>


</div>$ curl 10.244.1.31:8080/color
darkblue$
$ curl 10.244.1.31:8080/color
darkblue$
$ exit
logout
```
