---
layout: single
title:  "Kubeadm Join - Adding Other Worker Nodes"
date:   2022-03-29 13:55:04 +0530
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

# Kubeadm Join Again!
## Verifying Pre-requisites

So far we have two nodes (`k8s1` and `k8s2`) in our new Kubernetes cluster setup using the `kubeadm` tool.

Let us add one more node `k8s3` to this cluster using the same method that we followed so far.
### Letting iptables see bridged traffic
```sh
lab@k8s3:~$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
> br_netfilter
> EOF
br_netfilter
lab@k8s3:~$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
lab@k8s3:~$
lab@k8s3:~$ sudo sysctl --system
* Applying /etc/sysctl.d/10-console-messages.conf ...
kernel.printk = 4 4 1 7
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
kernel.kptr_restrict = 1
* Applying /etc/sysctl.d/10-link-restrictions.conf ...
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/10-magic-sysrq.conf ...
kernel.sysrq = 176
* Applying /etc/sysctl.d/10-network-security.conf ...
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.all.rp_filter = 2
* Applying /etc/sysctl.d/10-ptrace.conf ...
kernel.yama.ptrace_scope = 1
* Applying /etc/sysctl.d/10-zeropage.conf ...
vm.mmap_min_addr = 65536
* Applying /etc/sysctl.d/30-brave.conf ...
* Applying /usr/lib/sysctl.d/30-tracker.conf ...
fs.inotify.max_user_watches = 65536
* Applying /usr/lib/sysctl.d/50-default.conf ...
net.ipv4.conf.default.promote_secondaries = 1
sysctl: setting key "net.ipv4.conf.all.promote_secondaries": Invalid argument
net.ipv4.ping_group_range = 0 2147483647
net.core.default_qdisc = fq_codel
fs.protected_regular = 1
fs.protected_fifos = 1
* Applying /usr/lib/sysctl.d/50-pid-max.conf ...
kernel.pid_max = 4194304
* Applying /etc/sysctl.d/99-sysctl.conf ...
net.ipv4.ip_forward = 1
* Applying /etc/sysctl.d/k8s.conf ...
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
* Applying /usr/lib/sysctl.d/protect-links.conf ...
fs.protected_fifos = 1
fs.protected_hardlinks = 1
fs.protected_regular = 2
fs.protected_symlinks = 1
* Applying /etc/sysctl.conf ...
net.ipv4.ip_forward = 1
lab@k8s3:~$
```
### Install Docker 

```sh
lab@k8s3:~$ sudo apt-get update
Hit:1 http://packages.microsoft.com/repos/code stable InRelease
Hit:2 https://apt.releases.hashicorp.com focal InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu focal InRelease
Hit:4 https://dl.google.com/linux/chrome/deb stable InRelease
Hit:5 https://packages.microsoft.com/ubuntu/20.04/prod focal InRelease
Hit:6 https://brave-browser-apt-release.s3.brave.com stable InRelease
Get:7 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Hit:8 https://brave-browser-apt-beta.s3.brave.com stable InRelease
Get:9 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:10 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:11 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [1680 kB]
Get:12 http://security.ubuntu.com/ubuntu focal-security/main i386 Packages [408 kB]
Get:13 http://us.archive.ubuntu.com/ubuntu focal-updates/main i386 Packages [623 kB]
Get:14 http://us.archive.ubuntu.com/ubuntu focal-updates/main Translation-en [317 kB]
Get:15 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 DEP-11 Metadata [277 kB]
Get:16 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 c-n-f Metadata [14.8 kB]
Get:17 http://us.archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [893 kB]
Get:18 http://us.archive.ubuntu.com/ubuntu focal-updates/restricted Translation-en [127 kB]
Get:19 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [913 kB]
Get:20 http://us.archive.ubuntu.com/ubuntu focal-updates/universe i386 Packages [673 kB]
Get:21 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 DEP-11 Metadata [390 kB]
Get:22 http://us.archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 DEP-11 Metadata [944 B]
Get:23 http://us.archive.ubuntu.com/ubuntu focal-backports/main amd64 DEP-11 Metadata [7996 B]
Get:24 http://us.archive.ubuntu.com/ubuntu focal-backports/universe i386 Packages [12.9 kB]
Get:25 http://us.archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [22.7 kB]
Get:26 http://us.archive.ubuntu.com/ubuntu focal-backports/universe Translation-en [15.5 kB]
Get:27 http://us.archive.ubuntu.com/ubuntu focal-backports/universe amd64 DEP-11 Metadata [30.8 kB]
Get:28 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [1349 kB]
Get:29 http://security.ubuntu.com/ubuntu focal-security/main amd64 DEP-11 Metadata [40.6 kB]
Get:30 http://security.ubuntu.com/ubuntu focal-security/main amd64 c-n-f Metadata [9804 B]
Get:31 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [695 kB]
Get:32 http://security.ubuntu.com/ubuntu focal-security/universe i386 Packages [547 kB]
Get:33 http://security.ubuntu.com/ubuntu focal-security/universe amd64 DEP-11 Metadata [66.3 kB]
Get:34 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 DEP-11 Metadata [2464 B]
Hit:35 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Fetched 9452 kB in 3s (2792 kB/s)
Reading package lists... Done
lab@k8s3:~$
```

