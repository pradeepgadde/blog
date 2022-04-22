---
layout: single
title:  "Kubeadm Tokens and Print  Join Command"
date:   2022-04-04 10:55:04 +0530
categories: Kubernetes
tags: kubeadm minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/k-proxy-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---


# Kubernetes Kube-Proxy IPTables

Kubernetes *Services* are implemented using iptables rules (with default config) on all nodes. Every time a *Service* has been altered, created, deleted or *Endpoints* of a *Service* have changed, the `kube-apiserver` contacts every node's `kube-proxy` to update the iptables rules according to the current state.



> Cluster Nodes

```sh
lab@k8s1:~$ kubectl get nodes -o wide
NAME   STATUS   ROLES                  AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s1   Ready    control-plane,master   6d14h   v1.22.8   10.210.40.172   <none>        Ubuntu 20.04.4 LTS   5.13.0-37-generic   docker://20.10.14
k8s2   Ready    <none>                 6d13h   v1.22.8   10.210.40.173   <none>        Ubuntu 20.04.4 LTS   5.13.0-37-generic   docker://20.10.14
k8s3   Ready    <none>                 6d11h   v1.22.8   10.210.40.175   <none>        Ubuntu 20.04.4 LTS   5.13.0-37-generic   docker://20.10.14
lab@k8s1:~$
```

> Kube-system namespace components
```sh
lab@k8s1:~$ kubectl get pods -n kube-system
NAME                           READY   STATUS    RESTARTS        AGE
coredns-78fcd69978-cvpx2       1/1     Running   0               6d10h
coredns-78fcd69978-hf5sj       1/1     Running   0               6d10h
etcd-k8s1                      1/1     Running   0               5d4h
kube-apiserver-k8s1            1/1     Running   2 (5d19h ago)   6d14h
kube-controller-manager-k8s1   1/1     Running   2 (5d19h ago)   6d14h
kube-flannel-ds-lhcwb          1/1     Running   0               6d14h
kube-flannel-ds-ph9gg          1/1     Running   0               6d13h
kube-flannel-ds-xm28z          1/1     Running   0               6d11h
kube-proxy-brrvs               1/1     Running   0               6d14h
kube-proxy-cdl2t               1/1     Running   0               6d13h
kube-proxy-v8r74               1/1     Running   0               6d11h
kube-scheduler-k8s1            1/1     Running   3 (5d19h ago)   6d14h
lab@k8s1:~$
```

> Kube-Proxy Logs
```sh
lab@k8s1:~$ kubectl logs kube-proxy-brrvs -n kube-system
I0330 09:28:56.497083       1 node.go:172] Successfully retrieved node IP: 10.210.40.172
I0330 09:28:56.497156       1 server_others.go:140] Detected node IP 10.210.40.172
W0330 09:28:56.498045       1 server_others.go:565] Unknown proxy mode "", assuming iptables proxy
I0330 09:28:56.608251       1 server_others.go:206] kube-proxy running in dual-stack mode, IPv4-primary
I0330 09:28:56.608336       1 server_others.go:212] Using iptables Proxier.
I0330 09:28:56.608368       1 server_others.go:219] creating dualStackProxier for iptables.
W0330 09:28:56.609876       1 server_others.go:495] detect-local-mode set to ClusterCIDR, but no IPv6 cluster CIDR defined, , defaulting to no-op detect-local for IPv6
I0330 09:28:56.614112       1 server.go:649] Version: v1.22.8
I0330 09:28:56.618235       1 conntrack.go:52] Setting nf_conntrack_max to 131072
I0330 09:28:56.639543       1 config.go:315] Starting service config controller
I0330 09:28:56.639568       1 shared_informer.go:240] Waiting for caches to sync for service config
I0330 09:28:56.639617       1 config.go:224] Starting endpoint slice config controller
I0330 09:28:56.639625       1 shared_informer.go:240] Waiting for caches to sync for endpoint slice config
I0330 09:28:56.746010       1 shared_informer.go:247] Caches are synced for endpoint slice config
I0330 09:28:56.746067       1 shared_informer.go:247] Caches are synced for service config
E0330 09:29:10.157053       1 event_broadcaster.go:253] Server rejected event '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta:v1.ObjectMeta{Name:"k8s1.16e11e8ea77f02d9", GenerateName:"", Namespace:"default", SelfLink:"", UID:"", ResourceVersion:"", Generation:0, CreationTimestamp:v1.Time{Time:time.Time{wall:0x0, ext:0, loc:(*time.Location)(nil)}}, DeletionTimestamp:(*v1.Time)(nil), DeletionGracePeriodSeconds:(*int64)(nil), Labels:map[string]string(nil), Annotations:map[string]string(nil), OwnerReferences:[]v1.OwnerReference(nil), Finalizers:[]string(nil), ClusterName:"", ManagedFields:[]v1.ManagedFieldsEntry(nil)}, EventTime:v1.MicroTime{Time:time.Time{wall:0xc089269626108e2e, ext:796548160, loc:(*time.Location)(0x2d87400)}}, Series:(*v1.EventSeries)(nil), ReportingController:"kube-proxy", ReportingInstance:"kube-proxy-k8s1", Action:"StartKubeProxy", Reason:"Starting", Regarding:v1.ObjectReference{Kind:"Node", Namespace:"", Name:"k8s1", UID:"k8s1", APIVersion:"", ResourceVersion:"", FieldPath:""}, Related:(*v1.ObjectReference)(nil), Note:"", Type:"Normal", DeprecatedSource:v1.EventSource{Component:"", Host:""}, DeprecatedFirstTimestamp:v1.Time{Time:time.Time{wall:0x0, ext:0, loc:(*time.Location)(nil)}}, DeprecatedLastTimestamp:v1.Time{Time:time.Time{wall:0x0, ext:0, loc:(*time.Location)(nil)}}, DeprecatedCount:0}': 'etcdserver: request timed out' (will not retry!)
lab@k8s1:~$
```
From `Using iptables Proxier`, We can see that the Kube-proxy is using IPtables.



