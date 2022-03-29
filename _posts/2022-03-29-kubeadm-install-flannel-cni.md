---
layout: single
title:  "Install Flannel CNI in Kubeadm cluster"
date:   2022-03-29 11:25:04 +0530
categories: Kubernetes
tags: kubeadm
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/flannel.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
## Install Flannel CNI

```sh
lab@desktop:~$ kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
lab@desktop:~$
```
```sh
lab@desktop:~$ kubectl get pods -A
NAMESPACE     NAME                              READY   STATUS              RESTARTS      AGE
kube-system   coredns-78fcd69978-hkvsl          0/1     ContainerCreating   0             11m
kube-system   coredns-78fcd69978-rvtkt          0/1     ContainerCreating   0             11m
kube-system   etcd-desktop                      1/1     Running             1             11m
kube-system   kube-apiserver-desktop            1/1     Running             1             11m
kube-system   kube-controller-manager-desktop   1/1     Running             1             11m
kube-system   kube-flannel-ds-tqfl5             0/1     CrashLoopBackOff    4 (87s ago)   3m15s
kube-system   kube-proxy-ck8n2                  1/1     Running             0             11m
kube-system   kube-scheduler-desktop            1/1     Running             1             11m
lab@desktop:~$
```

```sh
lab@desktop:~$ kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
desktop   Ready    control-plane,master   13m   v1.22.0
lab@desktop:~$
```

```sh
lab@desktop:~$ kubectl describe nodes
Name:               desktop
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=desktop
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 29 Mar 2022 06:39:41 -0700
Taints:             node-role.kubernetes.io/master:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  desktop
  AcquireTime:     <unset>
  RenewTime:       Tue, 29 Mar 2022 06:51:50 -0700
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Tue, 29 Mar 2022 06:48:18 -0700   Tue, 29 Mar 2022 06:39:38 -0700   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Tue, 29 Mar 2022 06:48:18 -0700   Tue, 29 Mar 2022 06:39:38 -0700   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Tue, 29 Mar 2022 06:48:18 -0700   Tue, 29 Mar 2022 06:39:38 -0700   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Tue, 29 Mar 2022 06:48:18 -0700   Tue, 29 Mar 2022 06:48:08 -0700   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.210.40.172
  Hostname:    desktop
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
  System UUID:                18962942-410c-c225-65ca-e0a7de38d7cd
  Boot ID:                    79681989-c2a6-478c-bb51-7c46b1596fba
  Kernel Version:             5.13.0-37-generic
  OS Image:                   Ubuntu 20.04.4 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://20.10.14
  Kubelet Version:            v1.22.0
  Kube-Proxy Version:         v1.22.0
Non-terminated Pods:          (8 in total)
  Namespace                   Name                               CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                               ------------  ----------  ---------------  -------------  ---
  kube-system                 coredns-78fcd69978-hkvsl           100m (5%)     0 (0%)      70Mi (3%)        170Mi (9%)     12m
  kube-system                 coredns-78fcd69978-rvtkt           100m (5%)     0 (0%)      70Mi (3%)        170Mi (9%)     12m
  kube-system                 etcd-desktop                       100m (5%)     0 (0%)      100Mi (5%)       0 (0%)         12m
  kube-system                 kube-apiserver-desktop             250m (12%)    0 (0%)      0 (0%)           0 (0%)         12m
  kube-system                 kube-controller-manager-desktop    200m (10%)    0 (0%)      0 (0%)           0 (0%)         12m
  kube-system                 kube-flannel-ds-tqfl5              100m (5%)     100m (5%)   50Mi (2%)        50Mi (2%)      4m7s
  kube-system                 kube-proxy-ck8n2                   0 (0%)        0 (0%)      0 (0%)           0 (0%)         12m
  kube-system                 kube-scheduler-desktop             100m (5%)     0 (0%)      0 (0%)           0 (0%)         12m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                950m (47%)   100m (5%)
  memory             290Mi (15%)  390Mi (20%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)
Events:
  Type    Reason                   Age                From        Message
  ----    ------                   ----               ----        -------
  Normal  Starting                 12m                kube-proxy
  Normal  NodeHasSufficientMemory  12m (x6 over 12m)  kubelet     Node desktop status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    12m (x5 over 12m)  kubelet     Node desktop status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     12m (x5 over 12m)  kubelet     Node desktop status is now: NodeHasSufficientPID
  Normal  Starting                 12m                kubelet     Starting kubelet.
  Normal  NodeHasSufficientMemory  12m                kubelet     Node desktop status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    12m                kubelet     Node desktop status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     12m                kubelet     Node desktop status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  12m                kubelet     Updated Node Allocatable limit across pods
  Normal  NodeReady                3m48s              kubelet     Node desktop status is now: NodeReady
lab@desktop:~$ kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
desktop   Ready    control-plane,master   13m   v1.22.0
lab@desktop:~$
```