```sh
lab@k8s3:~$ sudo apt-get install \
>     ca-certificates \
>     curl \
>     gnupg \
>     lsb-release
Reading package lists... Done
Building dependency tree
Reading state information... Done
lsb-release is already the newest version (11.1.0ubuntu2).
lsb-release set to manually installed.
ca-certificates is already the newest version (20210119~20.04.2).
curl is already the newest version (7.68.0-1ubuntu2.7).
gnupg is already the newest version (2.2.19-3ubuntu2.1).
gnupg set to manually installed.

The following package was automatically installed and is no longer required:
  libfwupdplugin1
Use 'sudo apt autoremove' to remove it.
0 upgraded, 0 newly installed, 0 to remove and 7 not upgraded.
lab@k8s3:~$
```



```sh
lab@k8s3:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
lab@k8s3:~$
```

```sh
lab@k8s3:~$ echo \
>   "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
>   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
lab@k8s3:~$
```



```sh
lab@k8s3:~$ sudo apt-get update
Hit:1 https://brave-browser-apt-release.s3.brave.com stable InRelease
Hit:3 http://packages.microsoft.com/repos/code stable InRelease
Hit:4 https://brave-browser-apt-beta.s3.brave.com stable InRelease
Hit:5 https://apt.releases.hashicorp.com focal InRelease
Hit:6 https://packages.microsoft.com/ubuntu/20.04/prod focal InRelease
Get:7 https://download.docker.com/linux/ubuntu focal InRelease [57.7 kB]
Hit:8 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:9 http://us.archive.ubuntu.com/ubuntu focal InRelease
Hit:10 https://dl.google.com/linux/chrome/deb stable InRelease
Hit:11 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:12 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease
Get:2 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [9383 B]
Get:13 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages [15.5 kB]
Fetched 82.5 kB in 1s (61.2 kB/s)
Reading package lists... Done
lab@k8s3:~$
```