Create a Pod

```sh
lab@k8s1:~$ kubectl run nginx --image=nginx
pod/nginx created
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl get pods nginx -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          68s   10.244.3.38   k8s3   <none>           <none>
lab@k8s1:~$
```

Expose the Pod

```sh
lab@k8s1:~$ kubectl expose pod nginx --name=my-nginx-svc --port=3000 --target-port=80
service/my-nginx-svc exposed
lab@k8s1:~$
```

Verify the Services

```sh
lab@k8s1:~$ kubectl get svc
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP    6d14h
my-nginx-svc   ClusterIP   10.98.69.149   <none>        3000/TCP   4s
lab@k8s1:~$
```

We can see that a ClusterIP service called `my-nginx-svc` got created with a Cluster-IP of `10.98.69.149` and port of `3000`.

Verify the reachability with ClusterIP:Port 

```sh
lab@k8s1:~$ curl 10.98.69.149:3000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
lab@k8s1:~$
```

Verify the endpoints

```sh
lab@k8s1:~$ kubectl get ep
NAME           ENDPOINTS            AGE
kubernetes     192.168.100.1:6443   6d14h
my-nginx-svc   10.244.3.38:80       2m9s
lab@k8s1:~$
```

The endpoint `10.244.3.38:80` is nothing but the `nginx` pod that we created earlier.

Lets verify the IPtables

