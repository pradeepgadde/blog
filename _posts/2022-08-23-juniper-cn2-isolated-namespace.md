---
layout: single
title:  "Juniper CN2 Isolated Namespace"
date:   2022-08-23 01:55:04 +0530
categories: Kubernetes
tags: CN2 
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/junos.png
  og_image: /assets/images/junos.png
  teaser: /assets/images/junos.png
  caption: "Photo credit: [**Juniper Networks**](https://www.juniper.net/documentation/product/us/en/cloud-native-contrail-networking)"
  actions:
    - label: "Learn more"
      url: "https://www.juniper.net/documentation/product/us/en/cloud-native-contrail-networking"

author:
  name     : "Cloud-Native Contrail Networking (CN2)"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Blog"

    text: "Checkout other topics"
    nav: my-sidebar
---

# CN2 Isolated Namespace

## Reference

[https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/topics/topic-map/cn-cloud-native-isloated-namespace.html](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/topics/topic-map/cn-cloud-native-isloated-namespace.html)

## Namespaces

By default, namespaces are not isolated.

You can create an isolated namespace to isolate a pod from other pods, without explicitly configuring a network policy.

To create an isolated namespace, we need to add a label, `core.juniper.net/isolated-namespace: "true" ` to the namespace.

```yaml
pradeep@CN2% cat ns-isolated.yaml 
apiVersion: v1 
kind: Namespace 
metadata: 
   name: isolated-namespace-demo
   labels: 
     core.juniper.net/isolated-namespace: "true" 
      
pradeep@CN2% 
```

```sh
pradeep@CN2% kubectl apply -f ns-isolated.yaml 
namespace/isolated-namespace-demo created
pradeep@CN2% 
```

```sh
pradeep@CN2% kubectl get ns                   
NAME                                   STATUS   AGE
contrail                               Active   4d7h
contrail-analytics                     Active   4d7h
contrail-deploy                        Active   4d7h
contrail-k8s-kubemanager-mk-contrail   Active   4d7h
contrail-system                        Active   4d7h
default                                Active   4d7h
isolated-namespace-demo                Active   2s
kube-node-lease                        Active   4d7h
kube-public                            Active   4d7h
kube-system                            Active   4d7h
pradeep@CN2% 
```

```sh
pradeep@CN2% kubectl describe ns isolated-namespace-demo 
Name:         isolated-namespace-demo
Labels:       core.juniper.net/clusterName=contrail-k8s-kubemanager-mk
              core.juniper.net/isolated-namespace=true
              kubernetes.io/metadata.name=isolated-namespace-demo
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
pradeep@CN2%
```
Lets create another namespace, this time without the label, making it the standard namespace (non-isolated).

```sh
pradeep@CN2% cat non-isolated-ns.yaml 
apiVersion: v1 
kind: Namespace 
metadata: 
   name: non-isolated-namespace-demo

pradeep@CN2% 
```

```sh
pradeep@CN2% kubectl apply -f non-isolated-ns.yaml        
namespace/non-isolated-namespace-demo created
pradeep@CN2%
```

```sh
pradeep@CN2% kubectl get ns                               
NAME                                   STATUS   AGE
contrail                               Active   4d7h
contrail-analytics                     Active   4d7h
contrail-deploy                        Active   4d7h
contrail-k8s-kubemanager-mk-contrail   Active   4d7h
contrail-system                        Active   4d7h
default                                Active   4d7h
isolated-namespace-demo                Active   2m48s
kube-node-lease                        Active   4d7h
kube-public                            Active   4d7h
kube-system                            Active   4d7h
non-isolated-namespace-demo            Active   7s
pradeep@CN2%
```

```sh
pradeep@CN2% cat non-isolated-ns.yaml 
apiVersion: v1 
kind: Namespace 
metadata: 
   name: non-isolated-namespace-demo

pradeep@CN2% kubectl describe ns non-isolated-namespace-demo  
Name:         non-isolated-namespace-demo
Labels:       core.juniper.net/clusterName=contrail-k8s-kubemanager-mk
              kubernetes.io/metadata.name=non-isolated-namespace-demo
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
pradeep@CN2% 

```