```sh
lab@k8s3:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libfwupdplugin1
Use 'sudo apt autoremove' to remove it.
The following additional packages will be installed:
  docker-ce-rootless-extras docker-scan-plugin git git-man liberror-perl pigz slirp4netns
Suggested packages:
  aufs-tools cgroupfs-mount | cgroup-lite git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui gitk gitweb git-cvs
  git-mediawiki git-svn
The following NEW packages will be installed:
  containerd.io docker-ce docker-ce-cli docker-ce-rootless-extras docker-scan-plugin git git-man liberror-perl pigz slirp4netns
0 upgraded, 10 newly installed, 0 to remove and 7 not upgraded.
Need to get 102 MB of archives.
After this operation, 443 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://us.archive.ubuntu.com/ubuntu focal/universe amd64 pigz amd64 2.4-1 [57.4 kB]
Get:2 https://download.docker.com/linux/ubuntu focal/stable amd64 containerd.io amd64 1.5.11-1 [22.9 MB]
Get:3 http://us.archive.ubuntu.com/ubuntu focal/main amd64 liberror-perl all 0.17029-1 [26.5 kB]
Get:4 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 git-man all 1:2.25.1-1ubuntu3.2 [884 kB]
Get:5 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 git amd64 1:2.25.1-1ubuntu3.2 [4554 kB]
Get:6 http://us.archive.ubuntu.com/ubuntu focal/universe amd64 slirp4netns amd64 0.4.3-1 [74.3 kB]
Get:7 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-ce-cli amd64 5:20.10.14~3-0~ubuntu-focal [41.0 MB]
Get:8 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-ce amd64 5:20.10.14~3-0~ubuntu-focal [20.9 MB]
Get:9 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-ce-rootless-extras amd64 5:20.10.14~3-0~ubuntu-focal [7932 kB]
Get:10 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-scan-plugin amd64 0.17.0~ubuntu-focal [3521 kB]
Fetched 102 MB in 5s (19.6 MB/s)
Selecting previously unselected package pigz.
(Reading database ... 214950 files and directories currently installed.)
Preparing to unpack .../0-pigz_2.4-1_amd64.deb ...
Unpacking pigz (2.4-1) ...
Selecting previously unselected package containerd.io.
Preparing to unpack .../1-containerd.io_1.5.11-1_amd64.deb ...
Unpacking containerd.io (1.5.11-1) ...
Selecting previously unselected package docker-ce-cli.
Preparing to unpack .../2-docker-ce-cli_5%3a20.10.14~3-0~ubuntu-focal_amd64.deb ...
Unpacking docker-ce-cli (5:20.10.14~3-0~ubuntu-focal) ...
Selecting previously unselected package docker-ce.
Preparing to unpack .../3-docker-ce_5%3a20.10.14~3-0~ubuntu-focal_amd64.deb ...
Unpacking docker-ce (5:20.10.14~3-0~ubuntu-focal) ...
Selecting previously unselected package docker-ce-rootless-extras.
Preparing to unpack .../4-docker-ce-rootless-extras_5%3a20.10.14~3-0~ubuntu-focal_amd64.deb ...
Unpacking docker-ce-rootless-extras (5:20.10.14~3-0~ubuntu-focal) ...
Selecting previously unselected package docker-scan-plugin.
Preparing to unpack .../5-docker-scan-plugin_0.17.0~ubuntu-focal_amd64.deb ...
Unpacking docker-scan-plugin (0.17.0~ubuntu-focal) ...
Selecting previously unselected package liberror-perl.
Preparing to unpack .../6-liberror-perl_0.17029-1_all.deb ...
Unpacking liberror-perl (0.17029-1) ...
Selecting previously unselected package git-man.
Preparing to unpack .../7-git-man_1%3a2.25.1-1ubuntu3.2_all.deb ...
Unpacking git-man (1:2.25.1-1ubuntu3.2) ...
Selecting previously unselected package git.
Preparing to unpack .../8-git_1%3a2.25.1-1ubuntu3.2_amd64.deb ...
Unpacking git (1:2.25.1-1ubuntu3.2) ...
Selecting previously unselected package slirp4netns.
Preparing to unpack .../9-slirp4netns_0.4.3-1_amd64.deb ...
Unpacking slirp4netns (0.4.3-1) ...
Setting up slirp4netns (0.4.3-1) ...
Setting up docker-scan-plugin (0.17.0~ubuntu-focal) ...
Setting up liberror-perl (0.17029-1) ...
Setting up containerd.io (1.5.11-1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
Setting up docker-ce-cli (5:20.10.14~3-0~ubuntu-focal) ...
Setting up pigz (2.4-1) ...
Setting up git-man (1:2.25.1-1ubuntu3.2) ...
Setting up docker-ce-rootless-extras (5:20.10.14~3-0~ubuntu-focal) ...
Setting up docker-ce (5:20.10.14~3-0~ubuntu-focal) ...
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /lib/systemd/system/docker.service.
Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /lib/systemd/system/docker.socket.
Setting up git (1:2.25.1-1ubuntu3.2) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for systemd (245.4-4ubuntu3.15) ...
lab@k8s3:~$
```

