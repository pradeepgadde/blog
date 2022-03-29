---
layout: single
title:  "Kubernetes DNS"
date:   2022-03-21 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
show_date: true
header:
  teaser: /assets/images/coredns.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
 
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubernetes DNS

When we setup our minikube cluster, it is configured to use the CoreDNS addon or its precursor, kube-dns.

We can verify the same.

```sh
pradeep@learnk8s$ kubectl get pods -n kube-system
NAME                                       READY   STATUS    RESTARTS        AGE
calico-kube-controllers-8594699699-dztlm   1/1     Running   2 (120m ago)    47h
calico-node-gqvw6                          1/1     Running   2 (119m ago)    47h
calico-node-qdbcf                          1/1     Running   1 (120m ago)    47h
calico-node-sw74l                          1/1     Running   1 (118m ago)    47h
coredns-64897985d-58btq                    1/1     Running   2 (120m ago)    47h
etcd-minikube                              1/1     Running   1 (120m ago)    47h
kube-apiserver-minikube                    1/1     Running   3 (120m ago)    47h
kube-controller-manager-minikube           1/1     Running   3 (120m ago)    47h
kube-proxy-7k4lb                           1/1     Running   1 (118m ago)    47h
kube-proxy-gm2dh                           1/1     Running   1 (119m ago)    47h
kube-proxy-hvkqd                           1/1     Running   1 (120m ago)    47h
kube-scheduler-minikube                    1/1     Running   2 (120m ago)    47h
storage-provisioner                        1/1     Running   10 (107m ago)   47h
```
Describe the coredns pod, to get additional information.
```sh
pradeep@learnk8s$ kubectl describe pods coredns-64897985d-58btq -n kube-system
Name:                 coredns-64897985d-58btq
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 minikube/172.16.30.6
Start Time:           Sat, 19 Mar 2022 23:48:56 +0530
Labels:               k8s-app=kube-dns
                      pod-template-hash=64897985d
Annotations:          <none>
Status:               Running
IP:                   10.88.0.4
IPs:
  IP:           10.88.0.4
Controlled By:  ReplicaSet/coredns-64897985d
Containers:
  coredns:
    Container ID:  docker://c866d7b12f5f086407e04f3fd4e500889f860bfd55790e1e2a4870fad1052330
    Image:         k8s.gcr.io/coredns/coredns:v1.8.6
    Image ID:      docker-pullable://k8s.gcr.io/coredns/coredns@sha256:5b6ec0d6de9baaf3e92d0f66cd96a25b9edbce8716f5f15dcd1a616b3abd590e
    Ports:         53/UDP, 53/TCP, 9153/TCP
    Host Ports:    0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
    State:          Running
      Started:      Mon, 21 Mar 2022 21:07:03 +0530
    Last State:     Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Sun, 20 Mar 2022 19:25:02 +0530
      Finished:     Mon, 21 Mar 2022 21:06:40 +0530
    Ready:          True
    Restart Count:  2
    Limits:
      memory:  170Mi
    Requests:
      cpu:        100m
      memory:     70Mi
    Liveness:     http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
    Readiness:    http-get http://:8181/ready delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /etc/coredns from config-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rhcln (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      coredns
    Optional:  false
  kube-api-access-rhcln:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 CriticalAddonsOnly op=Exists
                             node-role.kubernetes.io/control-plane:NoSchedule
                             node-role.kubernetes.io/master:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                  From     Message
  ----     ------     ----                 ----     -------
  Warning  Unhealthy  107m                 kubelet  Liveness probe failed: Get "http://10.88.0.4:8080/health": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
  Warning  Unhealthy  107m (x2 over 107m)  kubelet  Readiness probe failed: Get "http://10.88.0.4:8181/ready": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
pradeep@learnk8s$
```

## Create a simple Pod to use as a test environment 
```yaml
pradeep@learnk8s$ cat dnsutils.yaml
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  namespace: default
spec:
  containers:
  - name: dnsutils
    image: k8s.gcr.io/e2e-test-images/jessie-dnsutils:1.3
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always

pradeep@learnk8s$
```

