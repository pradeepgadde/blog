---
layout: single
title:  "Kubernetes Security Contexts"
date:   2022-02-19 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/psp-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Security Contexts


### Security Context

To understand security context, first let us try to create a Pod without any security context and  check few things like, `whoami`, `id`, and `ps` command outputs to verify the current user inside the Pod.

```yaml
pradeep@learnk8s$ cat no-security-context-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-security-context-demo
spec:

  containers:
  - name: no-security-context-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
```
```shell
pradeep@learnk8s$ kubectl create -f no-security-context-demo.yaml
pod/no-security-context-demo created
```
Login to the Pod, and launch the `shell`.

```shell
pradeep@learnk8s$ kubectl exec -it no-security-context-demo -- sh
/ # whoami
root
/ # id
uid=0(root) gid=0(root) groups=10(wheel)
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 sleep 1h
    9 root      0:00 sh
   18 root      0:00 ps
/ # exit 0
```
From all three command outputs, it is clear that it is the `root` user with id `0`.

Now, we can make use of a security context to define privilege and access control settings for a Pod or Container. For example, based on user ID (UID) and group ID (GID).

Let us modify the Pod defintion to include `securityContext` and create another Pod.

```yaml
pradeep@learnk8s$ cat security-context-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000

  containers:
  - name: security-context-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
```
```shell
pradeep@learnk8s$ kubectl create -f security-context-demo.yaml
pod/security-context-demo created
```
```shell
pradeep@learnk8s$ kubectl get pods | grep security
no-security-context-demo    1/1     Running            0                 9m24s
security-context-demo       1/1     Running            0                 31s
```
```shell
pradeep@learnk8s$ kubectl exec -it security-context-demo -- sh
/ $ whoami
whoami: unknown uid 1000
/ $ id
uid=1000 gid=0(root)
/ $ ps
PID   USER     TIME  COMMAND
    1 1000      0:00 sleep 1h
    7 1000      0:00 sh
   16 1000      0:00 ps
/ $ cat /etc/passwd
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
bin:x:2:2:bin:/bin:/bin/false
sys:x:3:3:sys:/dev:/bin/false
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/false
www-data:x:33:33:www-data:/var/www:/bin/false
operator:x:37:37:Operator:/var:/bin/false
nobody:x:65534:65534:nobody:/home:/bin/false
/ $
```
We can see the difference. Now, inside the Pod, its all the user with ID `1000` as defined in our security context.

As we are still inside the `security-context-demo` pod, let us check few things like `date` and try to modify the `date` value.
```shell
/ $ date
Sat Feb 19 01:06:08 UTC 2022
/ $ date -s 202206020937
date: can't set date: Operation not permitted
Thu Jun  2 09:37:00 UTC 2022
/ $ date
Sat Feb 19 01:08:11 UTC 2022
/ $
```
It says that `date: can't set date: Operation not permitted` and date did not change.

Let us create a new Pod definition to run the container as `root` user with `SYS_TIME` capability.

```yaml
pradeep@learnk8s$ cat security-context-cap-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-cap
spec:
  containers:
  - name: security-context-demo-cap
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      runAsUser: 0
      capabilities:
        add: ["SYS_TIME"]
```
```shell
pradeep@learnk8s$ kubectl create -f security-context-cap-demo.yaml
pod/security-context-demo-cap created
```
```shell
pradeep@learnk8s$ kubectl exec -it security-context-demo-cap -- sh
/ # id
uid=0(root) gid=0(root) groups=10(wheel)
/ # whoami
root
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 sleep 1h
    9 root      0:00 sh
   18 root      0:00 ps
/ # date -s 202206020937
Thu Jun  2 09:37:00 UTC 2022
/ # date
Sat Feb 19 13:29:05 UTC 2022
```
This time, it is the root user and setting date worked fine (partially!). Important difference to observe is that there is no `Operation not permitted` warning. It seems to took the new date, but issuing a date again, still shows the current date. I haven't explored this further yet. 
