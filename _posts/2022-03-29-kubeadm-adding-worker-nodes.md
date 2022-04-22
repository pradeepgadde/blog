---
layout: single
title:  "Kubeadm Join - Adding Worker Nodes"
date:   2022-03-29 11:55:04 +0530
categories: Kubernetes
tags: kubeadm
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/node-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubeadm Join
## Pre-requisites
```sh
lab@k8s2:~$ lsmod | grep br_netfilter
```
```sh
lab@k8s2:~$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
> br_netfilter
> EOF
[sudo] password for lab:
br_netfilter
lab@k8s2:~$
```
```sh
lab@k8s2:~$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
lab@k8s2:~$

```
```sh
lab@k8s2:~$sudo sysctl --system
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
* Applying /usr/lib/sysctl.d/protect-links.conf ...
fs.protected_fifos = 1
fs.protected_hardlinks = 1
fs.protected_regular = 2
fs.protected_symlinks = 1
* Applying /etc/sysctl.conf ...
net.ipv4.ip_forward = 1
lab@k8s2:~$
```

## Install Docker

```sh
lab@k8s2:~$ sudo apt-get update
Hit:1 http://us.archive.ubuntu.com/ubuntu focal InRelease
Hit:3 https://brave-browser-apt-beta.s3.brave.com stable InRelease
Hit:4 https://brave-browser-apt-release.s3.brave.com stable InRelease
Hit:5 https://apt.releases.hashicorp.com focal InRelease
Hit:6 https://dl.google.com/linux/chrome/deb stable InRelease
Hit:7 http://packages.microsoft.com/repos/code stable InRelease
Get:8 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:9 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Hit:10 https://packages.microsoft.com/ubuntu/20.04/prod focal InRelease
Get:11 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:12 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [1680 kB]
Get:13 http://security.ubuntu.com/ubuntu focal-security/main i386 Packages [408 kB]
Get:14 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [1349 kB]
Get:15 http://us.archive.ubuntu.com/ubuntu focal-updates/main i386 Packages [623 kB]
Get:16 http://us.archive.ubuntu.com/ubuntu focal-updates/main Translation-en [317 kB]
Get:17 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 DEP-11 Metadata [278 kB]
Get:18 http://us.archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [893 kB]
Get:19 http://us.archive.ubuntu.com/ubuntu focal-updates/restricted Translation-en [127 kB]
Get:20 http://us.archive.ubuntu.com/ubuntu focal-updates/universe i386 Packages [673 kB]
Get:21 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [913 kB]
Get:22 http://us.archive.ubuntu.com/ubuntu focal-updates/universe amd64 DEP-11 Metadata [391 kB]
Get:23 http://us.archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 DEP-11 Metadata [944 B]
Get:24 http://us.archive.ubuntu.com/ubuntu focal-backports/main amd64 DEP-11 Metadata [8004 B]
Get:25 http://us.archive.ubuntu.com/ubuntu focal-backports/universe amd64 DEP-11 Metadata [30.8 kB]
Get:26 http://security.ubuntu.com/ubuntu focal-security/main amd64 DEP-11 Metadata [40.7 kB]
Get:27 http://security.ubuntu.com/ubuntu focal-security/main amd64 c-n-f Metadata [9804 B]
Get:28 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [695 kB]
Get:29 http://security.ubuntu.com/ubuntu focal-security/universe i386 Packages [547 kB]
Get:30 http://security.ubuntu.com/ubuntu focal-security/universe amd64 DEP-11 Metadata [66.3 kB]
Get:31 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 DEP-11 Metadata [2464 B]
Hit:2 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Fetched 9387 kB in 3s (3170 kB/s)
Reading package lists... Done
lab@k8s2:~$
```

```sh
lab@k8s2:~$ sudo apt-get install \
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
lab@k8s2:~$
```

```sh
lab@k8s2:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
lab@k8s2:~$
```

```sh
lab@k8s2:~$     echo \
>       "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
>       $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
lab@k8s2:~$
```

