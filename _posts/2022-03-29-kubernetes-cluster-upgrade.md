---
layout: single
title:  "Kubernetes Cluster Upgrade"
date:   2022-03-29 15:55:04 +0530
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

# Kubernetes Cluster Upgrade

Verify the current running version of the cluster .

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE     VERSION
k8s1   Ready    control-plane,master   3h45m   v1.22.0
k8s2   Ready    <none>                 152m    v1.22.0
k8s3   Ready    <none>                 62m     v1.22.0
lab@k8s1:~$
```
To upgrade your cluster smoothly to a newer version, you can use the 	`kubeadm upgrade` command.

```sh
lab@k8s1:~$ sudo kubeadm upgrade -h
Upgrade your cluster smoothly to a newer version with this command

Usage:
  kubeadm upgrade [flags]
  kubeadm upgrade [command]

Available Commands:
  apply       Upgrade your Kubernetes cluster to the specified version
  diff        Show what differences would be applied to existing static pod manifests. See also: kubeadm upgrade apply --dry-run
  node        Upgrade commands for a node in the cluster
  plan        Check which versions are available to upgrade to and validate whether your current cluster is upgradeable. To skip the internet check, pass in the optional [version] parameter

Flags:
  -h, --help   help for upgrade

Global Flags:
      --add-dir-header           If true, adds the file directory to the header of the log messages
      --log-file string          If non-empty, use this log file
      --log-file-max-size uint   Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)
      --one-output               If true, only write logs to their native severity level (vs also writing to each lower severity level)
      --rootfs string            [EXPERIMENTAL] The path to the 'real' host root filesystem.
      --skip-headers             If true, avoid header prefixes in the log messages
      --skip-log-headers         If true, avoid headers when opening log files
  -v, --v Level                  number for the log level verbosity

Use "kubeadm upgrade [command] --help" for more information about a command.
lab@k8s1:~$
```



Let us check the upgrade plan.



```sh
lab@k8s1:~$ sudo kubeadm upgrade plan
[sudo] password for lab:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.22.8
[upgrade/versions] kubeadm version: v1.22.0
I0329 11:03:04.323845  575665 version.go:255] remote version is much newer: v1.23.5; falling back to: stable-1.22
[upgrade/versions] Target version: v1.22.8
[upgrade/versions] Latest version in the v1.22 series: v1.22.8

