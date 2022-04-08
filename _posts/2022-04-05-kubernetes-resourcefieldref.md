---
layout: single
title:  "Kubernetes Using Container fields as values for environment variables"
date:   2022-04-04 10:50:04 +0530
categories: Kubernetes
tags: kubeadm minikube
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


# Use Container fields as values for environment variables

 In this exercise, you use Container fields as the values for environment variables. Here is the configuration file for a Pod that has one container:

```yaml
pradeep@learnk8s$ cat resourcefieldref-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resourcefieldref-demo
spec:
  containers:
    - name: resourcefieldref-demo
      image: k8s.gcr.io/busybox:1.24
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_CPU_REQUEST MY_CPU_LIMIT;
          printenv MY_MEM_REQUEST MY_MEM_LIMIT;
          sleep 10;
        done;
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      env:
        - name: MY_CPU_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: resourcefieldref-demo
              resource: requests.cpu
        - name: MY_CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: resourcefieldref-demo
              resource: limits.cpu
        - name: MY_MEM_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: resourcefieldref-demo
              resource: requests.memory
        - name: MY_MEM_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: resourcefieldref-demo
              resource: limits.memory
  restartPolicy: Never

```



```sh
pradeep@learnk8s$ kubectl create -f resourcefieldref-demo.yaml
pod/resourcefieldref-demo created
```



```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
fieldref-demo           1/1     Running   0          11m   10.244.205.195   minikube-m02   <none>           <none>
resourcefieldref-demo   1/1     Running   0          65s   10.244.205.196   minikube-m02   <none>           <none>
```



```sh
pradeep@learnk8s$ kubectl logs resourcefieldref-demo

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864

1
1
33554432
67108864
```



```sh
pradeep@learnk8s$ kubectl describe pods resourcefieldref-demo
Name:         resourcefieldref-demo
Namespace:    default
Priority:     0
Node:         minikube-m02/192.168.177.32
Start Time:   Mon, 04 Apr 2022 22:14:44 +0530
Labels:       <none>
Annotations:  cni.projectcalico.org/containerID: bb3594009ddd4bcac025336e96167c14f49b273a6ceaf4106fb5b22d574a04a9
              cni.projectcalico.org/podIP: 10.244.205.196/32
              cni.projectcalico.org/podIPs: 10.244.205.196/32
Status:       Running
IP:           10.244.205.196
IPs:
  IP:  10.244.205.196
Containers:
  resourcefieldref-demo:
    Container ID:  docker://7de83ba72972e4bb7d9d769d80e0f2171d1e65e90ecd289f886acdd599ec9f05
    Image:         k8s.gcr.io/busybox:1.24
    Image ID:      docker-pullable://k8s.gcr.io/busybox@sha256:4bdd623e848417d96127e16037743f0cd8b528c026e9175e22a84f639eca58ff
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
    Args:
      while true; do echo -en '\n'; printenv MY_CPU_REQUEST MY_CPU_LIMIT; printenv MY_MEM_REQUEST MY_MEM_LIMIT; sleep 10; done;
    State:          Running
      Started:      Mon, 04 Apr 2022 22:15:06 +0530
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     250m
      memory:  64Mi
    Requests:
      cpu:     125m
      memory:  32Mi
    Environment:
      MY_CPU_REQUEST:  1 (requests.cpu)
      MY_CPU_LIMIT:    1 (limits.cpu)
      MY_MEM_REQUEST:  33554432 (requests.memory)
      MY_MEM_LIMIT:    67108864 (limits.memory)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kqs2m (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-kqs2m:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m7s  default-scheduler  Successfully assigned default/resourcefieldref-demo to minikube-m02
  Normal  Pulling    2m5s  kubelet            Pulling image "k8s.gcr.io/busybox:1.24"
  Normal  Pulled     106s  kubelet            Successfully pulled image "k8s.gcr.io/busybox:1.24" in 18.709565279s
  Normal  Created    106s  kubelet            Created container resourcefieldref-demo
  Normal  Started    106s  kubelet            Started container resourcefieldref-demo
```