```sh
lab@k8s2:~$ sudo apt-get update
Hit:2 https://packages.microsoft.com/ubuntu/20.04/prod focal InRelease
Hit:3 http://packages.microsoft.com/repos/code stable InRelease
Hit:4 https://dl.google.com/linux/chrome/deb stable InRelease
Hit:5 https://brave-browser-apt-release.s3.brave.com stable InRelease
Get:6 https://download.docker.com/linux/ubuntu focal InRelease [57.7 kB]
Hit:7 https://brave-browser-apt-beta.s3.brave.com stable InRelease
Hit:8 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:9 http://us.archive.ubuntu.com/ubuntu focal InRelease
Hit:10 http://us.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:11 https://apt.releases.hashicorp.com focal InRelease
Hit:12 http://us.archive.ubuntu.com/ubuntu focal-backports InRelease
Get:13 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages [15.5 kB]
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [9383 B]
Fetched 82.5 kB in 2s (48.7 kB/s)
Reading package lists... Done
lab@k8s2:~$
```

```sh
lab@k8s2:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io
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
Fetched 102 MB in 3s (36.3 MB/s)
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
lab@k8s2:~$
```
Verify Docker installation.

```sh
lab@k8s2:~$ sudo docker run hello-world
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

lab@k8s2:~$
```

### Initiate Kubeadm Join

```sh
lab@k8s2:~$ sudo kubeadm join 192.168.100.1:6443 --token 7rwrhs.s4yudk2p3yyz1hlg \
> --discovery-token-ca-cert-hash sha256:45ea28506b18239a67e90374fcd186d9de899f33861be7d7b5f1873f1c49bb6f
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
[kubelet-check] Initial timeout of 40s passed.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
error execution phase kubelet-start: error uploading crisocket: timed out waiting for the condition
To see the stack trace of this error execute with --v=5 or higher
lab@k8s2:~$
```

Hmm, we have seen this problem, right!  

```sh
lab@k8s2:~$ sudo vi /etc/docker/daemon.json
lab@k8s2:~$ sudo cat /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
lab@k8s2:~$ sudo mkdir -p /etc/systemd/system/docker.service.d
lab@k8s2:~$ sudo systemctl daemon-reload
lab@k8s2:~$ sudo systemctl restart docker
```



```sh
lab@k8s2:~$ sudo kubeadm join 192.168.100.1:6443 --token 7rwrhs.s4yudk2p3yyz1hlg --discovery-token-ca-cert-hash sha256:45ea28506b18239a67e90374fcd186d9de899f33861be7d7b5f1873f1c49bb6f
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
	[ERROR Port-10250]: Port 10250 is in use
	[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
lab@k8s2:~$
```

Let us reset the work done by `kubeadm join`.

```sh
lab@k8s2:~$ sudo kubeadm reset
[reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
W0329 08:26:32.344127   11557 removeetcdmember.go:80] [reset] No kubeadm config, using etcd pod spec to get data directory
[reset] No etcd config found. Assuming external etcd
[reset] Please, manually reset etcd to prevent further issues
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[reset] Deleting contents of stateful directories: [/var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
lab@k8s2:~$
```



```sh
lab@k8s2:~$ sudo kubeadm join 192.168.100.1:6443 --token 7rwrhs.s4yudk2p3yyz1hlg --discovery-token-ca-cert-hash sha256:45ea28506b18239a67e90374fcd186d9de899f33861be7d7b5f1873f1c49bb6f
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
error execution phase kubelet-start: a Node with name "k8s2" and status "Ready" already exists in the cluster. You must delete the existing Node or change the name of this new joining Node
To see the stack trace of this error execute with --v=5 or higher
lab@k8s2:~$ exit
```