### Verify Docker

```sh
lab@k8s3:~$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:bfea6278a0a267fad2634554f4f0c6f31981eea41c553fdf5a83e95a41d40c38
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

lab@k8s3:~$
```



```sh
lab@k8s3:~$ sudo vi /etc/docker/daemon.json
lab@k8s3:~$ sudo mkdir -p /etc/systemd/system/docker.service.d
lab@k8s3:~$ sudo systemctl daemon-reload
lab@k8s3:~$ sudo systemctl restart docker
lab@k8s3:~$
```

### Verify the Kubelet Status

```sh
lab@k8s3:~$ sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Tue 2022-03-29 09:55:12 PDT; 858ms ago
       Docs: https://kubernetes.io/docs/home/
    Process: 7433 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (c>
   Main PID: 7433 (code=exited, status=1/FAILURE)

Mar 29 09:55:12 k8s3 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
Mar 29 09:55:12 k8s3 systemd[1]: kubelet.service: Failed with result 'exit-code'.
```

### Initiate Kubeadm Join

```sh
lab@k8s3:~$ sudo kubeadm join 192.168.100.1:6443 --token 7rwrhs.s4yudk2p3yyz1hlg \
> --discovery-token-ca-cert-hash sha256:45ea28506b18239a67e90374fcd186d9de899f33861be7d7b5f1873f1c49bb6f
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

lab@k8s3:~$
```



Yay! Once again, we added a new node to the existing cluster.

Before switching to the control-plane node, let us verify the `kubelet` status once again.

```sh
lab@k8s3:~$ sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Tue 2022-03-29 10:00:05 PDT; 14s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 7772 (kubelet)
      Tasks: 14 (limit: 2285)
     Memory: 69.2M
     CGroup: /system.slice/kubelet.service
             └─7772 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet>

Mar 29 10:00:07 k8s3 kubelet[7772]: I0329 10:00:07.392992    7772 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume >
Mar 29 10:00:07 k8s3 kubelet[7772]: I0329 10:00:07.393139    7772 reconciler.go:157] "Reconciler: start to sync state"
Mar 29 10:00:08 k8s3 kubelet[7772]: I0329 10:00:08.506664    7772 request.go:665] Waited for 1.009988697s due to client-side throttling>
Mar 29 10:00:11 k8s3 kubelet[7772]: I0329 10:00:11.068223    7772 cni.go:239] "Unable to update cni config" err="no networks found in />
Mar 29 10:00:11 k8s3 kubelet[7772]: E0329 10:00:11.503786    7772 kubelet.go:2332] "Container runtime network not ready" networkReady=">
Mar 29 10:00:11 k8s3 kubelet[7772]: I0329 10:00:11.554450    7772 pod_container_deletor.go:79] "Container not found in pod's containers>
Mar 29 10:00:15 k8s3 kubelet[7772]: I0329 10:00:15.890817    7772 transport.go:135] "Certificate rotation detected, shutting down clien>
Mar 29 10:00:16 k8s3 kubelet[7772]: I0329 10:00:16.083111    7772 cni.go:239] "Unable to update cni config" err="no networks found in />
Mar 29 10:00:16 k8s3 kubelet[7772]: E0329 10:00:16.522584    7772 kubelet.go:2332] "Container runtime network not ready" networkReady=">
Mar 29 10:00:16 k8s3 kubelet[7772]: E0329 10:00:16.590968    7772 cadvisor_stats_provider.go:415] "Partial failure issuing cadvisor.Con>
lab@k8s3:~$
```