```sh
lab@k8s1:~$ sudo iptables-save
# Generated by iptables-save v1.8.4 on Mon Apr  4 22:19:25 2022
*mangle
:PREROUTING ACCEPT [85611925:14193920045]
:INPUT ACCEPT [85611917:14193918979]
:FORWARD ACCEPT [8:1066]
:OUTPUT ACCEPT [85137391:14856492735]
:POSTROUTING ACCEPT [85138429:14856571195]
:KUBE-KUBELET-CANARY - [0:0]
:KUBE-PROXY-CANARY - [0:0]
:LIBVIRT_PRT - [0:0]
-A POSTROUTING -j LIBVIRT_PRT
-A LIBVIRT_PRT -o virbr0 -p udp -m udp --dport 68 -j CHECKSUM --checksum-fill
COMMIT
# Completed on Mon Apr  4 22:19:25 2022
# Generated by iptables-save v1.8.4 on Mon Apr  4 22:19:25 2022
*filter
:INPUT ACCEPT [175964:28858642]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [175626:30251694]
:DOCKER - [0:0]
:DOCKER-ISOLATION-STAGE-1 - [0:0]
:DOCKER-ISOLATION-STAGE-2 - [0:0]
:DOCKER-USER - [0:0]
:KUBE-EXTERNAL-SERVICES - [0:0]
:KUBE-FIREWALL - [0:0]
:KUBE-FORWARD - [0:0]
:KUBE-KUBELET-CANARY - [0:0]
:KUBE-NODEPORTS - [0:0]
:KUBE-PROXY-CANARY - [0:0]
:KUBE-SERVICES - [0:0]
:LIBVIRT_FWI - [0:0]
:LIBVIRT_FWO - [0:0]
:LIBVIRT_FWX - [0:0]
:LIBVIRT_INP - [0:0]
:LIBVIRT_OUT - [0:0]
-A INPUT -m comment --comment "kubernetes health check service ports" -j KUBE-NODEPORTS
-A INPUT -m conntrack --ctstate NEW -m comment --comment "kubernetes externally-visible service portals" -j KUBE-EXTERNAL-SERVICES
-A INPUT -j KUBE-FIREWALL
-A INPUT -j LIBVIRT_INP
-A FORWARD -m comment --comment "kubernetes forwarding rules" -j KUBE-FORWARD
-A FORWARD -m conntrack --ctstate NEW -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A FORWARD -m conntrack --ctstate NEW -m comment --comment "kubernetes externally-visible service portals" -j KUBE-EXTERNAL-SERVICES
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
-A FORWARD -j LIBVIRT_FWX
-A FORWARD -j LIBVIRT_FWI
-A FORWARD -j LIBVIRT_FWO
-A FORWARD -s 10.244.0.0/16 -m comment --comment "flanneld forward" -j ACCEPT
-A FORWARD -d 10.244.0.0/16 -m comment --comment "flanneld forward" -j ACCEPT
-A OUTPUT -m conntrack --ctstate NEW -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A OUTPUT -j KUBE-FIREWALL
-A OUTPUT -j LIBVIRT_OUT
-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2
-A DOCKER-ISOLATION-STAGE-1 -j RETURN
-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP
-A DOCKER-ISOLATION-STAGE-2 -j RETURN
-A DOCKER-USER -j RETURN
-A KUBE-FIREWALL -m comment --comment "kubernetes firewall for dropping marked packets" -m mark --mark 0x8000/0x8000 -j DROP
-A KUBE-FIREWALL ! -s 127.0.0.0/8 -d 127.0.0.0/8 -m comment --comment "block incoming localnet connections" -m conntrack ! --ctstate RELATED,ESTABLISHED,DNAT -j DROP
-A KUBE-FORWARD -m conntrack --ctstate INVALID -j DROP
-A KUBE-FORWARD -m comment --comment "kubernetes forwarding rules" -m mark --mark 0x4000/0x4000 -j ACCEPT
-A KUBE-FORWARD -m comment --comment "kubernetes forwarding conntrack pod source rule" -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A KUBE-FORWARD -m comment --comment "kubernetes forwarding conntrack pod destination rule" -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A LIBVIRT_FWI -d 192.168.122.0/24 -o virbr0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A LIBVIRT_FWI -o virbr0 -j REJECT --reject-with icmp-port-unreachable
-A LIBVIRT_FWO -s 192.168.122.0/24 -i virbr0 -j ACCEPT
-A LIBVIRT_FWO -i virbr0 -j REJECT --reject-with icmp-port-unreachable
-A LIBVIRT_FWX -i virbr0 -o virbr0 -j ACCEPT
-A LIBVIRT_INP -i virbr0 -p udp -m udp --dport 53 -j ACCEPT
-A LIBVIRT_INP -i virbr0 -p tcp -m tcp --dport 53 -j ACCEPT
-A LIBVIRT_INP -i virbr0 -p udp -m udp --dport 67 -j ACCEPT
-A LIBVIRT_INP -i virbr0 -p tcp -m tcp --dport 67 -j ACCEPT
-A LIBVIRT_OUT -o virbr0 -p udp -m udp --dport 53 -j ACCEPT
-A LIBVIRT_OUT -o virbr0 -p tcp -m tcp --dport 53 -j ACCEPT
-A LIBVIRT_OUT -o virbr0 -p udp -m udp --dport 68 -j ACCEPT
-A LIBVIRT_OUT -o virbr0 -p tcp -m tcp --dport 68 -j ACCEPT
COMMIT
# Completed on Mon Apr  4 22:19:25 2022
# Generated by iptables-save v1.8.4 on Mon Apr  4 22:19:25 2022
*nat
:PREROUTING ACCEPT [17:1461]
:INPUT ACCEPT [17:1461]
:OUTPUT ACCEPT [1940:116900]
:POSTROUTING ACCEPT [1920:115200]
:DOCKER - [0:0]
:KUBE-KUBELET-CANARY - [0:0]
:KUBE-MARK-DROP - [0:0]
:KUBE-MARK-MASQ - [0:0]
:KUBE-NODEPORTS - [0:0]
:KUBE-POSTROUTING - [0:0]
:KUBE-PROXY-CANARY - [0:0]
:KUBE-SEP-2NK74F4BXH6S2KMP - [0:0]
:KUBE-SEP-DSBJGVCOU4RGUKVK - [0:0]
:KUBE-SEP-EMMXYKITEUOEZBH3 - [0:0]
:KUBE-SEP-FVQSBIWR5JTECIVC - [0:0]
:KUBE-SEP-KGEJJ2QZOQ6ELXUE - [0:0]
:KUBE-SEP-LASJGFFJP3UOS6RQ - [0:0]
:KUBE-SEP-LPGSDLJ3FDW46N4W - [0:0]
:KUBE-SEP-QCMSUSADBJSJISDV - [0:0]
:KUBE-SERVICES - [0:0]
:KUBE-SVC-ERIFXISQEP7F7OF4 - [0:0]
:KUBE-SVC-JD5MR3NA4I4DYORP - [0:0]
:KUBE-SVC-NE3M6G7FP5C7DVRO - [0:0]
:KUBE-SVC-NPX46M4PTMTKRN6Y - [0:0]
:KUBE-SVC-TCOU7JCQXEZGVUNU - [0:0]
:LIBVIRT_PRT - [0:0]
-A PREROUTING -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A PREROUTING -d 10.210.52.212/32 -p tcp -m tcp --dport 2201 -j DNAT --to-destination 172.25.11.1:22
-A PREROUTING -d 10.210.52.212/32 -p tcp -m tcp --dport 2202 -j DNAT --to-destination 172.25.11.2:22
-A PREROUTING -d 10.210.52.212/32 -p tcp -m tcp --dport 2203 -j DNAT --to-destination 172.25.11.3:22
-A PREROUTING -d 10.210.52.212/32 -p tcp -m tcp --dport 2204 -j DNAT --to-destination 172.25.11.4:22
-A PREROUTING -d 10.210.52.212/32 -p tcp -m tcp --dport 2205 -j DNAT --to-destination 172.25.11.5:22
-A PREROUTING -d 10.210.52.212/32 -p tcp -m tcp --dport 2206 -j DNAT --to-destination 172.25.11.6:22
-A PREROUTING -d 10.210.52.212/32 -p tcp -m tcp --dport 2207 -j DNAT --to-destination 172.25.11.7:22
-A PREROUTING -d 10.210.52.212/32 -p tcp -m tcp --dport 2208 -j DNAT --to-destination 172.25.11.8:22
-A PREROUTING -d 10.210.52.212/32 -p tcp -m tcp --dport 2209 -j DNAT --to-destination 172.25.11.9:22
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
-A POSTROUTING -j LIBVIRT_PRT
-A POSTROUTING -d 172.25.11.0/24 -o eth1 -p tcp -m tcp --dport 22 -j SNAT --to-source 172.25.11.254
-A POSTROUTING -o eth0 -j MASQUERADE
-A POSTROUTING -s 10.244.0.0/16 -d 10.244.0.0/16 -m comment --comment "flanneld masq" -j RETURN
-A POSTROUTING -s 10.244.0.0/16 ! -d 224.0.0.0/4 -m comment --comment "flanneld masq" -j MASQUERADE --random-fully
-A POSTROUTING ! -s 10.244.0.0/16 -d 10.244.0.0/24 -m comment --comment "flanneld masq" -j RETURN
-A POSTROUTING ! -s 10.244.0.0/16 -d 10.244.0.0/16 -m comment --comment "flanneld masq" -j MASQUERADE --random-fully
-A DOCKER -i docker0 -j RETURN
-A KUBE-MARK-DROP -j MARK --set-xmark 0x8000/0x8000
-A KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000
-A KUBE-POSTROUTING -m mark ! --mark 0x4000/0x4000 -j RETURN
-A KUBE-POSTROUTING -j MARK --set-xmark 0x4000/0x0
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -j MASQUERADE --random-fully
-A KUBE-SEP-2NK74F4BXH6S2KMP -s 10.244.2.21/32 -m comment --comment "kube-system/kube-dns:dns" -j KUBE-MARK-MASQ
-A KUBE-SEP-2NK74F4BXH6S2KMP -p udp -m comment --comment "kube-system/kube-dns:dns" -m udp -j DNAT --to-destination 10.244.2.21:53
-A KUBE-SEP-DSBJGVCOU4RGUKVK -s 10.244.3.38/32 -m comment --comment "default/my-nginx-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-DSBJGVCOU4RGUKVK -p tcp -m comment --comment "default/my-nginx-svc" -m tcp -j DNAT --to-destination 10.244.3.38:80
-A KUBE-SEP-EMMXYKITEUOEZBH3 -s 192.168.100.1/32 -m comment --comment "default/kubernetes:https" -j KUBE-MARK-MASQ
-A KUBE-SEP-EMMXYKITEUOEZBH3 -p tcp -m comment --comment "default/kubernetes:https" -m tcp -j DNAT --to-destination 192.168.100.1:6443
-A KUBE-SEP-FVQSBIWR5JTECIVC -s 10.244.0.5/32 -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-MARK-MASQ
-A KUBE-SEP-FVQSBIWR5JTECIVC -p tcp -m comment --comment "kube-system/kube-dns:metrics" -m tcp -j DNAT --to-destination 10.244.0.5:9153
-A KUBE-SEP-KGEJJ2QZOQ6ELXUE -s 10.244.2.21/32 -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-MARK-MASQ
-A KUBE-SEP-KGEJJ2QZOQ6ELXUE -p tcp -m comment --comment "kube-system/kube-dns:metrics" -m tcp -j DNAT --to-destination 10.244.2.21:9153
-A KUBE-SEP-LASJGFFJP3UOS6RQ -s 10.244.0.5/32 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-MARK-MASQ
-A KUBE-SEP-LASJGFFJP3UOS6RQ -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp" -m tcp -j DNAT --to-destination 10.244.0.5:53
-A KUBE-SEP-LPGSDLJ3FDW46N4W -s 10.244.0.5/32 -m comment --comment "kube-system/kube-dns:dns" -j KUBE-MARK-MASQ
-A KUBE-SEP-LPGSDLJ3FDW46N4W -p udp -m comment --comment "kube-system/kube-dns:dns" -m udp -j DNAT --to-destination 10.244.0.5:53
-A KUBE-SEP-QCMSUSADBJSJISDV -s 10.244.2.21/32 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-MARK-MASQ
-A KUBE-SEP-QCMSUSADBJSJISDV -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp" -m tcp -j DNAT --to-destination 10.244.2.21:53
-A KUBE-SERVICES -d 10.96.0.1/32 -p tcp -m comment --comment "default/kubernetes:https cluster IP" -m tcp --dport 443 -j KUBE-SVC-NPX46M4PTMTKRN6Y
-A KUBE-SERVICES -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:metrics cluster IP" -m tcp --dport 9153 -j KUBE-SVC-JD5MR3NA4I4DYORP
-A KUBE-SERVICES -d 10.96.0.10/32 -p udp -m comment --comment "kube-system/kube-dns:dns cluster IP" -m udp --dport 53 -j KUBE-SVC-TCOU7JCQXEZGVUNU
-A KUBE-SERVICES -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-SVC-ERIFXISQEP7F7OF4
-A KUBE-SERVICES -d 10.98.69.149/32 -p tcp -m comment --comment "default/my-nginx-svc cluster IP" -m tcp --dport 3000 -j KUBE-SVC-NE3M6G7FP5C7DVRO
-A KUBE-SERVICES -m comment --comment "kubernetes service nodeports; NOTE: this must be the last rule in this chain" -m addrtype --dst-type LOCAL -j KUBE-NODEPORTS
-A KUBE-SVC-ERIFXISQEP7F7OF4 ! -s 10.244.0.0/16 -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-MARK-MASQ
-A KUBE-SVC-ERIFXISQEP7F7OF4 -m comment --comment "kube-system/kube-dns:dns-tcp" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-LASJGFFJP3UOS6RQ
-A KUBE-SVC-ERIFXISQEP7F7OF4 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-SEP-QCMSUSADBJSJISDV
-A KUBE-SVC-JD5MR3NA4I4DYORP ! -s 10.244.0.0/16 -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:metrics cluster IP" -m tcp --dport 9153 -j KUBE-MARK-MASQ
-A KUBE-SVC-JD5MR3NA4I4DYORP -m comment --comment "kube-system/kube-dns:metrics" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-FVQSBIWR5JTECIVC
-A KUBE-SVC-JD5MR3NA4I4DYORP -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-SEP-KGEJJ2QZOQ6ELXUE
-A KUBE-SVC-NE3M6G7FP5C7DVRO ! -s 10.244.0.0/16 -d 10.98.69.149/32 -p tcp -m comment --comment "default/my-nginx-svc cluster IP" -m tcp --dport 3000 -j KUBE-MARK-MASQ
-A KUBE-SVC-NE3M6G7FP5C7DVRO -m comment --comment "default/my-nginx-svc" -j KUBE-SEP-DSBJGVCOU4RGUKVK
-A KUBE-SVC-NPX46M4PTMTKRN6Y ! -s 10.244.0.0/16 -d 10.96.0.1/32 -p tcp -m comment --comment "default/kubernetes:https cluster IP" -m tcp --dport 443 -j KUBE-MARK-MASQ
-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes:https" -j KUBE-SEP-EMMXYKITEUOEZBH3
-A KUBE-SVC-TCOU7JCQXEZGVUNU ! -s 10.244.0.0/16 -d 10.96.0.10/32 -p udp -m comment --comment "kube-system/kube-dns:dns cluster IP" -m udp --dport 53 -j KUBE-MARK-MASQ
-A KUBE-SVC-TCOU7JCQXEZGVUNU -m comment --comment "kube-system/kube-dns:dns" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-LPGSDLJ3FDW46N4W
-A KUBE-SVC-TCOU7JCQXEZGVUNU -m comment --comment "kube-system/kube-dns:dns" -j KUBE-SEP-2NK74F4BXH6S2KMP
-A LIBVIRT_PRT -s 192.168.122.0/24 -d 224.0.0.0/24 -j RETURN
-A LIBVIRT_PRT -s 192.168.122.0/24 -d 255.255.255.255/32 -j RETURN
-A LIBVIRT_PRT -s 192.168.122.0/24 ! -d 192.168.122.0/24 -p tcp -j MASQUERADE --to-ports 1024-65535
-A LIBVIRT_PRT -s 192.168.122.0/24 ! -d 192.168.122.0/24 -p udp -j MASQUERADE --to-ports 1024-65535
-A LIBVIRT_PRT -s 192.168.122.0/24 ! -d 192.168.122.0/24 -j MASQUERADE
COMMIT
# Completed on Mon Apr  4 22:19:25 2022
lab@k8s1:~$
```