```sh
lab@desktop:~$ kubectl -n kube-system logs kube-flannel-ds-tqfl5
I0329 13:51:08.319758       1 main.go:205] CLI flags config: {etcdEndpoints:http://127.0.0.1:4001,http://127.0.0.1:2379 etcdPrefix:/coreos.com/network etcdKeyfile: etcdCertfile: etcdCAFile: etcdUsername: etcdPassword: version:false kubeSubnetMgr:true kubeApiUrl: kubeAnnotationPrefix:flannel.alpha.coreos.com kubeConfigFile: iface:[] ifaceRegex:[] ipMasq:true subnetFile:/run/flannel/subnet.env publicIP: publicIPv6: subnetLeaseRenewMargin:60 healthzIP:0.0.0.0 healthzPort:0 iptablesResyncSeconds:5 iptablesForwardRules:true netConfPath:/etc/kube-flannel/net-conf.json setNodeNetworkUnavailable:true}
W0329 13:51:08.319842       1 client_config.go:614] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0329 13:51:08.508927       1 kube.go:120] Waiting 10m0s for node controller to sync
I0329 13:51:08.509010       1 kube.go:378] Starting kube subnet manager
I0329 13:51:09.509083       1 kube.go:127] Node controller sync successful
I0329 13:51:09.509106       1 main.go:225] Created subnet manager: Kubernetes Subnet Manager - desktop
I0329 13:51:09.509112       1 main.go:228] Installing signal handlers
I0329 13:51:09.509281       1 main.go:454] Found network config - Backend type: vxlan
I0329 13:51:09.509301       1 match.go:189] Determining IP address of default interface
I0329 13:51:09.509761       1 match.go:242] Using interface with name eth0 and address 10.210.40.172
I0329 13:51:09.509777       1 match.go:264] Defaulting external address to interface address (10.210.40.172)
I0329 13:51:09.509853       1 vxlan.go:138] VXLAN config: VNI=1 Port=0 GBP=false Learning=false DirectRouting=false
E0329 13:51:09.510127       1 main.go:317] Error registering network: failed to acquire lease: node "desktop" pod cidr not assigned
I0329 13:51:09.510168       1 main.go:434] Stopping shutdownHandler...
lab@desktop:~$
```
Look at the error `Error registering network: failed to acquire lease: node "desktop" pod cidr not assigned`.

```sh
lab@k8s1:~$ kubectl get pods -A
NAMESPACE     NAME                              READY   STATUS              RESTARTS      AGE
kube-system   coredns-78fcd69978-hkvsl          0/1     ContainerCreating   0             15m
kube-system   coredns-78fcd69978-rvtkt          0/1     ContainerCreating   0             15m
kube-system   etcd-desktop                      1/1     Running             1             15m
kube-system   kube-apiserver-desktop            1/1     Running             1             15m
kube-system   kube-controller-manager-desktop   1/1     Running             1             15m
kube-system   kube-flannel-ds-tqfl5             0/1     CrashLoopBackOff    6 (57s ago)   7m2s
kube-system   kube-proxy-ck8n2                  1/1     Running             0             15m
kube-system   kube-scheduler-desktop            1/1     Running             1             15m
lab@k8s1:~$

```

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

Choose a Pod network add-on, and verify whether it requires any arguments to be passed to `kubeadm init`. Depending on which third-party provider you choose, you might need to set the `--pod-network-cidr` to a provider-specific value. See [Installing a Pod network add-on](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network).