## Pods
Let us create four pods. Two pods in the isolated-namespace and one pod in the non-isloated namespace and the last one in the default namespace.

### Isolated Namespace
```yaml
pradeep@CN2% cat p1.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: isolated-namespace-demo
  
spec:
  containers:
  - name: pod1
    image: gcr.io/google-containers/toolbox
    command: ["bash","-c","while true; do sleep 60s; done"]
    securityContext:
      privileged: true
pradeep@CN2%
```

```yaml
pradeep@CN2% cat p2.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: isolated-namespace-demo
  
    
spec:
  containers:
  - name: pod2
    image: gcr.io/google-containers/toolbox
    command: ["bash","-c","while true; do sleep 60s; done"]
    securityContext:
      privileged: true
pradeep@CN2% 
```
### Non-Isolated Namespace
```yaml
pradeep@CN2% cat p3.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod3
  namespace: non-isolated-namespace-demo
  
    
spec:
  containers:
  - name: pod3
    image: gcr.io/google-containers/toolbox
    command: ["bash","-c","while true; do sleep 60s; done"]
    securityContext:
      privileged: true
pradeep@CN2% 
```
### Default Namespace 

```yaml
pradeep@CN2% cat p4.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod4
  namespace: default
  annotations:
    k8s.v1.cni.cncf.io/networks: vn1
    
spec:
  containers:
  - name: pod4
    image: gcr.io/google-containers/toolbox
    command: ["bash","-c","while true; do sleep 60s; done"]
    securityContext:
      privileged: true
pradeep@CN2% 

```

```sh
pradeep@CN2% kubectl apply -f p1.yaml 
pod/pod1 created
pradeep@CN2% kubectl apply -f p2.yaml 
pod/pod2 created
pradeep@CN2% kubectl apply -f p3.yaml 
pod/pod3 created
pradeep@CN2% kubectl apply -f p4.yaml 
pod/pod4 created
pradeep@CN2%
```

```sh
pradeep@CN2% kubectl get pods -n non-isolated-namespace-demo -o wide
NAME   READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
pod3   1/1     Running   0          112s   10.244.0.7   minikube   <none>           <none>
pradeep@CN2%
```

```sh
pradeep@CN2% kubectl get pods -n isolated-namespace-demo -o wide 
NAME   READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
pod1   1/1     Running   0          2m6s   10.244.0.5   minikube   <none>           <none>
pod2   1/1     Running   0          2m3s   10.244.0.6   minikube   <none>           <none>
pradeep@CN2%
```

```sh
pradeep@CN2% kubectl get pods  -o wide
NAME      READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
pod4      1/1     Running   0          2m6s   10.244.0.8   minikube   <none>           <none>
vn1-pod   1/1     Running   0          13h    10.244.0.3   minikube   <none>           <none>
vn2-pod   1/1     Running   0          13h    10.244.0.4   minikube   <none>           <none>
pradeep@CN2% 
```

## Verify Connectivity between the Pods

```sh
pradeep@CN2% kubectl exec -it pod1 -n isolated-namespace-demo -- ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
46: eth0@if47: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:ad:18:f2:00:c6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.5/16 brd 10.244.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::281c:20ff:fe2d:1053/64 scope link 
       valid_lft forever preferred_lft forever
pradeep@CN2%
```

```sh
pradeep@CN2% kubectl exec -it pod2 -n isolated-namespace-demo -- ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
48: eth0@if49: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:20:66:86:19:4b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.6/16 brd 10.244.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::489e:95ff:fe0e:77d7/64 scope link 
       valid_lft forever preferred_lft forever
pradeep@CN2%
```



### Pod1 in Isolated Namespace

Pods in isolated namespace are able to communicate with only other pods in the same isolated namespace.

