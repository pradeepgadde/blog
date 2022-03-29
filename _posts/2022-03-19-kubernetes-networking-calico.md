---
layout: single
title:  "Kubernetes Networking with Calico CNI"
date:   2022-03-19 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
show_date: true
header:
  teaser: /assets/images/calico.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
  
    text: "Checkout other topics"
    nav: my-sidebar
---

Kubernetes Networking with Calico CNI

Let us build another minikube cluster, this time with the Calico CNI. Minikube supports many CNIs.

From the `minikube start -h` output, we can see the supported CNI plug-ins list.

```sh
 --cni='': CNI plug-in to use. Valid options: auto, bridge, calico, cilium, flannel, kindnet, or path to a CNI
manifest (default: auto)
```
Start the 3-node minikube cluster with `calico` CNI plugin.

```sh
pradeep@learnk8s$ minikube start --nodes=3 --cni=calico
😄  minikube v1.25.2 on Darwin 12.2.1
✨  Automatically selected the hyperkit driver
👍  Starting control plane node minikube in cluster minikube
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring Calico (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner

👍  Starting worker node minikube-m02 in cluster minikube
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ NO_PROXY=172.16.30.6
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ env NO_PROXY=172.16.30.6
🔎  Verifying Kubernetes components...

👍  Starting worker node minikube-m03 in cluster minikube
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ NO_PROXY=172.16.30.6,172.16.30.7
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ env NO_PROXY=172.16.30.6
    ▪ env NO_PROXY=172.16.30.6,172.16.30.7
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
pradeep@learnk8s$
```
Verify the cluster node IPs: `172.16.30.6`, `172.16.30.7`, and `172.16.30.8`.

```sh
pradeep@learnk8s$ kubectl get nodes -o wide
NAME           STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE              KERNEL-VERSION   CONTAINER-RUNTIME
minikube       Ready    control-plane,master   22m   v1.23.3   172.16.30.6   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
minikube-m02   Ready    <none>                 20m   v1.23.3   172.16.30.7   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
minikube-m03   Ready    <none>                 17m   v1.23.3   172.16.30.8   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
pradeep@learnk8s$
```

Verify the list of Pods in all namespaces.

```sh
pradeep@learnk8s$ kubectl get pods -A -o wide
NAMESPACE     NAME                                       READY   STATUS    RESTARTS        AGE     IP            NODE           NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-8594699699-dztlm   1/1     Running   0               6m50s   10.88.0.3     minikube       <none>           <none>
kube-system   calico-node-gqvw6                          1/1     Running   1 (2m54s ago)   4m38s   172.16.30.7   minikube-m02   <none>           <none>
kube-system   calico-node-qdbcf                          1/1     Running   0               6m50s   172.16.30.6   minikube       <none>           <none>
kube-system   calico-node-sw74l                          1/1     Running   0               114s    172.16.30.8   minikube-m03   <none>           <none>
kube-system   coredns-64897985d-58btq                    1/1     Running   0               6m50s   10.88.0.2     minikube       <none>           <none>
kube-system   etcd-minikube                              1/1     Running   0               7m1s    172.16.30.6   minikube       <none>           <none>
kube-system   kube-apiserver-minikube                    1/1     Running   0               7m1s    172.16.30.6   minikube       <none>           <none>
kube-system   kube-controller-manager-minikube           1/1     Running   0               7m1s    172.16.30.6   minikube       <none>           <none>
kube-system   kube-proxy-7k4lb                           1/1     Running   0               114s    172.16.30.8   minikube-m03   <none>           <none>
kube-system   kube-proxy-gm2dh                           1/1     Running   0               4m38s   172.16.30.7   minikube-m02   <none>           <none>
kube-system   kube-proxy-hvkqd                           1/1     Running   0               6m50s   172.16.30.6   minikube       <none>           <none>
kube-system   kube-scheduler-minikube                    1/1     Running   0               7m1s    172.16.30.6   minikube       <none>           <none>
kube-system   storage-provisioner                        1/1     Running   1 (2m26s ago)   6m47s   172.16.30.6   minikube       <none>           <none>
pradeep@learnk8s$
```
There are two Pods using the 10.88.0.X subnet (`calico-kube-controllers` and `coredns` Pods ).