So, let us reset the `kubeadm init` one more time and install the control plane node with `pod network cidr`

```sh
lab@k8s1:~$ sudo kubeadm reset
[sudo] password for lab:
[reset] Reading configuration from the cluster...
[reset] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
The 'update-cluster-status' phase is deprecated and will be removed in a future release. Currently it performs no operation
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[reset] Deleting contents of stateful directories: [/var/lib/etcd /var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
lab@k8s1:~$

```

```sh
lab@k8s1:~$ sudo kubeadm init --apiserver-advertise-address=192.168.100.1 --pod-network-cidr=10.100.0.0/16
I0329 07:02:10.164374  210845 version.go:255] remote version is much newer: v1.23.5; falling back to: stable-1.22
[init] Using Kubernetes version: v1.22.8
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.100.1]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s1 localhost] and IPs [192.168.100.1 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s1 localhost] and IPs [192.168.100.1 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 9.007047 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s1 as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 8ih52b.vedarospntvcyb4o
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.100.1:6443 --token 8ih52b.vedarospntvcyb4o \
	--discovery-token-ca-cert-hash sha256:babc25f29b9fbfab98a7d5fe6d96ce58e31649e022452b7b6c6b4282bfbf3f51
lab@k8s1:~$
```



```sh
lab@desktop:~$ mkdir -p $HOME/.kube
lab@desktop:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
cp: overwrite '/home/lab/.kube/config'? y
lab@desktop:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
lab@desktop:~$
```
```sh
lab@desktop:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE     VERSION
k8s1   Ready    control-plane,master   5m23s   v1.22.0
lab@desktop:~$
```

```sh
lab@desktop:~$ kubectl describe nodes
Name:               k8s1
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s1
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 29 Mar 2022 07:02:25 -0700
Taints:             node-role.kubernetes.io/master:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  k8s1
  AcquireTime:     <unset>
  RenewTime:       Tue, 29 Mar 2022 07:08:26 -0700
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Tue, 29 Mar 2022 07:07:42 -0700   Tue, 29 Mar 2022 07:02:22 -0700   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Tue, 29 Mar 2022 07:07:42 -0700   Tue, 29 Mar 2022 07:02:22 -0700   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Tue, 29 Mar 2022 07:07:42 -0700   Tue, 29 Mar 2022 07:02:22 -0700   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Tue, 29 Mar 2022 07:07:42 -0700   Tue, 29 Mar 2022 07:02:39 -0700   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.210.40.172
  Hostname:    k8s1
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
  System UUID:                18962942-410c-c225-65ca-e0a7de38d7cd
  Boot ID:                    79681989-c2a6-478c-bb51-7c46b1596fba
  Kernel Version:             5.13.0-37-generic
  OS Image:                   Ubuntu 20.04.4 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://20.10.14
  Kubelet Version:            v1.22.0
  Kube-Proxy Version:         v1.22.0
Non-terminated Pods:          (7 in total)
  Namespace                   Name                            CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                            ------------  ----------  ---------------  -------------  ---
  kube-system                 coredns-78fcd69978-fpm5b        100m (5%)     0 (0%)      70Mi (3%)        170Mi (9%)     5m53s
  kube-system                 coredns-78fcd69978-gkrfq        100m (5%)     0 (0%)      70Mi (3%)        170Mi (9%)     5m53s
  kube-system                 etcd-k8s1                       100m (5%)     0 (0%)      100Mi (5%)       0 (0%)         6m6s
  kube-system                 kube-apiserver-k8s1             250m (12%)    0 (0%)      0 (0%)           0 (0%)         6m6s
  kube-system                 kube-controller-manager-k8s1    200m (10%)    0 (0%)      0 (0%)           0 (0%)         6m8s
  kube-system                 kube-proxy-n6x56                0 (0%)        0 (0%)      0 (0%)           0 (0%)         5m54s
  kube-system                 kube-scheduler-k8s1             100m (5%)     0 (0%)      0 (0%)           0 (0%)         6m9s
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                850m (42%)   0 (0%)
  memory             240Mi (12%)  340Mi (18%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)
Events:
  Type    Reason                   Age                    From        Message
  ----    ------                   ----                   ----        -------
  Normal  Starting                 5m52s                  kube-proxy
  Normal  NodeAllocatableEnforced  6m17s                  kubelet     Updated Node Allocatable limit across pods
  Normal  Starting                 6m17s                  kubelet     Starting kubelet.
  Normal  NodeHasSufficientPID     6m16s (x3 over 6m17s)  kubelet     Node k8s1 status is now: NodeHasSufficientPID
  Normal  NodeHasNoDiskPressure    6m16s (x3 over 6m17s)  kubelet     Node k8s1 status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientMemory  6m16s (x4 over 6m17s)  kubelet     Node k8s1 status is now: NodeHasSufficientMemory
  Normal  Starting                 6m7s                   kubelet     Starting kubelet.
  Normal  NodeHasNoDiskPressure    6m6s                   kubelet     Node k8s1 status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     6m6s                   kubelet     Node k8s1 status is now: NodeHasSufficientPID
  Normal  NodeNotReady             6m6s                   kubelet     Node k8s1 status is now: NodeNotReady
  Normal  NodeAllocatableEnforced  6m6s                   kubelet     Updated Node Allocatable limit across pods
  Normal  NodeHasSufficientMemory  6m6s                   kubelet     Node k8s1 status is now: NodeHasSufficientMemory
  Normal  NodeReady                5m56s                  kubelet     Node k8s1 status is now: NodeReady
lab@desktop:~$
```

