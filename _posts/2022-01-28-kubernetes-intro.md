---
layout: single
title:  "Welcome to Kubernetes"
date:   2022-01-28 8:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
 
header:
  overlay_image: /assets/images/kubernetes.png
  og_image: /assets/images/kubernetes.png
  caption: "Photo credit: [**Kubernetes**](https://kubernetes.io)"
  teaser: /assets/images/kubernetes.png
  actions:
    - label: "Learn more"
      url: "https://kubernetes.io"
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubernetes   

## Introduction 
Google has created a resource in the form of a comic to understand why Kubernetes and  what problems can be addressed by Kubernetes. Here is the link to the same.

[https://cloud.google.com/kubernetes-engine/kubernetes-comic](https://cloud.google.com/kubernetes-engine/kubernetes-comic) 

And here is an extract:

![Kubernetes-Comic]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes-comic.png)

## Learning Resources
While there are many resources out there, personally I have used these resources:

[Introduction to Kubernetes by Linux Foundation](https://www.edx.org/course/introduction-to-kubernetes)

then, the most popular Udemy course on this subject: 

[Certified Kubernetes Administrator (CKA) with Practice Tests by Mumshad Mannambeth](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/), by [KodeKloud](https://kodekloud.com)

and the official Linux Foundation course 
[LFS258](https://training.linuxfoundation.org/training/kubernetes-fundamentals-lfs258-cka-exam-bundle/)

Also, I got the opportunity to attend some of the O'Reilly Live Classes–

[Certified Kubernetes Administrator (CKA) Exam Prep, hosted by Benjamin Muschko](https://github.com/bmuschko/cka-crash-course),

and

[Certified Kubernetes Administrator (CKA) Crash Course, presented by Sander van Vugt](https://github.com/sandervanvugt/cka).

Of course, the official :link: [Kubernetes Documentation](https://kubernetes.io/docs)

## Lab Setup

In this post, let's explore the world of Kubernetes right from our workstation. All the examples shown here are executed on a machine running macOS. 

To get started, we need to install three packages: `minikube`, `hyperkit`, and `kubectl`.
If you already have some kind of virtualization software like Vmware Fusion or Virtualbox, hyperkit is not mandatory, but I prefer to use hyperkit (which seems to be the case with minikube also).



### Minikube

#### Download

```shell
pradeep@learnk8s$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 
```
```sh
pradeep@learnk8s$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 68.6M  100 68.6M    0     0  6977k      0  0:00:10  0:00:10 --:--:-- 8368k
```



#### Install

```shell
pradeep@learnk8s$ sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```
```sh
pradeep@learnk8s$ sudo install minikube-darwin-amd64 /usr/local/bin/minikube
Password:
```

```shell
pradeep@learnk8s$ minikube version
minikube version: v1.25.1
commit: 3e64b11ed75e56e4898ea85f96b2e4af0301f43d
```
### Hyperkit

#### Download
If not, install Brew using, https://brew.sh/

#### Install
if you have Brew package manager, run: 
```shell
pradeep@learnk8s$ brew install hyperkit
```
```sh
pradeep@learnk8s$ brew install hyperkit
Running `brew update --preinstall`...
==> Auto-updated Homebrew!
Updated 3 taps (azure/functions, homebrew/core and homebrew/cask).
==> New Formulae
aws-auth                   koka                       opendht
bvm                        kubekey                    postgraphile
cloudflared                ltex-ls                    sdl12-compat
fortran-language-server    mapproxy                   spidermonkey@78
gemgen                     mu-repo                    testkube
go@1.17                    observerward               trivy
==> Updated Formulae
Updated 736 formulae.
==> Renamed Formulae
richmd -> rich-cli
==> New Casks
hepta                      paddle-easydl              prowlarr
jetbrains-gateway          pingnoo                    supernotes
macast                     poker-copilot              write
==> Updated Casks
Updated 486 casks.
==> Deleted Casks
optimal-layout      password-assistant  pd-runner           profilemanager

==> Downloading https://ghcr.io/v2/homebrew/core/libev/manifests/4.33
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/libev/blobs/sha256:de9342ba34cf
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sh
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/hyperkit/manifests/0.20200908
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/hyperkit/blobs/sha256:26a203b17
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sh
######################################################################## 100.0%
==> Installing dependencies for hyperkit: libev
==> Installing hyperkit dependency: libev
==> Pouring libev--4.33.monterey.bottle.tar.gz
🍺  /usr/local/Cellar/libev/4.33: 12 files, 483.4KB
==> Installing hyperkit
==> Pouring hyperkit--0.20200908.catalina.bottle.tar.gz
Error: The `brew link` step did not complete successfully
The formula built, but is not symlinked into /usr/local
Could not symlink bin/hyperkit
Target /usr/local/bin/hyperkit
already exists. You may want to remove it:
  rm '/usr/local/bin/hyperkit'

To force the link and overwrite all conflicting files:
  brew link --overwrite hyperkit

To list all files that would be deleted:
  brew link --overwrite --dry-run hyperkit

Possible conflicting files are:
/usr/local/bin/hyperkit -> /Applications/Docker.app/Contents/Resources/bin/com.docker.hyperkit
==> Summary
🍺  /usr/local/Cellar/hyperkit/0.20200908: 5 files, 4.3MB
==> Running `brew cleanup hyperkit`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```
```shell
pradeep@learnk8s$ hyperkit -v
hyperkit: 0.20200908

Homepage: https://github.com/docker/hyperkit
License: BSD
```

### Kubectl 
The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs. 

#### Download
https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/

#### Install
```shell
pradeep@learnk8s$ brew install kubectl 
```

```shell
pradeep@learnk8s$ brew install kubectl
==> Downloading https://ghcr.io/v2/homebrew/core/kubernetes-cli/manifests/1.23.5
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/kubernetes-cli/blobs/sha256:cfa
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sh
######################################################################## 100.0%
==> Pouring kubernetes-cli--1.23.5.monterey.bottle.tar.gz
Error: The `brew link` step did not complete successfully
The formula built, but is not symlinked into /usr/local
Could not symlink bin/kubectl
Target /usr/local/bin/kubectl
already exists. You may want to remove it:
  rm '/usr/local/bin/kubectl'

To force the link and overwrite all conflicting files:
  brew link --overwrite kubernetes-cli

To list all files that would be deleted:
  brew link --overwrite --dry-run kubernetes-cli

Possible conflicting files are:
/usr/local/bin/kubectl -> /Applications/Docker.app/Contents/Resources/bin/kubectl
==> Caveats
zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
==> Summary
🍺  /usr/local/Cellar/kubernetes-cli/1.23.5: 227 files, 56.8MB
==> Running `brew cleanup kubernetes-cli`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```

```shell
pradeep@learnk8s$ kubectl version
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.2", GitCommit:"9d142434e3af351a628bffee3939e64c681afa4d", GitTreeState:"clean", BuildDate:"2022-01-19T17:27:51Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.1", GitCommit:"86ec240af8cbd1b60bcc4c03c20da9b98005b92e", GitTreeState:"clean", BuildDate:"2021-12-16T11:34:54Z", GoVersion:"go1.17.5", Compiler:"gc", Platform:"linux/amd64"}
```



### All in one Minikube Cluster (macOS)

```shell
pradeep@learnk8s$ minikube start
😄  minikube v1.25.1 on Darwin 11.6.2
✨  Automatically selected the hyperkit driver. Other choices: vmware, virtualbox, ssh
👍  Starting control plane node minikube in cluster minikube
🔥  Creating hyperkit VM (CPUs=2, Memory=4000MB, Disk=20000MB) ...
❗  This VM is having trouble accessing https://k8s.gcr.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.23.1 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner

❗  /usr/local/bin/kubectl is version 1.21.2, which may have incompatibilites with Kubernetes 1.23.1.
    ▪ Want kubectl v1.23.1? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
 
```



#### Verify
```shell
pradeep@learnk8s$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

```shell
pradeep@learnk8s$ kubectl get pods
No resources found in default namespace.
```

```shell
pradeep@learnk8s$ kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS       AGE
kube-system   coredns-64897985d-nvzjq            1/1     Running   0              5m7s
kube-system   etcd-minikube                      1/1     Running   0              5m7s
kube-system   kube-apiserver-minikube            1/1     Running   0              5m7s
kube-system   kube-controller-manager-minikube   1/1     Running   0              5m7s
kube-system   kube-proxy-2ch8q                   1/1     Running   0              5m7s
kube-system   kube-scheduler-minikube            1/1     Running   0              5m7s
kube-system   storage-provisioner                1/1     Running   0 					    5m7s
```

### Multi-node Minikube Cluster (macOS) 

```shell
pradeep@learnk8s$ minikube start --nodes 2 -p k8s
😄  [k8s] minikube v1.25.1 on Darwin 11.6.2
✨  Automatically selected the hyperkit driver. Other choices: vmware, virtualbox, ssh
👍  Starting control plane node k8s in cluster k8s
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
❗  This VM is having trouble accessing https://k8s.gcr.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.23.1 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ kubelet.cni-conf-dir=/etc/cni/net.mk
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting worker node k8s-m02 in cluster k8s
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.177.17
❗  This VM is having trouble accessing https://k8s.gcr.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.23.1 on Docker 20.10.12 ...
    ▪ env NO_PROXY=192.168.177.17
🔎  Verifying Kubernetes components...

❗  /usr/local/bin/kubectl is version 1.21.2, which may have incompatibilites with Kubernetes 1.23.1.
    ▪ Want kubectl v1.23.1? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "k8s" cluster and "default" namespace by default 
```
#### Verify
```shell
pradeep@learnk8s$ minikube status -p k8s
k8s
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

k8s-m02
type: Worker
host: Running
kubelet: Running
```

### Minikube Cluster with a different runtime (containerd)

Read this post on Kubernetes Blog

https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/

```shell
pradeep@learnk8s$ minikube start --container-runtime=containerd
😄  minikube v1.25.1 on Darwin 11.6.2
✨  Automatically selected the hyperkit driver. Other choices: vmware, virtualbox, ssh
👍  Starting control plane node minikube in cluster minikube
💾  Downloading Kubernetes v1.23.1 preload ...
    > preloaded-images-k8s-v16-v1...: 571.57 MiB / 571.57 MiB  100.00% 3.17 MiB
🔥  Creating hyperkit VM (CPUs=2, Memory=4000MB, Disk=20000MB) ...
❗  This VM is having trouble accessing https://k8s.gcr.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
📦  Preparing Kubernetes v1.23.1 on containerd 1.4.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

❗  /usr/local/bin/kubectl is version 1.21.2, which may have incompatibilites with Kubernetes 1.23.1.
    ▪ Want kubectl v1.23.1? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

```

![kubernetes-logo]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes-logo.png)