Login to the `controlplane` node and check the CNI directory.
```sh
pradeep@learnk8s$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ cat /etc/cni/net.d/
.keep                      10-calico.conflist         87-podman-bridge.conflist  calico-kubeconfig
$ cat /etc/cni/net.d/10-calico.conflist
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "log_file_path": "/var/log/calico/cni/cni.log",
      "datastore_type": "kubernetes",
      "nodename": "minikube",
      "mtu": 0,
      "ipam": {
          "type": "calico-ipam"
      },
      "policy": {
          "type": "k8s"
      },
      "kubernetes": {
          "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "portmap",
      "snat": true,
      "capabilities": {"portMappings": true}
    },
    {
      "type": "bandwidth",
      "capabilities": {"bandwidth": true}
    }
  ]
}$
```
Apart from the Calico , there is a config file for Podman as well. Let us check that as well.

```sh
$ cat /etc/cni/net.d/87-podman-bridge.conflist
{
  "cniVersion": "0.4.0",
  "name": "podman",
  "plugins": [
    {
      "type": "bridge",
      "bridge": "cni-podman0",
      "isGateway": true,
      "ipMasq": true,
      "hairpinMode": true,
      "ipam": {
        "type": "host-local",
        "routes": [{ "dst": "0.0.0.0/0" }],
        "ranges": [
          [
            {
              "subnet": "10.88.0.0/16",
              "gateway": "10.88.0.1"
            }
          ]
        ]
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    },
    {
      "type": "firewall"
    },
    {
      "type": "tuning"
    }
  ]
}
$
```

From this, it is clear that while starting the minikube cluster, those two Pods got the IP from the Podman CNI, which seems to compete with the Calico CNI.

This issue is reported here: https://github.com/kubernetes/kubernetes/issues/107687 

Let us create a new Pod in the default namespace and check which IP will be obtained.

```sh
pradeep@learnk8s$ kubectl run nginx --image=nginx
pod/nginx created
```
Verify the IP of the new Pod and the node on which it got hosted.

```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          56s   10.244.205.193   minikube-m02   <none>           <none>
```

The new nginx pod got the IP address of `10.244.205.193` and is running on the `minikube-m02` node.

Let us log in to this `minikube-m02` node and verify the routing table and list of interfaces.

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
    link/ether 22:a7:1a:d9:be:42 brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.7/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 84337sec preferred_lft 84337sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:d3:aa:60:ec brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1480 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
    inet 10.244.205.192/32 scope global tunl0
       valid_lft forever preferred_lft forever