```sh
pradeep@learnk8s$ kubectl create -f dnsutils.yaml
pod/dnsutils created
pradeep@learnk8s$
```
Once that Pod is running, you can exec `nslookup` in that environment.
```sh
pradeep@learnk8s$ kubectl exec -i -t dnsutils -- nslookup kubernetes.default
;; connection timed out; no servers could be reached

command terminated with exit code 1
pradeep@learnk8s$
```
As the nslookup command failed, check the following:

```sh
pradeep@learnk8s$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          47h
web          NodePort    10.105.47.136   <none>        8080:31270/TCP   111m
web2         NodePort    10.106.73.151   <none>        8080:31691/TCP   102m
```
`kubernetes` service is running the `default` namespace.
Check the `kube-dns` service in the `kube-system` namespace.
```sh
pradeep@learnk8s$ kubectl get svc -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   47h
pradeep@learnk8s$
```
It is running fine. Let us check what `nameserver` is being used by this Pod.

```sh
pradeep@learnk8s$ kubectl exec -ti dnsutils -- cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

This confirms that the `dnsutils` pod is using the correct nameserver.

Are DNS endpoints exposed?
You can verify that DNS endpoints are exposed by using the `kubectl get endpoints` command.

```sh
pradeep@learnk8s$ kubectl get ep kube-dns -n kube-system
NAME       ENDPOINTS                                  AGE
kube-dns   10.88.0.4:53,10.88.0.4:53,10.88.0.4:9153   47h
pradeep@learnk8s$
```
Hmm, the endpoints are coming from the Podman CNI but not from the Calico CNI that we are using in this cluster. You remember the race condition issue  between Calico and Podman CNIs?!

Let us get the replicaset ID for the coredns service and delete it.
```sh
pradeep@learnk8s$ kubectl get rs -n kube-system
NAME                                 DESIRED   CURRENT   READY   AGE
calico-kube-controllers-8594699699   1         1         1       47h
coredns-64897985d                    1         1         1       47h
```
Deleting the replicaset
```sh
pradeep@learnk8s$ kubectl delete rs coredns-64897985d -n kube-system
replicaset.apps "coredns-64897985d" deleted
pradeep@learnk8s$
```
Immediately, the replicaset got re-created. Check the AGE!

```sh
pradeep@learnk8s$ kubectl get rs -n kube-system
NAME                                 DESIRED   CURRENT   READY   AGE
calico-kube-controllers-8594699699   1         1         1       47h
coredns-64897985d                    1         1         0       2s
pradeep@learnk8s$
```
Let us check the endpoints again.
```sh
pradeep@learnk8s$ kubectl get ep kube-dns -n kube-system
NAME       ENDPOINTS                                                 AGE
kube-dns   10.244.205.196:53,10.244.205.196:53,10.244.205.196:9153   47h
pradeep@learnk8s$
```
Now that the endpoints are correctly pointing to the Calico CNI IP addresses, verify the nslookup again.

```sh
pradeep@learnk8s$ kubectl exec -ti dnsutils -- cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

```sh
pradeep@learnk8s$ kubectl exec -i -t dnsutils -- nslookup kubernetes.default
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	kubernetes.default.svc.cluster.local
Address: 10.96.0.1

pradeep@learnk8s$
```

Just to verify the coredns status.

```sh
pradeep@learnk8s$ kubectl get pods -n kube-system
NAME                                       READY   STATUS    RESTARTS        AGE
calico-kube-controllers-8594699699-dztlm   1/1     Running   2 (132m ago)    47h
calico-node-gqvw6                          1/1     Running   2 (131m ago)    47h
calico-node-qdbcf                          1/1     Running   1 (132m ago)    47h
calico-node-sw74l                          1/1     Running   1 (130m ago)    47h
coredns-64897985d-kwp95                    1/1     Running   0               74s
etcd-minikube                              1/1     Running   1 (132m ago)    47h
kube-apiserver-minikube                    1/1     Running   3 (132m ago)    47h
kube-controller-manager-minikube           1/1     Running   3 (132m ago)    47h
kube-proxy-7k4lb                           1/1     Running   1 (130m ago)    47h
kube-proxy-gm2dh                           1/1     Running   1 (131m ago)    47h
kube-proxy-hvkqd                           1/1     Running   1 (132m ago)    47h
kube-scheduler-minikube                    1/1     Running   2 (132m ago)    47h
storage-provisioner                        1/1     Running   10 (118m ago)   47h
pradeep@learnk8s$
```

