---
layout: single
title:  "Kubernetes Services"
date:   2022-01-28 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/svc-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"

    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Services


### Services
- ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType.
- NodePort: Exposes the Service on each Node's IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You'll be able to contact the NodePort Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`. 
- LoadBalancer: Exposes the Service externally using a cloud provider's load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.

####  Default Services
From the default namespace
```shell
pradeep@learnk8s$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   33m
```
From all namespaces
```shell
pradeep@learnk8s$ kubectl get services -A
NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  33m
kube-system   kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   33m
```

#### NodePort type service
Expose the deployment as a NodePort type Service
```shell
pradeep@learnk8s$ kubectl expose deployment demo --type=NodePort --port=8080
service/demo exposed
```
```shell
pradeep@learnk8s$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
demo         NodePort    10.101.87.60   <none>        8080:32298/TCP   4s
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          35m
```
Note the port number `32298` assigned to the service

#### Access NodePort type Service
```shell
pradeep@learnk8s$ kubectl get nodes -o wide
NAME      STATUS   ROLES                  AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE              KERNEL-VERSION   CONTAINER-RUNTIME
k8s       Ready    control-plane,master   38m   v1.23.1   192.168.177.27   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
k8s-m02   Ready    <none>                 37m   v1.23.1   192.168.177.28   <none>        Buildroot 2021.02.4   4.19.202         docker://20.10.12
```
From Minikube SSH session, with `k8s-m02` node IP, and NodePort `32298`
```shell
$ curl 192.168.177.28:32298
CLIENT VALUES:
client_address=10.244.1.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://192.168.177.28:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=192.168.177.28:32298
user-agent=curl/7.78.0
BODY:
-no body in request-$
```
with `k8s` node IP and NodePort `32298`
```shell
$ curl 192.168.177.27:32298
CLIENT VALUES:
client_address=10.244.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://192.168.177.27:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=192.168.177.27:32298
user-agent=curl/7.78.0
BODY:
-no body in request-$
```

#### ClusterIP Service
Let's delete the service
```shell
pradeep@learnk8s$ kubectl delete svc demo
service "demo" deleted
```
and expose the same deployment as a ClusterIP (the default ServiceType). If we don't specify any type, its the ClusterIP.

```shell
pradeep@learnk8s$ kubectl expose deployment demo  --port=8080
service/demo exposed
```

```shell
pradeep@learnk8s$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
demo         ClusterIP   10.103.185.242   <none>        8080/TCP   5s
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP    45m
```
Note the ClusterIP assigned to this service `10.103.185.242`.

#### Access Pods with ClusterIP
```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.103.185.242:8080
CLIENT VALUES:
client_address=192.168.177.27
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://10.103.185.242:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=10.103.185.242:8080
user-agent=curl/7.78.0
BODY:
-no body in request-$
```

#### LoadBalancer Service
```shell
pradeep@learnk8s$ kubectl delete service demo
service "demo" deleted
```
```shell
pradeep@learnk8s$ kubectl expose deployment demo –type=LoadBalancer --port=8080
service/demo exposed
```
```shell
pradeep@learnk8s$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
demo         LoadBalancer   10.98.252.85   <pending>     8080:30646/TCP   3s
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP          50m
```
Open another terminal and initiate `minikube tunnel`
```shell
pradeep@learnk8s$ minikube tunnel -p k8s
Password:
Status:
	machine: k8s
	pid: 95721
	route: 10.96.0.0/12 -> 192.168.177.27
	minikube: Running
	services: [demo]
    errors:
		minikube: no errors
		router: no errors
		loadbalancer emulator: no errors
```
Return to the first terminal, Verify that `EXTERNAL-IP` is populated now
```shell
pradeep@learnk8s$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
demo         LoadBalancer   10.98.252.85   10.98.252.85   8080:30646/TCP   3m27s
kubernetes   ClusterIP      10.96.0.1      <none>         443/TCP          53m
```
NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.

#### Access Pods with LoadBalancer Service
```shell
pradeep@learnk8s$ minikube service demo -p k8s
|-----------|------|-------------|-----------------------------|
| NAMESPACE | NAME | TARGET PORT |             URL             |
|-----------|------|-------------|-----------------------------|
| default   | demo |        8080 | http://192.168.177.27:30646 |
|-----------|------|-------------|-----------------------------|
🎉  Opening service default/demo in default browser..
```

![kubernetes-lb-svc](/assets/images/kubernetes-lb-svc.png) 

