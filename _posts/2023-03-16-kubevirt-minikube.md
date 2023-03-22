---
layout: single
title:  "Getting Started with KubeVirt"
date:   2023-03-16 8:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
 
header:
  overlay_image: /assets/images/kubevirt.png
  og_image: /assets/images/kubernetes.png
  caption: "Photo credit: [**Kubernetes**](https://kubernetes.io)"
  teaser: /assets/images/kubernetes.png
  actions:
    - label: "Learn more"
      url: "https://kubernetes.io"
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubevirt.png"
  
sidebar:
  - title: "Topics"
    nav: my-sidebar
---

# KubeVirt  

## Introduction 

```sh
base) pradeep:~$minikube start --cni=flannel
😄  minikube v1.25.2 on Darwin 13.1
✨  Using the hyperkit driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🎉  minikube 1.29.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.29.0
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🔄  Restarting existing hyperkit VM for "minikube" ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
🔗  Configuring Flannel (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image kubernetesui/metrics-scraper:v1.0.7
    ▪ Using image kubernetesui/dashboard:v2.3.1
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass, dashboard
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
(base) pradeep:~$
```
```sh
(base) pradeep:~$kubectl version             
Client Version: version.Info{Major:"1", Minor:"22", GitVersion:"v1.22.5", GitCommit:"5c99e2ac2ff9a3c549d9ca665e7bc05a3e18f07e", GitTreeState:"clean", BuildDate:"2021-12-16T08:38:33Z", GoVersion:"go1.16.12", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:19:12Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"linux/amd64"}
(base) pradeep:~$
```
Enable KubeVirt addon
```sh
(base) pradeep:~$minikube addons enable kubevirt
    ▪ Using image bitnami/kubectl:1.17
🌟  The 'kubevirt' addon is enabled
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.phase}"
Deploying%                                                                                                                                                                                                  (base) pradeep:~$
```

After some time,

```sh
base) pradeep:~$kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.phase}"

Deployed%                                                                                                                                                                                                   (base) pradeep:~$
```

By default KubeVirt will deploy 7 pods, 3 services, 1 daemonset, 3 deployment apps, 3 replica sets.

```sh
(base) pradeep:~$kubectl get all -n kubevirt
Warning: kubevirt.io/v1 VirtualMachineInstancePresets is now deprecated and will be removed in v2.
NAME                                   READY   STATUS    RESTARTS   AGE
pod/virt-api-5d88d98cfc-pgppj          1/1     Running   0          3m1s
pod/virt-controller-6657ddcd7f-86zbm   1/1     Running   0          2m25s
pod/virt-controller-6657ddcd7f-npx5s   1/1     Running   0          2m25s
pod/virt-handler-k2wc2                 1/1     Running   0          2m25s
pod/virt-operator-6848b95c99-bpq4m     1/1     Running   0          4m10s
pod/virt-operator-6848b95c99-w9n2p     1/1     Running   0          4m9s

NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubevirt-operator-webhook     ClusterIP   10.102.228.252   <none>        443/TCP   3m6s
service/kubevirt-prometheus-metrics   ClusterIP   None             <none>        443/TCP   3m6s
service/virt-api                      ClusterIP   10.99.136.97     <none>        443/TCP   3m6s
service/virt-exportproxy              ClusterIP   10.110.78.235    <none>        443/TCP   3m6s

NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/virt-handler   1         1         1       1            1           kubernetes.io/os=linux   2m25s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/virt-api          1/1     1            1           3m1s
deployment.apps/virt-controller   2/2     2            2           2m25s
deployment.apps/virt-operator     2/2     2            2           4m10s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/virt-api-5d88d98cfc          1         1         1       3m1s
replicaset.apps/virt-controller-6657ddcd7f   2         2         2       2m25s
replicaset.apps/virt-operator-6848b95c99     2         2         2       4m10s

NAME                            AGE    PHASE
kubevirt.kubevirt.io/kubevirt   4m8s   Deployed
(base) pradeep:~$

```
check logs of the kubevirt-install-manager pod:

```sh
(base) pradeep:~$kubectl logs pod/kubevirt-install-manager -n kube-system

Installing KubeVirt version: v0.59.0
namespace/kubevirt created
customresourcedefinition.apiextensions.k8s.io/kubevirts.kubevirt.io created
priorityclass.scheduling.k8s.io/kubevirt-cluster-critical created
clusterrole.rbac.authorization.k8s.io/kubevirt.io:operator created
serviceaccount/kubevirt-operator created
role.rbac.authorization.k8s.io/kubevirt-operator created
rolebinding.rbac.authorization.k8s.io/kubevirt-operator-rolebinding created
clusterrole.rbac.authorization.k8s.io/kubevirt-operator created
clusterrolebinding.rbac.authorization.k8s.io/kubevirt-operator created
deployment.apps/virt-operator created
Using software emulation
kubevirt.kubevirt.io/kubevirt created
(base) pradeep:~$

```

## Virtctl

KubeVirt provides an additional binary called *virtctl* for quick access to the serial and graphical ports of a VM and also handle start/stop operations.

```sh
(base) pradeep:~$kubectl krew install virt
Updated the local copy of plugin index.
  New plugins available:
    * applier
    * colorize-applied
    * community-images
    * confirm
    * count
    * crane
    * discover
    * execws
    * foreach
    * kc
    * kluster-capacity
    * kubescape
    * nodepools
    * oomd
    * permissions
    * ttsum
    * unlimited
Installing plugin: virt
Installed plugin: virt
\
 | Use this plugin:
 | 	kubectl virt
 | Documentation:
 | 	https://github.com/kubevirt/kubectl-virt-plugin
 | Caveats:
 | \
 |  | virt plugin is a wrapper for virtctl originating from the KubeVirt project. In order to use virtctl you will
 |  | need to have KubeVirt installed on your Kubernetes cluster to use it. See https://kubevirt.io/ for details
 |  | 
 |  | See
 |  | 
 |  |   https://kubevirt.io/user-guide/docs/latest/using-virtual-machines/graphical-and-console-access.html
 |  | 
 |  | for a usage example
 | /
/
WARNING: You installed plugin "virt" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.
(base) pradeep:~$

```

With this, we  have deployed KubeVirt on Minikube.
