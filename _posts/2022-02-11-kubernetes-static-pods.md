---
layout: single
title:  "Kubernetes Static Pods"
date:   2022-02-11 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/kubernetes.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
    
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Static Pods


### Static Pods

*Static Pods* are managed directly by the kubelet daemon on a specific node, without the API server observing them. Unlike Pods that are managed by the control plane; instead, the kubelet watches each static Pod (and restarts it if it fails).

The Pod names will be suffixed with the node hostname with a leading hyphen.

Manifests are standard Pod definitions in JSON or YAML format in a specific directory. Use the staticPodPath: `<the directory>` field in the kubelet configuration file, which periodically scans the directory and creates/deletes static Pods as YAML/JSON files appear/disappear there. 

Let's explore our minikube environment to see if any Static Pods are there.

First SSH to the minikube node and search for the `kubelet` process. The result shows all configuration parameteres used by the `kubelet`.
```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ps -aux | grep kubelet
root        4244 11.6  4.5 1946788 100436 ?      Ssl  Feb08 238:40 /var/lib/minikube/binaries/v1.23.1/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --cni-conf-dir=/etc/cni/net.mk --config=/var/lib/kubelet/config.yaml --container-runtime=docker --hostname-override=k8s --housekeeping-interval=5m --kubeconfig=/etc/kubernetes/kubelet.conf --network-plugin=cni --node-ip=192.168.177.27
root      412294 15.3 13.5 1042556 296888 ?      Ssl  01:40  10:27 kube-apiserver --advertise-address=192.168.177.27 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/var/lib/minikube/certs/ca.crt --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota --enable-bootstrap-token-auth=true --etcd-cafile=/var/lib/minikube/certs/etcd/ca.crt --etcd-certfile=/var/lib/minikube/certs/apiserver-etcd-client.crt --etcd-keyfile=/var/lib/minikube/certs/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/var/lib/minikube/certs/apiserver-kubelet-client.crt --kubelet-client-key=/var/lib/minikube/certs/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/var/lib/minikube/certs/front-proxy-client.crt --proxy-client-key-file=/var/lib/minikube/certs/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/var/lib/minikube/certs/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=8443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/var/lib/minikube/certs/sa.pub --service-account-signing-key-file=/var/lib/minikube/certs/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/var/lib/minikube/certs/apiserver.crt --tls-private-key-file=/var/lib/minikube/certs/apiserver.key
docker    429542  0.0  0.0   3348   448 pts/0    S+   02:48   0:00 grep kubelet
```
As seen above, the kubelet config settings are present in `--config=/var/lib/kubelet/config.yaml`. You can explore this file to get the `staticPodPath`.
```shell
$ more /var/lib/kubelet/config.yaml | grep static
staticPodPath: /etc/kubernetes/manifests
```
Now that we know the `staticPodPath`, lets see what manifests are currently defined.

```shell
$ ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml	kube-controller-manager.yaml  kube-scheduler.yaml
$
```
There are four manifest files defined in this location. Looking at the name of the files, you can relate them to the Kubernetes core components.

Let us re-visit the `kube-system` namespace and get all the pods running there.
```shell
$ exit
logout
pradeep@learnk8s$ kubectl get pods -n kube-system
NAME                          READY   STATUS    RESTARTS        AGE
coredns-64897985d-r9tzv       1/1     Running   8 (114m ago)    6d2h
etcd-k8s                      1/1     Running   2 (114m ago)    6d2h
kindnet-jpxdd                 1/1     Running   5 (8h ago)      6d2h
kindnet-p77mb                 1/1     Running   50 (118m ago)   6d2h
kube-apiserver-k8s            1/1     Running   3 (8h ago)      6d2h
kube-controller-manager-k8s   1/1     Running   4 (113m ago)    6d2h
kube-proxy-fszkr              1/1     Running   1 (2d15h ago)   6d2h
kube-proxy-m747v              1/1     Running   1               6d2h
kube-scheduler-k8s            1/1     Running   2 (8h ago)      6d2h
storage-provisioner           1/1     Running   49 (81m ago)    6d2h
```
Pay special attention to the `-k8s` string in the name of the Pods: `etcd-k8s`, `kube-apiserver-k8s`, `kube-controller-manager-k8`, and `kube-scheduler-k8s`. As mentioned earlier, The StaticPod names will be suffixed with the node hostname (`k8s` in our case) with a leading hyphen.


#### Create a Static Pod in the worker node
Now that we have seen the static pods deployed by the system during cluster setup, let us manually deploy an `nginx` pod in the second node (`k8s-m02`) of our minikube cluster `k8s`.

For this first, we need to SSH to that node: `k8s-m02`. To do this, add `-n k8s-m02` option to the minikube ssh command that we have been using so far.

```shell
pradeep@learnk8s$ minikube ssh -n k8s-m02 -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ip a show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether be:02:3c:97:9d:85 brd ff:ff:ff:ff:ff:ff
    inet 192.168.177.28/24 brd 192.168.177.255 scope global dynamic eth0
       valid_lft 47485sec preferred_lft 47485sec

```

This confirms that we logged in to the `k8s-m02` node which has the IP address: 192.168.177.28. 

Create a new file called `nginx.yaml` in the staicPodPath location, that is `/etc/kubernetes/manifests/`.

```yaml
$ sudo vi /etc/kubernetes/manifests/nginx.yaml
$ cat /etc/kubernetes/manifests/nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
$ exit
logout
```
By the presence of this file in the `k8s-m02` node, the kubelet creates this Pod automatically now.

To verify,
```shell
pradeep@learnk8s$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
demo-6c54f77c95-mgz7f   1/1     Running   1          5d22h
demo-6c54f77c95-q679r   1/1     Running   1          4d22h
demo-6c54f77c95-qqzbf   1/1     Running   1          4d22h
demo-6c54f77c95-vjgc2   1/1     Running   1          4d22h
demo-6c54f77c95-wv78b   1/1     Running   1          5d22h
demo-ds-gtcf7           1/1     Running   0          14h
demo-ds-kkw4g           1/1     Running   0          14h
nginx-k8s-m02           1/1     Running   0          18s
nginx-manual            1/1     Running   1          4d21h
nginx-no-tolerate       1/1     Running   1          4d14h
nginx-node-selector     1/1     Running   0          2d15h
nginx-taint-demo        1/1     Running   1          4d14h
with-node-affinity      1/1     Running   0          2d15h
```

Our nginx static pod is successfully Running on `k8s-m02` node, as seen above, as pod named `nginx-k8s-m02`. 

Another way to confirm that this Pod is indeed running on the `k8s-m02` node is 
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep m02
demo-ds-gtcf7           1/1     Running   0          14h     10.244.1.5    k8s-m02   <none>           <none>
nginx-k8s-m02           1/1     Running   0          18m     10.244.1.6    k8s-m02   <none>           <none>
nginx-node-selector     1/1     Running   0          2d15h   10.244.1.2    k8s-m02   <none>           <none>
with-node-affinity      1/1     Running   0          2d15h   10.244.1.3    k8s-m02   <none>           <none>
```

