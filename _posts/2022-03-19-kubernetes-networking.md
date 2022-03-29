---
layout: single
title:  "Kubernetes Networking with Minikube (Kindnet CNI)"
date:   2022-03-19 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
show_date: true
header:
  teaser: /assets/images/kindnet.jpeg
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
    
    text: "Checkout other topics"
    nav: my-sidebar
---

Hello!

Welcome to Kubernetes Networking.

Let us setup a fresh minikube cluster with default settings and with 3 nodes in it. 

```sh
pradeep@learnk8s$ minikube start --nodes=3
😄  minikube v1.25.2 on Darwin 12.2.1
✨  Automatically selected the hyperkit driver
💾  Downloading driver docker-machine-driver-hyperkit:
    > docker-machine-driver-hyper...: 65 B / 65 B [----------] 100.00% ? p/s 0s
    > docker-machine-driver-hyper...: 8.35 MiB / 8.35 MiB  100.00% 9.87 MiB p/s
🔑  The 'hyperkit' driver requires elevated permissions. The following commands will be executed:

    $ sudo chown root:wheel /Users/pradeep/.minikube/bin/docker-machine-driver-hyperkit
    $ sudo chmod u+s /Users/pradeep/.minikube/bin/docker-machine-driver-hyperkit


Password:
💿  Downloading VM boot image ...
    > minikube-v1.25.2.iso.sha256: 65 B / 65 B [-------------] 100.00% ? p/s 0s
    > minikube-v1.25.2.iso: 237.06 MiB / 237.06 MiB [] 100.00% 6.12 MiB p/s 39s
👍  Starting control plane node minikube in cluster minikube
💾  Downloading Kubernetes v1.23.3 preload ...
    > preloaded-images-k8s-v17-v1...: 505.68 MiB / 505.68 MiB  100.00% 9.37 MiB
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ kubelet.cni-conf-dir=/etc/cni/net.mk
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting worker node minikube-m02 in cluster minikube
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ NO_PROXY=172.16.30.3
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ env NO_PROXY=172.16.30.3
🔎  Verifying Kubernetes components...

👍  Starting worker node minikube-m03 in cluster minikube
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ NO_PROXY=172.16.30.3,172.16.30.4
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ env NO_PROXY=172.16.30.3
    ▪ env NO_PROXY=172.16.30.3,172.16.30.4
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
pradeep@learnk8s$
```
Our cluster nodes are assinged the IPs: `172.16.30.3`, `172.16.30.4`, and `172.16.30.5` respectively.

Also, note that the CNI config directory `kubelet.cni-conf-dir=/etc/cni/net.mk` and minikube is configuring CNI (Container Networking Interface) during the start.

```shell
pradeep@learnk8s$ kubectl get nodes -o wide
NAME           STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE              KERNEL-VERSION   CONTAINER-RUNTIME
minikube       Ready    control-plane,master   6m54s   v1.23.3   172.16.30.3   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
minikube-m02   Ready    <none>                 4m43s   v1.23.3   172.16.30.4   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
minikube-m03   Ready    <none>                 74s     v1.23.3   172.16.30.5   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
pradeep@learnk8s$
```
Let us take a look at all the pods in this newly deployed cluster. Currently, all the pods are in the `kube-system` namespace. The pods that are of interest to us in this post are the ones starting with the name `kindnet`.

So what is Kindnet? Kindnet is a simple CNI plugin for Kubernetes with IPv4 and IPv6 support that provides the Cluster Networking.

```sh
pradeep@learnk8s$ kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-64897985d-llrjh            1/1     Running   0          7m25s
kube-system   etcd-minikube                      1/1     Running   0          7m40s
kube-system   kindnet-4mrvw                      1/1     Running   0          2m2s
kube-system   kindnet-cpv4s                      1/1     Running   0          5m31s
kube-system   kindnet-xqjlm                      1/1     Running   0          7m26s
kube-system   kube-apiserver-minikube            1/1     Running   0          7m37s
kube-system   kube-controller-manager-minikube   1/1     Running   0          7m37s
kube-system   kube-proxy-b97w8                   1/1     Running   0          7m26s
kube-system   kube-proxy-gtlw7                   1/1     Running   0          2m2s
kube-system   kube-proxy-rts4b                   1/1     Running   0          5m31s
kube-system   kube-scheduler-minikube            1/1     Running   0          7m37s
kube-system   storage-provisioner                1/1     Running   1          7m35s
```

