---
layout: single
title:  "Kubernetes Host Aliases"
date:   2022-04-22 10:55:04 +0530
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

# Kubernetes Host Aliases

We can add entries to a Pod's `/etc/hosts` file to provide Pod-level override of hostname resolution when DNS and other options are not applicable.  HostAliases field in PodSpec achieves this purpose.

Manual editing of `/etc/hosts` is not suggested because the file is managed by the `kubelet` and can be overwritten on during Pod creation/restart.

Before adding any host aliases, let us check the contents of `/etc/hosts` file of a container.

Create a busybox container and run the command to display the contents of the hosts file.

```sh
pradeep@learnk8s$ kubectl run busybox --image=busybox --command -- cat /etc/hosts
pod/busybox created
```
Verify Pod is created or not

```sh
pradeep@learnk8s$ kubectl get pods
NAME      READY   STATUS              RESTARTS   AGE
busybox   0/1     ContainerCreating   0          5s
```
It is still in `ContainerCreating` state, verify again
```sh
pradeep@learnk8s$ kubectl get pods
NAME      READY   STATUS      RESTARTS   AGE
busybox   0/1     Completed   0          7s
```
Now that we see `Completed` state, let us check the logs of this container.

```sh
pradeep@learnk8s$ kubectl logs busybox
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.244.151.18	busybox
```
As expected, in the logs, we do see the expected output, the `Kubernetes-managed hosts file`.

Apart from the default entries for the localhost, we do see one entry at the end, with an IP, `10.244.151.18	busybox` which is nothing but it's own IP address.

We can verify this with the `-o wide` option as well.

```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME      READY   STATUS             RESTARTS      AGE   IP              NODE           NOMINATED NODE   READINESS GATES
busybox   0/1     CrashLoopBackOff   1 (13s ago)   23s   10.244.151.18   minikube-m03   <none>           <none>
pradeep@learnk8s$ 

```
Now let us add some custom entries to this hosts file with the hostalias.
We can make use of the `kubectl explain pod.spec` command to get all the supportedd spec values, once we have the list, just explain the one we are interested in currently (hostAliases).

```sh
pradeep@learnk8s$ kubectl explain pod.spec.hostAliases 
KIND:     Pod
VERSION:  v1

RESOURCE: hostAliases <[]Object>

DESCRIPTION:
     HostAliases is an optional list of hosts and IPs that will be injected into
     the pod's hosts file if specified. This is only valid for non-hostNetwork
     pods.

     HostAlias holds the mapping between IP and hostnames that will be injected
     as an entry in the pod's hosts file.

FIELDS:
   hostnames	<[]string>
     Hostnames for the above IP address.

   ip	<string>
     IP address of the host file entry.

pradeep@learnk8s$ 
```
We can see it takes two fields, `hostnames` and `ip`.

Let us run the same command as earlier but with the `--dry-run=client` option along with `-o yaml` to get the YAML manifest, to which we can add the hostAliases.

```sh
pradeep@learnk8s$ kubectl run busybox --image=busybox --dry-run=client -o yaml --command -- cat /etc/hosts
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - cat
    - /etc/hosts
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
pradeep@learnk8s$
```
Let us save this output to a file named `pod-with-hostalias.yaml` 
```sh
pradeep@learnk8s$ kubectl run busybox --image=busybox --dry-run=client -o yaml --command -- cat /etc/hosts > pod-with-hostalias.yaml
```
Edit this file to add hostAliases. For example one for `kubernetes.io` and another for `pradeepgadde.com` with some random IPs.

```sh
pradeep@learnk8s$ vi pod-with-hostalias.yaml 
pradeep@learnk8s$ cat pod-with-hostalias.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox
  name: busybox
spec:
  hostAliases:
  - ip: "127.0.0.1"
    hostnames:
    - "pradeepgadde.com"
  - ip: "192.168.1.1"
    hostnames:
    - "kubernetes.io"
  containers:
  - command:
    - cat
    - /etc/hosts
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
Create a Pod from this manifest,
```sh
pradeep@learnk8s$ kubectl create -f pod-with-hostalias.yaml 
pod/busybox created
```
Verify that the Pod is created
```sh
pradeep@learnk8s$ kubectl get pods
NAME      READY   STATUS      RESTARTS   AGE
busybox   0/1     Completed   0          6s
```

As we see `Completed` state, let us check the logs.
```sh
pradeep@learnk8s$ kubectl logs busybox
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
172.17.0.3	busybox

# Entries added by HostAliases.
127.0.0.1	pradeepgadde.com
192.168.1.1	kubernetes.io
pradeep@learnk8s$ 
```

We can see that, now, there are two sections , apart from the previous entries, we do see a new section called `Entries added by HostAliases.`

Also, the entry for its own IP is still there, like earlier.

```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME      READY   STATUS             RESTARTS       AGE    IP           NODE       NOMINATED NODE   READINESS GATES
busybox   0/1     CrashLoopBackOff   5 (105s ago)   5m3s   172.17.0.3   minikube   <none>           <none>
```

This confirms that hostAliases in the Pod specification is working.