Are DNS queries being received/processed?
You can verify if queries are being received by CoreDNS by adding the `log` plugin to the CoreDNS configuration (aka Corefile). The CoreDNS Corefile is held in a ConfigMap named `coredns`. 

```sh
pradeep@learnk8s$ kubectl get cm -n kube-system
NAME                                 DATA   AGE
calico-config                        4      47h
coredns                              1      47h
extension-apiserver-authentication   6      47h
kube-proxy                           2      47h
kube-root-ca.crt                     1      47h
kubeadm-config                       1      47h
kubelet-config-1.23                  1      47h
pradeep@learnk8s$
```
Describe the config map.
```sh
pradeep@learnk8s$ kubectl describe cm coredns -n kube-system
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    hosts {
       192.168.64.1 host.minikube.internal
       fallthrough
    }
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}


BinaryData
====

Events:  <none>
pradeep@learnk8s$
```

Edit this configmap and add the `log` plugin.

```sh
pradeep@learnk8s$ kubectl edit cm coredns -n kube-system
configmap/coredns edited
pradeep@learnk8s$
```

Describe it again and verify the change

```sh
pradeep@learnk8s$ kubectl describe cm coredns -n kube-system
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    log
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    hosts {
       192.168.64.1 host.minikube.internal
       fallthrough
    }
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}


BinaryData
====

Events:  <none>
pradeep@learnk8s$
```
After saving the changes, it may take up to minute or two for Kubernetes to propagate these changes to the CoreDNS pods.

Next, make some queries and view the logs per the sections above in this document. If CoreDNS pods are receiving the queries, you should see them in the logs.


```sh
pradeep@learnk8s$ kubectl -n kube-system logs coredns-64897985d-kwp95
.:53
[INFO] plugin/reload: Running configuration MD5 = 08e2b174e0f0a30a2e82df9c995f4a34
CoreDNS-1.8.6
linux/amd64, go1.17.1, 13a9191
[INFO] Reloading
[INFO] plugin/health: Going into lameduck mode for 5s
pradeep@learnk8s$ kubectl exec -i -t dnsutils -- nslookup kubernetes.default
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	kubernetes.default.svc.cluster.local
Address: 10.96.0.1

pradeep@learnk8s$ kubectl -n kube-system logs coredns-64897985d-kwp95
.:53
[INFO] plugin/reload: Running configuration MD5 = 08e2b174e0f0a30a2e82df9c995f4a34
CoreDNS-1.8.6
linux/amd64, go1.17.1, 13a9191
[INFO] Reloading
[INFO] plugin/health: Going into lameduck mode for 5s
[INFO] plugin/reload: Running configuration MD5 = 07d7c5ad4525bf2c472eaef020d0184d
[INFO] Reloading complete
[INFO] 127.0.0.1:39552 - 1628 "HINFO IN 1824978633801001221.7972855182251791512. udp 57 false 512" NXDOMAIN qr,rd,ra 132 1.058936054s
[INFO] 10.244.151.4:41947 - 42256 "A IN kubernetes.default.default.svc.cluster.local. udp 62 false 512" NXDOMAIN qr,aa,rd 155 0.000469847s
[INFO] 10.244.151.4:55247 - 43436 "A IN kubernetes.default.svc.cluster.local. udp 54 false 512" NOERROR qr,aa,rd 106 0.000357148s
pradeep@learnk8s$
```

Let us make some more queries,

```sh
pradeep@learnk8s$ kubectl exec -i -t dnsutils -- nslookup web.default
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	web.default.svc.cluster.local
Address: 10.105.47.136

pradeep@learnk8s$ kubectl exec -i -t dnsutils -- nslookup web2.default
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	web2.default.svc.cluster.local
Address: 10.106.73.151

pradeep@learnk8s$ kubectl exec -i -t dnsutils -- nslookup web3.default
Server:		10.96.0.10
Address:	10.96.0.10#53

** server can't find web3.default: NXDOMAIN

command terminated with exit code 1
pradeep@learnk8s$
```