8: calic440f455693@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1480 qdisc noqueue state UP group default
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 0
$ ip route
default via 172.16.30.1 dev eth0 proto dhcp src 172.16.30.7 metric 1024
10.244.120.64/26 via 172.16.30.6 dev tunl0 proto bird onlink
10.244.151.0/26 via 172.16.30.8 dev tunl0 proto bird onlink
blackhole 10.244.205.192/26 proto bird
10.244.205.193 dev calic440f455693 scope link
172.16.30.0/24 dev eth0 proto kernel scope link src 172.16.30.7
172.16.30.1 dev eth0 proto dhcp scope link src 172.16.30.7 metric 1024
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
$
```

There are two special interfaces, `5: tunl0@NONE:` and `8: calic440f455693@if5:`. We can see that it is an `ipip` tunnel and tunnel interface (`tunl0`) has the IP address of `10.244.205.192/32`.  

From the routing table, there are two /26 routes pointing to the `tunl0` interface. The `10.244.120.64/26 via 172.16.30.6` and `10.244.151.0/26 via 172.16.30.8` entries in the routing table, shows  the `Calico` assigned subnets on the other two nodes, and the routes for those remote subnets via the `IPIP` tunnel interface.

From within the `minikube-m02` node, issue the `docker ps` command the retrieve the container ID of the `nginx` container.

```sh
$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS     NAMES
0d1f5d390956   nginx                  "/docker-entrypoint.…"   28 minutes ago   Up 28 minutes             k8s_nginx_nginx_default_5c5b022b-70d0-4e59-bbba-35a9bb43aa5c_0
6b67d9586b86   k8s.gcr.io/pause:3.6   "/pause"                 28 minutes ago   Up 28 minutes             k8s_POD_nginx_default_5c5b022b-70d0-4e59-bbba-35a9bb43aa5c_0
6f65ec9ee321   5ef66b403f4f           "start_runit"            39 minutes ago   Up 39 minutes             k8s_calico-node_calico-node-gqvw6_kube-system_6a835837-d36e-4366-aefe-cedc09d2f148_1
3d0be715b116   9b7cc9982109           "/usr/local/bin/kube…"   41 minutes ago   Up 41 minutes             k8s_kube-proxy_kube-proxy-gm2dh_kube-system_770d601e-7b93-42fa-8019-5976ae95684e_0
9c077d1f8374   k8s.gcr.io/pause:3.6   "/pause"                 41 minutes ago   Up 41 minutes             k8s_POD_calico-node-gqvw6_kube-system_6a835837-d36e-4366-aefe-cedc09d2f148_0
4b2360ad79a9   k8s.gcr.io/pause:3.6   "/pause"                 41 minutes ago   Up 41 minutes             k8s_POD_kube-proxy-gm2dh_kube-system_770d601e-7b93-42fa-8019-5976ae95684e_0
$
```

Get the `Pid` of the `nginx` container with the `docker inspect` command.
```sh
$ docker inspect 0d1f5d390956 | grep Pid
            "Pid": 11380,
            "PidMode": "",
            "PidsLimit": null,
$
```

With the `nsenter` command, verify the list of interfaces inside the `nginx` container.
```sh
$ sudo nsenter -t 11380 -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
3: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
5: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1480 qdisc noqueue state UP group default
    link/ether e6:ea:af:be:ec:53 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.205.193/32 brd 10.244.205.193 scope global eth0
       valid_lft forever preferred_lft forever
$
```

We can see that the Pod IP is configured on the `5: eth0@if8:` interface. From the `@if8`, we can see the link to the `8: calic440f455693@if5:` interface on the host (like the `veth` interfaces in the case of Kindnet CNI). 

Look at the routing table of the `nginx` pod.
```sh
$ sudo nsenter -t 11380 -n ip route
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0 scope link
$
```

Exit the `minikube-m02` node and use the `kubectl describe nodes` command and filter for the PodCIDRs.

```sh
pradeep@learnk8s$ kubectl describe nodes | grep -e Name -e PodCIDR
Name:               minikube
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
  Namespace                   Name                                        CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
Name:               minikube-m02
PodCIDR:                      10.244.1.0/24
PodCIDRs:                     10.244.1.0/24
  Namespace                   Name                 CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
Name:               minikube-m03
PodCIDR:                      10.244.2.0/24
PodCIDRs:                     10.244.2.0/24
  Namespace                   Name                 CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
pradeep@learnk8s$
```

Currently, there is only one pod (`nginx`) in the cluster. Similar to what we have verified on the `minikube-m02` node, do the same on the other two nodes as well.

From the `minikube` node:
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
    link/ether 9e:55:51:7f:e1:98 brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.6/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 82845sec preferred_lft 82845sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:92:37:51:98 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: cni-podman0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 36:85:e8:76:d2:1c brd ff:ff:ff:ff:ff:ff
    inet 10.88.0.1/16 brd 10.88.255.255 scope global cni-podman0
       valid_lft forever preferred_lft forever
6: veth5a561323@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni-podman0 state UP group default
    link/ether f6:67:49:87:51:36 brd ff:ff:ff:ff:ff:ff link-netnsid 0
7: vethb6ed82ba@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni-podman0 state UP group default
    link/ether 22:b5:fe:8a:37:59 brd ff:ff:ff:ff:ff:ff link-netnsid 1
8: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1480 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
    inet 10.244.120.64/32 scope global tunl0
       valid_lft forever preferred_lft forever
$ ip route
default via 172.16.30.1 dev eth0 proto dhcp src 172.16.30.6 metric 1024
10.88.0.0/16 dev cni-podman0 proto kernel scope link src 10.88.0.1
blackhole 10.244.120.64/26 proto bird
10.244.151.0/26 via 172.16.30.8 dev tunl0 proto bird onlink
10.244.205.192/26 via 172.16.30.7 dev tunl0 proto bird onlink
172.16.30.0/24 dev eth0 proto kernel scope link src 172.16.30.6
172.16.30.1 dev eth0 proto dhcp scope link src 172.16.30.6 metric 1024
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
$ exit
logout
pradeep@learnk8s$
```
On the `controlplane` node, there are extra interfaces coming from the `Podman CNI`.