That was the complete list, let us narrow down the results just for the `my-nginx-svc` service.

```sh
lab@k8s1:~$ sudo iptables-save | grep nginx
-A KUBE-SEP-DSBJGVCOU4RGUKVK -s 10.244.3.38/32 -m comment --comment "default/my-nginx-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-DSBJGVCOU4RGUKVK -p tcp -m comment --comment "default/my-nginx-svc" -m tcp -j DNAT --to-destination 10.244.3.38:80
-A KUBE-SERVICES -d 10.98.69.149/32 -p tcp -m comment --comment "default/my-nginx-svc cluster IP" -m tcp --dport 3000 -j KUBE-SVC-NE3M6G7FP5C7DVRO
-A KUBE-SVC-NE3M6G7FP5C7DVRO ! -s 10.244.0.0/16 -d 10.98.69.149/32 -p tcp -m comment --comment "default/my-nginx-svc cluster IP" -m tcp --dport 3000 -j KUBE-MARK-MASQ
-A KUBE-SVC-NE3M6G7FP5C7DVRO -m comment --comment "default/my-nginx-svc" -j KUBE-SEP-DSBJGVCOU4RGUKVK
lab@k8s1:~$
```

We can see there is a DNAT rule, to translate from clusterIP:Port to 10.244.3.38:80