Switch to the control-plane node (`k8s1`)

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE     VERSION
k8s1   Ready    control-plane,master   165m    v1.22.0
k8s2   Ready    <none>                 92m     v1.22.0
k8s3   Ready    <none>                 2m15s   v1.22.0
lab@k8s1:~$
```

Describe the new node `k8s3`

```sh
lab@k8s1:~$ kubectl describe node k8s3
Name:               k8s3
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s3
                    kubernetes.io/os=linux
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"46:57:f7:c6:05:51"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 10.210.40.175
                    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 29 Mar 2022 10:00:06 -0700
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  k8s3
  AcquireTime:     <unset>
  RenewTime:       Tue, 29 Mar 2022 10:03:00 -0700
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Tue, 29 Mar 2022 10:00:29 -0700   Tue, 29 Mar 2022 10:00:29 -0700   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Tue, 29 Mar 2022 10:00:36 -0700   Tue, 29 Mar 2022 10:00:06 -0700   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Tue, 29 Mar 2022 10:00:36 -0700   Tue, 29 Mar 2022 10:00:06 -0700   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Tue, 29 Mar 2022 10:00:36 -0700   Tue, 29 Mar 2022 10:00:06 -0700   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Tue, 29 Mar 2022 10:00:36 -0700   Tue, 29 Mar 2022 10:00:36 -0700   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.210.40.175
  Hostname:    k8s3
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
  System UUID:                e6ee2942-fc1b-488a-6add-25e5744a15ee
  Boot ID:                    3c77369e-54ee-471e-979c-7c730ee356d8
  Kernel Version:             5.13.0-37-generic
  OS Image:                   Ubuntu 20.04.4 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://20.10.14
  Kubelet Version:            v1.22.0
  Kube-Proxy Version:         v1.22.0
