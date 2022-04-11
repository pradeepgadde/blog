---
layout: single
title:  "Kubernetes Quality of Service"
date:   2022-04-11 03:59:04 +0530
categories: Kubernetes
tags: kubeadm minikube
author: "Pradeep Gadde"
toc: true
toc_sticky: true
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

# Kubernetes Quality of Service

While [describing Pods](https://pradeepgadde.com/blog/kubernetes/2022/02/04/kubectl-describe-explain.html), I used to notice one line about `QoS Class: ` and it used to be mostly `BestEffort` as shown in the description `QoS Class: BestEffort`. Later when applying limits, noticed a change from `BestEffort` to `Burstable`. I thought these were the only two options. Today, I got chance to explore this topic and find out that there is a third option, `Guaranteed` as well.

According to Kubernetes Documentation, when Kubernetes creates a Pod it assigns one of these QoS classes to the Pod:

- Guaranteed

- Burstable

- BestEffort

  | QoS Class  | Condition                                                    |
  | ---------- | ------------------------------------------------------------ |
  | Guaranteed | Every Container in the Pod must have a memory, CPU limit and a memory, CPU request, also the limit must be equal to the limit value |
  | Burstable  | The Pod does not meet the criteria for QoS class Guaranteed and at least one Container in the Pod has a memory or CPU request. |
  | BestEffort | The Containers in the Pod must not have any memory or CPU limits or requests. |

  

Let us test these three QoS options here.

## BestEffort

```yaml
pradeep@learnk8s$ cat besteffort.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-besteffort
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-besteffort
    image: nginx

```



```sh
pradeep@learnk8s$ kubectl create -f besteffort.yaml 
pod/qos-demo-besteffort created
```



```sh
pradeep@learnk8s$ kubectl get pods -n qos-example
NAME                  READY   STATUS    RESTARTS   AGE
qos-demo-besteffort   1/1     Running   0          94s
```



```sh
pradeep@learnk8s$ kubectl describe pods -n qos-example
Name:         qos-demo-besteffort
Namespace:    qos-example
Priority:     0
Node:         minikube-m02/172.16.30.7
Start Time:   Mon, 11 Apr 2022 08:32:39 +0530
Labels:       <none>
Annotations:  cni.projectcalico.org/containerID: 48d330b8d0089f9c010fee4a0c8b142229491c76c04d432f59de216c8f07821c
              cni.projectcalico.org/podIP: 10.244.205.204/32
              cni.projectcalico.org/podIPs: 10.244.205.204/32
Status:       Running
IP:           10.244.205.204
IPs:
  IP:  10.244.205.204
Containers:
  qos-demo-besteffort:
    Container ID:   docker://4382b8719874f73a37847bb07e6111adaa66fa1cdb069af71b281b688eedf92d
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2275af0f20d71b293916f1958f8497f987b8d8fd8113df54635f2a5915002bf1
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 11 Apr 2022 08:33:27 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-x4pmx (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-x4pmx:
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
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m8s  default-scheduler  Successfully assigned qos-example/qos-demo-besteffort to minikube-m02
  Normal  Pulling    2m3s  kubelet            Pulling image "nginx"
  Normal  Pulled     81s   kubelet            Successfully pulled image "nginx" in 41.600660801s
  Normal  Created    81s   kubelet            Created container qos-demo-besteffort
  Normal  Started    81s   kubelet            Started container qos-demo-besteffort
 
```

From this, as expected, we see the `QoS Class` as `BestEffort`, because our Pod did not have any memory or CPU limits or requests.



## Burstable

```yaml
pradeep@learnk8s$ cat burstable.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-burstable
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-burstable
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"

```



```sh
pradeep@learnk8s$ kubectl get pods -n qos-example
NAME                  READY   STATUS    RESTARTS   AGE
qos-demo-besteffort   1/1     Running   0          5m15s
qos-demo-burstable    1/1     Running   0          38s
```



Let us describe the new pod

```sh
pradeep@learnk8s$ kubectl describe pods qos-demo-burstable -n qos-example
Name:         qos-demo-burstable
Namespace:    qos-example
Priority:     0
Node:         minikube-m03/172.16.30.8
Start Time:   Mon, 11 Apr 2022 08:37:16 +0530
Labels:       <none>
Annotations:  cni.projectcalico.org/containerID: 0c37628815b45cb2111696cb6512fe9d89c58772d17f878550bfe13d08002b20
              cni.projectcalico.org/podIP: 10.244.151.12/32
              cni.projectcalico.org/podIPs: 10.244.151.12/32
Status:       Running
IP:           10.244.151.12
IPs:
  IP:  10.244.151.12
Containers:
  qos-demo-burstable:
    Container ID:   docker://17fe659a6280979b6dc08277a2482ec742a1fd54ebbbaf0a5689441ec45bdf33
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2275af0f20d71b293916f1958f8497f987b8d8fd8113df54635f2a5915002bf1
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 11 Apr 2022 08:37:21 +0530
    Ready:          True
    Restart Count:  0
    Limits:
      memory:  200Mi
    Requests:
      memory:     100Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-42ppk (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-42ppk:
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
  Normal  Scheduled  59s   default-scheduler  Successfully assigned qos-example/qos-demo-burstable to minikube-m03
  Normal  Pulling    57s   kubelet            Pulling image "nginx"
  Normal  Pulled     54s   kubelet            Successfully pulled image "nginx" in 2.813492871s
  Normal  Created    54s   kubelet            Created container qos-demo-burstable
  Normal  Started    54s   kubelet            Started container qos-demo-burstable
 
```

In this example, our Pod has a memory limit and request. Requested memory (100Mi) is less than the limits (200Mi) set, so `Burstable` QoS class is assigned.



## Guaranteed

```yaml
pradeep@learnk8s$ cat guaranteed.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-guaranteed
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-guaranteed
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "300m"
      requests:
        memory: "200Mi"
        cpu: "300m"

```

Create the pod

```sh
pradeep@learnk8s$ kubectl create -f guaranteed.yaml 
pod/qos-demo-guaranteed created
```

List all Pods

```sh
pradeep@learnk8s$ kubectl get pods -n qos-example
NAME                  READY   STATUS    RESTARTS   AGE
qos-demo-besteffort   1/1     Running   0          11m
qos-demo-burstable    1/1     Running   0          6m46s
qos-demo-guaranteed   1/1     Running   0          34s
```

Describe the new pod

```sh
pradeep@learnk8s$ kubectl describe pods qos-demo-guaranteed -n qos-example
Name:         qos-demo-guaranteed
Namespace:    qos-example
Priority:     0
Node:         minikube-m03/172.16.30.8
Start Time:   Mon, 11 Apr 2022 08:43:28 +0530
Labels:       <none>
Annotations:  cni.projectcalico.org/containerID: 0216b63d97760d90d9bb3235a4a9db09ababf751cf87971b0fa91e925c9f8fe3
              cni.projectcalico.org/podIP: 10.244.151.13/32
              cni.projectcalico.org/podIPs: 10.244.151.13/32
Status:       Running
IP:           10.244.151.13
IPs:
  IP:  10.244.151.13
Containers:
  qos-demo-guaranteed:
    Container ID:   docker://e4cdb0a437d2890dd8c81896d5feece2ba0a166a2fef2fb0e46bc62dd7b16d7f
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2275af0f20d71b293916f1958f8497f987b8d8fd8113df54635f2a5915002bf1
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 11 Apr 2022 08:43:36 +0530
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     300m
      memory:  200Mi
    Requests:
      cpu:        300m
      memory:     200Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mvkxt (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-mvkxt:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m11s  default-scheduler  Successfully assigned qos-example/qos-demo-guaranteed to minikube-m03
  Normal  Pulling    2m7s   kubelet            Pulling image "nginx"
  Normal  Pulled     2m4s   kubelet            Successfully pulled image "nginx" in 2.746371427s
  Normal  Created    2m4s   kubelet            Created container qos-demo-guaranteed
  Normal  Started    2m3s   kubelet            Started container qos-demo-guaranteed

```

In this example, the Container has a memory limit and a memory request, both equal to 200 MiB. The Container has a CPU limit and a CPU request, both equal to 300 milliCPU, meeting the conditions for `Guaranteed` QoS Class.

This line from the description confirms the same.

`QoS Class:                   Guaranteed`



## Pod with two containers

```yaml
pradeep@learnk8s$ cat mixed-qos.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-mixed-qos
  namespace: qos-example
spec:
  containers:

  - name: qos-demo-mixed-1
    image: nginx
    resources:
      requests:
        memory: "200Mi"

  - name: qos-demo-mixed-2
    image: redis


```

Create the pod with two containers

```sh
pradeep@learnk8s$ kubectl create -f mixed-qos.yaml 
pod/qos-demo-mixed-qos created
```
List all the pods

```sh
pradeep@learnk8s$ kubectl get pods -n qos-example
NAME                  READY   STATUS    RESTARTS   AGE
qos-demo-besteffort   1/1     Running   0          18m
qos-demo-burstable    1/1     Running   0          13m
qos-demo-guaranteed   1/1     Running   0          7m38s
qos-demo-mixed-qos    2/2     Running   0          39s
```
Describe the latest pod with two containers and verify what QoS class is assgined to this Pod.

```sh
pradeep@learnk8s$ kubectl describe pod qos-demo-mixed-qos -n qos-example
Name:         qos-demo-mixed-qos
Namespace:    qos-example
Priority:     0
Node:         minikube-m02/172.16.30.7
Start Time:   Mon, 11 Apr 2022 08:50:27 +0530
Labels:       <none>
Annotations:  cni.projectcalico.org/containerID: 8cd859d87e66cc326fdaa1b443fd82591a699bdccca564b897c953ffca46a12d
              cni.projectcalico.org/podIP: 10.244.205.205/32
              cni.projectcalico.org/podIPs: 10.244.205.205/32
Status:       Running
IP:           10.244.205.205
IPs:
  IP:  10.244.205.205
Containers:
  qos-demo-mixed-1:
    Container ID:   docker://4c46c37e173185999ac60a0c857d19638e25fe6170b7be720110a56b1fa54b1a
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2275af0f20d71b293916f1958f8497f987b8d8fd8113df54635f2a5915002bf1
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 11 Apr 2022 08:50:33 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      memory:     200Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bvptc (ro)
  qos-demo-mixed-2:
    Container ID:   docker://a0b03ecae8aab3c35b0aa5167ebdbc919166ba5a8b49541394f33120bd2f29b1
    Image:          redis
    Image ID:       docker-pullable://redis@sha256:69a3ab2516b560690e37197b71bc61ba245aafe4525ebdece1d8a0bc5669e3e2
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 11 Apr 2022 08:50:44 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bvptc (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-bvptc:
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
  Normal  Scheduled  2m6s  default-scheduler  Successfully assigned qos-example/qos-demo-mixed-qos to minikube-m02
  Normal  Pulling    2m4s  kubelet            Pulling image "nginx"
  Normal  Pulled     2m1s  kubelet            Successfully pulled image "nginx" in 2.869276565s
  Normal  Created    2m1s  kubelet            Created container qos-demo-mixed-1
  Normal  Started    2m    kubelet            Started container qos-demo-mixed-1
  Normal  Pulling    2m    kubelet            Pulling image "redis"
  Normal  Pulled     110s  kubelet            Successfully pulled image "redis" in 9.722976428s
  Normal  Created    110s  kubelet            Created container qos-demo-mixed-2
  Normal  Started    109s  kubelet            Started container qos-demo-mixed-2
 
```

In this example Pod, one container specifies a memory request of 200 MiB. The other container does not specify any requests or limits. This Pod meets the criteria for QoS class `Burstable`. That is, it does not meet the criteria for QoS class `Guaranteed`, and one of its Containers has a memory request.

This line from the description confirms the same.
`QoS Class:                   Burstable`

## Clean up
Let us clean up all resources in this `qos-example` namespace.

```sh
pradeep@learnk8s$ kubectl delete ns qos-example
namespace "qos-example" deleted
```
```sh
pradeep@learnk8s$ kubectl get pods -n qos-example
No resources found in qos-example namespace.
```

This concludes our discussion on Kubernetes QoS.