From the `minikube-m03` node:
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
    link/ether de:b8:1d:5e:d9:c0 brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.8/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 83040sec preferred_lft 83040sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:66:0b:71:73 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1480 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
    inet 10.244.151.0/32 scope global tunl0
       valid_lft forever preferred_lft forever
$ ip route
default via 172.16.30.1 dev eth0 proto dhcp src 172.16.30.8 metric 1024
10.244.120.64/26 via 172.16.30.6 dev tunl0 proto bird onlink
blackhole 10.244.151.0/26 proto bird
10.244.205.192/26 via 172.16.30.7 dev tunl0 proto bird onlink
172.16.30.0/24 dev eth0 proto kernel scope link src 172.16.30.8
172.16.30.1 dev eth0 proto dhcp scope link src 172.16.30.8 metric 1024
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
$
```
On other nodes also, we do see similar `tunl0` interface and routes for Calico subnets (/26s) via the `IPIP` tunnel.
For the local subnet, there is a `blackhole` route: for example on the `minikube-m03` node `blackhole 10.244.151.0/26 proto bird`, and on `minikube-m02` node `blackhole 10.244.205.192/26 proto bird`, on the `minikube` node `blackhole 10.244.120.64/26 proto bird`.


Let us manually schedule a Pod on the `minikube-m03` node.
```yaml
pradeep@learnk8s$ cat my-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-manual
spec:
  nodeName: minikube-m03
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
NAME           READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
nginx          1/1     Running   0          53m   10.244.205.193   minikube-m02   <none>           <none>
nginx-manual   1/1     Running   0          39s   10.244.151.1     minikube-m03   <none>           <none>
pradeep@learnk8s$
```

As expected, the `nginx-manual` pod got an IP from the `10.244.151.0/26` subnet.

Log back to the `minikube-m03` node and check the routing table and list of interfaces.
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
    link/ether de:b8:1d:5e:d9:c0 brd ff:ff:ff:ff:ff:ff
    inet 172.16.30.8/24 brd 172.16.30.255 scope global dynamic eth0
       valid_lft 82414sec preferred_lft 82414sec
3: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:66:0b:71:73 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
5: tunl0@NONE: <NOARP,UP,LOWER_UP> mtu 1480 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
    inet 10.244.151.0/32 scope global tunl0
       valid_lft forever preferred_lft forever
8: califba6dd09590@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1480 qdisc noqueue state UP group default
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netnsid 0
$ ip route
default via 172.16.30.1 dev eth0 proto dhcp src 172.16.30.8 metric 1024
10.244.120.64/26 via 172.16.30.6 dev tunl0 proto bird onlink
blackhole 10.244.151.0/26 proto bird
10.244.151.1 dev califba6dd09590 scope link
10.244.205.192/26 via 172.16.30.7 dev tunl0 proto bird onlink
172.16.30.0/24 dev eth0 proto kernel scope link src 172.16.30.8
172.16.30.1 dev eth0 proto dhcp scope link src 172.16.30.8 metric 1024
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
$
```

Compared to the previous output (when there were no pods on this node), there is a new interface `8: califba6dd09590@if5:` and new route entry for the Pod IP `10.244.151.1 dev califba6dd09590 scope link`.

Get the container ID and Pid of this `nginx-manual` container.