lab@k8s1:~$
```



We will be upgrading the control-plane node first, drain this node and mark it as unschedulable.

```sh
lab@k8s1:~$ kubectl drain k8s1 --ignore-daemonsets
node/k8s1 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-lhcwb, kube-system/kube-proxy-brrvs
node/k8s1 drained
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS                     ROLES                  AGE     VERSION
k8s1   Ready,SchedulingDisabled   control-plane,master   3h51m   v1.22.0
k8s2   Ready                      <none>                 157m    v1.22.0
k8s3   Ready                      <none>                 67m     v1.22.0
lab@k8s1:~$
```

Upgrade the kubeadm tool, then the control-plane components, and finally kubelet.

```sh
lab@k8s1:~$ sudo apt update
Hit:1 https://download.docker.com/linux/ubuntu focal InRelease
Hit:2 http://us.archive.ubuntu.com/ubuntu focal InRelease
Hit:3 http://packages.microsoft.com/repos/code stable InRelease
Hit:4 https://apt.releases.hashicorp.com focal InRelease
Get:6 https://dl.google.com/linux/chrome/deb stable InRelease [1811 B]
Hit:7 https://packages.microsoft.com/ubuntu/20.04/prod focal InRelease
Get:8 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Hit:9 https://brave-browser-apt-beta.s3.brave.com stable InRelease
Hit:10 https://brave-browser-apt-release.s3.brave.com stable InRelease
Get:11 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:12 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:13 https://dl.google.com/linux/chrome/deb stable/main amd64 Packages [1089 B]
Get:14 http://us.archive.ubuntu.com/ubuntu focal-updates/main i386 Packages [623 kB]
Get:15 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [1680 kB]
Get:16 http://security.ubuntu.com/ubuntu focal-security/main amd64 DEP-11 Metadata [40.8 kB]
Get:17 http://us.archive.ubuntu.com/ubuntu focal-updates/main Translation-en [317 kB]
Get:18 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 DEP-11 Metadata [277 kB]
Get:19 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 c-n-f Metadata [14.8 kB]
Get:20 http://us.archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [893 kB]
Get:21 http://security.ubuntu.com/ubuntu focal-security/main amd64 c-n-f Metadata [9804 B]
Get:22 http://us.archive.ubuntu.com/ubuntu focal-updates/restricted Translation-en [127 kB]
Get:23 http://security.ubuntu.com/ubuntu focal-security/universe amd64 DEP-11 Metadata [66.3 kB]
Get:24 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [913 kB]
Get:25 http://us.archive.ubuntu.com/ubuntu focal-updates/universe i386 Packages [673 kB]
Hit:5 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Get:26 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 DEP-11 Metadata [391 kB]
Get:27 http://us.archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 DEP-11 Metadata [944 B]
Get:28 http://us.archive.ubuntu.com/ubuntu focal-backports/main amd64 DEP-11 Metadata [8004 B]
Get:29 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 DEP-11 Metadata [2464 B]
Get:30 http://us.archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [22.7 kB]
Get:31 http://us.archive.ubuntu.com/ubuntu focal-backports/universe i386 Packages [12.9 kB]
Get:32 http://us.archive.ubuntu.com/ubuntu focal-backports/universe Translation-en [15.5 kB]
Get:33 http://us.archive.ubuntu.com/ubuntu focal-backports/universe amd64 DEP-11 Metadata [30.8 kB]
Fetched 6457 kB in 3s (2336 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
7 packages can be upgraded. Run 'apt list --upgradable' to see them.
lab@k8s1:~$
```

Download the latest `kubeadm` version (1.22.8, based on the upgrade plan suggestion)
```sh
lab@k8s1:~$ sudo apt install kubeadm=1.22.8-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libfwupdplugin1
Use 'sudo apt autoremove' to remove it.
The following held packages will be changed:
  kubeadm
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 6 not upgraded.
Need to get 8724 kB of archives.
After this operation, 20.5 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.22.8-00 [8724 kB]
Fetched 8724 kB in 2s (4563 kB/s)
(Reading database ... 216135 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.22.8-00_amd64.deb ...
Unpacking kubeadm (1.22.8-00) over (1.22.0-00) ...
Setting up kubeadm (1.22.8-00) ...
lab@k8s1:~$

```

### Apply the upgrade 
```sh
lab@k8s1:~$ sudo kubeadm upgrade apply v1.22.8
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.22.8"
[upgrade/versions] Cluster version: v1.22.8
[upgrade/versions] kubeadm version: v1.22.8
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.22.8"...
Static pod: kube-apiserver-k8s1 hash: a89d3c659a9a39d514d8076764b40bdb
Static pod: kube-controller-manager-k8s1 hash: f305f1a23bcb8c027327189d9d3ff5ac
Static pod: kube-scheduler-k8s1 hash: 67fe8e36dda02fbff101beddbcbe5879
[upgrade/etcd] Upgrading to TLS for etcd
Static pod: etcd-k8s1 hash: 91e64623622aeb865a09c79cf82eb4a5
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Current and new manifests of etcd are equal, skipping upgrade
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests555972790"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Current and new manifests of kube-apiserver are equal, skipping upgrade
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Current and new manifests of kube-controller-manager are equal, skipping upgrade
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Current and new manifests of kube-scheduler are equal, skipping upgrade
[upgrade/postupgrade] Applying label node-role.kubernetes.io/control-plane='' to Nodes with label node-role.kubernetes.io/master='' (deprecated)
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.22.8". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
lab@k8s1:~$
```

Yay! SUCCESS! Your cluster was upgraded to "v1.22.8". Enjoy!
As per the instructions given at the end of the upgrade apply command,  let us upgrade the kubelet as well.

```sh
lab@k8s1:~$ sudo apt install kubelet=1.22.8-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libfwupdplugin1
Use 'sudo apt autoremove' to remove it.
The following held packages will be changed:
  kubelet
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 6 not upgraded.
Need to get 19.1 MB of archives.
After this operation, 32.1 MB disk space will be freed.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.22.8-00 [19.1 MB]
Fetched 19.1 MB in 1s (17.2 MB/s)
(Reading database ... 216135 files and directories currently installed.)
Preparing to unpack .../kubelet_1.22.8-00_amd64.deb ...
Unpacking kubelet (1.22.8-00) over (1.22.0-00) ...
Setting up kubelet (1.22.8-00) ...
lab@k8s1:~$
```
Restart the kubelet service.

```sh
lab@k8s1:~$ sudo systemctl restart kubelet
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS                     ROLES                  AGE     VERSION
k8s1   Ready,SchedulingDisabled   control-plane,master   3h58m   v1.22.8
k8s2   Ready                      <none>                 165m    v1.22.0
k8s3   Ready                      <none>                 74m     v1.22.0
lab@k8s1:~$
```

Now that the control-plane is upgraded, mark it is Schedulable again (by uncording it!)

```sh
lab@k8s1:~$ kubectl uncordon k8s1
node/k8s1 uncordoned
lab@k8s1:~$
```
Verify that `k8s1` is Ready!

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE    VERSION
k8s1   Ready    control-plane,master   4h     v1.22.8
k8s2   Ready    <none>                 166m   v1.22.0
k8s3   Ready    <none>                 76m    v1.22.0
lab@k8s1:~$
```
Now, drain the first worker node, `k8s2`.

```sh
lab@k8s1:~$ kubectl drain k8s2 --ignore-daemonsets
node/k8s2 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-ph9gg, kube-system/kube-proxy-cdl2t
evicting pod kube-system/coredns-78fcd69978-vfms9
evicting pod default/web-79d88c97d6-6msxc
evicting pod default/web-79d88c97d6-nktjl
pod/web-79d88c97d6-6msxc evicted
pod/web-79d88c97d6-nktjl evicted
pod/coredns-78fcd69978-vfms9 evicted
node/k8s2 evicted
lab@k8s1:~$
```
Verify the status of the `k8s2` node
```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS                     ROLES                  AGE    VERSION
k8s1   Ready                      control-plane,master   4h1m   v1.22.8
k8s2   Ready,SchedulingDisabled   <none>                 168m   v1.22.0
k8s3   Ready                      <none>                 77m    v1.22.0
lab@k8s1:~$
```

Now, let us upgrade this node to the new version v1.22.8.
Login to the `k8s2` node 

```sh
lab@k8s2:~$ sudo apt update
[sudo] password for lab:
Hit:1 http://packages.microsoft.com/repos/code stable InRelease
Hit:3 https://download.docker.com/linux/ubuntu focal InRelease
Get:4 https://dl.google.com/linux/chrome/deb stable InRelease [1811 B]
Hit:5 https://packages.microsoft.com/ubuntu/20.04/prod focal InRelease
Hit:6 https://apt.releases.hashicorp.com focal InRelease
Hit:7 https://brave-browser-apt-release.s3.brave.com stable InRelease
Hit:8 http://us.archive.ubuntu.com/ubuntu focal InRelease
Get:9 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:10 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Hit:11 https://brave-browser-apt-beta.s3.brave.com stable InRelease
Get:12 https://dl.google.com/linux/chrome/deb stable/main amd64 Packages [1089 B]
Get:13 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:14 http://security.ubuntu.com/ubuntu focal-security/main amd64 DEP-11 Metadata [40.8 kB]
Get:15 http://security.ubuntu.com/ubuntu focal-security/universe amd64 DEP-11 Metadata [66.3 kB]
Get:16 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 DEP-11 Metadata [277 kB]
Get:17 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 DEP-11 Metadata [2464 B]
Get:18 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 c-n-f Metadata [14.8 kB]
Get:19 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 DEP-11 Metadata [391 kB]
Get:20 http://us.archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 DEP-11 Metadata [944 B]
Get:21 http://us.archive.ubuntu.com/ubuntu focal-backports/main amd64 DEP-11 Metadata [8004 B]
Get:22 http://us.archive.ubuntu.com/ubuntu focal-backports/universe i386 Packages [12.9 kB]
Get:23 http://us.archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [22.7 kB]
Get:24 http://us.archive.ubuntu.com/ubuntu focal-backports/universe Translation-en [15.5 kB]
Get:25 http://us.archive.ubuntu.com/ubuntu focal-backports/universe amd64 DEP-11 Metadata [30.8 kB]
Hit:2 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Fetched 1222 kB in 3s (398 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
7 packages can be upgraded. Run 'apt list --upgradable' to see them.
lab@k8s2:~$
```
Install latest `kubeadm` version.

```sh
lab@k8s2:~$ sudo apt install kubeadm=1.22.8-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libfwupdplugin1
Use 'sudo apt autoremove' to remove it.
The following held packages will be changed:
  kubeadm
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 6 not upgraded.
Need to get 8724 kB of archives.
After this operation, 20.5 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.22.8-00 [8724 kB]
Fetched 8724 kB in 1s (11.6 MB/s)
(Reading database ... 216135 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.22.8-00_amd64.deb ...
Unpacking kubeadm (1.22.8-00) over (1.22.0-00) ...
Setting up kubeadm (1.22.8-00) ...
lab@k8s2:~$

```

### Kubeadm Upgrade Node

Upgrade commands for a node in the cluster

```sh
lab@k8s2:~$ sudo kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
lab@k8s2:~$
```

Update the kubelet to the latest.

```sh
lab@k8s2:~$ sudo apt install kubelet=1.22.8-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libfwupdplugin1
Use 'sudo apt autoremove' to remove it.
The following held packages will be changed:
  kubelet
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 6 not upgraded.
Need to get 19.1 MB of archives.
After this operation, 32.1 MB disk space will be freed.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.22.8-00 [19.1 MB]
Fetched 19.1 MB in 1s (19.6 MB/s)
(Reading database ... 216135 files and directories currently installed.)
Preparing to unpack .../kubelet_1.22.8-00_amd64.deb ...
Unpacking kubelet (1.22.8-00) over (1.22.0-00) ...
Setting up kubelet (1.22.8-00) ...
lab@k8s2:~$
```
Restart the kubelet service.

```sh
lab@k8s2:~$ sudo systemctl restart kubelet
lab@k8s2:~$
```
Exit this worker node, and login to the control-plane node.

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS                     ROLES                  AGE    VERSION
k8s1   Ready                      control-plane,master   4h6m   v1.22.8
k8s2   Ready,SchedulingDisabled   <none>                 173m   v1.22.8
k8s3   Ready                      <none>                 83m    v1.22.0
lab@k8s1:~$
```
We can see that the first worker node (`k8s2`) got upgraded. 
Uncordon this node now.

```sh
lab@k8s1:~$ kubectl uncordon k8s2
node/k8s2 uncordoned
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE    VERSION
k8s1   Ready    control-plane,master   4h7m   v1.22.8
k8s2   Ready    <none>                 174m   v1.22.8
k8s3   Ready    <none>                 84m    v1.22.0
lab@k8s1:~$
```

Similarly, we can upgrade the `k8s3` node as well.

Before that, lets verify what all are running on this node.

```sh
lab@k8s1:~$ kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    6/6     6            6           61m
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP           NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4bsz5   1/1     Running   0          8m1s   10.244.3.8   k8s3   <none>           <none>
web-79d88c97d6-4kqj6   1/1     Running   0          60m    10.244.3.2   k8s3   <none>           <none>
web-79d88c97d6-hvdrs   1/1     Running   0          49m    10.244.3.5   k8s3   <none>           <none>
web-79d88c97d6-q4l2x   1/1     Running   0          60m    10.244.3.3   k8s3   <none>           <none>
web-79d88c97d6-rs5gk   1/1     Running   0          49m    10.244.3.4   k8s3   <none>           <none>
web-79d88c97d6-rsklz   1/1     Running   0          8m1s   10.244.3.7   k8s3   <none>           <none>
lab@k8s1:~$
```

Drain this last node to be upgraded.

```sh
lab@k8s1:~$ kubectl drain k8s3 --ignore-daemonsets
node/k8s3 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-xm28z, kube-system/kube-proxy-v8r74
evicting pod kube-system/coredns-78fcd69978-5jrm2
evicting pod default/web-79d88c97d6-hvdrs
evicting pod default/web-79d88c97d6-4bsz5
evicting pod default/web-79d88c97d6-4kqj6
evicting pod default/web-79d88c97d6-rs5gk
evicting pod default/web-79d88c97d6-q4l2x
evicting pod default/web-79d88c97d6-rsklz
pod/web-79d88c97d6-4bsz5 evicted
pod/web-79d88c97d6-hvdrs evicted
pod/web-79d88c97d6-rsklz evicted
pod/web-79d88c97d6-rs5gk evicted
pod/web-79d88c97d6-q4l2x evicted
pod/web-79d88c97d6-4kqj6 evicted
pod/coredns-78fcd69978-5jrm2 evicted
node/k8s3 evicted
lab@k8s1:~$
```
Verify 
```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4w4nl   1/1     Running   0          26s   10.244.2.14   k8s2   <none>           <none>
web-79d88c97d6-7ktck   1/1     Running   0          26s   10.244.2.15   k8s2   <none>           <none>
web-79d88c97d6-96sd9   1/1     Running   0          26s   10.244.2.12   k8s2   <none>           <none>
web-79d88c97d6-9tzlh   1/1     Running   0          26s   10.244.2.11   k8s2   <none>           <none>
web-79d88c97d6-kngc4   1/1     Running   0          26s   10.244.2.10   k8s2   <none>           <none>
web-79d88c97d6-p5vfg   1/1     Running   0          26s   10.244.2.13   k8s2   <none>           <none>
lab@k8s1:~$
```
All six pods of the deployment are now running on the other worker node, `k8s2`.
Login to the `k8s3` node.

```sh
lab@k8s3:~$ sudo apt update
[sudo] password for lab:
Hit:1 http://packages.microsoft.com/repos/code stable InRelease
Get:3 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Hit:4 https://brave-browser-apt-release.s3.brave.com stable InRelease
Hit:5 http://us.archive.ubuntu.com/ubuntu focal InRelease
Hit:6 https://download.docker.com/linux/ubuntu focal InRelease
Hit:7 https://packages.microsoft.com/ubuntu/20.04/prod focal InRelease
Hit:8 https://brave-browser-apt-beta.s3.brave.com stable InRelease
Get:9 https://dl.google.com/linux/chrome/deb stable InRelease [1811 B]
Hit:10 https://apt.releases.hashicorp.com focal InRelease
Get:11 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:2 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [9383 B]
Get:12 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:13 https://dl.google.com/linux/chrome/deb stable/main amd64 Packages [1074 B]
Get:14 http://security.ubuntu.com/ubuntu focal-security/main amd64 DEP-11 Metadata [40.8 kB]
Get:15 http://security.ubuntu.com/ubuntu focal-security/universe amd64 DEP-11 Metadata [66.3 kB]
Get:16 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 DEP-11 Metadata [277 kB]
Get:17 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 DEP-11 Metadata [2464 B]
Get:18 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 DEP-11 Metadata [391 kB]
Get:19 http://us.archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 DEP-11 Metadata [944 B]
Get:20 http://us.archive.ubuntu.com/ubuntu focal-backports/main amd64 DEP-11 Metadata [8004 B]
Get:21 http://us.archive.ubuntu.com/ubuntu focal-backports/universe amd64 DEP-11 Metadata [30.8 kB]
Fetched 1166 kB in 2s (651 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
8 packages can be upgraded. Run 'apt list --upgradable' to see them.
lab@k8s3:~$
```
Install latest `kubeadm` version.

```sh
lab@k8s3:~$ sudo apt install kubeadm=1.22.8-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libfwupdplugin1
Use 'sudo apt autoremove' to remove it.
The following held packages will be changed:
  kubeadm
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 7 not upgraded.
Need to get 8724 kB of archives.
After this operation, 20.5 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.22.8-00 [8724 kB]
Fetched 8724 kB in 1s (10.3 MB/s)
(Reading database ... 216135 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.22.8-00_amd64.deb ...
Unpacking kubeadm (1.22.8-00) over (1.22.0-00) ...
Setting up kubeadm (1.22.8-00) ...
lab@k8s3:~$

```
Upgrade this node.

```sh
lab@k8s3:~$ sudo kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
lab@k8s3:~$
```
As suggested at the end, Now you should go ahead and upgrade the kubelet package using your package manager.

```sh
lab@k8s3:~$ sudo apt install kubelet=1.22.8-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libfwupdplugin1
Use 'sudo apt autoremove' to remove it.
The following held packages will be changed:
  kubelet
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 7 not upgraded.
Need to get 19.1 MB of archives.
After this operation, 32.1 MB disk space will be freed.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.22.8-00 [19.1 MB]
Fetched 19.1 MB in 1s (19.7 MB/s)
(Reading database ... 216135 files and directories currently installed.)
Preparing to unpack .../kubelet_1.22.8-00_amd64.deb ...
Unpacking kubelet (1.22.8-00) over (1.22.0-00) ...
Setting up kubelet (1.22.8-00) ...
lab@k8s3:~$

```

Restart `kubelet`.
```sh
lab@k8s3:~$ sudo systemctl restart kubelet
lab@k8s3:~$
```

Exit and go back to the control-plane node.
```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS                     ROLES                  AGE     VERSION
k8s1   Ready                      control-plane,master   4h16m   v1.22.8
k8s2   Ready                      <none>                 3h3m    v1.22.8
k8s3   Ready,SchedulingDisabled   <none>                 92m     v1.22.8
lab@k8s1:~$
```
Finally, all three nodes are upgrade to the latest version `v1.22.8`.

Uncordon the `k8s3` node, to make it available for workloads.

```sh
lab@k8s1:~$ kubectl uncordon k8s3
node/k8s3 uncordoned
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE     VERSION
k8s1   Ready    control-plane,master   4h17m   v1.22.8
k8s2   Ready    <none>                 3h4m    v1.22.8
k8s3   Ready    <none>                 93m     v1.22.8
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP            NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4w4nl   1/1     Running   0          7m23s   10.244.2.14   k8s2   <none>           <none>
web-79d88c97d6-7ktck   1/1     Running   0          7m23s   10.244.2.15   k8s2   <none>           <none>
web-79d88c97d6-96sd9   1/1     Running   0          7m23s   10.244.2.12   k8s2   <none>           <none>
web-79d88c97d6-9tzlh   1/1     Running   0          7m23s   10.244.2.11   k8s2   <none>           <none>
web-79d88c97d6-kngc4   1/1     Running   0          7m23s   10.244.2.10   k8s2   <none>           <none>
web-79d88c97d6-p5vfg   1/1     Running   0          7m23s   10.244.2.13   k8s2   <none>           <none>
lab@k8s1:~$
```
Scale the deployment again, to confirm that the pods are getting scheduled on both nodes after the cluster upgrade.


```sh
lab@k8s1:~$ kubectl scale deployment web --replicas=9
deployment.apps/web scaled
lab@k8s1:~
```
```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP            NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4w4nl   1/1     Running   0          7m49s   10.244.2.14   k8s2   <none>           <none>
web-79d88c97d6-7ktck   1/1     Running   0          7m49s   10.244.2.15   k8s2   <none>           <none>
web-79d88c97d6-8tvsk   1/1     Running   0          9s      10.244.3.10   k8s3   <none>           <none>
web-79d88c97d6-96sd9   1/1     Running   0          7m49s   10.244.2.12   k8s2   <none>           <none>
web-79d88c97d6-9tzlh   1/1     Running   0          7m49s   10.244.2.11   k8s2   <none>           <none>
web-79d88c97d6-brtgx   1/1     Running   0          9s      10.244.3.9    k8s3   <none>           <none>
web-79d88c97d6-kngc4   1/1     Running   0          7m49s   10.244.2.10   k8s2   <none>           <none>
web-79d88c97d6-p5vfg   1/1     Running   0          7m49s   10.244.2.13   k8s2   <none>           <none>
web-79d88c97d6-rbhpr   1/1     Running   0          9s      10.244.3.11   k8s3   <none>           <none>
lab@k8s1:~$
```

We can see that all three new pods got scheduled on the `k8s3` node confirming that all nodes are working fine.

This concludes our discussion on the Kubernetes cluster upgrade procedure using the `kubeadm upgrade` utility.

