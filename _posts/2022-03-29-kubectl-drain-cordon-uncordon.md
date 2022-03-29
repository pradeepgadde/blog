---
layout: single
title:  "Kubectl Drain, Cordon, and Uncordon"
date:   2022-03-29 14:55:04 +0530
categories: Kubernetes
tags: kubeadm
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/kubeadm.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubectl Drain, Cordon, and Uncordon



In our newly deployed Kubernetes cluster, verify if any resources present in the `default` namespace.

```sh
lab@k8s1:~$ kubectl get pods
No resources found in default namespace.
```

Let us create a deployment with four replicas.

```sh
lab@k8s1:~$ kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0 --replicas=4
deployment.apps/web created
lab@k8s1:~$
```

Verify the IP addresses and nodes assigned to this deployment.

```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4kqj6   1/1     Running   0          9s    10.244.3.2   k8s3   <none>           <none>
web-79d88c97d6-djfs9   1/1     Running   0          9s    10.244.2.3   k8s2   <none>           <none>
web-79d88c97d6-q4l2x   1/1     Running   0          9s    10.244.3.3   k8s3   <none>           <none>
web-79d88c97d6-xf4qm   1/1     Running   0          9s    10.244.2.2   k8s2   <none>           <none>
lab@k8s1:~$
```

Though, there are three nodes in our cluster, all the pods were created on only two nodes: `k8s2` and `k8s3`. This is because of Taints applied to the `control-plane` node.



```sh
lab@k8s1:~$ kubectl describe nodes | grep -e Taint -e Name:
Name:               k8s1
Taints:             node-role.kubernetes.io/master:NoSchedule
Name:               k8s2
Taints:             <none>
Name:               k8s3
Taints:             <none>
lab@k8s1:~$
```

We can see that, there is a `node-role.kubernetes.io/master:NoSchedule` Taint applied on the `k8s1` node which is preventing any pods from being scheduled on this node.



```sh
lab@k8s1:~$ kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    4/4     4            4           5m21s
lab@k8s1:~$
```

### Kubectl Drain

For maintenance reasons, we can take any node out of service, and move all the running applications to other nodes with the help of `kubectl drain` command.

```sh
lab@k8s1:~$ kubectl drain -h
Drain node in preparation for maintenance.

 The given node will be marked unschedulable to prevent new pods from arriving. 'drain' evicts the pods if the API
server supports https://kubernetes.io/docs/concepts/workloads/pods/disruptions/ . Otherwise, it will use normal DELETE
to delete the pods. The 'drain' evicts or deletes all pods except mirror pods (which cannot be deleted through the API
server).  If there are daemon set-managed pods, drain will not proceed without --ignore-daemonsets, and regardless it
will not delete any daemon set-managed pods, because those pods would be immediately replaced by the daemon set
controller, which ignores unschedulable markings.  If there are any pods that are neither mirror pods nor managed by a
replication controller, replica set, daemon set, stateful set, or job, then drain will not delete any pods unless you
use --force.  --force will also allow deletion to proceed if the managing resource of one or more pods is missing.

 'drain' waits for graceful termination. You should not operate on the machine until the command completes.

 When you are ready to put the node back into service, use kubectl uncordon, which will make the node schedulable again.

 https://kubernetes.io/images/docs/kubectl_drain.svg

Examples:
  # Drain node "foo", even if there are pods not managed by a replication controller, replica set, job, daemon set or
stateful set on it
  kubectl drain foo --force

  # As above, but abort if there are pods not managed by a replication controller, replica set, job, daemon set or
stateful set, and use a grace period of 15 minutes
  kubectl drain foo --grace-period=900

Options:
      --chunk-size=500: Return large lists in chunks rather than all at once. Pass 0 to disable. This flag is beta and
may change in the future.
      --delete-emptydir-data=false: Continue even if there are pods using emptyDir (local data that will be deleted when
the node is drained).
      --disable-eviction=false: Force drain to use delete, even if eviction is supported. This will bypass checking
PodDisruptionBudgets, use with caution.
      --dry-run='none': Must be "none", "server", or "client". If client strategy, only print the object that would be
sent, without sending it. If server strategy, submit server-side request without persisting the resource.
      --force=false: Continue even if there are pods not managed by a ReplicationController, ReplicaSet, Job, DaemonSet
or StatefulSet.
      --grace-period=-1: Period of time in seconds given to each pod to terminate gracefully. If negative, the default
value specified in the pod will be used.
      --ignore-daemonsets=false: Ignore DaemonSet-managed pods.
      --ignore-errors=false: Ignore errors occurred between drain nodes in group.
      --pod-selector='': Label selector to filter pods on the node
  -l, --selector='': Selector (label query) to filter on
      --skip-wait-for-delete-timeout=0: If pod DeletionTimestamp older than N seconds, skip waiting for the pod.
Seconds must be greater than 0 to skip.
      --timeout=0s: The length of time to wait before giving up, zero means infinite

Usage:
  kubectl drain NODE [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
lab@k8s1:~$
```