All the queries worked as expected. We did not have any service named `web3` in the `default` namespace, so it failed.

Let us review the coredns logs again.
```sh
pradeep@learnk8s$ kubectl -n kube-system logs coredns-64897985d-kwp95
.:53
[INFO] plugin/reload: Running configuration MD5 = 08e2b174e0f0a30a2e82df9c995f4a34
CoreDNS-1.8.6
linux/amd64, go1.17.1, 13a9191
[INFO] Reloading
[INFO] plugin/health: Going into lameduck mode for 5s
[INFO] plugin/reload: Running configuration MD5 = 07d7c5ad4525bf2c472eaef020d0184d
[INFO] Reloading complete
[INFO] 127.0.0.1:39552 - 1628 "HINFO IN 1824978633801001221.7972855182251791512. udp 57 false 512" NXDOMAIN qr,rd,ra 132 1.058936054s
[INFO] 10.244.151.4:41947 - 42256 "A IN kubernetes.default.default.svc.cluster.local. udp 62 false 512" NXDOMAIN qr,aa,rd 155 0.000469847s
[INFO] 10.244.151.4:55247 - 43436 "A IN kubernetes.default.svc.cluster.local. udp 54 false 512" NOERROR qr,aa,rd 106 0.000357148s
[INFO] 10.244.151.4:46146 - 25028 "A IN web.default.default.svc.cluster.local. udp 55 false 512" NXDOMAIN qr,aa,rd 148 0.000302078s
[INFO] 10.244.151.4:43513 - 15523 "A IN web.default.svc.cluster.local. udp 47 false 512" NOERROR qr,aa,rd 92 0.000412597s
[INFO] 10.244.151.4:41650 - 22470 "A IN web2.default.default.svc.cluster.local. udp 56 false 512" NXDOMAIN qr,aa,rd 149 0.000398918s
[INFO] 10.244.151.4:33214 - 54758 "A IN web2.default.svc.cluster.local. udp 48 false 512" NOERROR qr,aa,rd 94 0.00031088s
[INFO] 10.244.151.4:34263 - 58079 "A IN web3.default.default.svc.cluster.local. udp 56 false 512" NXDOMAIN qr,aa,rd 149 0.000433005s
[INFO] 10.244.151.4:52587 - 56008 "A IN web3.default.svc.cluster.local. udp 48 false 512" NXDOMAIN qr,aa,rd 141 0.000420513s
[INFO] 10.244.151.4:49172 - 35844 "A IN web3.default.cluster.local. udp 44 false 512" NXDOMAIN qr,aa,rd 137 0.000302094s
[INFO] 10.244.151.4:44833 - 3203 "A IN web3.default. udp 30 false 512" NXDOMAIN qr,rd,ra 105 0.18944251s
pradeep@learnk8s$
```

Are you in the right namespace for the service?
DNS queries that don't specify a namespace are limited to the pod's namespace.

If the namespace of the pod and service differ, the DNS query must include the namespace of the service.

This query is limited to the pod's namespace:

```sh
pradeep@learnk8s$ kubectl exec -i -t dnsutils -- nslookup ingress-nginx-controller
Server:		10.96.0.10
Address:	10.96.0.10#53

** server can't find ingress-nginx-controller: NXDOMAIN

command terminated with exit code 1
pradeep@learnk8s$
```
This failed because, `ingress-nginx-controller` svc is not present in the `default` namespace.

