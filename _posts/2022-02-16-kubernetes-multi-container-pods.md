---
layout: single
title:  "Kubernetes Multi-container Pods"
date:   2022-02-16 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/pod-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Multi Container Pods


### Multi Container Pods

As per Kubernetes documentation, The primary reason that Pods can have multiple containers is to support helper applications that assist a primary application. Typical examples of helper applications are data pullers, data pushers, and proxies. Helper and primary applications often need to communicate with each other. Typically this is done through a shared filesystem, as shown in this example.

We have not yet discussed volumes, but for now, it is sufficient to know that it is a storage component.

```yaml
pradeep@learnk8s$ cat multi-container-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-containers
spec:

  restartPolicy: Never

  volumes:
  - name: common-data
    emptyDir: {}

  containers:

  - name: nginx
    image: nginx
    volumeMounts:
    - name: common-data
      mountPath: /usr/share/nginx/html

  - name: ubuntu
    image: ubuntu
    volumeMounts:
    - name: common-data
      mountPath: /ubuntu-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the Ubuntu container which is visible in the nginx container as they are sharing the same storage common-data > /ubuntu-data/index.html"]
```

```shell
pradeep@learnk8s$ kubectl create -f multi-container-pod.yaml
pod/multi-containers created
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep multi
multi-containers            1/2     NotReady    0              8m3s   10.244.1.34   k8s-m02   <none>           <none>
```
If you notice, though we have defined two containers (nginx and ubuntu) in this Pod definition, there is only a single IP (10.244.1.34 in this case). This is important to understand. All containers in a pod share the same network resources.

Also, READY column shows `1/2` meaning, there are two containers but only one is Running. Let us find out more on this by describing it.
```shell
pradeep@learnk8s$ kubectl describe pods multi-containers
Name:         multi-containers
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Wed, 16 Feb 2022 07:19:50 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.1.34
IPs:
  IP:  10.244.1.34
Containers:
  nginx:
    Container ID:   docker://1cb2560ba88bf7f5a100c74246f10adecf6ce026d9f6031e43947c32ba021d3c
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 16 Feb 2022 07:20:09 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from common-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7jtgb (ro)
  ubuntu:
    Container ID:  docker://d896bd674b06da660ee2e179d104af43344628dcf49edeb90630dc33fa63197e
    Image:         ubuntu
    Image ID:      docker-pullable://ubuntu@sha256:669e010b58baf5beb2836b253c1fd5768333f0d1dbcb834f7c07a4dc93f474be
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
    Args:
      -c
      echo Hello from the Ubuntu container which is visible in the nginx container as they are sharing the same storage common-data > /ubuntu-data/index.html
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 16 Feb 2022 07:20:20 +0530
      Finished:     Wed, 16 Feb 2022 07:20:20 +0530
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /ubuntu-data from common-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7jtgb (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  common-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  kube-api-access-7jtgb:
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
  Normal  Scheduled  3m8s   default-scheduler  Successfully assigned default/multi-containers to k8s-m02
  Normal  Pulling    3m6s   kubelet            Pulling image "nginx"
  Normal  Pulled     2m49s  kubelet            Successfully pulled image "nginx" in 17.095268831s
  Normal  Created    2m49s  kubelet            Created container nginx
  Normal  Started    2m49s  kubelet            Started container nginx
  Normal  Pulling    2m49s  kubelet            Pulling image "ubuntu"
  Normal  Pulled     2m38s  kubelet            Successfully pulled image "ubuntu" in 10.692871602s
  Normal  Created    2m38s  kubelet            Created container ubuntu
  Normal  Started    2m38s  kubelet            Started container ubuntu
```
We can see that the nginx container is Running but the ubuntu container is terminated with Reason as Completed. That means, Ubuntu container has finished its task. The task that we have given to this container is to echo a message and write that to a file. That's all.

To continue the verification, login to the nginx container. Remember the `kubectl exec` command?!

```shell
pradeep@learnk8s$ kubectl exec -it multi-containers -c nginx -- /bin/bash

root@multi-containers:/# curl localhost
Hello from the Ubuntu container which is visible in the nginx container as they are sharing the same storage common-data
root@multi-containers:/# exit
exit
```
The one difference that you might have noticed is the use of `-c` option here. When there are multiple containers in a pod, we need to be specific about which container we want to work with, by specifying its name. In this case , we want to login to the shell (bash) of the nginx container, and henc the `-c nginx` option. Also, we specify the command (`/bin/bash`) that we want to execute after the two dashes followed by a space (`--`).

We logged in to the nginx container, but the data written by the ubuntu container is visible here as both of them are using the common storage volume called `common-data`. 
