---
layout: single
title:  "Create a K8s cluster with Kind"
date:   2022-05-23 11:55:04 +0530
categories: Kubernetes
tags: kind
author: "Pradeep Gadde"
classes: wide
show_date: true
header:
  teaser: /assets/images/kind.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
    text: "Checkout other topics"
    nav: my-sidebar
---

In the previous post, we just installed `kind` using the source code and looked at the help options.

Let us create our first cluster using `kind`.

```sh
pradeep:~$kind create cluster
ERROR: failed to create cluster: failed to list nodes: command "docker ps -a --filter label=io.x-k8s.kind.cluster=kind --format '{{.Names}}'" failed with error: exit status 1
Command Output: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
pradeep:~$
```

I have not started Docker Desktop yet, hence the above error.

Let us start it and re-run the same command.

```sh
pradeep:~$kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.24.0) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
pradeep:~$
```

```sh
pradeep:~$kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:65477
CoreDNS is running at https://127.0.0.1:65477/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
pradeep:~$
```
```sh
pradeep:~$kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   75s   v1.24.0
```
```
pradeep:~$kubectl get nodes -o wide
NAME                 STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION      CONTAINER-RUNTIME
kind-control-plane   Ready    control-plane   81s   v1.24.0   172.18.0.2    <none>        Ubuntu 21.10   5.10.104-linuxkit   containerd://1.6.4
pradeep:~$
```

Let us describe the `kind-control-plane` node.
```sh
pradeep:~$kubectl describe node
Name:               kind-control-plane
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kind-control-plane
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 23 May 2022 21:06:22 +0530
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  kind-control-plane
  AcquireTime:     <unset>
  RenewTime:       Mon, 23 May 2022 21:08:05 +0530
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Mon, 23 May 2022 21:06:44 +0530   Mon, 23 May 2022 21:06:22 +0530   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Mon, 23 May 2022 21:06:44 +0530   Mon, 23 May 2022 21:06:22 +0530   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Mon, 23 May 2022 21:06:44 +0530   Mon, 23 May 2022 21:06:22 +0530   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Mon, 23 May 2022 21:06:44 +0530   Mon, 23 May 2022 21:06:44 +0530   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  172.18.0.2
  Hostname:    kind-control-plane
Capacity:
  cpu:                4
  ephemeral-storage:  61255492Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             4028728Ki
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  61255492Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             4028728Ki
  pods:               110
System Info:
  Machine ID:                 7837ba6d6f7f45a8a6be3ec4fbdb9495
  System UUID:                ab26a89a-7b30-46ad-aca6-c9f11a6aca5d
  Boot ID:                    09c3d45f-d6d1-44ac-a72f-202aaf448f07
  Kernel Version:             5.10.104-linuxkit
  OS Image:                   Ubuntu 21.10
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.4
  Kubelet Version:            v1.24.0
  Kube-Proxy Version:         v1.24.0
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
ProviderID:                   kind://docker/kind/kind-control-plane
Non-terminated Pods:          (9 in total)
  Namespace                   Name                                          CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                          ------------  ----------  ---------------  -------------  ---
  kube-system                 coredns-6d4b75cb6d-jddfp                      100m (2%)     0 (0%)      70Mi (1%)        170Mi (4%)     95s
  kube-system                 coredns-6d4b75cb6d-xxg4c                      100m (2%)     0 (0%)      70Mi (1%)        170Mi (4%)     95s
  kube-system                 etcd-kind-control-plane                       100m (2%)     0 (0%)      100Mi (2%)       0 (0%)         107s
  kube-system                 kindnet-gqshf                                 100m (2%)     100m (2%)   50Mi (1%)        50Mi (1%)      95s
  kube-system                 kube-apiserver-kind-control-plane             250m (6%)     0 (0%)      0 (0%)           0 (0%)         105s
  kube-system                 kube-controller-manager-kind-control-plane    200m (5%)     0 (0%)      0 (0%)           0 (0%)         105s
  kube-system                 kube-proxy-fs4h5                              0 (0%)        0 (0%)      0 (0%)           0 (0%)         95s
  kube-system                 kube-scheduler-kind-control-plane             100m (2%)     0 (0%)      0 (0%)           0 (0%)         105s
  local-path-storage          local-path-provisioner-9cd9bd544-cdr6d        0 (0%)        0 (0%)      0 (0%)           0 (0%)         95s
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                950m (23%)  100m (2%)
  memory             290Mi (7%)  390Mi (9%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-1Gi      0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:
  Type    Reason                   Age                    From        Message
  ----    ------                   ----                   ----        -------
  Normal  Starting                 91s                    kube-proxy  
  Normal  NodeHasSufficientMemory  2m17s (x8 over 2m17s)  kubelet     Node kind-control-plane status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    2m17s (x8 over 2m17s)  kubelet     Node kind-control-plane status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     2m17s (x7 over 2m17s)  kubelet     Node kind-control-plane status is now: NodeHasSufficientPID
  Normal  Starting                 106s                   kubelet     Starting kubelet.
  Normal  NodeHasSufficientMemory  106s                   kubelet     Node kind-control-plane status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    106s                   kubelet     Node kind-control-plane status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     106s                   kubelet     Node kind-control-plane status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  105s                   kubelet     Updated Node Allocatable limit across pods
  Normal  NodeReady                85s                    kubelet     Node kind-control-plane status is now: NodeReady
pradeep:~$
```
Let us look at Pods
```sh
pradeep:~$kubectl get pods
No resources found in default namespace.
```
From all namespaces, this time.
```sh
pradeep:~$kubectl get pods -A
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-6d4b75cb6d-jddfp                     1/1     Running   0          2m33s
kube-system          coredns-6d4b75cb6d-xxg4c                     1/1     Running   0          2m33s
kube-system          etcd-kind-control-plane                      1/1     Running   0          2m45s
kube-system          kindnet-gqshf                                1/1     Running   0          2m33s
kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          2m43s
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          2m43s
kube-system          kube-proxy-fs4h5                             1/1     Running   0          2m33s
kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          2m43s
local-path-storage   local-path-provisioner-9cd9bd544-cdr6d       1/1     Running   0          2m33s
pradeep:~$
```

We can see it is very similar to the cluster setup via minikube.

Use the `kind get clusters` command, to see that we have one cluster named `kind`.

```sh
pradeep:~$kind get clusters
kind
pradeep:~$
```

Let us create a test nginx pod.
```sh
pradeep:~$kubectl run nginx --image=nginx
pod/nginx created
pradeep:~$
```

```sh
pradeep:~$kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          39s   10.244.0.5   kind-control-plane   <none>           <none>
pradeep:~$
```

We can see that the pod is successfully Running on the `kind-control-plane` node.

Let us delete this cluster now

```sh
pradeep:~$kind delete cluster
Deleting cluster "kind" ...
pradeep:~$
```

```sh
pradeep:~$kind get clusters
No kind clusters found.
pradeep:~$
```

This concludes our first test with `kind`.


