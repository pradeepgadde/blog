---
layout: single
title:  "Kubernetes Network Policy"
date:   2022-04-23 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/netpol-256.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

In our previous post on [Kubernetes Networking with Calico CNI](https://pradeepgadde.com/blog/kubernetes/2022/03/19/kubernetes-networking-calico.html) we looked into Calico CNI implemenation of Kubernetes networking and also we looked at an example network policy. In this post, let us take a look at another example of Kubernetes Network Policy.

To test the networking policy, let us create a simple deployment and expose it as a service.
```sh
pradeep@learnk8s$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
pradeep@learnk8s$ kubectl expose deployment nginx --port=80
service/nginx exposed
```

```sh
pradeep@learnk8s$ kubectl get all -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod/nginx-85b98978db-p4r8c   1/1     Running   0          28s   172.17.0.3   minikube   <none>           <none>

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   11h   <none>
service/nginx        ClusterIP   10.102.7.79   <none>        80/TCP    17s   app=nginx

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
deployment.apps/nginx   1/1     1            1           28s   nginx        nginx    app=nginx

NAME                               DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES   SELECTOR
replicaset.apps/nginx-85b98978db   1         1         1       28s   nginx        nginx    app=nginx,pod-template-hash=85b98978db
pradeep@learnk8s$ 
```
Now that a service is created, test it by accessing it from another Pod.

Create a Pod from busybox and launch the shell.

```sh
pradeep@learnk8s$ kubectl run busybox --rm -ti --image=busybox:1.28 -- /bin/sh 
If you don't see a command prompt, try pressing enter.
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
21: eth0@if22: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ #
```
Use `wget --spider` command to test the access to our initigial `nginx` pod service.

```sh
/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.102.7.79:80)
/ # ping nginx
PING nginx (10.102.7.79): 56 data bytes
^C
--- nginx ping statistics ---
7 packets transmitted, 0 packets received, 100% packet loss
```
While `wget --spider` does not download any files, test it again without the `--spider` option.

```sh
/ # cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
172.17.0.4	busybox
/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.102.7.79:80)
/ # wget nginx
Connecting to nginx (10.102.7.79:80)
index.html           100% |********************************************************|   615   0:00:00 ETA
/ # ls
bin         etc         index.html  root        tmp         var
dev         home        proc        sys         usr
/ # 
```
We can see that both Pods can communicate with each other, that is by default allowed.

Exit the busybox pod.
```sh
/ # exit
Session ended, resume using 'kubectl attach busybox -c busybox -i -t' command when the pod is running
pod "busybox" deleted
```
Exiting should cause the busybox Pod to be deleted also, because of the additional flags we have provided with the `kubectl run` command earlier.

```sh
pradeep@learnk8s$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-85b98978db-p4r8c   1/1     Running   0          7m46s
pradeep@learnk8s$ 
```

Now, let us limit this access by creating a Network Policy.

First verify if we have any network policies currently.

```sh
pradeep@learnk8s$ kubectl get netpol
No resources found in default namespace.
```

To limit the access to the `nginx` service so that only Pods with the label `access: true` can query it, create a NetworkPolicy object as follows:



```yaml
pradeep@learnk8s$ cat nginx-policy.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"
```

```sh
pradeep@learnk8s$ kubectl create -f nginx-policy.yaml 
networkpolicy.networking.k8s.io/access-nginx created
```

```sh
pradeep@learnk8s$ kubectl get netpol
NAME           POD-SELECTOR   AGE
access-nginx   app=nginx      4s
```

Describe our network policy
```sh
pradeep@learnk8s$ kubectl describe netpol
Name:         access-nginx
Namespace:    default
Created on:   2022-04-23 17:23:57 +0530 IST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=nginx
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      PodSelector: access=true
  Not affecting egress traffic
  Policy Types: Ingress
pradeep@learnk8s$ 
```

NetworkPolicy includes a `podSelector` which selects the grouping of Pods to which the policy applies. You can see this policy selects Pods with the label `app=nginx`. The label was automatically added to the Pod in the `nginx` Deployment. An empty podSelector selects all pods in the namespace.



Test access to the service when `access=true` label is not defined.

```sh
pradeep@learnk8s$ kubectl run busybox --rm -ti --image=busybox:1.28 -- /bin/sh

If you don't see a command prompt, try pressing enter.
/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.102.7.79:80)
/ # wget nginx
Connecting to nginx (10.102.7.79:80)
index.html           100% |********************************************************|   615   0:00:00 ETA
/ # 
```



We can see that, though we do not have the required label as per our current network policy definition, the pods are able to communicate with each other.

This is because our current network provider does not support the network policies. We started our minikube cluster with defaults, which does not support network policies.

```sh
pradeep@learnk8s$ minikube start 
😄  minikube v1.25.2 on Darwin 12.2.1
✨  Automatically selected the hyperkit driver
👍  Starting control plane node minikube in cluster minikube
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

```sh
pradeep@learnk8s$ kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS       AGE
default       nginx-85b98978db-p4r8c             1/1     Running   0              18m
kube-system   coredns-64897985d-srqsj            1/1     Running   5 (78m ago)    11h
kube-system   etcd-minikube                      1/1     Running   3 (78m ago)    11h
kube-system   kube-apiserver-minikube            1/1     Running   12             11h
kube-system   kube-controller-manager-minikube   1/1     Running   14 (72m ago)   11h
kube-system   kube-proxy-bsbvb                   1/1     Running   0              11h
kube-system   kube-scheduler-minikube            1/1     Running   5              11h
kube-system   storage-provisioner                1/1     Running   20 (33m ago)   11h
pradeep@learnk8s$
```



Let us stop and delete this cluster and launch it again with `Calico` CNI.

```sh
pradeep@learnk8s$ minikube stop
✋  Stopping node "minikube"  ...
🛑  1 node stopped.
pradeep@learnk8s$ minikube delete
🔥  Deleting "minikube" in hyperkit ...
💀  Removed all traces of the "minikube" cluster.
pradeep@learnk8s$ 
```



```sh
pradeep@learnk8s$ minikube start --cni=calico
😄  minikube v1.25.2 on Darwin 12.2.1
✨  Automatically selected the docker driver. Other choices: hyperkit, ssh
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
    > gcr.io/k8s-minikube/kicbase: 379.06 MiB / 379.06 MiB  100.00% 3.06 MiB p/
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring Calico (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
pradeep@learnk8s$ 

```

Look at the line `Configuring Calico (Container Networking Interface) ...`

```sh
pradeep@learnk8s$ kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-8594699699-npmdd   0/1     Running   0          3m7s
kube-system   calico-node-dk4f5                          1/1     Running   0          3m7s
kube-system   coredns-64897985d-qlzt6                    1/1     Running   0          3m7s
kube-system   etcd-minikube                              1/1     Running   0          3m18s
kube-system   kube-apiserver-minikube                    1/1     Running   0          3m18s
kube-system   kube-controller-manager-minikube           1/1     Running   0          3m18s
kube-system   kube-proxy-kzh2x                           1/1     Running   0          3m7s
kube-system   kube-scheduler-minikube                    1/1     Running   0          3m18s
kube-system   storage-provisioner                        1/1     Running   0          2m58s
pradeep@learnk8s$ 

```

As it is a new cluster, all of our previous work has gone! Let us build those two Pods and service once again quickly.



```sh
pradeep@learnk8s$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
```

```sh
pradeep@learnk8s$ kubectl expose deployment nginx --port=80
service/nginx exposed
```
Get all the resources with their labels.
```sh
pradeep@learnk8s$ kubectl get all --show-labels
NAME                         READY   STATUS    RESTARTS   AGE     LABELS
pod/nginx-85b98978db-t44nc   1/1     Running   0          4m32s   app=nginx,pod-template-hash=85b98978db

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     LABELS
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   21m     component=apiserver,provider=kubernetes
service/nginx        ClusterIP   10.96.213.189   <none>        80/TCP    4m23s   app=nginx

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
deployment.apps/nginx   1/1     1            1           4m32s   app=nginx

NAME                               DESIRED   CURRENT   READY   AGE     LABELS
replicaset.apps/nginx-85b98978db   1         1         1       4m32s   app=nginx,pod-template-hash=85b98978db
pradeep@learnk8s$ 
```

```sh
pradeep@learnk8s$ kubectl describe service nginx
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.213.189
IPs:               10.96.213.189
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.120.67:80
Session Affinity:  None
Events:            <none>
```

Apply the network policy.
```sh
pradeep@learnk8s$ kubectl create -f nginx-policy.yaml 
networkpolicy.networking.k8s.io/access-nginx created
```

```sh
pradeep@learnk8s$ kubectl describe netpol
Name:         access-nginx
Namespace:    default
Created on:   2022-04-23 17:58:00 +0530 IST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=nginx
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      PodSelector: access=true
  Not affecting egress traffic
  Policy Types: Ingress
pradeep@learnk8s$ 

```

Test now without the required matching labels applied.
```sh
pradeep@learnk8s$ kubectl run busybox --rm -ti --image=busybox:1.28 -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.96.213.189:80)
wget: download timed out
/ # wget nginx --timeout=1
Connecting to nginx (10.96.213.189:80)
wget: download timed out
/ # 
```
The network policy seems to be working.

Exit out of this Pod, and create a new pod, this time with matching labels.
```sh
/ # exit
Session ended, resume using 'kubectl attach busybox -c busybox -i -t' command when the pod is running
pod "busybox" deleted
```

```sh
pradeep@learnk8s$ kubectl run busybox --rm -ti --labels="access=true" --image=busybox:1.28 -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.96.213.189:80)
/ # wget nginx
Connecting to nginx (10.96.213.189:80)
index.html           100% |********************************************************|   615   0:00:00 ETA
/ # exit
Session ended, resume using 'kubectl attach busybox -c busybox -i -t' command when the pod is running
pod "busybox" deleted
pradeep@learnk8s$ 
```

This concludes our testing of a simple Kubernetes Network policy.