And similar rules present in other two nodes (`k8s2` and `k8s3`) of the cluster as well.



```sh
lab@k8s2:~$ sudo iptables-save | grep nginx
[sudo] password for lab:
-A KUBE-SEP-DSBJGVCOU4RGUKVK -s 10.244.3.38/32 -m comment --comment "default/my-nginx-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-DSBJGVCOU4RGUKVK -p tcp -m comment --comment "default/my-nginx-svc" -m tcp -j DNAT --to-destination 10.244.3.38:80
-A KUBE-SERVICES -d 10.98.69.149/32 -p tcp -m comment --comment "default/my-nginx-svc cluster IP" -m tcp --dport 3000 -j KUBE-SVC-NE3M6G7FP5C7DVRO
-A KUBE-SVC-NE3M6G7FP5C7DVRO ! -s 10.244.0.0/16 -d 10.98.69.149/32 -p tcp -m comment --comment "default/my-nginx-svc cluster IP" -m tcp --dport 3000 -j KUBE-MARK-MASQ
-A KUBE-SVC-NE3M6G7FP5C7DVRO -m comment --comment "default/my-nginx-svc" -j KUBE-SEP-DSBJGVCOU4RGUKVK
lab@k8s2:~$
```

```sh
lab@k8s3:~$ sudo iptables-save | grep nginx
[sudo] password for lab:
-A KUBE-SEP-DSBJGVCOU4RGUKVK -s 10.244.3.38/32 -m comment --comment "default/my-nginx-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-DSBJGVCOU4RGUKVK -p tcp -m comment --comment "default/my-nginx-svc" -m tcp -j DNAT --to-destination 10.244.3.38:80
-A KUBE-SERVICES -d 10.98.69.149/32 -p tcp -m comment --comment "default/my-nginx-svc cluster IP" -m tcp --dport 3000 -j KUBE-SVC-NE3M6G7FP5C7DVRO
-A KUBE-SVC-NE3M6G7FP5C7DVRO ! -s 10.244.0.0/16 -d 10.98.69.149/32 -p tcp -m comment --comment "default/my-nginx-svc cluster IP" -m tcp --dport 3000 -j KUBE-MARK-MASQ
-A KUBE-SVC-NE3M6G7FP5C7DVRO -m comment --comment "default/my-nginx-svc" -j KUBE-SEP-DSBJGVCOU4RGUKVK
lab@k8s3:~$
```