Communication to other namespaces including the default namespace is not working.

```sh
pradeep@CN2% kubectl exec -it pod1 -n isolated-namespace-demo -- ping 10.244.0.6
PING 10.244.0.6 (10.244.0.6) 56(84) bytes of data.
64 bytes from 10.244.0.6: icmp_seq=1 ttl=63 time=1.64 ms
64 bytes from 10.244.0.6: icmp_seq=2 ttl=63 time=0.099 ms
64 bytes from 10.244.0.6: icmp_seq=3 ttl=63 time=0.144 ms
^C
--- 10.244.0.6 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2009ms
rtt min/avg/max/mdev = 0.099/0.630/1.648/0.720 ms
pradeep@CN2% kubectl exec -it pod1 -n isolated-namespace-demo -- ping 10.244.0.7
PING 10.244.0.7 (10.244.0.7) 56(84) bytes of data.
^C
--- 10.244.0.7 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3057ms

command terminated with exit code 1
pradeep@CN2% kubectl exec -it pod1 -n isolated-namespace-demo -- ping 10.244.0.8
PING 10.244.0.8 (10.244.0.8) 56(84) bytes of data.
^C
--- 10.244.0.8 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3106ms

command terminated with exit code 1
pradeep@CN2% 

```

### Pod2 in Isolated Namespace

```sh
pradeep@CN2% kubectl exec -it pod2 -n isolated-namespace-demo -- ping 10.244.0.5
PING 10.244.0.5 (10.244.0.5) 56(84) bytes of data.
64 bytes from 10.244.0.5: icmp_seq=1 ttl=63 time=5.43 ms
64 bytes from 10.244.0.5: icmp_seq=2 ttl=63 time=0.120 ms
^C
--- 10.244.0.5 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.120/2.779/5.438/2.659 ms
pradeep@CN2% 
```

```sh
pradeep@CN2% kubectl exec -it pod2 -n isolated-namespace-demo -- ping 10.244.0.7
PING 10.244.0.7 (10.244.0.7) 56(84) bytes of data.
^C
--- 10.244.0.7 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3110ms

command terminated with exit code 1
pradeep@CN2%
```

```sh
pradeep@CN2% kubectl exec -it pod2 -n isolated-namespace-demo -- ping 10.244.0.8
PING 10.244.0.8 (10.244.0.8) 56(84) bytes of data.
^C
--- 10.244.0.8 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2029ms

command terminated with exit code 1
pradeep@CN2% 
```



### Pod3 in Non Isolated Namespace
Pods in non isolated namespace are able to communicate with pods in the non-isolated namespace as well as the default namespace.

```sh
pradeep@CN2% kubectl exec -it pod3 -n non-isolated-namespace-demo -- ip a           
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
50: eth0@if51: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:f3:17:4e:0d:72 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.7/16 brd 10.244.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::3048:8eff:fe81:e84f/64 scope link 
       valid_lft forever preferred_lft forever
pradeep@CN2% 
```