Let us drain the `k8s2` node.

```sh
lab@k8s1:~$ kubectl drain k8s2
node/k8s2 cordoned
DEPRECATED WARNING: Aborting the drain command in a list of nodes will be deprecated in v1.23.
The new behavior will make the drain command go through all nodes even if one or more nodes failed during the drain.
For now, users can try such experience via: --ignore-errors
error: unable to drain node "k8s2", aborting command...

There are pending nodes to be drained:
 k8s2
error: cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-system/kube-flannel-ds-ph9gg, kube-system/kube-proxy-cdl2t
lab@k8s1:~$
```
The drain command failed with  `error: cannot delete DaemonSet-managed Pods`.

We can still see the pods running on the `k8s2` node.

```sh
lab@k8s1:~$ kubectl get pods  -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4kqj6   1/1     Running   0          9m12s   10.244.3.2   k8s3   <none>           <none>
web-79d88c97d6-djfs9   1/1     Running   0          9m12s   10.244.2.3   k8s2   <none>           <none>
web-79d88c97d6-q4l2x   1/1     Running   0          9m12s   10.244.3.3   k8s3   <none>           <none>
web-79d88c97d6-xf4qm   1/1     Running   0          9m12s   10.244.2.2   k8s2   <none>           <none>
lab@k8s1:~$
```

The previous command (drain) output also suggests the solution.

```sh
lab@k8s1:~$ kubectl drain k8s2 --ignore-daemonsets
node/k8s2 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-ph9gg, kube-system/kube-proxy-cdl2t
evicting pod default/web-79d88c97d6-xf4qm
evicting pod default/web-79d88c97d6-djfs9
pod/web-79d88c97d6-djfs9 evicted
pod/web-79d88c97d6-xf4qm evicted
node/k8s2 evicted
lab@k8s1:~$
```
We can see that the node is `cordoned` and all pods are `evicted`.

```sh
lab@k8s1:~$ kubectl get pods  -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4kqj6   1/1     Running   0          12m   10.244.3.2   k8s3   <none>           <none>
web-79d88c97d6-hvdrs   1/1     Running   0          44s   10.244.3.5   k8s3   <none>           <none>
web-79d88c97d6-q4l2x   1/1     Running   0          12m   10.244.3.3   k8s3   <none>           <none>
web-79d88c97d6-rs5gk   1/1     Running   0          44s   10.244.3.4   k8s3   <none>           <none>
lab@k8s1:~$
```
Now, all four pods of our deployment are running only on the other node `k8s3`.  There are two new pods ( check the `AGE` column).

