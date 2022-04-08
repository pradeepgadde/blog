---
layout: single
title:  "Kubernetes Container Health"
date:   2022-04-04 10:57:04 +0530
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


# Kubernetes Container Health

Kubernetes makes use of probes to determine if a container is healthy or not.

There are different type of probes:

- Readiness Probes
- Liveness Probes
- Startup Probes



## Liveness Probes

Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides liveness probes to detect and remedy such situations.

Do you work? Yes , it is fine.

Do you work? No, let us restart!

There are many health verification methods

- exec.command
- httpGet
- tcpSocket
- gRPC

In this sample,  For the first 30 seconds of the container's life, there is a `/tmp/healthy` file. So during the first 30 seconds, the command `cat /tmp/healthy` returns a success code. After 30 seconds, `cat /tmp/healthy` returns a failure code.


```yaml
lab@k8s1:~$ cat liveness-probe-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness-probe-demo
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5

```

```sh
lab@k8s1:~$ kubectl create -f liveness-probe-demo.yaml
pod/liveness-exec created
lab@k8s1:~$
```

```sh

lab@k8s1:~$ kubectl describe pods liveness-exec
Name:         liveness-exec
Namespace:    default
Priority:     0
Node:         k8s3/10.210.40.175
Start Time:   Mon, 04 Apr 2022 20:21:53 -0700
Labels:       test=liveness
Annotations:  <none>
Status:       Running
IP:           10.244.3.36
IPs:
  IP:  10.244.3.36
Containers:
  liveness-probe-demo:
    Container ID:  docker://5f28058be1b5dd76f156edf5ca442b0e40686cba176b6c36e9a530572153d2c4
    Image:         k8s.gcr.io/busybox
    Image ID:      docker-pullable://k8s.gcr.io/busybox@sha256:d8d3bc2c183ed2f9f10e7258f84971202325ee6011ba137112e01e30f206de67
    Port:          <none>
    Host Port:     <none>
    Args:
      /bin/sh
      -c
      touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    State:          Running
      Started:      Mon, 04 Apr 2022 20:21:56 -0700
    Ready:          True
    Restart Count:  0
    Liveness:       exec [cat /tmp/healthy] delay=5s timeout=1s period=5s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r6znh (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-r6znh:
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
  Type     Reason     Age               From               Message

----     ------     ----              ----               -------
  Normal   Scheduled  53s               default-scheduler  Successfully assigned default/liveness-exec to k8s3
  Normal   Pulling    52s               kubelet            Pulling image "k8s.gcr.io/busybox"
  Normal   Pulled     50s               kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 1.857223923s
  Normal   Created    50s               kubelet            Created container liveness-probe-demo
  Normal   Started    50s               kubelet            Started container liveness-probe-demo
  Warning  Unhealthy  8s (x3 over 18s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    8s                kubelet            Container liveness-probe-demo failed liveness probe, will be restarted
lab@k8s1:~$

```

```sh
lab@k8s1:~$ kubectl get pods liveness-exec
NAME            READY   STATUS    RESTARTS     AGE
liveness-exec   1/1     Running   1 (4s ago)   79s
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl describe pods liveness-exec
Name:         liveness-exec
Namespace:    default
Priority:     0
Node:         k8s3/10.210.40.175
Start Time:   Mon, 04 Apr 2022 20:21:53 -0700
Labels:       test=liveness
Annotations:  <none>
Status:       Running
IP:           10.244.3.36
IPs:
  IP:  10.244.3.36
Containers:
  liveness-probe-demo:
    Container ID:  docker://97d24d601a4bac093ea55787438bedd84450776e6a629d94ddc702480a421ec2
    Image:         k8s.gcr.io/busybox
    Image ID:      docker-pullable://k8s.gcr.io/busybox@sha256:d8d3bc2c183ed2f9f10e7258f84971202325ee6011ba137112e01e30f206de67
    Port:          <none>
    Host Port:     <none>
    Args:
      /bin/sh
      -c
      touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    State:          Running
      Started:      Mon, 04 Apr 2022 20:23:09 -0700
    Last State:     Terminated
      Reason:       Error
      Exit Code:    137
      Started:      Mon, 04 Apr 2022 20:21:56 -0700
      Finished:     Mon, 04 Apr 2022 20:23:08 -0700
    Ready:          True
    Restart Count:  1
    Liveness:       exec [cat /tmp/healthy] delay=5s timeout=1s period=5s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r6znh (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-r6znh:
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
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  87s                default-scheduler  Successfully assigned default/liveness-exec to k8s3
  Normal   Pulled     84s                kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 1.857223923s
  Warning  Unhealthy  42s (x3 over 52s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    42s                kubelet            Container liveness-probe-demo failed liveness probe, will be restarted
  Normal   Pulling    12s (x2 over 86s)  kubelet            Pulling image "k8s.gcr.io/busybox"
  Normal   Created    11s (x2 over 84s)  kubelet            Created container liveness-probe-demo
  Normal   Started    11s (x2 over 84s)  kubelet            Started container liveness-probe-demo
  Normal   Pulled     11s                kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 902.140746ms
lab@k8s1:~$
```

At the bottom of the output, there are messages indicating that the liveness probes have failed, and the containers have been killed and recreated

```sh
lab@k8s1:~$ kubectl get pods liveness-exec
NAME            READY   STATUS    RESTARTS     AGE
liveness-exec   1/1     Running   2 (7s ago)   2m37s
lab@k8s1:~$
```
The output shows that `RESTARTS` has been incremented