```sh
pradeep@CN2% kubectl exec -it pod3 -n non-isolated-namespace-demo -- ping 10.244.0.5
PING 10.244.0.5 (10.244.0.5) 56(84) bytes of data.
64 bytes from 10.244.0.5: icmp_seq=1 ttl=63 time=2.53 ms
64 bytes from 10.244.0.5: icmp_seq=2 ttl=63 time=0.185 ms
64 bytes from 10.244.0.5: icmp_seq=3 ttl=63 time=0.105 ms
^C
--- 10.244.0.5 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2012ms
rtt min/avg/max/mdev = 0.105/0.942/2.537/1.128 ms

pradeep@CN2% kubectl exec -it pod3 -n non-isolated-namespace-demo -- ping 10.244.0.6
PING 10.244.0.6 (10.244.0.6) 56(84) bytes of data.
64 bytes from 10.244.0.6: icmp_seq=1 ttl=63 time=10.7 ms
64 bytes from 10.244.0.6: icmp_seq=2 ttl=63 time=0.094 ms
64 bytes from 10.244.0.6: icmp_seq=3 ttl=63 time=4.38 ms
64 bytes from 10.244.0.6: icmp_seq=4 ttl=63 time=0.325 ms
^C
--- 10.244.0.6 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3038ms
rtt min/avg/max/mdev = 0.094/3.877/10.708/4.297 ms

pradeep@CN2% kubectl exec -it pod3 -n non-isolated-namespace-demo -- ping 10.244.0.8
PING 10.244.0.8 (10.244.0.8) 56(84) bytes of data.
64 bytes from 10.244.0.8: icmp_seq=1 ttl=63 time=3.33 ms
64 bytes from 10.244.0.8: icmp_seq=2 ttl=63 time=0.175 ms
64 bytes from 10.244.0.8: icmp_seq=3 ttl=63 time=0.190 ms
64 bytes from 10.244.0.8: icmp_seq=4 ttl=63 time=0.112 ms
^C
--- 10.244.0.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3042ms
rtt min/avg/max/mdev = 0.112/0.952/3.331/1.373 ms
pradeep@CN2% 

```

### Pod4 in Default Namespace
Pods in the default  namespace are able to communicate with both pods in the  isolated namespace and non-isolated namespace.

```sh
pradeep@CN2% kubectl exec -it pod4  -- ip a                   
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
52: eth0@if53: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:12:59:0f:cb:c3 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.8/16 brd 10.244.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::b80a:fff:fe14:21a4/64 scope link 
       valid_lft forever preferred_lft forever
54: eth1@if55: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:1a:fc:e5:63:6e brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.10.3/24 brd 172.16.10.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::6cb4:4aff:fe07:bf99/64 scope link 
       valid_lft forever preferred_lft forever
pradeep@CN2% 

```

```sh
pradeep@CN2% kubectl exec -it pod4 -- ping 10.244.0.5                               
PING 10.244.0.5 (10.244.0.5) 56(84) bytes of data.
64 bytes from 10.244.0.5: icmp_seq=1 ttl=63 time=3.04 ms
64 bytes from 10.244.0.5: icmp_seq=2 ttl=63 time=0.097 ms
64 bytes from 10.244.0.5: icmp_seq=3 ttl=63 time=0.113 ms
^C
--- 10.244.0.5 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2010ms
rtt min/avg/max/mdev = 0.097/1.084/3.043/1.385 ms

pradeep@CN2% kubectl exec -it pod4 -- ping 10.244.0.6
PING 10.244.0.6 (10.244.0.6) 56(84) bytes of data.
64 bytes from 10.244.0.6: icmp_seq=1 ttl=63 time=1.47 ms
64 bytes from 10.244.0.6: icmp_seq=2 ttl=63 time=0.156 ms
64 bytes from 10.244.0.6: icmp_seq=3 ttl=63 time=0.281 ms
^C
--- 10.244.0.6 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.156/0.638/1.479/0.597 ms

pradeep@CN2% kubectl exec -it pod4 -- ping 10.244.0.7
PING 10.244.0.7 (10.244.0.7) 56(84) bytes of data.
64 bytes from 10.244.0.7: icmp_seq=1 ttl=63 time=1.96 ms
64 bytes from 10.244.0.7: icmp_seq=2 ttl=63 time=0.196 ms
64 bytes from 10.244.0.7: icmp_seq=3 ttl=63 time=0.127 ms
64 bytes from 10.244.0.7: icmp_seq=4 ttl=63 time=0.623 ms
^C
--- 10.244.0.7 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3078ms
rtt min/avg/max/mdev = 0.127/0.727/1.965/0.739 ms
pradeep@CN2% 
```

I was expecting that the isolated namespace resources are completely isolated in both directions. But it looks like if the communication is initiated from outside, towards the isolated namespace resources, it still works.  

So, it looks like the restrictions are in place only for the Egress but not the Ingress. I am not sure, if this is by design or not.