```sh
lab@k8s1:~$ kubectl describe node k8s2
Name:               k8s2
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s2
                    kubernetes.io/os=linux
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"92:01:5e:01:fe:01"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 10.210.40.173
                    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 29 Mar 2022 08:29:34 -0700
Taints:             node.kubernetes.io/unschedulable:NoSchedule
Unschedulable:      true
Lease:
  HolderIdentity:  k8s2
  AcquireTime:     <unset>
  RenewTime:       Tue, 29 Mar 2022 10:38:33 -0700
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Tue, 29 Mar 2022 08:29:40 -0700   Tue, 29 Mar 2022 08:29:40 -0700   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Tue, 29 Mar 2022 10:35:11 -0700   Tue, 29 Mar 2022 08:29:34 -0700   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Tue, 29 Mar 2022 10:35:11 -0700   Tue, 29 Mar 2022 08:29:34 -0700   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Tue, 29 Mar 2022 10:35:11 -0700   Tue, 29 Mar 2022 08:29:34 -0700   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Tue, 29 Mar 2022 10:35:11 -0700   Tue, 29 Mar 2022 08:29:44 -0700   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.210.40.173
  Hostname:    k8s2
Capacity:
  cpu:                2
  ephemeral-storage:  64320836Ki
  hugepages-2Mi:      0
  memory:             2025204Ki
  pods:               110
Allocatable:
  cpu:                2
  ephemeral-storage:  59278082360
  hugepages-2Mi:      0
  memory:             1922804Ki
  pods:               110
System Info:
  Machine ID:                 596c7f47034440028cd05f4d0fa9c753
  System UUID:                02cd2942-ef22-71c9-90d0-54187982487f
  Boot ID:                    ed911925-bdd7-4d98-aa4e-eaa3a689bad2
  Kernel Version:             5.13.0-37-generic
  OS Image:                   Ubuntu 20.04.4 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://20.10.14
  Kubelet Version:            v1.22.0
  Kube-Proxy Version:         v1.22.0
PodCIDR:                      10.244.2.0/24
PodCIDRs:                     10.244.2.0/24
Non-terminated Pods:          (2 in total)
  Namespace                   Name                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                     ------------  ----------  ---------------  -------------  ---
  kube-system                 kube-flannel-ds-ph9gg    100m (5%)     100m (5%)   50Mi (2%)        50Mi (2%)      129m
  kube-system                 kube-proxy-cdl2t         0 (0%)        0 (0%)      0 (0%)           0 (0%)         129m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                100m (5%)  100m (5%)
  memory             50Mi (2%)  50Mi (2%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:
  Type    Reason              Age    From     Message
  ----    ------              ----   ----     -------
  Normal  NodeNotSchedulable  5m31s  kubelet  Node k8s2 status is now: NodeNotSchedulable
lab@k8s1:~$
```

From the events section, it is confirmed that `k8s2`is  in `NodeNotSchedulable` state now.

### Kubectl Uncordon
Once the maintenance activity (on `k8s2` node ) is completed, we can change the status to Schedulable with the `kubectl uncordon` command.

```sh
lab@k8s1:~$ kubectl uncordon -h
Mark node as schedulable.

Examples:
  # Mark node "foo" as schedulable
  kubectl uncordon foo

Options:
      --dry-run='none': Must be "none", "server", or "client". If client strategy, only print the object that would be
sent, without sending it. If server strategy, submit server-side request without persisting the resource.
  -l, --selector='': Selector (label query) to filter on

Usage:
  kubectl uncordon NODE [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
lab@k8s1:~$
```

Let us uncordon the `k8s2` node.
```sh
lab@k8s1:~$ kubectl uncordon k8s2
node/k8s2 uncordoned
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get events
LAST SEEN   TYPE     REASON                    OBJECT                      MESSAGE
9m54s       Normal   NodeNotSchedulable        node/k8s2                   Node k8s2 status is now: NodeNotSchedulable
72s         Normal   NodeSchedulable           node/k8s2                   Node k8s2 status is now: NodeSchedulable
<ommitted>
```

Let us see our deployment status.
```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4kqj6   1/1     Running   0          19m     10.244.3.2   k8s3   <none>           <none>
web-79d88c97d6-hvdrs   1/1     Running   0          7m48s   10.244.3.5   k8s3   <none>           <none>
web-79d88c97d6-q4l2x   1/1     Running   0          19m     10.244.3.3   k8s3   <none>           <none>
web-79d88c97d6-rs5gk   1/1     Running   0          7m48s   10.244.3.4   k8s3   <none>           <none>
lab@k8s1:~$
```

All the pods are still running on the `k8s3` node only. This is because, existing pods will not be affected by `uncordon` action. If any new pods are created, they will be scheduled on the uncordoned node `k8s2`.

To test this, let us scale this deployment from four to six replicas.

