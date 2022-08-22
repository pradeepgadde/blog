---
layout: single
title:  "Juniper CN2 Virtual Network Router "
date:   2022-08-22 11:55:04 +0530
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

# CN2-Virtual Network Router

## Reference
[https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/topics/concept/Contrail_Network_Policy_Implementation_in_CN2.html#xd_a29cf6a0c60ca860-3566e1ce-17b7d3148e0--7e89__section_rrx_pl4_ftb](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/topics/concept/Contrail_Network_Policy_Implementation_in_CN2.html#xd_a29cf6a0c60ca860-3566e1ce-17b7d3148e0--7e89__section_rrx_pl4_ftb) 

## VirtualNetworkRouter Connecting Two Virtual Networks
In this post, let us start from scratch, with no Pods, no Virtual Networks.

```sh
pradeep@CN2 % kubectl get pods         
No resources found in default namespace.
pradeep@CN2 % kubectl get vn  
No resources found in default namespace.
pradeep@CN2 % kubectl get ri
No resources found in default namespace.
pradeep@CN2 % kubectl get vnr
No resources found in default namespace.
pradeep@CN2 % kubectl get rt 
NAME                   STATE     AGE
target-64512-8000000   Success   3d13h
target-64512-8000001   Success   3d13h
target-64512-8000002   Success   3d13h
target-64512-8000003   Success   3d13h
target-64512-8000004   Success   3d13h
target-64512-8000005   Success   3d13h
target-64512-8000006   Success   3d13h
target-64512-8000007   Success   3d13h
target-64512-8000008   Success   3d13h
pradeep@CN2 %
pradeep@CN2 % kubectl get net-attach-def
No resources found in default namespace.
pradeep@CN2 % 
```



Let us create two network attachments and thus two virtual networks, two subnets.

In this case, we are creating two virtual networks, with defaults. We have not explicitly assigned any route targets.

## NetworkAttachmentDefinition

```yaml
pradeep@CN2 % cat vn1.yaml 
  apiVersion: "k8s.cni.cncf.io/v1"
  kind: NetworkAttachmentDefinition
  metadata:
    name: vn1
    namespace: default
    annotations:
      juniper.net/networks: '{
        "ipamV4Subnet": "172.16.10.0/24"
      }'
      
  spec:
    config: '{
    "cniVersion": "0.3.1",
    "name": "vn1",
    "type": "contrail-k8s-cni"
  }'
pradeep@CN2 %
```



```yaml
pradeep@CN2 % cat vn2.yaml 
  apiVersion: "k8s.cni.cncf.io/v1"
  kind: NetworkAttachmentDefinition
  metadata:
    name: vn2
    namespace: default
    annotations:
      juniper.net/networks: '{
        "ipamV4Subnet": "172.16.20.0/24"
      }'
      
  spec:
    config: '{
    "cniVersion": "0.3.1",
    "name": "vn2",
    "type": "contrail-k8s-cni"
  }'

pradeep@CN2 % 
```

```sh
pradeep@CN2 % kubectl apply -f vn1.yaml 
networkattachmentdefinition.k8s.cni.cncf.io/vn1 created
pradeep@CN2 % kubectl apply -f vn2.yaml 
networkattachmentdefinition.k8s.cni.cncf.io/vn2 created
pradeep@CN2 % kubectl get net-attach-def,vn,subnet
NAME                                              AGE
networkattachmentdefinition.k8s.cni.cncf.io/vn1   12s
networkattachmentdefinition.k8s.cni.cncf.io/vn2   8s

NAME                                           VNI   IP FAMILIES   STATE     AGE
virtualnetwork.core.contrail.juniper.net/vn1   5     v4            Success   12s
virtualnetwork.core.contrail.juniper.net/vn2   6     v4            Success   8s

NAME                                      CIDR             USAGE   STATE     AGE
subnet.core.contrail.juniper.net/vn1-v4   172.16.10.0/24   1.17%   Success   12s
subnet.core.contrail.juniper.net/vn2-v4   172.16.20.0/24   1.17%   Success   8s
pradeep@CN2 % 
```

```sh
pradeep@CN2 % kubectl get ri,rt
NAME                                            ROUTETARGET     STATE     AGE
routinginstance.core.contrail.juniper.net/vn1   64512:8000009   Success   35s
routinginstance.core.contrail.juniper.net/vn2   64512:8000010   Success   31s

NAME                                                         STATE     AGE
routetarget.core.contrail.juniper.net/target-64512-8000000   Success   3d13h
routetarget.core.contrail.juniper.net/target-64512-8000001   Success   3d13h
routetarget.core.contrail.juniper.net/target-64512-8000002   Success   3d13h
routetarget.core.contrail.juniper.net/target-64512-8000003   Success   3d13h
routetarget.core.contrail.juniper.net/target-64512-8000004   Success   3d13h
routetarget.core.contrail.juniper.net/target-64512-8000005   Success   3d13h
routetarget.core.contrail.juniper.net/target-64512-8000006   Success   3d13h
routetarget.core.contrail.juniper.net/target-64512-8000007   Success   3d13h
routetarget.core.contrail.juniper.net/target-64512-8000008   Success   3d13h
routetarget.core.contrail.juniper.net/target-64512-8000009   Success   35s
routetarget.core.contrail.juniper.net/target-64512-8000010   Success   31s
pradeep@CN2 % 
```
## Pod
Let us create two pods and associated each of them with a different network.
`vn1-pod` is assigned to `vn1` and `vn2-pod` is assigned to `vn2` virtual network.
One difference in the Pod definition file is the `securityContext`. Without this, we will not be able to do network connectivity checks like ping, or addition of static routes etc.

```sh
pradeep@CN2 % cat vn1-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: vn1-pod
  namespace: default
  annotations:
    k8s.v1.cni.cncf.io/networks: vn1
    
spec:
  containers:
  - name: vn1-pod
    image: gcr.io/google-containers/toolbox
    command: ["bash","-c","while true; do sleep 60s; done"]
    securityContext:
      privileged: true
pradeep@CN2 %
```

```sh
pradeep@CN2 % cat vn2-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: vn2-pod
  namespace: default
  annotations:
    k8s.v1.cni.cncf.io/networks: vn2
    
spec:
  containers:
  - name: vn2-pod
    image: gcr.io/google-containers/toolbox
    command: ["bash","-c","while true; do sleep 60s; done"]
    securityContext:
      privileged: true
pradeep@CN2 % 
```

```sh
pradeep@CN2 % kubectl apply -f vn1-pod.yaml 
pod/vn1-pod created
pradeep@CN2 % kubectl apply -f vn2-pod.yaml 
pod/vn2-pod created
pradeep@CN2 % 
```
## vn1-pod
Verify that the Pods are running and obtained the IP address from the defaultPodNetwork on first interface and from the virtual network on the second interface.

```sh
pradeep@CN2 % kubectl get pods -o wide     
NAME      READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
vn1-pod   1/1     Running   0          104s   10.244.0.3   minikube   <none>           <none>
vn2-pod   1/1     Running   0          100s   10.244.0.4   minikube   <none>           <none>
pradeep@CN2 % 
```
From the below output, we can see that vn1-pod is assigned the `172.16.10.2` address from the `vn1` virtual network.

```sh
pradeep@CN2 % kubectl exec -it vn1-pod -- ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
30: eth0@if31: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:18:03:b3:70:f3 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.3/16 brd 10.244.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::60f3:5ff:fe37:89f9/64 scope link 
       valid_lft forever preferred_lft forever
32: eth1@if33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:86:d2:f8:b8:a8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.10.2/24 brd 172.16.10.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::9061:aff:fe4a:ac75/64 scope link 
       valid_lft forever preferred_lft forever
pradeep@CN2 % 
```

Verify the current routing table of `vn1-pod`. The default route points to the first interface `eth0.

```sh
pradeep@CN2 % kubectl exec -it vn1-pod -- ip route
default via 10.244.0.1 dev eth0 
10.244.0.0/16 dev eth0 proto kernel scope link src 10.244.0.3 
172.16.10.0/24 dev eth1 proto kernel scope link src 172.16.10.2 
pradeep@CN2 % 
```

Add a static route to the `vn2` subnet and verify the route table again.

```sh
pradeep@CN2 % kubectl exec -it vn1-pod -- ip route add 172.16.20.0/24 via 172.16.10.1
pradeep@CN2 % kubectl exec -it vn1-pod -- ip route                                   
default via 10.244.0.1 dev eth0 
10.244.0.0/16 dev eth0 proto kernel scope link src 10.244.0.3 
172.16.10.0/24 dev eth1 proto kernel scope link src 172.16.10.2 
172.16.20.0/24 via 172.16.10.1 dev eth1 
pradeep@CN2 % 
```

## vn2-pod

```sh
pradeep@CN2 % kubectl exec -it vn2-pod -- ip a    
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
34: eth0@if35: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:06:6f:e3:7f:59 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.4/16 brd 10.244.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::8e5:1aff:fecc:9fbb/64 scope link 
       valid_lft forever preferred_lft forever
36: eth1@if37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:3c:a5:0d:53:cb brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.20.2/24 brd 172.16.20.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::9836:19ff:fecd:8002/64 scope link 
       valid_lft forever preferred_lft forever
pradeep@CN2 %
```

```sh
pradeep@CN2 % kubectl exec -it vn2-pod -- ip route
default via 10.244.0.1 dev eth0 
10.244.0.0/16 dev eth0 proto kernel scope link src 10.244.0.4 
172.16.20.0/24 dev eth1 proto kernel scope link src 172.16.20.2 
pradeep@CN2 %
```
```sh
pradeep@CN2 % kubectl exec -it vn2-pod -- ip route add 172.16.10.0/24 via 172.16.20.1
pradeep@CN2 %
```
```sh
pradeep@CN2 % kubectl exec -it vn2-pod -- ip route                                   
default via 10.244.0.1 dev eth0 
10.244.0.0/16 dev eth0 proto kernel scope link src 10.244.0.4 
172.16.10.0/24 via 172.16.20.1 dev eth1 
172.16.20.0/24 dev eth1 proto kernel scope link src 172.16.20.2 
pradeep@CN2 % 
```
Verify reachability to the gateway or next-hop IP address 
```sh
pradeep@CN2 % kubectl exec -it vn2-pod -- ping 172.16.20.1
PING 172.16.20.1 (172.16.20.1) 56(84) bytes of data.
64 bytes from 172.16.20.1: icmp_seq=1 ttl=64 time=1.37 ms
64 bytes from 172.16.20.1: icmp_seq=2 ttl=64 time=0.483 ms
64 bytes from 172.16.20.1: icmp_seq=3 ttl=64 time=4.74 ms
^C
--- 172.16.20.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.483/2.201/4.744/1.834 ms
pradeep@CN2 % 
```
Verify reachability to the vn1-pod IP address

```sh
pradeep@CN2 % kubectl exec -it vn2-pod -- ping 172.16.10.2
PING 172.16.10.2 (172.16.10.2) 56(84) bytes of data.
^C
--- 172.16.10.2 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3055ms

command terminated with exit code 1
pradeep@CN2 % 
```
Verify the other way, ping vn2-pod IP from the vn1-pod

```sh
pradeep@CN2 % kubectl exec -it vn1-pod -- ping 172.16.20.2
PING 172.16.20.2 (172.16.20.2) 56(84) bytes of data.
^C
--- 172.16.20.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2077ms

command terminated with exit code 1
pradeep@CN2 % 
```
Ping the `vn1-pod` gateway/next-hop
```sh
pradeep@CN2 % kubectl exec -it vn1-pod -- ping 172.16.10.1
PING 172.16.10.1 (172.16.10.1) 56(84) bytes of data.
64 bytes from 172.16.10.1: icmp_seq=1 ttl=64 time=3.91 ms
64 bytes from 172.16.10.1: icmp_seq=2 ttl=64 time=0.518 ms
64 bytes from 172.16.10.1: icmp_seq=3 ttl=64 time=0.617 ms
^C
--- 172.16.10.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2053ms
rtt min/avg/max/mdev = 0.518/1.684/3.919/1.581 ms
pradeep@CN2 % 

```



Though route is present in each pod and the next-hop is reachable, the two Pods are not able to communicate with each other.

>Pods in VN1 cannot connect to pods in VN2. This is the default behavior of VirtualNetworks in Cloud-Native Contrail Networking.

## VirtualNetworkRouter

Typically, `VirtualNetwork` traffic is isolated to maintain tenant  separation. In Cloud-Native Contrail Networking, `VirtualNetworkRouter` (VNR) performs route leaking. Route leaking establishes connectivity between `VirtualNetworks` by importing routing instances (RI) and the routing tables associated with these instances. As a result, devices on one routing table can access  resources from devices on another routing table.



```yaml
pradeep@CN2 % cat vnr-demo.yaml 
apiVersion: core.contrail.juniper.net/v1alpha1
kind: VirtualNetworkRouter
metadata:
  
  name: vnr-1
  annotations:
    core.juniper.net/display-name: vnr-1
  labels:
      vnr: Cust1-vnr
spec:
  type: mesh
  virtualNetworkSelector:
    matchExpressions:
      - key: customer
        operator: In
        values:
           - Cust1
pradeep@CN2 % 
```

 A `type: mesh` VNR   with the name `vnr-1` establishes connectivity between the two  `VirtualNetworks` using `matchExpressions` `customer: Cust1`. The VNR imports the RI and routing table of `vn1`  to `vn2` and vice versa. Since `vnr-1` is a mesh-type VNR, all pods in connected `VirtualNetworks` communicate with each other.

In this example, there are two `VirtualNetworks` (`vn1`,   `vn2`) in namespace `default`. 

Let us see if both virtual networks  have the `label` `customer: Cust1`. 

```sh
pradeep@CN2 % kubectl get vn --show-labels 
NAME   VNI   IP FAMILIES   STATE     AGE   LABELS
vn1    5     v4            Success   22m   back-reference.core.juniper.net/c6dbefed51ec7143c19e3d9285d04ea8f7343de4632174839c9549f1=Subnet_default_vn1-v4
vn2    6     v4            Success   22m   back-reference.core.juniper.net/54f620398eebd1cfc552b90b3f8ab758524eed64d368f07cf2a545f1=Subnet_default_vn2-v4
pradeep@CN2 % 

```

As the `VirtualNetworkSelector` is looking for a key value of `customer: Cust1`, let us label our VirtualNetworks with this key.

```sh
pradeep@CN2 % kubectl label vn vn1 customer=Cust1
virtualnetwork.core.contrail.juniper.net/vn1 labeled
pradeep@CN2 % kubectl label vn vn2 customer=Cust1
virtualnetwork.core.contrail.juniper.net/vn2 labeled
```

Verify the new labels

```sh
pradeep@CN2 % kubectl get vn --show-labels       
NAME   VNI   IP FAMILIES   STATE     AGE   LABELS
vn1    5     v4            Success   37m   back-reference.core.juniper.net/c6dbefed51ec7143c19e3d9285d04ea8f7343de4632174839c9549f1=Subnet_default_vn1-v4,customer=Cust1
vn2    6     v4            Success   37m   back-reference.core.juniper.net/54f620398eebd1cfc552b90b3f8ab758524eed64d368f07cf2a545f1=Subnet_default_vn2-v4,customer=Cust1
pradeep@CN2 % 
```

```sh
pradeep@CN2 % kubectl get vn -l customer=Cust1
NAME   VNI   IP FAMILIES   STATE     AGE
vn1    5     v4            Success   37m
vn2    6     v4            Success   37m
pradeep@CN2 %
```

Each `VirtualNetwork` contains a single pod. The `VirtualNetwork` `vn1` contains `vn1-pod`. The          `VirtualNetwork` `vn2` contains `vn2-pod`.

Let us create the VNR

```sh
pradeep@CN2 % kubectl apply -f vnr-demo.yaml 
virtualnetworkrouter.core.contrail.juniper.net/vnr-1 created
pradeep@CN2 %
```

```sh
pradeep@CN2 % kubectl get vnr               
NAME    TYPE   STATE     AGE
vnr-1   mesh   Success   4s
pradeep@CN2 % 
```

```sh
pradeep@CN2 % kubectl describe vnr
Name:         vnr-1
Namespace:    default
Labels:       vnr=Cust1-vnr
Annotations:  core.juniper.net/display-name: vnr-1
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         VirtualNetworkRouter
Metadata:
  Creation Timestamp:  2022-08-22T09:07:14Z
  Finalizers:
    remove-vnr-rt-from-vn-ri.finalizers.core.juniper.net
    vnr-routinginstance-delete.finalizers.core.juniper.net
  Generation:  1
  Managed Fields:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:core.juniper.net/display-name:
          f:kubectl.kubernetes.io/last-applied-configuration:
        f:labels:
          .:
          f:vnr:
      f:spec:
        f:type:
        f:virtualNetworkSelector:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2022-08-22T09:07:14Z
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .:
          v:"remove-vnr-rt-from-vn-ri.finalizers.core.juniper.net":
          v:"vnr-routinginstance-delete.finalizers.core.juniper.net":
      f:status:
        f:observation:
        f:state:
    Manager:         manager
    Operation:       Update
    Time:            2022-08-22T09:07:15Z
  Resource Version:  195914
  UID:               54f5fe54-431e-4a1f-96b3-e8ada9fab2f4
Spec:
  Fq Name:
    default-domain
    default
    vnr-1
  Import:
  Type:  mesh
  Virtual Network Selector:
    Match Expressions:
      Key:       customer
      Operator:  In
      Values:
        Cust1
Status:
  Observation:  
  State:        Success
Events:         <none>
pradeep@CN2 % 

```

```sh
pradeep@CN2 % kubectl get ri     
NAME    ROUTETARGET     STATE     AGE
vn1     64512:8000009   Success   40m
vn2     64512:8000010   Success   40m
vnr-1   64512:8000011   Success   79s
pradeep@CN2 % kubectl get rt
NAME                   STATE     AGE
target-64512-8000000   Success   3d14h
target-64512-8000001   Success   3d14h
target-64512-8000002   Success   3d14h
target-64512-8000003   Success   3d14h
target-64512-8000004   Success   3d14h
target-64512-8000005   Success   3d14h
target-64512-8000006   Success   3d14h
target-64512-8000007   Success   3d14h
target-64512-8000008   Success   3d14h
target-64512-8000009   Success   40m
target-64512-8000010   Success   40m
target-64512-8000011   Success   81s
pradeep@CN2 % 

```

```sh
pradeep@CN2 % kubectl describe ri vnr-1 
Name:         vnr-1
Namespace:    default
Labels:       back-reference.core.juniper.net/ada66443f9ca45a891a89ead4c2b344332d592e81a488c0e423aeb4c=RouteTarget_target-64512-8000011
Annotations:  <none>
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         RoutingInstance
Metadata:
  Creation Timestamp:  2022-08-22T09:07:14Z
  Finalizers:
    route-target-delete-default.finalizers.core.juniper.net
    route-target-number-deallocation.finalizers.core.juniper.net
  Generation:  1
  Managed Fields:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .:
          v:"route-target-delete-default.finalizers.core.juniper.net":
          v:"route-target-number-deallocation.finalizers.core.juniper.net":
        f:labels:
          .:
          f:back-reference.core.juniper.net/ada66443f9ca45a891a89ead4c2b344332d592e81a488c0e423aeb4c:
        f:ownerReferences:
          .:
          k:{"uid":"54f5fe54-431e-4a1f-96b3-e8ada9fab2f4"}:
            .:
            f:apiVersion:
            f:blockOwnerDeletion:
            f:controller:
            f:kind:
            f:name:
            f:uid:
      f:status:
        f:defaultRouteTargetReference:
          .:
          f:apiVersion:
          f:attributes:
          f:kind:
          f:name:
        f:isDefault:
        f:observation:
        f:state:
    Manager:    manager
    Operation:  Update
    Time:       2022-08-22T09:07:14Z
  Owner References:
    API Version:           core.contrail.juniper.net/v1alpha1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  VirtualNetworkRouter
    Name:                  vnr-1
    UID:                   54f5fe54-431e-4a1f-96b3-e8ada9fab2f4
  Resource Version:        195907
  UID:                     3553f161-ffc9-44d0-b6a3-037fd1770154
Spec:
  Fq Name:
    default-domain
    default
    vnr-1
  Parent:
Status:
  Default Route Target Reference:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
    Fq Name:
      target-64512-8000011
    Kind:       RouteTarget
    Name:       target-64512-8000011
    UID:        1ee0ed57-7307-45b4-8c11-c5771e2be92e
  Is Default:   true
  Observation:  
  State:        Success
Events:         <none>
pradeep@CN2 % 

```



```sh
pradeep@CN2 % kubectl describe ri vn1  
Name:         vn1
Namespace:    default
Labels:       back-reference.core.juniper.net/3acfef9839cb888c5103a243eb04c2b34931b92c32a156257cd0d501=RouteTarget_target-64512-8000009
              back-reference.core.juniper.net/ada66443f9ca45a891a89ead4c2b344332d592e81a488c0e423aeb4c=RouteTarget_target-64512-8000011
              core.juniper.net/parent=2b292ddea1bd7937355c5b143bfd86b1aa46500d307ae7655af90db4
Annotations:  <none>
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         RoutingInstance
Metadata:
  Creation Timestamp:  2022-08-22T08:27:59Z
  Finalizers:
    route-target-delete-default.finalizers.core.juniper.net
    route-target-number-deallocation.finalizers.core.juniper.net
  Generation:  1
  Managed Fields:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .:
          v:"route-target-delete-default.finalizers.core.juniper.net":
          v:"route-target-number-deallocation.finalizers.core.juniper.net":
        f:labels:
          f:back-reference.core.juniper.net/3acfef9839cb888c5103a243eb04c2b34931b92c32a156257cd0d501:
        f:ownerReferences:
          .:
          k:{"uid":"e6ccbe59-d49b-4c25-bbd4-c2383940a800"}:
            .:
            f:apiVersion:
            f:blockOwnerDeletion:
            f:controller:
            f:kind:
            f:name:
            f:uid:
      f:spec:
        f:parent:
          f:apiVersion:
          f:kind:
          f:name:
          f:namespace:
          f:uid:
      f:status:
        f:defaultRouteTargetReference:
          .:
          f:apiVersion:
          f:attributes:
          f:kind:
          f:name:
        f:isDefault:
        f:observation:
        f:state:
        f:virtualNetworkRouterRouteTargetReferences:
          .:
          f:default/vnr-1:
            .:
            f:routeTargetReferences:
    Manager:    manager
    Operation:  Update
    Time:       2022-08-22T09:07:14Z
  Owner References:
    API Version:           core.contrail.juniper.net/v1alpha1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  VirtualNetwork
    Name:                  vn1
    UID:                   e6ccbe59-d49b-4c25-bbd4-c2383940a800
  Resource Version:        195911
  UID:                     ff8f37c4-4338-4b0e-81a3-399b93287045
Spec:
  Fq Name:
    default-domain
    default
    vn1
    vn1
  Parent:
    API Version:  core.contrail.juniper.net/v1alpha1
    Kind:         VirtualNetwork
    Name:         vn1
    Namespace:    default
    UID:          e6ccbe59-d49b-4c25-bbd4-c2383940a800
Status:
  Default Route Target Reference:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
    Fq Name:
      target-64512-8000009
    Kind:       RouteTarget
    Name:       target-64512-8000009
    UID:        f3c296c9-abeb-4dfb-a235-75d8d06907ac
  Is Default:   true
  Observation:  
  State:        Success
  Virtual Network Router Route Target References:
    default/vnr-1:
      Route Target References:
        API Version:  core.contrail.juniper.net/v1alpha1
        Attributes:
        Fq Name:
          target-64512-8000011
        Kind:              RouteTarget
        Name:              target-64512-8000011
        Resource Version:  195905
        UID:               1ee0ed57-7307-45b4-8c11-c5771e2be92e
Events:                    <none>
pradeep@CN2 % 

```



```sh
pradeep@CN2 % kubectl describe ri vn2
Name:         vn2
Namespace:    default
Labels:       back-reference.core.juniper.net/744aa6b17be94e6f45b4e33d1fa6468a2f1a7a3907b5687ab2a8ae3c=RouteTarget_target-64512-8000010
              back-reference.core.juniper.net/ada66443f9ca45a891a89ead4c2b344332d592e81a488c0e423aeb4c=RouteTarget_target-64512-8000011
              core.juniper.net/parent=955ff7225033c263b793b2cbecb8fb70c6e79f1236ec1e335552a1e2
Annotations:  <none>
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         RoutingInstance
Metadata:
  Creation Timestamp:  2022-08-22T08:28:03Z
  Finalizers:
    route-target-delete-default.finalizers.core.juniper.net
    route-target-number-deallocation.finalizers.core.juniper.net
  Generation:  1
  Managed Fields:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .:
          v:"route-target-delete-default.finalizers.core.juniper.net":
          v:"route-target-number-deallocation.finalizers.core.juniper.net":
        f:labels:
          f:back-reference.core.juniper.net/744aa6b17be94e6f45b4e33d1fa6468a2f1a7a3907b5687ab2a8ae3c:
        f:ownerReferences:
          .:
          k:{"uid":"646d6d34-8580-4dfe-b7b0-92ba209d64e4"}:
            .:
            f:apiVersion:
            f:blockOwnerDeletion:
            f:controller:
            f:kind:
            f:name:
            f:uid:
      f:spec:
        f:parent:
          f:apiVersion:
          f:kind:
          f:name:
          f:namespace:
          f:uid:
      f:status:
        f:defaultRouteTargetReference:
          .:
          f:apiVersion:
          f:attributes:
          f:kind:
          f:name:
        f:isDefault:
        f:observation:
        f:state:
        f:virtualNetworkRouterRouteTargetReferences:
          .:
          f:default/vnr-1:
            .:
            f:routeTargetReferences:
    Manager:    manager
    Operation:  Update
    Time:       2022-08-22T09:07:15Z
  Owner References:
    API Version:           core.contrail.juniper.net/v1alpha1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  VirtualNetwork
    Name:                  vn2
    UID:                   646d6d34-8580-4dfe-b7b0-92ba209d64e4
  Resource Version:        195913
  UID:                     ca2e9826-da26-4a5d-b4ab-ca2d3cdd1efb
Spec:
  Fq Name:
    default-domain
    default
    vn2
    vn2
  Parent:
    API Version:  core.contrail.juniper.net/v1alpha1
    Kind:         VirtualNetwork
    Name:         vn2
    Namespace:    default
    UID:          646d6d34-8580-4dfe-b7b0-92ba209d64e4
Status:
  Default Route Target Reference:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
    Fq Name:
      target-64512-8000010
    Kind:       RouteTarget
    Name:       target-64512-8000010
    UID:        1a5308cf-e24d-46bf-ab60-d762c70b6838
  Is Default:   true
  Observation:  
  State:        Success
  Virtual Network Router Route Target References:
    default/vnr-1:
      Route Target References:
        API Version:  core.contrail.juniper.net/v1alpha1
        Attributes:
        Fq Name:
          target-64512-8000011
        Kind:              RouteTarget
        Name:              target-64512-8000011
        Resource Version:  195905
        UID:               1ee0ed57-7307-45b4-8c11-c5771e2be92e
Events:                    <none>
pradeep@CN2 % 

```

In the description of both vn1 and vn2 virtual networks, we can see `Virtual Network Router Route Target References:
    default/vnr-1` under the Status section.

Along with the Default Route Target Reference, there is an additional   Virtual Network Router Route Target Reference.

## Verify Connectivity between Virtual Networks
Now that the VirtualNetworkRouter is created, verify the reachability again.

From VN1 to VN2
```sh
pradeep@CN2 % kubectl exec -it vn1-pod -- ping 172.16.20.2
PING 172.16.20.2 (172.16.20.2) 56(84) bytes of data.
64 bytes from 172.16.20.2: icmp_seq=1 ttl=63 time=8.64 ms
64 bytes from 172.16.20.2: icmp_seq=2 ttl=63 time=0.143 ms
64 bytes from 172.16.20.2: icmp_seq=3 ttl=63 time=0.067 ms
^C
--- 172.16.20.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2058ms
rtt min/avg/max/mdev = 0.067/2.950/8.640/4.023 ms
pradeep@CN2 %
```
From VN2 to VN1
```sh
pradeep@CN2 % kubectl exec -it vn2-pod -- ping 172.16.10.2
PING 172.16.10.2 (172.16.10.2) 56(84) bytes of data.
64 bytes from 172.16.10.2: icmp_seq=1 ttl=63 time=4.26 ms
64 bytes from 172.16.10.2: icmp_seq=2 ttl=63 time=0.140 ms
64 bytes from 172.16.10.2: icmp_seq=3 ttl=63 time=0.117 ms
^C
--- 172.16.10.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2014ms
rtt min/avg/max/mdev = 0.117/1.508/4.267/1.950 ms
pradeep@CN2 % 

```

This confirms that our newly created VirtualNetworkRouter has enabled communication between the two virtual networks.