```sh
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m39s                default-scheduler  Successfully assigned default/liveness-exec to k8s3
  Normal   Pulled     2m37s                kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 1.857223923s
  Normal   Pulled     84s                  kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 902.140746ms
  Warning  Unhealthy  40s (x6 over 2m5s)   kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    40s (x2 over 115s)   kubelet            Container liveness-probe-demo failed liveness probe, will be restarted
  Normal   Pulling    10s (x3 over 2m39s)  kubelet            Pulling image "k8s.gcr.io/busybox"
  Normal   Created    9s (x3 over 2m37s)   kubelet            Created container liveness-probe-demo
  Normal   Started    9s (x3 over 2m37s)   kubelet            Started container liveness-probe-demo
  Normal   Pulled     9s                   kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 449.070061ms
lab@k8s1:~$
```



## Readiness Probe

Are you Ready ? Yes, Incoming traffic accepted

Are you Ready? No, Incoming traffic is NOT accepted.

Unlike Liveness Probe, the Pod will NOT be Restated.

Readiness probes are configured similarly to liveness probes. The only difference is that you use the `readinessProbe` field instead of the `livenessProbe` field.

```yaml
lab@k8s1:~$ cat readiness-probe-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-exec
spec:
  containers:
  - name: readiness-probe-demo
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - sleep 20; touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl create -f readiness-probe-demo.yaml
pod/readiness-exec created
```

```sh
lab@k8s1:~$ kubectl get pods readiness-exec
NAME             READY   STATUS    RESTARTS   AGE
readiness-exec   1/1     Running   0          43s
```

```sh
lab@k8s1:~$ kubectl describe pods readiness-exec
Name:         readiness-exec
Namespace:    default
Priority:     0
Node:         k8s3/10.210.40.175
Start Time:   Mon, 04 Apr 2022 20:44:00 -0700
Labels:       test=readiness
Annotations:  <none>
Status:       Running
IP:           10.244.3.37
IPs:
  IP:  10.244.3.37
Containers:
  readiness-probe-demo:
    Container ID:  docker://658df52da674bebe6bafb35863b5973f1e8eefe3a52d7335513ddb3365047d84
    Image:         k8s.gcr.io/busybox
    Image ID:      docker-pullable://k8s.gcr.io/busybox@sha256:d8d3bc2c183ed2f9f10e7258f84971202325ee6011ba137112e01e30f206de67
    Port:          <none>
    Host Port:     <none>
    Args:
      /bin/sh
      -c
      sleep 20; touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    State:          Running
      Started:      Mon, 04 Apr 2022 20:44:01 -0700
    Ready:          True
    Restart Count:  0
    Readiness:      exec [cat /tmp/healthy] delay=5s timeout=1s period=5s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-s5r52 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-s5r52:
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
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  57s               default-scheduler  Successfully assigned default/readiness-exec to k8s3
  Normal   Pulling    57s               kubelet            Pulling image "k8s.gcr.io/busybox"
  Normal   Pulled     57s               kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 472.133563ms
  Normal   Created    57s               kubelet            Created container readiness-probe-demo
  Normal   Started    57s               kubelet            Started container readiness-probe-demo
  Warning  Unhealthy  3s (x4 over 48s)  kubelet            Readiness probe failed: cat: can't open '/tmp/healthy': No such file or directory
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get pods readiness-exec
NAME             READY   STATUS    RESTARTS   AGE
readiness-exec   0/1     Running   0          84s
lab@k8s1:~$
```

It is no longer in `Ready` state because of `Readiness probe failed: cat: can't open '/tmp/healthy': No such file or directory`.



```sh
lab@k8s1:~$ kubectl describe pods readiness-exec
Name:         readiness-exec
Namespace:    default
Priority:     0
Node:         k8s3/10.210.40.175
Start Time:   Mon, 04 Apr 2022 20:44:00 -0700
Labels:       test=readiness
Annotations:  <none>
Status:       Running
IP:           10.244.3.37
IPs:
  IP:  10.244.3.37
Containers:
  readiness-probe-demo:
    Container ID:  docker://658df52da674bebe6bafb35863b5973f1e8eefe3a52d7335513ddb3365047d84
    Image:         k8s.gcr.io/busybox
    Image ID:      docker-pullable://k8s.gcr.io/busybox@sha256:d8d3bc2c183ed2f9f10e7258f84971202325ee6011ba137112e01e30f206de67
    Port:          <none>
    Host Port:     <none>
    Args:
      /bin/sh
      -c
      sleep 20; touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    State:          Running
      Started:      Mon, 04 Apr 2022 20:44:01 -0700
    Ready:          False
    Restart Count:  0
    Readiness:      exec [cat /tmp/healthy] delay=5s timeout=1s period=5s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-s5r52 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-api-access-s5r52:
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
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  2m18s               default-scheduler  Successfully assigned default/readiness-exec to k8s3
  Normal   Pulling    2m17s               kubelet            Pulling image "k8s.gcr.io/busybox"
  Normal   Pulled     2m17s               kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 472.133563ms
  Normal   Created    2m17s               kubelet            Created container readiness-probe-demo
  Normal   Started    2m17s               kubelet            Started container readiness-probe-demo
  Warning  Unhealthy  8s (x21 over 2m8s)  kubelet            Readiness probe failed: cat: can't open '/tmp/healthy': No such file or directory
lab@k8s1:~$
```

We can see the condition of `  Ready             False` and ` Warning  Unhealthy  8s (x21 over 2m8s) `.

After some more time,

```sh
 Warning  Unhealthy  109s (x58 over 6m44s)  kubelet            Readiness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

```sh
lab@k8s1:~$ kubectl get pods readiness-exec
NAME             READY   STATUS    RESTARTS   AGE
readiness-exec   0/1     Running   0          6m42s
lab@k8s1:~$
```

The pod is still Running but not Ready, and there are NO (zero) Restarts.