Let us delete the service now.

```sh
lab@k8s1:~$ kubectl delete service my-nginx-svc
service "my-nginx-svc" deleted

```

Check the IPtables again

```sh
lab@k8s1:~$ sudo iptables-save | grep nginx
[sudo] password for lab:
lab@k8s1:~$
```

Right after deleting the `my-nginx-svc` service, corresponding iptables are removed automatically.



Similarly, we can verify the related IPtable entries for the default `kube-dns` service in the `kube-system` namespace.

```sh
lab@k8s1:~$ sudo iptables-save | grep kube-dns
-A KUBE-SEP-2NK74F4BXH6S2KMP -s 10.244.2.21/32 -m comment --comment "kube-system/kube-dns:dns" -j KUBE-MARK-MASQ
-A KUBE-SEP-2NK74F4BXH6S2KMP -p udp -m comment --comment "kube-system/kube-dns:dns" -m udp -j DNAT --to-destination 10.244.2.21:53
-A KUBE-SEP-FVQSBIWR5JTECIVC -s 10.244.0.5/32 -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-MARK-MASQ
-A KUBE-SEP-FVQSBIWR5JTECIVC -p tcp -m comment --comment "kube-system/kube-dns:metrics" -m tcp -j DNAT --to-destination 10.244.0.5:9153
-A KUBE-SEP-KGEJJ2QZOQ6ELXUE -s 10.244.2.21/32 -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-MARK-MASQ
-A KUBE-SEP-KGEJJ2QZOQ6ELXUE -p tcp -m comment --comment "kube-system/kube-dns:metrics" -m tcp -j DNAT --to-destination 10.244.2.21:9153
-A KUBE-SEP-LASJGFFJP3UOS6RQ -s 10.244.0.5/32 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-MARK-MASQ
-A KUBE-SEP-LASJGFFJP3UOS6RQ -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp" -m tcp -j DNAT --to-destination 10.244.0.5:53
-A KUBE-SEP-LPGSDLJ3FDW46N4W -s 10.244.0.5/32 -m comment --comment "kube-system/kube-dns:dns" -j KUBE-MARK-MASQ
-A KUBE-SEP-LPGSDLJ3FDW46N4W -p udp -m comment --comment "kube-system/kube-dns:dns" -m udp -j DNAT --to-destination 10.244.0.5:53
-A KUBE-SEP-QCMSUSADBJSJISDV -s 10.244.2.21/32 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-MARK-MASQ
-A KUBE-SEP-QCMSUSADBJSJISDV -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp" -m tcp -j DNAT --to-destination 10.244.2.21:53
-A KUBE-SERVICES -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:metrics cluster IP" -m tcp --dport 9153 -j KUBE-SVC-JD5MR3NA4I4DYORP
-A KUBE-SERVICES -d 10.96.0.10/32 -p udp -m comment --comment "kube-system/kube-dns:dns cluster IP" -m udp --dport 53 -j KUBE-SVC-TCOU7JCQXEZGVUNU
-A KUBE-SERVICES -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-SVC-ERIFXISQEP7F7OF4
-A KUBE-SVC-ERIFXISQEP7F7OF4 ! -s 10.244.0.0/16 -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-MARK-MASQ
-A KUBE-SVC-ERIFXISQEP7F7OF4 -m comment --comment "kube-system/kube-dns:dns-tcp" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-LASJGFFJP3UOS6RQ
-A KUBE-SVC-ERIFXISQEP7F7OF4 -m comment --comment "kube-system/kube-dns:dns-tcp" -j KUBE-SEP-QCMSUSADBJSJISDV
-A KUBE-SVC-JD5MR3NA4I4DYORP ! -s 10.244.0.0/16 -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:metrics cluster IP" -m tcp --dport 9153 -j KUBE-MARK-MASQ
-A KUBE-SVC-JD5MR3NA4I4DYORP -m comment --comment "kube-system/kube-dns:metrics" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-FVQSBIWR5JTECIVC
-A KUBE-SVC-JD5MR3NA4I4DYORP -m comment --comment "kube-system/kube-dns:metrics" -j KUBE-SEP-KGEJJ2QZOQ6ELXUE
-A KUBE-SVC-TCOU7JCQXEZGVUNU ! -s 10.244.0.0/16 -d 10.96.0.10/32 -p udp -m comment --comment "kube-system/kube-dns:dns cluster IP" -m udp --dport 53 -j KUBE-MARK-MASQ
-A KUBE-SVC-TCOU7JCQXEZGVUNU -m comment --comment "kube-system/kube-dns:dns" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-LPGSDLJ3FDW46N4W
-A KUBE-SVC-TCOU7JCQXEZGVUNU -m comment --comment "kube-system/kube-dns:dns" -j KUBE-SEP-2NK74F4BXH6S2KMP
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get ep kube-dns -n kube-system
NAME       ENDPOINTS                                                AGE
kube-dns   10.244.0.5:53,10.244.2.21:53,10.244.0.5:53 + 3 more...   6d15h
lab@k8s1:~$ kubectl describe ep kube-dns -n kube-system
Name:         kube-dns
Namespace:    kube-system
Labels:       k8s-app=kube-dns
              kubernetes.io/cluster-service=true
              kubernetes.io/name=CoreDNS
Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2022-03-29T14:16:25Z
Subsets:
  Addresses:          10.244.0.5,10.244.2.21
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    dns-tcp  53    TCP
    dns      53    UDP
    metrics  9153  TCP

Events:  <none>
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get pods -n kube-system -o wide | grep coredns
coredns-78fcd69978-cvpx2       1/1     Running   0               6d11h   10.244.0.5      k8s1   <none>           <none>
coredns-78fcd69978-hf5sj       1/1     Running   0               6d11h   10.244.2.21     k8s2   <none>           <none>
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get svc -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   6d15h
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl describe svc -n kube-system
Name:              kube-dns
Namespace:         kube-system
Labels:            k8s-app=kube-dns
                   kubernetes.io/cluster-service=true
                   kubernetes.io/name=CoreDNS
Annotations:       prometheus.io/port: 9153
                   prometheus.io/scrape: true
Selector:          k8s-app=kube-dns
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.0.10
IPs:               10.96.0.10
Port:              dns  53/UDP
TargetPort:        53/UDP
Endpoints:         10.244.0.5:53,10.244.2.21:53
Port:              dns-tcp  53/TCP
TargetPort:        53/TCP
Endpoints:         10.244.0.5:53,10.244.2.21:53
Port:              metrics  9153/TCP
TargetPort:        9153/TCP
Endpoints:         10.244.0.5:9153,10.244.2.21:9153
Session Affinity:  None
Events:            <none>
lab@k8s1:~$
```