```sh
pradeep@learnk8s$ kubectl -n kube-system logs coredns-64897985d-kwp95
.:53
[INFO] plugin/reload: Running configuration MD5 = 08e2b174e0f0a30a2e82df9c995f4a34
CoreDNS-1.8.6
linux/amd64, go1.17.1, 13a9191
[INFO] Reloading
[INFO] plugin/health: Going into lameduck mode for 5s
[INFO] plugin/reload: Running configuration MD5 = 07d7c5ad4525bf2c472eaef020d0184d
[INFO] Reloading complete
[INFO] 127.0.0.1:39552 - 1628 "HINFO IN 1824978633801001221.7972855182251791512. udp 57 false 512" NXDOMAIN qr,rd,ra 132 1.058936054s
[INFO] 10.244.151.4:41947 - 42256 "A IN kubernetes.default.default.svc.cluster.local. udp 62 false 512" NXDOMAIN qr,aa,rd 155 0.000469847s
[INFO] 10.244.151.4:55247 - 43436 "A IN kubernetes.default.svc.cluster.local. udp 54 false 512" NOERROR qr,aa,rd 106 0.000357148s
[INFO] 10.244.151.4:46146 - 25028 "A IN web.default.default.svc.cluster.local. udp 55 false 512" NXDOMAIN qr,aa,rd 148 0.000302078s
[INFO] 10.244.151.4:43513 - 15523 "A IN web.default.svc.cluster.local. udp 47 false 512" NOERROR qr,aa,rd 92 0.000412597s
[INFO] 10.244.151.4:41650 - 22470 "A IN web2.default.default.svc.cluster.local. udp 56 false 512" NXDOMAIN qr,aa,rd 149 0.000398918s
[INFO] 10.244.151.4:33214 - 54758 "A IN web2.default.svc.cluster.local. udp 48 false 512" NOERROR qr,aa,rd 94 0.00031088s
[INFO] 10.244.151.4:34263 - 58079 "A IN web3.default.default.svc.cluster.local. udp 56 false 512" NXDOMAIN qr,aa,rd 149 0.000433005s
[INFO] 10.244.151.4:52587 - 56008 "A IN web3.default.svc.cluster.local. udp 48 false 512" NXDOMAIN qr,aa,rd 141 0.000420513s
[INFO] 10.244.151.4:49172 - 35844 "A IN web3.default.cluster.local. udp 44 false 512" NXDOMAIN qr,aa,rd 137 0.000302094s
[INFO] 10.244.151.4:44833 - 3203 "A IN web3.default. udp 30 false 512" NXDOMAIN qr,rd,ra 105 0.18944251s
[INFO] 10.244.151.4:50251 - 50181 "A IN ingress-nginx-controller.default.svc.cluster.local. udp 68 false 512" NXDOMAIN qr,aa,rd 161 0.000381672s
[INFO] 10.244.151.4:52033 - 12007 "A IN ingress-nginx-controller.svc.cluster.local. udp 60 false 512" NXDOMAIN qr,aa,rd 153 0.000844354s
[INFO] 10.244.151.4:50003 - 30556 "A IN ingress-nginx-controller.cluster.local. udp 56 false 512" NXDOMAIN qr,aa,rd 149 0.000453492s
[INFO] 10.244.151.4:41810 - 19431 "A IN ingress-nginx-controller. udp 42 false 512" NXDOMAIN qr,rd,ra 117 0.057777871s
pradeep@learnk8s$
```

We can see that the query is done with the `default` namespace: `ingress-nginx-controller.default.svc.cluster.local.`

Let us list all active services in all namespaces.
```sh
pradeep@learnk8s$ kubectl get svc -A
NAMESPACE       NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
default         kubernetes                           ClusterIP   10.96.0.1        <none>        443/TCP                      47h
default         web                                  NodePort    10.105.47.136    <none>        8080:31270/TCP               137m
default         web2                                 NodePort    10.106.73.151    <none>        8080:31691/TCP               127m
ingress-nginx   ingress-nginx-controller             NodePort    10.101.85.4      <none>        80:30419/TCP,443:30336/TCP   147m
ingress-nginx   ingress-nginx-controller-admission   ClusterIP   10.104.163.136   <none>        443/TCP                      147m
kube-system     kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       47h
pradeep@learnk8s$
```

Now that we see `ingress-nginx-controller` in the `ingress-nginx` namespace, let us query for the correct name.

```sh
pradeep@learnk8s$ kubectl exec -i -t dnsutils -- nslookup ingress-nginx-controller.ingress-nginx
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	ingress-nginx-controller.ingress-nginx.svc.cluster.local
Address: 10.101.85.4
```

We can see the name resolution is successful this time.