There are three kindnet pods, and if we check the IP address of these pods, all of them are having the same IP address as that of their node.

```sh
pradeep@learnk8s$ kubectl get pods -A -o wide | grep kindnet
kube-system   kindnet-4mrvw                      1/1     Running   0          6m17s   172.16.30.5   minikube-m03   <none>           <none>
kube-system   kindnet-cpv4s                      1/1     Running   0          9m46s   172.16.30.4   minikube-m02   <none>           <none>
kube-system   kindnet-xqjlm                      1/1     Running   0          11m     172.16.30.3   minikube       <none>           <none>
pradeep@learnk8s$
```
Let us login to the first node, and check its routing table and all the interfaces currently configured on this node.


```sh
pradeep@learnk8s$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether d6:df:bb:d6:c7:bc brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.3/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 85395sec preferred_lft 85395sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:df:e0:80:59 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: veth2b322de6@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 8e:09:87:56:8c:50 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.1/32 brd 10.244.0.1 scope global veth2b322de6
       valid_lft forever preferred_lft forever
$ ip route
default via 172.16.30.1 dev eth0 proto dhcp src 172.16.30.3 metric 1024
10.244.0.2 dev veth2b322de6 scope host
10.244.1.0/24 via 172.16.30.4 dev eth0
10.244.2.0/24 via 172.16.30.5 dev eth0
172.16.30.0/24 dev eth0 proto kernel scope link src 172.16.30.3
172.16.30.1 dev eth0 proto dhcp scope link src 172.16.30.3 metric 1024
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
```

This `minikube` node has the IP address of `172.16.30.3/24` on the `eth0` interface and  there is a static route to `10.244.1.0/24` with next-hop as the `minikube-m02` node with IP address of `172.16.30.4` and similarly another static route to `10.244.2.0/24` with next-hop as `172.16.30.5` which is the IP address of the the `minikube-m03` node.

There is another virtual interface (`veth`) called `veth2b322de6@if4` with an IP address of `10.244.0.1/32`. So what are these subnets `10.244.X.0/24` used for?

If we look at the `kindnet` CNI configuration file (located at the path given in the `minikube start` output shown above), on this `minikube` node,  we can see that there is a subnet `10.244.0.0/24` range defined.

```json
$ cat /etc/cni/net.mk/10-kindnet.conflist
{
	"cniVersion": "0.3.1",
	"name": "kindnet",
	"plugins": [
	{
		"type": "ptp",
		"ipMasq": false,
		"ipam": {
			"type": "host-local",
			"dataDir": "/run/cni-ipam-state",
			"routes": [


				{ "dst": "0.0.0.0/0" }
			],
			"ranges": [


				[ { "subnet": "10.244.0.0/24" } ]
			]
		}
		,
		"mtu": 1500

	},
	{
		"type": "portmap",
		"capabilities": {
			"portMappings": true
		}
	}
	]
}
```
Similarly, let us verify the same details in other two nodes of the cluster.

First, login to the `minikube-m02` node.

```sh
pradeep@learnk8s$ minikube ssh -n minikube-m02
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 2e:6c:52:09:b4:bb brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.4/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 84780sec preferred_lft 84780sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:b9:25:87:9f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
$ ip route
default via 172.16.30.1 dev eth0 proto dhcp src 172.16.30.4 metric 1024
10.244.0.0/24 via 172.16.30.3 dev eth0
10.244.2.0/24 via 172.16.30.5 dev eth0
172.16.30.0/24 dev eth0 proto kernel scope link src 172.16.30.4
172.16.30.1 dev eth0 proto dhcp scope link src 172.16.30.4 metric 1024
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
$ cat /etc/cni/net.mk/10-kindnet.conflist

{
	"cniVersion": "0.3.1",
	"name": "kindnet",
	"plugins": [
	{
		"type": "ptp",
		"ipMasq": false,
		"ipam": {
			"type": "host-local",
			"dataDir": "/run/cni-ipam-state",
			"routes": [


				{ "dst": "0.0.0.0/0" }
			],
			"ranges": [


				[ { "subnet": "10.244.1.0/24" } ]
			]
		}
		,
		"mtu": 1500

	},
	{
		"type": "portmap",
		"capabilities": {
			"portMappings": true
		}
	}
	]
}
$
```
We can see a similar setup. Local subnet range defined in the CNI (kindnet) configuration file is the `10.244.1.0/24` subnet and two static routes (`10.244.0.0/24` via `172.16.30.3` and another static route `10.244.2.0/24 via 172.16.30.5`).