```sh
lab@k8s1:~$ kubectl scale deployment web --replicas=6
deployment.apps/web scaled
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4kqj6   1/1     Running   0          21m   10.244.3.2   k8s3   <none>           <none>
web-79d88c97d6-6msxc   1/1     Running   0          18s   10.244.2.5   k8s2   <none>           <none>
web-79d88c97d6-hvdrs   1/1     Running   0          10m   10.244.3.5   k8s3   <none>           <none>
web-79d88c97d6-nktjl   1/1     Running   0          18s   10.244.2.4   k8s2   <none>           <none>
web-79d88c97d6-q4l2x   1/1     Running   0          21m   10.244.3.3   k8s3   <none>           <none>
web-79d88c97d6-rs5gk   1/1     Running   0          10m   10.244.3.4   k8s3   <none>           <none>
lab@k8s1:~$
```

We can see that both of the new pods are assigned to the `k8s2` node.
This confirms `uncordon` worked fine.

Let us try `drain` on the `k8s1` node.
```sh
lab@k8s1:~$ kubectl drain k8s1
node/k8s1 cordoned
DEPRECATED WARNING: Aborting the drain command in a list of nodes will be deprecated in v1.23.
The new behavior will make the drain command go through all nodes even if one or more nodes failed during the drain.
For now, users can try such experience via: --ignore-errors
error: unable to drain node "k8s1", aborting command...

There are pending nodes to be drained:
 k8s1
error: cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-system/kube-flannel-ds-lhcwb, kube-system/kube-proxy-brrvs
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl drain k8s1 --ignore-daemonsets
node/k8s1 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-lhcwb, kube-system/kube-proxy-brrvs
evicting pod kube-system/coredns-78fcd69978-bhc9k
evicting pod kube-system/coredns-78fcd69978-2s8w7
pod/coredns-78fcd69978-bhc9k evicted
pod/coredns-78fcd69978-2s8w7 evicted
node/k8s1 evicted
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS                     ROLES                  AGE     VERSION
k8s1   Ready,SchedulingDisabled   control-plane,master   3h32m   v1.22.0
k8s2   Ready                      <none>                 139m    v1.22.0
k8s3   Ready                      <none>                 48m     v1.22.0
lab@k8s1:~$
```



### Kubectl Cordon

```sh
lab@k8s1:~$ kubectl cordon -h
Mark node as unschedulable.

Examples:
  # Mark node "foo" as unschedulable
  kubectl cordon foo

Options:
      --dry-run='none': Must be "none", "server", or "client". If client strategy, only print the object that would be
sent, without sending it. If server strategy, submit server-side request without persisting the resource.
  -l, --selector='': Selector (label query) to filter on

Usage:
  kubectl cordon NODE [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl cordon k8s3
node/k8s3 cordoned
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS                     ROLES                  AGE     VERSION
k8s1   Ready,SchedulingDisabled   control-plane,master   3h36m   v1.22.0
k8s2   Ready                      <none>                 143m    v1.22.0
k8s3   Ready,SchedulingDisabled   <none>                 52m     v1.22.0
lab@k8s1:~$
```

We can see that the Scheduling is Disabled on both `k8s3` and `k8s1`. The difference between drain and cordon is that existing pods will not be evicted with the cordon command.



```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4kqj6   1/1     Running   0          27m     10.244.3.2   k8s3   <none>           <none>
web-79d88c97d6-6msxc   1/1     Running   0          6m19s   10.244.2.5   k8s2   <none>           <none>
web-79d88c97d6-hvdrs   1/1     Running   0          16m     10.244.3.5   k8s3   <none>           <none>
web-79d88c97d6-nktjl   1/1     Running   0          6m19s   10.244.2.4   k8s2   <none>           <none>
web-79d88c97d6-q4l2x   1/1     Running   0          27m     10.244.3.3   k8s3   <none>           <none>
web-79d88c97d6-rs5gk   1/1     Running   0          16m     10.244.3.4   k8s3   <none>           <none>
lab@k8s1:~$
```

On `k8s3` node which is cordoned, we still see the old pods in Running state.

Let us uncordon both of them.



```sh
lab@k8s1:~$ kubectl uncordon k8s1 k8s3
node/k8s1 uncordoned
node/k8s3 uncordoned
lab@k8s1:~$
```

```sh
 kubectl get nodes
NAME   STATUS   ROLES                  AGE     VERSION
k8s1   Ready    control-plane,master   3h39m   v1.22.0
k8s2   Ready    <none>                 146m    v1.22.0
k8s3   Ready    <none>                 55m     v1.22.0
lab@k8s1:~$
```

This concludes our discussion on kubectl drain, cordon, and uncordon utilities.