```sh
lab@desktop:~$ kubectl get pods -A
NAMESPACE     NAME                           READY   STATUS              RESTARTS   AGE
kube-system   coredns-78fcd69978-fpm5b       0/1     ContainerCreating   0          6m38s
kube-system   coredns-78fcd69978-gkrfq       0/1     ContainerCreating   0          6m38s
kube-system   etcd-k8s1                      1/1     Running             0          6m51s
kube-system   kube-apiserver-k8s1            1/1     Running             0          6m51s
kube-system   kube-controller-manager-k8s1   1/1     Running             0          6m53s
kube-system   kube-proxy-n6x56               1/1     Running             0          6m39s
kube-system   kube-scheduler-k8s1            1/1     Running             0          6m54s
lab@desktop:~$
```

```sh
lab@desktop:~$ kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
lab@desktop:~$
```

```sh
lab@desktop:~$ kubectl get pods -A
NAMESPACE     NAME                           READY   STATUS              RESTARTS      AGE
kube-system   coredns-78fcd69978-fpm5b       0/1     ContainerCreating   0             7m48s
kube-system   coredns-78fcd69978-gkrfq       0/1     ContainerCreating   0             7m48s
kube-system   etcd-k8s1                      1/1     Running             0             8m1s
kube-system   kube-apiserver-k8s1            1/1     Running             0             8m1s
kube-system   kube-controller-manager-k8s1   1/1     Running             0             8m3s
kube-system   kube-flannel-ds-999w2          0/1     Error               2 (23s ago)   31s
kube-system   kube-proxy-n6x56               1/1     Running             0             7m49s
kube-system   kube-scheduler-k8s1            1/1     Running             0             8m4s
lab@desktop:~$
```