```sh
$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED             STATUS             PORTS     NAMES
244b50fad9d4   nginx                  "/docker-entrypoint.…"   5 minutes ago       Up 5 minutes                 k8s_nginx_nginx-manual_default_ffbeaf88-b368-48af-b181-fc30cb49406a_0
e7117e519923   k8s.gcr.io/pause:3.6   "/pause"                 5 minutes ago       Up 5 minutes                 k8s_POD_nginx-manual_default_ffbeaf88-b368-48af-b181-fc30cb49406a_0
c4c993bc1f2e   calico/node            "start_runit"            About an hour ago   Up About an hour             k8s_calico-node_calico-node-sw74l_kube-system_0e30bbad-8370-4907-ab20-ce81450ad13c_0
a3482f106e3a   9b7cc9982109           "/usr/local/bin/kube…"   About an hour ago   Up About an hour             k8s_kube-proxy_kube-proxy-7k4lb_kube-system_f284e305-e71b-40b0-a715-796d0733bc03_0
25a914d158da   k8s.gcr.io/pause:3.6   "/pause"                 About an hour ago   Up About an hour             k8s_POD_calico-node-sw74l_kube-system_0e30bbad-8370-4907-ab20-ce81450ad13c_0
abce35d2a0ae   k8s.gcr.io/pause:3.6   "/pause"                 About an hour ago   Up About an hour             k8s_POD_kube-proxy-7k4lb_kube-system_f284e305-e71b-40b0-a715-796d0733bc03_0
$
```
```sh
$ docker inspect 244b50fad9d4 | grep Pid
            "Pid": 42125,
            "PidMode": "",
            "PidsLimit": null,
$
```

Use the `nsenter` command and verify the Pod interfaces and route table.
```sh
$ nsenter -t 42125 -n ip a
nsenter: cannot open /proc/42125/ns/net: Permission denied
$ sudo nsenter -t 42125 -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
3: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
5: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1480 qdisc noqueue state UP group default
    link/ether 9a:93:7b:77:8f:c7 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.151.1/32 brd 10.244.151.1 scope global eth0
       valid_lft forever preferred_lft forever
$
```
The `eth0` interface (`5: eth0@if8`) of the Pod has the IP address `10.244.151.1/32` and is mapped to the  new `cali` interface `8: califba6dd09590@if5:` on the host.
```sh
$ sudo nsenter -t 42125 -n ip route
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0 scope link
$
```
Verify communication between the two Pods. `10.244.205.193` is the IP address of the `nginx` pod running on the other node `minikube-m02`.

```sh
$ sudo nsenter -t 42125 -n ping 10.244.205.193
PING 10.244.205.193 (10.244.205.193): 56 data bytes
64 bytes from 10.244.205.193: seq=0 ttl=62 time=17.321 ms
64 bytes from 10.244.205.193: seq=1 ttl=62 time=0.963 ms
64 bytes from 10.244.205.193: seq=2 ttl=62 time=0.891 ms
64 bytes from 10.244.205.193: seq=3 ttl=62 time=1.529 ms
^C
--- 10.244.205.193 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.891/5.176/17.321 ms
$
```
Verify communication to the outside cluster hosts, like any internet host ( for example 8.8.8.8)

```sh
$ sudo nsenter -t 42125 -n ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=115 time=20.260 ms
64 bytes from 8.8.8.8: seq=1 ttl=115 time=15.457 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 15.457/17.858/20.260 ms
$
```
Let us install `calicoctl` as a Pod itself.
```sh
pradeep@learnk8s$ kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calicoctl.yaml
serviceaccount/calicoctl created
pod/calicoctl created
clusterrole.rbac.authorization.k8s.io/calicoctl created
clusterrolebinding.rbac.authorization.k8s.io/calicoctl created
pradeep@learnk8s$
```
Verify that the `calicoctl` pod is running.