Login to the control-plane node.

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS     ROLES                  AGE     VERSION
k8s1   Ready      control-plane,master   71m     v1.22.0
k8s2   NotReady   <none>                 2m34s   v1.22.0
lab@k8s1:~$
```



Let us delete this `k8s2` node.

```sh
lab@k8s1:~$ kubectl delete nodes k8s2
node "k8s2" deleted
```
```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE   VERSION
k8s1   Ready    control-plane,master   72m   v1.22.0
lab@k8s1:~$
```

Log back to the worker node.

```sh
lab@k8s2:~$ sudo kubeadm join 192.168.100.1:6443 --token 7rwrhs.s4yudk2p3yyz1hlg --discovery-token-ca-cert-hash sha256:45ea28506b18239a67e90374fcd186d9de899f33861be7d7b5f1873f1c49bb6f
[sudo] password for lab:
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
lab@k8s2:~$
```
This seems to be coming from the previous failed attempt.
Let us delete it and attempt again.

```sh
lab@k8s2:~$ sudo rm /etc/kubernetes/pki/ca.crt
lab@k8s2:~$
```

```sh
lab@k8s2:~$ sudo kubeadm join 192.168.100.1:6443 --token 7rwrhs.s4yudk2p3yyz1hlg --discovery-token-ca-cert-hash sha256:45ea28506b18239a67e90374fcd186d9de899f33861be7d7b5f1873f1c49bb6f
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

lab@k8s2:~$
```



Congratulations!  `This node has joined the cluster:` message confirms the success of `kubeadm join` command.

```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE     VERSION
k8s1   Ready    control-plane,master   75m     v1.22.0
k8s2   Ready    <none>                 2m27s   v1.22.0
lab@k8s1:~$
```

Let us describe the new node `k8s2`.

```sh
lab@k8s1:~$ kubectl describe nodes k8s2
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
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  k8s2
  AcquireTime:     <unset>
  RenewTime:       Tue, 29 Mar 2022 08:32:28 -0700
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Tue, 29 Mar 2022 08:29:40 -0700   Tue, 29 Mar 2022 08:29:40 -0700   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Tue, 29 Mar 2022 08:29:44 -0700   Tue, 29 Mar 2022 08:29:34 -0700   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Tue, 29 Mar 2022 08:29:44 -0700   Tue, 29 Mar 2022 08:29:34 -0700   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Tue, 29 Mar 2022 08:29:44 -0700   Tue, 29 Mar 2022 08:29:34 -0700   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Tue, 29 Mar 2022 08:29:44 -0700   Tue, 29 Mar 2022 08:29:44 -0700   KubeletReady                 kubelet is posting ready status. AppArmor enabled
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
  kube-system                 kube-flannel-ds-ph9gg    100m (5%)     100m (5%)   50Mi (2%)        50Mi (2%)      2m55s
  kube-system                 kube-proxy-cdl2t         0 (0%)        0 (0%)      0 (0%)           0 (0%)         2m55s
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                100m (5%)  100m (5%)
  memory             50Mi (2%)  50Mi (2%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:
  Type    Reason                   Age                    From        Message
  ----    ------                   ----                   ----        -------
  Normal  Starting                 2m52s                  kube-proxy
  Normal  Starting                 7m18s                  kube-proxy
  Normal  NodeHasSufficientMemory  7m28s (x2 over 7m28s)  kubelet     Node k8s2 status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    7m28s (x2 over 7m28s)  kubelet     Node k8s2 status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     7m28s (x2 over 7m28s)  kubelet     Node k8s2 status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  7m28s                  kubelet     Updated Node Allocatable limit across pods
  Normal  Starting                 7m28s                  kubelet     Starting kubelet.
  Normal  NodeReady                7m8s                   kubelet     Node k8s2 status is now: NodeReady
  Normal  Starting                 2m55s                  kubelet     Starting kubelet.
  Normal  NodeHasNoDiskPressure    2m55s (x2 over 2m55s)  kubelet     Node k8s2 status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     2m55s (x2 over 2m55s)  kubelet     Node k8s2 status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  2m55s                  kubelet     Updated Node Allocatable limit across pods
  Normal  NodeHasSufficientMemory  2m55s (x2 over 2m55s)  kubelet     Node k8s2 status is now: NodeHasSufficientMemory
  Normal  NodeReady                2m45s                  kubelet     Node k8s2 status is now: NodeReady
lab@k8s1:~$
```



From the description, we can see that ` 10.244.2.0/24` CIDR block is assigned to this node, which is different from the subnet assigned to the  previous node.



Similarly we can add any number of worker nodes.