```sh
lab@desktop:~$ kubectl -n kube-system logs kube-flannel-ds-999w2
I0329 14:10:21.626796       1 main.go:205] CLI flags config: {etcdEndpoints:http://127.0.0.1:4001,http://127.0.0.1:2379 etcdPrefix:/coreos.com/network etcdKeyfile: etcdCertfile: etcdCAFile: etcdUsername: etcdPassword: version:false kubeSubnetMgr:true kubeApiUrl: kubeAnnotationPrefix:flannel.alpha.coreos.com kubeConfigFile: iface:[] ifaceRegex:[] ipMasq:true subnetFile:/run/flannel/subnet.env publicIP: publicIPv6: subnetLeaseRenewMargin:60 healthzIP:0.0.0.0 healthzPort:0 iptablesResyncSeconds:5 iptablesForwardRules:true netConfPath:/etc/kube-flannel/net-conf.json setNodeNetworkUnavailable:true}
W0329 14:10:21.626873       1 client_config.go:614] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0329 14:10:21.820778       1 kube.go:120] Waiting 10m0s for node controller to sync
I0329 14:10:21.820832       1 kube.go:378] Starting kube subnet manager
I0329 14:10:22.821365       1 kube.go:127] Node controller sync successful
I0329 14:10:22.821402       1 main.go:225] Created subnet manager: Kubernetes Subnet Manager - k8s1
I0329 14:10:22.821416       1 main.go:228] Installing signal handlers
I0329 14:10:22.822041       1 main.go:454] Found network config - Backend type: vxlan
I0329 14:10:22.822070       1 match.go:189] Determining IP address of default interface
I0329 14:10:22.822577       1 match.go:242] Using interface with name eth0 and address 10.210.40.172
I0329 14:10:22.822607       1 match.go:264] Defaulting external address to interface address (10.210.40.172)
I0329 14:10:22.822667       1 vxlan.go:138] VXLAN config: VNI=1 Port=0 GBP=false Learning=false DirectRouting=false
E0329 14:10:22.822907       1 main.go:317] Error registering network: failed to acquire lease: node "k8s1" pod cidr not assigned
W0329 14:10:22.823036       1 reflector.go:436] github.com/flannel-io/flannel/subnet/kube/kube.go:379: watch of *v1.Node ended with: an error on the server ("unable to decode an event from the watch stream: context canceled") has prevented the request from succeeding
I0329 14:10:22.823091       1 main.go:434] Stopping shutdownHandler...
lab@desktop:~$
```

Hmm. Still same error!

Upon further checking, I got to know that the --pod-network-cidr=10.244.0.0/16 option is a requirement for Flannel  and we shouldn't change that network address!

So, let us repeat the process again, with correct `--pod-network-cidr=10.244.0.0/16`.

```sh
lab@k8s1:~$ sudo kubeadm init --apiserver-advertise-address=192.168.100.1 --pod-network-cidr=10.244.0.0/16
[sudo] password for lab:
I0329 07:16:09.273635  413182 version.go:255] remote version is much newer: v1.23.5; falling back to: stable-1.22
[init] Using Kubernetes version: v1.22.8
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.100.1]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s1 localhost] and IPs [192.168.100.1 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s1 localhost] and IPs [192.168.100.1 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 9.003704 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s1 as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 7rwrhs.s4yudk2p3yyz1hlg
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.100.1:6443 --token 7rwrhs.s4yudk2p3yyz1hlg \
	--discovery-token-ca-cert-hash sha256:45ea28506b18239a67e90374fcd186d9de899f33861be7d7b5f1873f1c49bb6f
lab@k8s1:~$
```

