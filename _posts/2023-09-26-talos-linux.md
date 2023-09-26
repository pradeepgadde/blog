---
layout: single
title:  "Talos Linux Quick Start"
date:   2023-09-26 09:58:04 +0530
categories: Kubernetes
tags: Linux
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/talos.png
author:
  name     : "Talos"
  avatar   : "/assets/images/talos.png"

sidebar:
  - title: "Topics"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Talos Linux
According to their website (https://www.talos.dev/), Talos is a container optimized Linux distro; a reimagining of Linux for distributed systems such as Kubernetes. Designed to be as minimal as possible while still maintaining practicality. For these reasons, Talos has a number of features unique to it:

    it is immutable
    it is atomic
    it is ephemeral
    it is minimal
    it is secure by default
    it is managed via a single declarative configuration file and gRPC API

Talos can be deployed on container, cloud, virtualized, and bare metal platforms.

## Installation on MacOS

```
(base) pradeep:~$curl -sL https://talos.dev/install | sh

Downloading talosctl-darwin-amd64...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 74.1M  100 74.1M    0     0  1072k      0  0:01:10  0:01:10 --:--:-- 1686k
Download complete!

Validating checksum...
Checksum valid.

talosctl was successfully installed 🎉

Installed version:
Client:
	Tag:         v1.5.3
	SHA:         cb21c671
	Built:       
	Go version:  go1.20.8
	OS/Arch:     darwin/amd64
Server:

Now run:

  talosctl cluster create                  # install a local test cluster
  talosctl dashboard                       # If you have created a cluster, launch the dashboard

Looking for more? Visit https://talos.dev/latest

(base) pradeep:~$

```

## Create Cluster
Local Docker Cluster
The easiest way to try Talos is by using the CLI (`talosctl`) to create a cluster on a machine with docker installed.
```
(base) pradeep:~$talosctl cluster create
validating CIDR and reserving IPs
generating PKI and tokens
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
(base) pradeep:~$
```
After launching Docker

```
(base) pradeep:~$talosctl cluster create
validating CIDR and reserving IPs
generating PKI and tokens
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
(base) pradeep:~$talosctl cluster create
validating CIDR and reserving IPs
generating PKI and tokens
downloading ghcr.io/siderolabs/talos:v1.5.3
creating network talos-default
creating controlplane nodes
creating worker nodes
waiting for API
bootstrapping cluster
waiting for etcd to be healthy: OK
waiting for etcd members to be consistent across nodes: OK
waiting for etcd members to be control plane nodes: OK
waiting for apid to be ready: OK
waiting for all nodes memory sizes: OK
waiting for all nodes disk sizes: OK
waiting for kubelet to be healthy: OK
waiting for all nodes to finish boot sequence: OK
waiting for all k8s nodes to report: OK
waiting for all k8s nodes to report ready: OK
waiting for all control plane static pods to be running: OK
waiting for all control plane components to be ready: OK
waiting for kube-proxy to report ready: OK
waiting for coredns to report ready: OK
waiting for all k8s nodes to report schedulable: OK

merging kubeconfig into "/Users/pradeep/.kube/config"
PROVISIONER       docker
NAME              talos-default
NETWORK NAME      talos-default
NETWORK CIDR      10.5.0.0/24
NETWORK GATEWAY   10.5.0.1
NETWORK MTU       1500

NODES:

NAME                            TYPE           IP         CPU    RAM      DISK
/talos-default-controlplane-1   controlplane   10.5.0.2   2.00   2.1 GB   -
/talos-default-worker-1         worker         10.5.0.3   2.00   2.1 GB   -
(base) pradeep:~$

```

## Nodes

```
(base) pradeep:~$kubectl get nodes
NAME                           STATUS   ROLES           AGE     VERSION
talos-default-controlplane-1   Ready    control-plane   2m25s   v1.28.2
talos-default-worker-1         Ready    <none>          2m30s   v1.28.2
(base) pradeep:~$
```

## Node Details

```
(base) pradeep:~$kubectl get nodes -o wide
NAME                           STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION      CONTAINER-RUNTIME
talos-default-controlplane-1   Ready    control-plane   2m30s   v1.28.2   10.5.0.2      <none>        Talos (v1.5.3)   5.10.104-linuxkit   containerd://1.6.23
talos-default-worker-1         Ready    <none>          2m35s   v1.28.2   10.5.0.3      <none>        Talos (v1.5.3)   5.10.104-linuxkit   containerd://1.6.23
(base) pradeep:~$
```

## Pods

```
(base) pradeep:~$kubectl get pods -A
NAMESPACE     NAME                                                   READY   STATUS    RESTARTS        AGE
kube-system   coredns-78f679c54d-lmqzl                               1/1     Running   0               3m17s
kube-system   coredns-78f679c54d-pnbvv                               1/1     Running   0               3m16s
kube-system   kube-apiserver-talos-default-controlplane-1            1/1     Running   0               2m21s
kube-system   kube-controller-manager-talos-default-controlplane-1   1/1     Running   1 (3m34s ago)   2m16s
kube-system   kube-flannel-sprwf                                     1/1     Running   0               3m11s
kube-system   kube-flannel-stql2                                     1/1     Running   0               3m6s
kube-system   kube-proxy-4lt7z                                       1/1     Running   0               3m6s
kube-system   kube-proxy-7g6qc                                       1/1     Running   0               3m11s
kube-system   kube-scheduler-talos-default-controlplane-1            1/1     Running   2 (3m42s ago)   118s
(base) pradeep:~$
```

## All Resources

```
(base) pradeep:~$kubectl get all -A
NAMESPACE     NAME                                                       READY   STATUS    RESTARTS        AGE
kube-system   pod/coredns-78f679c54d-lmqzl                               1/1     Running   0               4m8s
kube-system   pod/coredns-78f679c54d-pnbvv                               1/1     Running   0               4m7s
kube-system   pod/kube-apiserver-talos-default-controlplane-1            1/1     Running   0               3m12s
kube-system   pod/kube-controller-manager-talos-default-controlplane-1   1/1     Running   1 (4m25s ago)   3m7s
kube-system   pod/kube-flannel-sprwf                                     1/1     Running   0               4m2s
kube-system   pod/kube-flannel-stql2                                     1/1     Running   0               3m57s
kube-system   pod/kube-proxy-4lt7z                                       1/1     Running   0               3m57s
kube-system   pod/kube-proxy-7g6qc                                       1/1     Running   0               4m2s
kube-system   pod/kube-scheduler-talos-default-controlplane-1            1/1     Running   2 (4m33s ago)   2m49s

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  4m22s
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   4m19s

NAMESPACE     NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/kube-flannel   2         2         2       2            2           <none>          4m23s
kube-system   daemonset.apps/kube-proxy     2         2         2       2            2           <none>          4m23s

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           4m20s

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-78f679c54d   2         2         2       4m8s
(base) pradeep:~$
```

## Dashboard

Use the`talosctl dashboard --nodes 10.5.0.2`command to launch the dashboard

```
talos-default-controlplane-1 (v1.5.3): uptime 12m17s, 4x1.1GHz, 6.3 GiB RAM, PROCS 32, CPU 7.0%, RAM 17.1%                                                                                                  
                                                                                                                                                                                                            
 UUID     n/a                                                                KUBERNETES         v1.28.2                         HOST         talos-default-controlplane-1                                   
 CLUSTER  talos-default                                                      KUBELET            Healthy                         IP           10.5.0.2/24                                                    
 STAGE    Running                                                            APISERVER          Healthy                         GW           10.5.0.1                                                       
 READY    True                                                               CONTROLLER-MANAGER Healthy                         CONNECTIVITY OK                                                             
 TYPE     controlplane                                                       SCHEDULER          Healthy                         DNS          8.8.8.8, 1.1.1.1                                               
 MACHINES 2                                                                                                                     NTP          pool.ntp.org                                                   
                                                                                                                                                                                                            
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 kern: warning: [2023-09-26T16:38:49.102759265Z]: grpcfuse: loading out-of-tree module taints kernel.                                                                                                       
 kern:    info: [2023-09-26T16:38:49.109000265Z]: grpcfuse module loaded                                                                                                                                    
 kern:    info: [2023-09-26T16:38:49.230532265Z]: Initializing XFRM netlink socket                                                                                                                          
 kern:    info: [2023-09-26T16:40:12.117658265Z]: br-1dbae07fb205: port 1(veth1db8214) entered blocking state                                                                                               
 kern:    info: [2023-09-26T16:40:12.122748265Z]: br-1dbae07fb205: port 1(veth1db8214) entered disabled state                                                                                               
 kern:    info: [2023-09-26T16:40:12.126373265Z]: device veth1db8214 entered promiscuous mode                                                                                                               
 kern:    info: [2023-09-26T16:40:12.510525265Z]: IPVS: ftp: loaded support on port[0] = 21                                                                                                                 
 kern:    info: [2023-09-26T16:40:12.877848265Z]: eth0: renamed from vethdf27c89                                                                                                                            
 kern:    info: [2023-09-26T16:40:12.888682265Z]: IPv6: ADDRCONF(NETDEV_CHANGE): veth1db8214: link becomes ready                                                                                            
 kern:    info: [2023-09-26T16:40:12.892128265Z]: br-1dbae07fb205: port 1(veth1db8214) entered blocking state                                                                                               
 kern:    info: [2023-09-26T16:40:12.895316265Z]: br-1dbae07fb205: port 1(veth1db8214) entered forwarding state                                                                                             
 kern:    info: [2023-09-26T16:40:12.900208265Z]: IPv6: ADDRCONF(NETDEV_CHANGE): br-1dbae07fb205: link becomes ready                                                                                        
 kern:    info: [2023-09-26T16:40:13.165518265Z]: br-1dbae07fb205: port 2(vethacbd704) entered blocking state                                                                                               
 kern:    info: [2023-09-26T16:40:13.170225265Z]: br-1dbae07fb205: port 2(vethacbd704) entered disabled state                                                                                               
 kern:    info: [2023-09-26T16:40:13.175009265Z]: device vethacbd704 entered promiscuous mode                                                                                                               
 kern:    info: [2023-09-26T16:40:13.418805265Z]: wireguard: WireGuard 1.0.0 loaded. See www.wireguard.com for information.                                                                                 
 kern:    info: [2023-09-26T16:40:13.422771265Z]: IPVS: ftp: loaded support on port[0] = 21                                                                                                                 
 kern:    info: [2023-09-26T16:40:13.423811265Z]: wireguard: Copyright (C) 2015-2019 Jason A. Donenfeld <Jason@zx2c4.com>. All Rights Reserved.                                                             
 kern:    info: [2023-09-26T16:40:13.740982265Z]: eth0: renamed from veth4e21746                                                                                                                            
 kern:    info: [2023-09-26T16:40:13.754857265Z]: IPv6: ADDRCONF(NETDEV_CHANGE): vethacbd704: link becomes ready                                                                                            
 kern:    info: [2023-09-26T16:40:13.758341265Z]: br-1dbae07fb205: port 2(vethacbd704) entered blocking state                                                                                               
 kern:    info: [2023-09-26T16:40:13.761262265Z]: br-1dbae07fb205: port 2(vethacbd704) entered forwarding state                                                                                             
 kern: warning: [2023-09-26T16:41:32.429273265Z]: hrtimer: interrupt took 1815878 ns                                                                                                                        
 kern:    info: [2023-09-26T16:43:42.875983265Z]: IPVS: ftp: loaded support on port[0] = 21                                                                                                                 
 kern:    info: [2023-09-26T16:43:42.876008265Z]: IPVS: ftp: loaded support on port[0] = 21                                                                                                                 
 kern:    info: [2023-09-26T16:44:22.279664265Z]: IPVS: ftp: loaded support on port[0] = 21                                                                                                                 
 kern:    info: [2023-09-26T16:44:22.309419265Z]: cni0: port 1(veth0e4ab069) entered blocking state                                                                                                         
 kern:    info: [2023-09-26T16:44:22.312008265Z]: cni0: port 1(veth0e4ab069) entered disabled state                                                                                                         
 kern:    info: [2023-09-26T16:44:22.315546265Z]: device veth0e4ab069 entered promiscuous mode                                                                                                              
 kern:    info: [2023-09-26T16:44:22.331424265Z]: IPv6: ADDRCONF(NETDEV_CHANGE): eth0: link becomes ready                                                                                                   
 kern:    info: [2023-09-26T16:44:22.334249265Z]: IPv6: ADDRCONF(NETDEV_CHANGE): veth0e4ab069: link becomes ready                                                                                           
 kern:    info: [2023-09-26T16:44:22.337391265Z]: cni0: port 1(veth0e4ab069) entered blocking state                                                                                                         
 kern:    info: [2023-09-26T16:44:22.339934265Z]: cni0: port 1(veth0e4ab069) entered forwarding state                                                                                                       
 kern:    info: [2023-09-26T16:44:22.660903265Z]: IPVS: ftp: loaded support on port[0] = 21                                                                                                                 
 kern:    info: [2023-09-26T16:44:22.708492265Z]: cni0: port 2(vetha8a7b83d) entered blocking state                                                                                                         
 kern:    info: [2023-09-26T16:44:22.711941265Z]: cni0: port 2(vetha8a7b83d) entered disabled state                                                                                                         
 kern:    info: [2023-09-26T16:44:22.716504265Z]: device vetha8a7b83d entered promiscuous mode                                                                                                              
 kern:    info: [2023-09-26T16:44:22.719746265Z]: cni0: port 2(vetha8a7b83d) entered blocking state                                                                                                         
 kern:    info: [2023-09-26T16:44:22.723021265Z]: cni0: port 2(vetha8a7b83d) entered forwarding state                                                                                                       
 kern:    info: [2023-09-26T16:44:22.745331265Z]: IPv6: ADDRCONF(NETDEV_CHANGE): vetha8a7b83d: link becomes ready                                                                                           
                                                                                                                                                                                                            
[10.5.0.2] --- [Summary] --- [F2: Monitor]──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```



## Launch a workload

```
(base) pradeep:~$kubectl run nginx --image=nginx
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
pod/nginx created
(base) pradeep:~$
```



```
(base) pradeep:~$kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE                     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          80s   10.244.0.2   talos-default-worker-1   <none>           <none>
(base) pradeep:~$
```



## Destroy Cluster

```
(base) pradeep:~$talosctl cluster destroy

destroying node talos-default-controlplane-1
destroying node talos-default-worker-1
destroying network talos-default
(base) pradeep:~$
```