Finally, let us check the same in the `minikube-m03` node.

```sh
pradeep@learnk8s$ minikube ssh -n minikube-m03
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether ce:26:e4:eb:95:56 brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.5/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 84738sec preferred_lft 84738sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:1a:9e:e4:0b brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
$ ip route
default via 172.16.30.1 dev eth0 proto dhcp src 172.16.30.5 metric 1024
10.244.0.0/24 via 172.16.30.3 dev eth0
10.244.1.0/24 via 172.16.30.4 dev eth0
172.16.30.0/24 dev eth0 proto kernel scope link src 172.16.30.5
172.16.30.1 dev eth0 proto dhcp scope link src 172.16.30.5 metric 1024
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
$ cat /etc/cni/net.mk/10-kindnet.conflist

{
	"cniVersion": "0.3.1",
	"name": "kindnet",
	"plugins": [
	{
		"type": "ptp",
		"ipMasq": false,
		"ipam": {
			"type": "host-local",
			"dataDir": "/run/cni-ipam-state",
			"routes": [


				{ "dst": "0.0.0.0/0" }
			],
			"ranges": [


				[ { "subnet": "10.244.2.0/24" } ]
			]
		}
		,
		"mtu": 1500

	},
	{
		"type": "portmap",
		"capabilities": {
			"portMappings": true
		}
	}
	]
}
$ exit
logout
```

Here also, very similar setup. `10.244.2.0/24` is the local subnet and remote subnets are cofnigured via static routes (`10.244.0.0/24 via 172.16.30.3` and `10.244.1.0/24 via 172.16.30.4`).

One thing to note is that, in both of the worker nodes, there isn't any `veth` interface present yet, unlike the `controleplane` node.

To understand, why let us get the IP addresses of all the running pods.

```sh
pradeep@learnk8s$ kubectl get pods -A -o wide
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
kube-system   coredns-64897985d-llrjh            1/1     Running   0          32m   10.244.0.2    minikube       <none>           <none>
kube-system   etcd-minikube                      1/1     Running   0          33m   172.16.30.3   minikube       <none>           <none>
kube-system   kindnet-4mrvw                      1/1     Running   0          27m   172.16.30.5   minikube-m03   <none>           <none>
kube-system   kindnet-cpv4s                      1/1     Running   0          31m   172.16.30.4   minikube-m02   <none>           <none>
kube-system   kindnet-xqjlm                      1/1     Running   0          33m   172.16.30.3   minikube       <none>           <none>
kube-system   kube-apiserver-minikube            1/1     Running   0          33m   172.16.30.3   minikube       <none>           <none>
kube-system   kube-controller-manager-minikube   1/1     Running   0          33m   172.16.30.3   minikube       <none>           <none>
kube-system   kube-proxy-b97w8                   1/1     Running   0          33m   172.16.30.3   minikube       <none>           <none>
kube-system   kube-proxy-gtlw7                   1/1     Running   0          27m   172.16.30.5   minikube-m03   <none>           <none>
kube-system   kube-proxy-rts4b                   1/1     Running   0          31m   172.16.30.4   minikube-m02   <none>           <none>
kube-system   kube-scheduler-minikube            1/1     Running   0          33m   172.16.30.3   minikube       <none>           <none>
kube-system   storage-provisioner                1/1     Running   1          33m   172.16.30.3   minikube       <none>           <none>
pradeep@learnk8s$
```
There is one Pod named `coredns-64897985d-llrjh` in the `kube-system` namespace with an IP address of `10.244.0.2` which is in the same subnet range defined in the CNI config on the `minikube` node.

Let us create our first pod (we can use any image, it does not matter). For test purposes, let us create a new pod using the `busybox` image.

```sh
pradeep@learnk8s$ kubectl run busybox --image=busybox
pod/busybox created
```

Verify the IP address assigned and the node on which it is running.
```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME      READY   STATUS      RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
busybox   0/1     Completed   0          12s   10.244.2.2   minikube-m03   <none>           <none>
pradeep@learnk8s$
```
We can see that the scheduler has assigned the `minikube-m03` node for this new pod and this `busybox` pod obtained its IP address `10.244.2.2` which is from the CNI assigned subnet (`10.244.2.0/24`) for this node.