PodCIDR:                      10.244.3.0/24
PodCIDRs:                     10.244.3.0/24
Non-terminated Pods:          (2 in total)
  Namespace                   Name                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                     ------------  ----------  ---------------  -------------  ---
  kube-system                 kube-flannel-ds-xm28z    100m (5%)     100m (5%)   50Mi (2%)        50Mi (2%)      2m57s
  kube-system                 kube-proxy-v8r74         0 (0%)        0 (0%)      0 (0%)           0 (0%)         2m57s
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                100m (5%)  100m (5%)
  memory             50Mi (2%)  50Mi (2%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:
  Type    Reason                   Age    From        Message
  ----    ------                   ----   ----        -------
  Normal  Starting                 2m46s  kube-proxy
  Normal  Starting                 2m57s  kubelet     Starting kubelet.
  Normal  NodeHasSufficientMemory  2m57s  kubelet     Node k8s3 status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    2m57s  kubelet     Node k8s3 status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     2m57s  kubelet     Node k8s3 status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  2m57s  kubelet     Updated Node Allocatable limit across pods
  Normal  NodeReady                2m27s  kubelet     Node k8s3 status is now: NodeReady
lab@k8s1:~$
```

This new node has been assigned the ` 10.244.3.0/24` as the PodCIDR subnet.

```sh
lab@k8s1:~$ ip route
default via 10.210.40.254 dev eth0
10.210.40.128/25 dev eth0 proto kernel scope link src 10.210.40.172
10.244.0.0/24 dev cni0 proto kernel scope link src 10.244.0.1
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink
10.244.3.0/24 via 10.244.3.0 dev flannel.1 onlink
169.254.0.0/16 dev eth1 scope link metric 1000
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.1
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown
lab@k8s1:~$
```

Focus on the three route entries

| 10.244.0.0/24 | cni0      |
| ------------- | --------- |
| 10.244.2.0/24 | flannel.1 |
| 10.244.3.0/24 | flannel.1 |

Look at the interface list, and pay attention to `flannel.1` and `cni0` interfaces.
```sh
lab@k8s1:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:a9:d3:20 brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    altname ens160
    inet 10.210.40.172/25 brd 10.210.40.255 scope global dynamic eth0
       valid_lft 892sec preferred_lft 892sec
    inet6 fe80::250:56ff:fea9:d320/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:a9:72:de brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    altname ens192
    inet 192.168.100.1/24 brd 192.168.100.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fea9:72de/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:5d:eb:e0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:5d:eb:e0 brd ff:ff:ff:ff:ff:ff
6: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:1f:f2:02:94 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
7: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default
    link/ether 52:1d:12:10:50:8f brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::501d:12ff:fe10:508f/64 scope link
       valid_lft forever preferred_lft forever
8: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether e6:63:4c:a2:58:f3 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/24 brd 10.244.0.255 scope global cni0
       valid_lft forever preferred_lft forever
    inet6 fe80::e463:4cff:fea2:58f3/64 scope link
       valid_lft forever preferred_lft forever
9: veth56b267f5@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default
    link/ether 12:bb:c8:7f:d9:30 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::10bb:c8ff:fe7f:d930/64 scope link
       valid_lft forever preferred_lft forever
10: veth6ed8edf6@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default
    link/ether 7a:93:ef:bc:93:a0 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::7893:efff:febc:93a0/64 scope link
       valid_lft forever preferred_lft forever
```

Also, there are two `veth` interfaces, which corresponds to the two `coredns` pods who got the IPs in the `cni0` subnet (10.244.0.2 and 10.244.0.3)

```sh
lab@k8s1:~$ kubectl get pods -A -o wide
NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE    IP              NODE   NOMINATED NODE   READINESS GATES
kube-system   coredns-78fcd69978-2s8w7       1/1     Running   0          175m   10.244.0.3      k8s1   <none>           <none>
kube-system   coredns-78fcd69978-bhc9k       1/1     Running   0          175m   10.244.0.2      k8s1   <none>           <none>
kube-system   etcd-k8s1                      1/1     Running   1          176m   10.210.40.172   k8s1   <none>           <none>
kube-system   kube-apiserver-k8s1            1/1     Running   1          176m   10.210.40.172   k8s1   <none>           <none>
kube-system   kube-controller-manager-k8s1   1/1     Running   0          176m   10.210.40.172   k8s1   <none>           <none>
kube-system   kube-flannel-ds-lhcwb          1/1     Running   0          171m   10.210.40.172   k8s1   <none>           <none>
kube-system   kube-flannel-ds-ph9gg          1/1     Running   0          102m   10.210.40.173   k8s2   <none>           <none>
kube-system   kube-flannel-ds-xm28z          1/1     Running   0          12m    10.210.40.175   k8s3   <none>           <none>
kube-system   kube-proxy-brrvs               1/1     Running   0          175m   10.210.40.172   k8s1   <none>           <none>
kube-system   kube-proxy-cdl2t               1/1     Running   0          102m   10.210.40.173   k8s2   <none>           <none>
kube-system   kube-proxy-v8r74               1/1     Running   0          12m    10.210.40.175   k8s3   <none>           <none>
kube-system   kube-scheduler-k8s1            1/1     Running   1          176m   10.210.40.172   k8s1   <none>           <none>
lab@k8s1:~$
```

### Verify the Routes and Interfaces in Worker Nodes

```sh
lab@k8s2:~$ ip route
default via 10.210.40.254 dev eth0
10.210.40.128/25 dev eth0 proto kernel scope link src 10.210.40.173
10.244.0.0/24 via 10.244.0.0 dev flannel.1 onlink
10.244.3.0/24 via 10.244.3.0 dev flannel.1 onlink
169.254.0.0/16 dev eth1 scope link metric 1000
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.2
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown
lab@k8s2:~$
```
```sh
lab@k8s2:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:a9:0d:a8 brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    altname ens160
    inet 10.210.40.173/25 brd 10.210.40.255 scope global dynamic eth0
       valid_lft 892sec preferred_lft 892sec
    inet6 fe80::250:56ff:fea9:da8/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:a9:65:97 brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    altname ens192
    inet 192.168.100.2/24 brd 192.168.100.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fea9:6597/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:5d:eb:e0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:5d:eb:e0 brd ff:ff:ff:ff:ff:ff
6: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:27:e4:04:ee brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:27ff:fee4:4ee/64 scope link
       valid_lft forever preferred_lft forever
9: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default
    link/ether 92:01:5e:01:fe:01 brd ff:ff:ff:ff:ff:ff
    inet 10.244.2.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::9001:5eff:fe01:fe01/64 scope link
       valid_lft forever preferred_lft forever
lab@k8s2:~$
```
From the second worker node, `k8s3`

```sh
lab@k8s3:~$ ip route
default via 10.210.40.254 dev eth0
10.210.40.128/25 dev eth0 proto kernel scope link src 10.210.40.175
10.244.0.0/24 via 10.244.0.0 dev flannel.1 onlink
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink
169.254.0.0/16 dev eth1 scope link metric 1000
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.3
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown
lab@k8s3:~$
```
```sh
lab@k8s3:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:a9:40:98 brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    altname ens160
    inet 10.210.40.175/25 brd 10.210.40.255 scope global dynamic eth0
       valid_lft 885sec preferred_lft 885sec
    inet6 fe80::250:56ff:fea9:4098/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:a9:05:46 brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    altname ens192
    inet 192.168.100.3/24 brd 192.168.100.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fea9:546/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:5d:eb:e0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:5d:eb:e0 brd ff:ff:ff:ff:ff:ff
6: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:de:c2:99:e5 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:deff:fec2:99e5/64 scope link
       valid_lft forever preferred_lft forever
9: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default
    link/ether 46:57:f7:c6:05:51 brd ff:ff:ff:ff:ff:ff
    inet 10.244.3.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::4457:f7ff:fec6:551/64 scope link
       valid_lft forever preferred_lft forever
lab@k8s3:~$
```
One difference w.r.t to the interface list is that, in both worker nodes, we do not see the `cni0` interface yet. This is because, we don't have any pods running on these two nodes.
We will revist this after spinning some pods.

```sh
lab@k8s1:~$ kubectl get pods
No resources found in default namespace.
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE    VERSION
k8s1   Ready    control-plane,master   3h1m   v1.22.0
k8s2   Ready    <none>                 108m   v1.22.0
k8s3   Ready    <none>                 17m    v1.22.0
lab@k8s1:~$
```
Now, we have a three node kubernetes cluster up and running, ready to serve our application workloads.

```sh
lab@k8s1:~$ kubectl get all -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-78fcd69978-2s8w7       1/1     Running   0          3h3m
kube-system   pod/coredns-78fcd69978-bhc9k       1/1     Running   0          3h3m
kube-system   pod/etcd-k8s1                      1/1     Running   1          3h3m
kube-system   pod/kube-apiserver-k8s1            1/1     Running   1          3h3m
kube-system   pod/kube-controller-manager-k8s1   1/1     Running   0          3h3m
kube-system   pod/kube-flannel-ds-lhcwb          1/1     Running   0          179m
kube-system   pod/kube-flannel-ds-ph9gg          1/1     Running   0          110m
kube-system   pod/kube-flannel-ds-xm28z          1/1     Running   0          20m
kube-system   pod/kube-proxy-brrvs               1/1     Running   0          3h3m
kube-system   pod/kube-proxy-cdl2t               1/1     Running   0          110m
kube-system   pod/kube-proxy-v8r74               1/1     Running   0          20m
kube-system   pod/kube-scheduler-k8s1            1/1     Running   1          3h3m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  3h4m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   3h3m

NAMESPACE     NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-flannel-ds   3         3         3       3            3           <none>                   179m
kube-system   daemonset.apps/kube-proxy        3         3         3       3            3           kubernetes.io/os=linux   3h3m

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           3h3m

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-78fcd69978   2         2         2       3h3m
lab@k8s1:~$
```
This concludes our initial cluster setup using the `kubeadm` tool.