```sh
pradeep@learnk8s$ kubectl get pods -n kube-system -o wide
NAME                                       READY   STATUS    RESTARTS      AGE   IP            NODE           NOMINATED NODE   READINESS GATES
calico-kube-controllers-8594699699-dztlm   1/1     Running   0             82m   10.88.0.3     minikube       <none>           <none>
calico-node-gqvw6                          1/1     Running   1 (78m ago)   80m   172.16.30.7   minikube-m02   <none>           <none>
calico-node-qdbcf                          1/1     Running   0             82m   172.16.30.6   minikube       <none>           <none>
calico-node-sw74l                          1/1     Running   0             77m   172.16.30.8   minikube-m03   <none>           <none>
calicoctl                                  1/1     Running   0             18s   172.16.30.7   minikube-m02   <none>           <none>
coredns-64897985d-58btq                    1/1     Running   0             82m   10.88.0.2     minikube       <none>           <none>
etcd-minikube                              1/1     Running   0             82m   172.16.30.6   minikube       <none>           <none>
kube-apiserver-minikube                    1/1     Running   0             82m   172.16.30.6   minikube       <none>           <none>
kube-controller-manager-minikube           1/1     Running   0             82m   172.16.30.6   minikube       <none>           <none>
kube-proxy-7k4lb                           1/1     Running   0             77m   172.16.30.8   minikube-m03   <none>           <none>
kube-proxy-gm2dh                           1/1     Running   0             80m   172.16.30.7   minikube-m02   <none>           <none>
kube-proxy-hvkqd                           1/1     Running   0             82m   172.16.30.6   minikube       <none>           <none>
kube-scheduler-minikube                    1/1     Running   0             82m   172.16.30.6   minikube       <none>           <none>
storage-provisioner                        1/1     Running   1 (77m ago)   82m   172.16.30.6   minikube       <none>           <none>
pradeep@learnk8s$
```

Verify the `calicoctl`

```sh
pradeep@learnk8s$ kubectl exec -ti -n kube-system calicoctl -- /calicoctl get nodes -o wide
Failed to get resources: Version mismatch.
Client Version:   v3.22.1
Cluster Version:  v3.20.0
Use --allow-version-mismatch to override.

command terminated with exit code 1
pradeep@learnk8s$
```

Repeat it with `--allow-version-mismatch`.
```sh
pradeep@learnk8s$ kubectl exec -ti -n kube-system calicoctl -- /calicoctl get nodes -o wide --allow-version-mismatch
NAME           ASN       IPV4             IPV6
minikube       (64512)   172.16.30.6/24
minikube-m02   (64512)   172.16.30.7/24
minikube-m03   (64512)   172.16.30.8/24

pradeep@learnk8s$
```

We can confirm that the `calicoctl` is working and we can see some BGP Autonomous systems shown as well. Currently all three nodes are part of the same ASN (`64512`).


Create an alias for the `calicoctl` 
```sh
pradeep@learnk8s$ alias calicoctl="kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch"
```

By default, calicoctl will attempt to read from the Kubernetes API using the default kubeconfig located at $(HOME)/.kube/config.

If the default kubeconfig does not exist, or you would like to specify alternative API access information, you can do so using the following configuration options.

```sh
pradeep@learnk8s$ DATASTORE_TYPE=kubernetes KUBECONFIG=~/.kube/config calicoctl get nodes
NAME
minikube
minikube-m02
minikube-m03

pradeep@learnk8s$
```

Verify all API-Resources installed by `Calico` CNI plugin.

```sh
pradeep@learnk8s$ kubectl api-resources | grep calico
bgpconfigurations                              crd.projectcalico.org/v1               false        BGPConfiguration
bgppeers                                       crd.projectcalico.org/v1               false        BGPPeer
blockaffinities                                crd.projectcalico.org/v1               false        BlockAffinity
clusterinformations                            crd.projectcalico.org/v1               false        ClusterInformation
felixconfigurations                            crd.projectcalico.org/v1               false        FelixConfiguration
globalnetworkpolicies                          crd.projectcalico.org/v1               false        GlobalNetworkPolicy
globalnetworksets                              crd.projectcalico.org/v1               false        GlobalNetworkSet
hostendpoints                                  crd.projectcalico.org/v1               false        HostEndpoint
ipamblocks                                     crd.projectcalico.org/v1               false        IPAMBlock
ipamconfigs                                    crd.projectcalico.org/v1               false        IPAMConfig
ipamhandles                                    crd.projectcalico.org/v1               false        IPAMHandle
ippools                                        crd.projectcalico.org/v1               false        IPPool
kubecontrollersconfigurations                  crd.projectcalico.org/v1               false        KubeControllersConfiguration
networkpolicies                                crd.projectcalico.org/v1               true         NetworkPolicy
networksets                                    crd.projectcalico.org/v1               true         NetworkSet
pradeep@learnk8s$
```
Verify some of these resources with `kubectl get` and `kubectl describe` commands.