Now, let us go back to the `minikube-m03` node and check the list of interfaces and see if there is anything new!

```sh
pradeep@learnk8s$ minikube ssh -n minikube-m03
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether ce:26:e4:eb:95:56 brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.5/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 83749sec preferred_lft 83749sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:1a:9e:e4:0b brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: veth07b7de9e@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 6a:9a:f7:af:ac:0f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.2.1/32 brd 10.244.2.1 scope global veth07b7de9e
       valid_lft forever preferred_lft forever
$
```
Compared to the initial setup, there is one new interface (#5, with the name `veth07b7de9e@if4`). This is very similar to the `controlplane` node now. 

This `veth` interface has the first IP address (`10.244.2.1/32`) from the CNI assigned subnet.

I tried to login to this container and check few things from inside the container, but before I do that the container crashed, so I have deleted it.
```sh
pradeep@learnk8s$ kubectl exec -it busybox -- /bin/bash
error: unable to upgrade connection: container not found ("busybox")
pradeep@learnk8s$ kubectl get pods
NAME      READY   STATUS             RESTARTS      AGE
busybox   0/1     CrashLoopBackOff   6 (63s ago)   7m3s
pradeep@learnk8s$ kubectl delete pod busybox
pod "busybox" deleted
```

Create another containter, this time using another image, `nginx` for test purposes.
```sh
pradeep@learnk8s$ kubectl run nginx --image=nginx
pod/nginx created
```

Verify the IP address of this new pod and the node on which it is running.

```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          25s   10.244.2.3   minikube-m03   <none>           <none>
pradeep@learnk8s$
```
Ah!, this pod also got assigned to the same node, `minikube-m03` and look at the IP address, the next IP address in the same range, `10.244.2.3`.


```sh
pradeep@learnk8s$ minikube ssh -n minikube-m03
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether ce:26:e4:eb:95:56 brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.5/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 83069sec preferred_lft 83069sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:1a:9e:e4:0b brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
6: veth37e46d8f@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 7e:96:f9:98:5f:4a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.2.1/32 brd 10.244.2.1 scope global veth37e46d8f
       valid_lft forever preferred_lft forever
$
```
From the routing table, we can see that the new Pod IP is routed via the `veth` interface `10.244.2.3 dev veth37e46d8f`.

```sh
$ ip route
default via 172.16.30.1 dev eth0 proto dhcp src 172.16.30.5 metric 1024
10.244.0.0/24 via 172.16.30.3 dev eth0
10.244.1.0/24 via 172.16.30.4 dev eth0
10.244.2.3 dev veth37e46d8f scope host
172.16.30.0/24 dev eth0 proto kernel scope link src 172.16.30.5
172.16.30.1 dev eth0 proto dhcp scope link src 172.16.30.5 metric 1024
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
$
```

Let us manually schedule a Pod on the `minikube-m02` node as well and observe.

```yaml
pradeep@learnk8s$ cat my-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-manual
spec:
  nodeName: minikube-m02
  containers:
  - image: nginx
    name: nginx
```
```sh
pradeep@learnk8s$ kubectl create -f my-pod.yaml
pod/nginx-manual created
```

```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
nginx-manual   1/1     Running   0          73s   10.244.1.2   minikube-m02   <none>           <none>
pradeep@learnk8s$
```
We can see that, this pod has been given an IP (`10.244.1.2`) from the 10.244.1.0/24 subnet allocated to `minikube-m02` node.

Let us login to this node and verify the list of currently running pods with the `docker ps` command.

```sh
pradeep@learnk8s$ minikube ssh -n minikube-m02
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS     NAMES
d6397d143118   nginx                  "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes             k8s_nginx_nginx-manual_default_950195d8-bc1d-420f-aede-fcd4a273f0e0_0
d2fb47206b71   k8s.gcr.io/pause:3.6   "/pause"                 4 minutes ago   Up 4 minutes             k8s_POD_nginx-manual_default_950195d8-bc1d-420f-aede-fcd4a273f0e0_0
ea4e0c180710   6de166512aa2           "/bin/kindnetd"          9 minutes ago   Up 9 minutes             k8s_kindnet-cni_kindnet-cpv4s_kube-system_5d3d3977-c5ff-4bea-917a-e0db52896da2_1
6ae7df286995   9b7cc9982109           "/usr/local/bin/kube…"   9 minutes ago   Up 9 minutes             k8s_kube-proxy_kube-proxy-rts4b_kube-system_9773fb68-a469-4cb2-827b-546076677b3b_1
d77c3e7c06e4   k8s.gcr.io/pause:3.6   "/pause"                 9 minutes ago   Up 9 minutes             k8s_POD_kindnet-cpv4s_kube-system_5d3d3977-c5ff-4bea-917a-e0db52896da2_1
5e18805db658   k8s.gcr.io/pause:3.6   "/pause"                 9 minutes ago   Up 9 minutes             k8s_POD_kube-proxy-rts4b_kube-system_9773fb68-a469-4cb2-827b-546076677b3b_1
$
```
Obtain the container ID for the `nginx-manual` pod. In this case ,it is `d6397d143118`.

Use the `docker inspect` command to get the PID of this container.

```sh
$ docker inspect d2fb47206b71 | grep Pid
            "Pid": 4200,
            "PidMode": "",
            "PidsLimit": null,
$ 
```
Now, take the `Pid` and use the `nsenter` command to issue the `ip a` command from inside this Pod namespace. This requires root privileges, so use `sudo` command.

```sh
$ sudo nsenter -t 4200 -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 52:a1:40:3b:54:18 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.1.2/24 brd 10.244.1.255 scope global eth0
       valid_lft forever preferred_lft forever
$
```

We can see that the Pod IP (`10.244.1.2`) is present on the interface ``#4` named `eth0@if5`.

Look at all the interfaces from this `minikube-m02` node.
```sh
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 2e:6c:52:09:b4:bb brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.4/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 85770sec preferred_lft 85770sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:41:3c:16:fe brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: vethfa21a27c@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether b2:26:8c:88:bf:d3 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.1.1/32 brd 10.244.1.1 scope global vethfa21a27c
       valid_lft forever preferred_lft forever
$
```
From the interface naming, we can see the link between the Pod namespace and the host.  The `4: eth0@if5:` interface with the IP address `10.244.1.2/24` is linked to the `5: vethfa21a27c@if4:` interface with the IP address `10.244.1.1/32`.

The name followed by the `@` symbol tells us the interface number. For example, inside the Pod namespace, it is shown as `@if5` for the interface number `4:`, this corresponds to interface number `5:` on the host, which is nothing but the `5: vethfa21a27c@if4:`. From this `5: vethfa21a27c@if4:` on the host, `@if4` corresponds to the interface number `4` on the Pod, which is the `eth0` interface indicated by `4: eth0@if5:`.

Now you can see the full picture of the internal connectivity. 

Just to confirm, this Pod hosted on `minikube-m02` can communicate with a Pod with IP address 10.244.0.2 hosted on the `minikube` node.
This can be confirmed by first looking at the Pod routing table, using the same `nsenter` command. There is a `default` route pointing to the gateway `10.244.1.1`.


```sh
$ sudo nsenter -t 4200 -n ip route
default via 10.244.1.1 dev eth0
10.244.1.0/24 via 10.244.1.1 dev eth0 src 10.244.1.2
10.244.1.1 dev eth0 scope link src 10.244.1.2
$
```

Ping to another Pod in the same cluster, but on another node.
```sh
$ sudo nsenter -t 4200 -n ping 10.244.0.2 -c 3
PING 10.244.0.2 (10.244.0.2): 56 data bytes
64 bytes from 10.244.0.2: seq=0 ttl=62 time=1.313 ms
64 bytes from 10.244.0.2: seq=1 ttl=62 time=1.980 ms
64 bytes from 10.244.0.2: seq=2 ttl=62 time=1.139 ms

--- 10.244.0.2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 1.139/1.477/1.980 ms
$
```

We can also see that using the same `default` route, this Pod can communicate with the outside cluster as well.

```sh
$ sudo nsenter -t 4200 -n ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=115 time=12.515 ms
64 bytes from 8.8.8.8: seq=1 ttl=115 time=12.238 ms
64 bytes from 8.8.8.8: seq=2 ttl=115 time=11.216 ms
64 bytes from 8.8.8.8: seq=3 ttl=115 time=11.303 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 11.216/11.818/12.515 ms
$
```

The following diagram summarizes our discussion so far.

![](/assets/images/k8s-minikube-kindnet.png)

This concludes our initial verification of the Kubernetes networking on the minikube with the default CNI (kindnet) which uses the simple `static` routes to facilitate communication across nodes. We will look at other CNIs later in other posts.