```sh
lab@k8s1:~$ mkdir -p $HOME/.kube
lab@k8s1:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
lab@k8s1:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl describe nodes
Name:               k8s1
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s1
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 29 Mar 2022 07:16:22 -0700
Taints:             node-role.kubernetes.io/master:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  k8s1
  AcquireTime:     <unset>
  RenewTime:       Tue, 29 Mar 2022 07:19:40 -0700
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Tue, 29 Mar 2022 07:16:36 -0700   Tue, 29 Mar 2022 07:16:19 -0700   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Tue, 29 Mar 2022 07:16:36 -0700   Tue, 29 Mar 2022 07:16:19 -0700   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Tue, 29 Mar 2022 07:16:36 -0700   Tue, 29 Mar 2022 07:16:19 -0700   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Tue, 29 Mar 2022 07:16:36 -0700   Tue, 29 Mar 2022 07:16:36 -0700   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.210.40.172
  Hostname:    k8s1
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
  System UUID:                18962942-410c-c225-65ca-e0a7de38d7cd
  Boot ID:                    79681989-c2a6-478c-bb51-7c46b1596fba
  Kernel Version:             5.13.0-37-generic
  OS Image:                   Ubuntu 20.04.4 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://20.10.14
  Kubelet Version:            v1.22.0
  Kube-Proxy Version:         v1.22.0
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
Non-terminated Pods:          (7 in total)
  Namespace                   Name                            CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                            ------------  ----------  ---------------  -------------  ---
  kube-system                 coredns-78fcd69978-2s8w7        100m (5%)     0 (0%)      70Mi (3%)        170Mi (9%)     3m1s
  kube-system                 coredns-78fcd69978-bhc9k        100m (5%)     0 (0%)      70Mi (3%)        170Mi (9%)     3m1s
  kube-system                 etcd-k8s1                       100m (5%)     0 (0%)      100Mi (5%)       0 (0%)         3m16s
  kube-system                 kube-apiserver-k8s1             250m (12%)    0 (0%)      0 (0%)           0 (0%)         3m16s
  kube-system                 kube-controller-manager-k8s1    200m (10%)    0 (0%)      0 (0%)           0 (0%)         3m16s
  kube-system                 kube-proxy-brrvs                0 (0%)        0 (0%)      0 (0%)           0 (0%)         3m1s
  kube-system                 kube-scheduler-k8s1             100m (5%)     0 (0%)      0 (0%)           0 (0%)         3m16s
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                850m (42%)   0 (0%)
  memory             240Mi (12%)  340Mi (18%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)
Events:
  Type    Reason                   Age                    From        Message
  ----    ------                   ----                   ----        -------
  Normal  Starting                 2m58s                  kube-proxy
  Normal  NodeHasSufficientMemory  3m26s (x5 over 3m27s)  kubelet     Node k8s1 status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    3m26s (x5 over 3m27s)  kubelet     Node k8s1 status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     3m26s (x4 over 3m27s)  kubelet     Node k8s1 status is now: NodeHasSufficientPID
  Normal  Starting                 3m17s                  kubelet     Starting kubelet.
  Normal  NodeHasNoDiskPressure    3m17s                  kubelet     Node k8s1 status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     3m17s                  kubelet     Node k8s1 status is now: NodeHasSufficientPID
  Normal  NodeHasSufficientMemory  3m17s                  kubelet     Node k8s1 status is now: NodeHasSufficientMemory
  Normal  NodeNotReady             3m16s                  kubelet     Node k8s1 status is now: NodeNotReady
  Normal  NodeAllocatableEnforced  3m16s                  kubelet     Updated Node Allocatable limit across pods
  Normal  NodeReady                3m6s                   kubelet     Node k8s1 status is now: NodeReady
lab@k8s1:~$
```

Look at the `PodCIDR:                      10.244.0.0/24` statement which was not seen earlier.

```sh
lab@k8s1:~$ kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```

```sh
lab@k8s1:~$ kubectl get pods -A
NAMESPACE     NAME                           READY   STATUS              RESTARTS   AGE
kube-system   coredns-78fcd69978-2s8w7       0/1     ContainerCreating   0          4m17s
kube-system   coredns-78fcd69978-bhc9k       0/1     ContainerCreating   0          4m17s
kube-system   etcd-k8s1                      1/1     Running             1          4m32s
kube-system   kube-apiserver-k8s1            1/1     Running             1          4m32s
kube-system   kube-controller-manager-k8s1   1/1     Running             0          4m32s
kube-system   kube-flannel-ds-lhcwb          1/1     Running             0          6s
kube-system   kube-proxy-brrvs               1/1     Running             0          4m17s
kube-system   kube-scheduler-k8s1            1/1     Running             1          4m32s
```
```sh
lab@k8s1:~$ kubectl get pods -A
NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE
kube-system   coredns-78fcd69978-2s8w7       1/1     Running   0          4m21s
kube-system   coredns-78fcd69978-bhc9k       1/1     Running   0          4m21s
kube-system   etcd-k8s1                      1/1     Running   1          4m36s
kube-system   kube-apiserver-k8s1            1/1     Running   1          4m36s
kube-system   kube-controller-manager-k8s1   1/1     Running   0          4m36s
kube-system   kube-flannel-ds-lhcwb          1/1     Running   0          10s
kube-system   kube-proxy-brrvs               1/1     Running   0          4m21s
kube-system   kube-scheduler-k8s1            1/1     Running   1          4m36s
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE     VERSION
k8s1   Ready    control-plane,master   5m27s   v1.22.0
lab@k8s1:~$
```

Yay! Congratulations, with this our control-plane node installation is complete (including the CNI).

You might have noticed, the `coredns` pods were in `Pending` state until the successful installation of CNI (Flannel in this case)