```sh
pradeep@learnk8s$ kubectl get bgppeers -A
No resources found
```
```sh
pradeep@learnk8s$ kubectl get ippools
NAME                  AGE
default-ipv4-ippool   97m
pradeep@learnk8s$ kubectl describe ippools
Name:         default-ipv4-ippool
Namespace:
Labels:       <none>
Annotations:  projectcalico.org/metadata: {"uid":"cf94cf43-8887-4528-a934-ca498f0422e1","creationTimestamp":"2022-03-19T18:20:36Z"}
API Version:  crd.projectcalico.org/v1
Kind:         IPPool
Metadata:
  Creation Timestamp:  2022-03-19T18:20:36Z
  Generation:          1
  Managed Fields:
    API Version:  crd.projectcalico.org/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:projectcalico.org/metadata:
      f:spec:
        .:
        f:blockSize:
        f:cidr:
        f:ipipMode:
        f:natOutgoing:
        f:nodeSelector:
        f:vxlanMode:
    Manager:         Go-http-client
    Operation:       Update
    Time:            2022-03-19T18:20:36Z
  Resource Version:  659
  UID:               8d9e076a-dae6-4bbd-8f44-920da83472ba
Spec:
  Block Size:     26
  Cidr:           10.244.0.0/16
  Ipip Mode:      Always
  Nat Outgoing:   true
  Node Selector:  all()
  Vxlan Mode:     Never
Events:           <none>
pradeep@learnk8s$
```

```sh
pradeep@learnk8s$ kubectl get ipamblocks
NAME                AGE
10-244-120-64-26    98m
10-244-151-0-26     93m
10-244-205-192-26   95m
pradeep@learnk8s$
```
We have seen these IPAM blocks already, when we verified the routing tables on each node.

```sh
pradeep@learnk8s$ kubectl get networkpolicies
No resources found in default namespace.
```

Let us create a network policy now.

```yaml
pradeep@learnk8s$ cat network-policy.yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
pradeep@learnk8s$
```

```sh
pradeep@learnk8s$ kubectl create -f network-policy.yaml
networkpolicy.networking.k8s.io/default-deny-ingress created
```
Verify that the network policy is created.

```sh
pradeep@learnk8s$ kubectl get netpol
NAME                   POD-SELECTOR   AGE
default-deny-ingress   <none>         2s
```
Describe it for more details.
```sh
pradeep@learnk8s$ kubectl describe netpol
Name:         default-deny-ingress
Namespace:    default
Created on:   2022-03-20 01:45:29 +0530 IST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    <none> (Selected pods are isolated for ingress connectivity)
  Not affecting egress traffic
  Policy Types: Ingress
pradeep@learnk8s$ cat network-policy.yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
pradeep@learnk8s$
```
If you recall, we have two pods in our cluster at the moment.
```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME           READY   STATUS    RESTARTS   AGE    IP               NODE           NOMINATED NODE   READINESS GATES
nginx          1/1     Running   0          105m   10.244.205.193   minikube-m02   <none>           <none>
nginx-manual   1/1     Running   0          52m    10.244.151.1     minikube-m03   <none>           <none>
```

Now that the network policy is applied, let us check the connectivity between these two pods again.

```sh
pradeep@learnk8s$ minikube ssh -n minikube-m02
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ docker ps | grep nginx
0d1f5d390956   nginx                  "/docker-entrypoint.…"   2 hours ago      Up 2 hours                k8s_nginx_nginx_default_5c5b022b-70d0-4e59-bbba-35a9bb43aa5c_0
6b67d9586b86   k8s.gcr.io/pause:3.6   "/pause"                 2 hours ago      Up 2 hours                k8s_POD_nginx_default_5c5b022b-70d0-4e59-bbba-35a9bb43aa5c_0
$ docker inspect 0d1f5d390956 | grep Pid
            "Pid": 11380,
            "PidMode": "",
            "PidsLimit": null,
$ sudo nsenter -t 11380 -n ping 10.244.151.1
PING 10.244.151.1 (10.244.151.1): 56 data bytes
^C
--- 10.244.151.1 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
$
```
From `nginx` pod, we are not able to ping to `nginx-manual` pod.

Similarly, verify the other way, ping from `nginx-manual` to `nginx` Pod (which was working earlier, that we tested already!).
```sh
pradeep@learnk8s$ minikube ssh -n minikube-m03
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ docker ps | grep nginx
244b50fad9d4   nginx                  "/docker-entrypoint.…"   55 minutes ago   Up 55 minutes             k8s_nginx_nginx-manual_default_ffbeaf88-b368-48af-b181-fc30cb49406a_0
e7117e519923   k8s.gcr.io/pause:3.6   "/pause"                 55 minutes ago   Up 55 minutes             k8s_POD_nginx-manual_default_ffbeaf88-b368-48af-b181-fc30cb49406a_0
$ docker inspect 244b50fad9d4 | grep Pid
            "Pid": 42125,
            "PidMode": "",
            "PidsLimit": null,
$ sudo nsenter -t 42125 -n ping 10.244.205.193
PING 10.244.205.193 (10.244.205.193): 56 data bytes
^C
--- 10.244.205.193 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss
$ exit 0
logout
pradeep@learnk8s$
```

With network policy applied, the communication is blocked.

Let us create another network policy, this time to allow all ingress.

```yaml
pradeep@learnk8s$ cat allow-ingress.yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress

```
```sh
pradeep@learnk8s$ kubectl create -f allow-ingress.yaml
networkpolicy.networking.k8s.io/allow-all-ingress created
```

```sh
pradeep@learnk8s$ kubectl get netpol
NAME                   POD-SELECTOR   AGE
allow-all-ingress      <none>         5s
default-deny-ingress   <none>         11m
```

Describe the new policy.
```sh
pradeep@learnk8s$ kubectl describe netpol allow-all-ingress
Name:         allow-all-ingress
Namespace:    default
Created on:   2022-03-20 01:56:32 +0530 IST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From: <any> (traffic not restricted by source)
  Not affecting egress traffic
  Policy Types: Ingress
pradeep@learnk8s$
```

Now that we have allowed the ingress traffic to all pods/all ports.

Verify again. Now that we know the Pid of the containers, those steps are not needed.

```sh
pradeep@learnk8s$ minikube ssh -n minikube-m02
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ sudo nsenter -t 11380 -n ping 10.244.151.1
PING 10.244.151.1 (10.244.151.1): 56 data bytes
64 bytes from 10.244.151.1: seq=0 ttl=62 time=3.834 ms
64 bytes from 10.244.151.1: seq=1 ttl=62 time=1.409 ms
64 bytes from 10.244.151.1: seq=2 ttl=62 time=0.723 ms
^C
--- 10.244.151.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.723/1.988/3.834 ms
$ exit 0
logout
```
Yay! Ping is working again.

Similarly, verify in the other direction.
```sh
pradeep@learnk8s$ minikube ssh -n minikube-m03
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ sudo nsenter -t 42125 -n ping 10.244.205.193
PING 10.244.205.193 (10.244.205.193): 56 data bytes
64 bytes from 10.244.205.193: seq=0 ttl=62 time=1.132 ms
64 bytes from 10.244.205.193: seq=1 ttl=62 time=1.672 ms
64 bytes from 10.244.205.193: seq=2 ttl=62 time=1.695 ms
^C
--- 10.244.205.193 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 1.132/1.499/1.695 ms
$ exit 0
logout
```

This completes our initial discussion on the Calico CNI. We just scratched the surface of it, there are many more features, which will be discussed in other posts.

