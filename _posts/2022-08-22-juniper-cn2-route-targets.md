---
layout: single
title:  "Juniper CN2 Route Targets "
date:   2022-08-22 13:55:04 +0530
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

# CN2 - Route Targets for Inter-VN Communication

## Reference

[https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/topics/concept/cn-cloud-native-route-targets.html](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/topics/concept/cn-cloud-native-route-targets.html)

## Common Route Targets for Inter-VN Communication

In this post, we will look at another way to achieve Inter-VirtualNetwork communication.

Cloud-Native Contrail Networking supports inter-virtual network routing using route targets. Specify common route targets to route traffic between your virtual networks. 

Lets start with a fresh setup without any Pods,VNs,VNRs.

```sh
pradga@pradga-mbp CN2 % kubectl get pods,vn,vnr,rt,ri,net-attach-def
NAME                                                         STATE     AGE
routetarget.core.contrail.juniper.net/target-64512-8000000   Success   3d19h
routetarget.core.contrail.juniper.net/target-64512-8000001   Success   3d19h
routetarget.core.contrail.juniper.net/target-64512-8000002   Success   3d19h
routetarget.core.contrail.juniper.net/target-64512-8000003   Success   3d19h
routetarget.core.contrail.juniper.net/target-64512-8000004   Success   3d19h
routetarget.core.contrail.juniper.net/target-64512-8000005   Success   3d19h
routetarget.core.contrail.juniper.net/target-64512-8000006   Success   3d19h
routetarget.core.contrail.juniper.net/target-64512-8000007   Success   3d19h
routetarget.core.contrail.juniper.net/target-64512-8000008   Success   3d19h
pradga@pradga-mbp CN2 % 
```

## NetworkAttachmentDefinition

Here we are going to create two VirtualNetworks both with the same 
` "routeTargetList": ["target:1234:567"]`.

```yaml
pradga@pradga-mbp CN2 % cat vn1-rt.yaml 
  apiVersion: "k8s.cni.cncf.io/v1"
  kind: NetworkAttachmentDefinition
  metadata:
    name: vn1
    namespace: default
    annotations:
      juniper.net/networks: '{
        "ipamV4Subnet": "172.16.10.0/24",
       
        "routeTargetList": ["target:1234:567"]
        
        
      }'
      
  spec:
    config: '{
    "cniVersion": "0.3.1",
    "name": "vn1",
    "type": "contrail-k8s-cni"
  }'
  
pradga@pradga-mbp CN2 %
```

```yaml
pradga@pradga-mbp CN2 % cat vn2-rt.yaml
  apiVersion: "k8s.cni.cncf.io/v1"
  kind: NetworkAttachmentDefinition
  metadata:
    name: vn2
    namespace: default
    annotations:
      juniper.net/networks: '{
        "ipamV4Subnet": "172.16.20.0/24",
       
        "routeTargetList": ["target:1234:567"]
        
        
      }'
      
  spec:
    config: '{
    "cniVersion": "0.3.1",
    "name": "vn2",
    "type": "contrail-k8s-cni"
  }'
pradga@pradga-mbp CN2 % 
```

```sh
pradga@pradga-mbp CN2 % kubectl apply -f vn1-rt.yaml 
networkattachmentdefinition.k8s.cni.cncf.io/vn1 created
pradga@pradga-mbp CN2 % kubectl apply -f vn2-rt.yaml
networkattachmentdefinition.k8s.cni.cncf.io/vn2 created
pradga@pradga-mbp CN2 % 
```

```sh
pradga@pradga-mbp CN2 % kubectl get net-attach-def,vn,subnet
NAME                                              AGE
networkattachmentdefinition.k8s.cni.cncf.io/vn1   21s
networkattachmentdefinition.k8s.cni.cncf.io/vn2   18s

NAME                                           VNI   IP FAMILIES   STATE     AGE
virtualnetwork.core.contrail.juniper.net/vn1   5     v4            Success   21s
virtualnetwork.core.contrail.juniper.net/vn2   6     v4            Success   18s

NAME                                      CIDR             USAGE   STATE     AGE
subnet.core.contrail.juniper.net/vn1-v4   172.16.10.0/24   1.17%   Success   21s
subnet.core.contrail.juniper.net/vn2-v4   172.16.20.0/24   1.17%   Success   18s
pradga@pradga-mbp CN2 % 
```

## Route Targets

```sh
pradga@pradga-mbp CN2 % kubectl get rt                      
NAME                   STATE     AGE
target-1234-567        Success   38s
target-64512-8000000   Success   3d19h
target-64512-8000001   Success   3d19h
target-64512-8000002   Success   3d19h
target-64512-8000003   Success   3d19h
target-64512-8000004   Success   3d19h
target-64512-8000005   Success   3d19h
target-64512-8000006   Success   3d19h
target-64512-8000007   Success   3d19h
target-64512-8000008   Success   3d19h
target-64512-8000009   Success   38s
target-64512-8000010   Success   34s
pradga@pradga-mbp CN2 % kubectl get ri
NAME   ROUTETARGET     STATE     AGE
vn1    64512:8000009   Success   42s
vn2    64512:8000010   Success   38s
pradga@pradga-mbp CN2 % 
```
## RoutingInstance

```sh
pradga@pradga-mbp CN2 % kubectl describe ri vn1
Name:         vn1
Namespace:    default
Labels:       back-reference.core.juniper.net/3acfef9839cb888c5103a243eb04c2b34931b92c32a156257cd0d501=RouteTarget_target-64512-8000009
              back-reference.core.juniper.net/b54a4327c2757f2a7613d6fb4864a0c8b92568a34ee4294af9e105a2=RouteTarget_target-1234-567
              core.juniper.net/parent=2b292ddea1bd7937355c5b143bfd86b1aa46500d307ae7655af90db4
Annotations:  <none>
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         RoutingInstance
Metadata:
  Creation Timestamp:  2022-08-22T14:47:36Z
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
          k:{"uid":"92222ffe-9a5b-48a7-bf68-07a3c471698f"}:
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
        f:routeTargetReferences:
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
    Time:       2022-08-22T14:47:36Z
  Owner References:
    API Version:           core.contrail.juniper.net/v1alpha1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  VirtualNetwork
    Name:                  vn1
    UID:                   92222ffe-9a5b-48a7-bf68-07a3c471698f
  Resource Version:        235879
  UID:                     e81fa7c4-0286-41d6-b800-5cfc51363e4c
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
    UID:          92222ffe-9a5b-48a7-bf68-07a3c471698f
  Route Target References:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
    Fq Name:
      target-1234-567
    Kind:              RouteTarget
    Name:              target-1234-567
    Resource Version:  235866
    UID:               7151bc59-522a-4a70-ad1b-f10441a5b812
Status:
  Default Route Target Reference:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
    Fq Name:
      target-64512-8000009
    Kind:       RouteTarget
    Name:       target-64512-8000009
    UID:        b726bb77-c5ee-48e2-84e2-d1e501fe29ce
  Is Default:   true
  Observation:  
  State:        Success
Events:         <none>
pradga@pradga-mbp CN2 % 
```

```sh
pradga@pradga-mbp CN2 % kubectl describe ri vn2
Name:         vn2
Namespace:    default
Labels:       back-reference.core.juniper.net/744aa6b17be94e6f45b4e33d1fa6468a2f1a7a3907b5687ab2a8ae3c=RouteTarget_target-64512-8000010
              back-reference.core.juniper.net/b54a4327c2757f2a7613d6fb4864a0c8b92568a34ee4294af9e105a2=RouteTarget_target-1234-567
              core.juniper.net/parent=955ff7225033c263b793b2cbecb8fb70c6e79f1236ec1e335552a1e2
Annotations:  <none>
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         RoutingInstance
Metadata:
  Creation Timestamp:  2022-08-22T14:47:40Z
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
          k:{"uid":"dc3cbef1-8a27-4310-a2d2-5d4a01f64f47"}:
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
        f:routeTargetReferences:
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
    Time:       2022-08-22T14:47:40Z
  Owner References:
    API Version:           core.contrail.juniper.net/v1alpha1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  VirtualNetwork
    Name:                  vn2
    UID:                   dc3cbef1-8a27-4310-a2d2-5d4a01f64f47
  Resource Version:        235910
  UID:                     f67b83ae-375d-4ce6-889b-96770ec9c5d9
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
    UID:          dc3cbef1-8a27-4310-a2d2-5d4a01f64f47
  Route Target References:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
    Fq Name:
      target-1234-567
    Kind:              RouteTarget
    Name:              target-1234-567
    Resource Version:  235866
    UID:               7151bc59-522a-4a70-ad1b-f10441a5b812
Status:
  Default Route Target Reference:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
    Fq Name:
      target-64512-8000010
    Kind:       RouteTarget
    Name:       target-64512-8000010
    UID:        8c10c274-c65e-4ad8-be36-8020c62057d4
  Is Default:   true
  Observation:  
  State:        Success
Events:         <none>
pradga@pradga-mbp CN2 % 
```
In the description of the RoutingInstances, we can see  `Route Target References` along with the `Default Route Target Reference`.

Each RI has their own Default Route Target  but there is a common Route Target between the two.


## Pods
Let us create two pods and associated each of them with a different network. vn1-pod is assigned to vn1 and vn2-pod is assigned to vn2 virtual network. One difference in the Pod definition file is the securityContext. Without this, we will not be able to do network connectivity checks like ping, or addition of static routes etc.

```yaml
pradga@pradga-mbp CN2 % cat vn1-pod.yaml 
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
pradga@pradga-mbp CN2 %
```

```yaml
pradga@pradga-mbp CN2 % cat vn2-pod.yaml 
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
pradga@pradga-mbp CN2 % 
```

```sh
pradga@pradga-mbp CN2 % kubectl apply -f vn1-pod.yaml 
pod/vn1-pod created
pradga@pradga-mbp CN2 % kubectl apply -f vn2-pod.yaml
pod/vn2-pod created
pradga@pradga-mbp CN2 % 
```

```sh
pradga@pradga-mbp CN2 % kubectl get pods -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
vn1-pod   1/1     Running   0          84s   10.244.0.3   minikube   <none>           <none>
vn2-pod   1/1     Running   0          80s   10.244.0.4   minikube   <none>           <none>
pradga@pradga-mbp CN2 % 
```
```sh
pradga@pradga-mbp CN2 % kubectl describe pods vn1-pod
Name:         vn1-pod
Namespace:    default
Priority:     0
Node:         minikube/192.168.177.47
Start Time:   Mon, 22 Aug 2022 20:20:22 +0530
Labels:       <none>
Annotations:  k8s.v1.cni.cncf.io/network-status:
                [{
                    "name": "contrail-k8s-kubemanager-mk-contrail/default-podnetwork",
                    "interface": "eth0",
                    "ips": [
                        "10.244.0.3"
                    ],
                    "mac": "02:33:8a:94:6a:25",
                    "default": true,
                    "dns": {}
                },{
                    "name": "default/vn1",
                    "interface": "eth1",
                    "ips": [
                        "172.16.10.2"
                    ],
                    "mac": "02:25:b2:d1:c1:5f",
                    "dns": {}
                }]
              k8s.v1.cni.cncf.io/networks: vn1
              kube-manager.juniper.net/vm-uuid: d8a27bea-70c5-4802-863e-66fbb3695817
Status:       Running
IP:           10.244.0.3
IPs:
  IP:  10.244.0.3
Containers:
  vn1-pod:
    Container ID:  cri-o://f3a5197144eb0507b4c70cda6322ed85002d3a2629a2ce2d6317b28925c8cf7e
    Image:         gcr.io/google-containers/toolbox
    Image ID:      gcr.io/google-containers/toolbox@sha256:4de42fa6aa0b8b85fa9839ca2eba4457335fa3840007cd2b72e5869c2af87d35
    Port:          <none>
    Host Port:     <none>
    Command:
      bash
      -c
      while true; do sleep 60s; done
    State:          Running
      Started:      Mon, 22 Aug 2022 20:21:15 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-g4746 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-g4746:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  12m   default-scheduler  Successfully assigned default/vn1-pod to minikube
  Normal  Pulling    12m   kubelet            Pulling image "gcr.io/google-containers/toolbox"
  Normal  Pulled     11m   kubelet            Successfully pulled image "gcr.io/google-containers/toolbox" in 22.189943665s
  Normal  Created    11m   kubelet            Created container vn1-pod
  Normal  Started    11m   kubelet            Started container vn1-pod
pradga@pradga-mbp CN2 % 
```

```sh
pradga@pradga-mbp CN2 % kubectl describe pods vn2-pod
Name:         vn2-pod
Namespace:    default
Priority:     0
Node:         minikube/192.168.177.47
Start Time:   Mon, 22 Aug 2022 20:20:26 +0530
Labels:       <none>
Annotations:  k8s.v1.cni.cncf.io/network-status:
                [{
                    "name": "contrail-k8s-kubemanager-mk-contrail/default-podnetwork",
                    "interface": "eth0",
                    "ips": [
                        "10.244.0.4"
                    ],
                    "mac": "02:f4:49:16:03:7b",
                    "default": true,
                    "dns": {}
                },{
                    "name": "default/vn2",
                    "interface": "eth1",
                    "ips": [
                        "172.16.20.2"
                    ],
                    "mac": "02:0e:30:fa:58:08",
                    "dns": {}
                }]
              k8s.v1.cni.cncf.io/networks: vn2
              kube-manager.juniper.net/vm-uuid: 1a00fe3d-39f5-444e-b4f8-0adfb5a4a1f6
Status:       Running
IP:           10.244.0.4
IPs:
  IP:  10.244.0.4
Containers:
  vn2-pod:
    Container ID:  cri-o://c0a0b8e7d08becf5986983b9850d6bf2b5ac3e3c6c708f55fc4503e1b6075ac5
    Image:         gcr.io/google-containers/toolbox
    Image ID:      gcr.io/google-containers/toolbox@sha256:4de42fa6aa0b8b85fa9839ca2eba4457335fa3840007cd2b72e5869c2af87d35
    Port:          <none>
    Host Port:     <none>
    Command:
      bash
      -c
      while true; do sleep 60s; done
    State:          Running
      Started:      Mon, 22 Aug 2022 20:21:37 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-p7796 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-p7796:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  13m   default-scheduler  Successfully assigned default/vn2-pod to minikube
  Normal  Pulling    13m   kubelet            Pulling image "gcr.io/google-containers/toolbox"
  Normal  Pulled     12m   kubelet            Successfully pulled image "gcr.io/google-containers/toolbox" in 39.835891441s
  Normal  Created    12m   kubelet            Created container vn2-pod
  Normal  Started    12m   kubelet            Started container vn2-pod
pradga@pradga-mbp CN2 % 
```


## Verify Communication
### vn1-pod

```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn1-pod -- ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
38: eth1@if39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:25:b2:d1:c1:5f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.10.2/24 brd 172.16.10.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::f4ca:c0ff:fe88:9919/64 scope link 
       valid_lft forever preferred_lft forever
40: eth0@if41: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:33:8a:94:6a:25 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.3/16 brd 10.244.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6c12:87ff:fe05:2bbe/64 scope link 
       valid_lft forever preferred_lft forever
pradga@pradga-mbp CN2 %
```

```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn1-pod -- ip route
default via 10.244.0.1 dev eth0 
10.244.0.0/16 dev eth0 proto kernel scope link src 10.244.0.3 
172.16.10.0/24 dev eth1 proto kernel scope link src 172.16.10.2 
```
```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn1-pod -- ip route add 172.16.20.0/24 via 172.16.10.1
pradga@pradga-mbp CN2 %
```

```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn1-pod -- ip route                                   
default via 10.244.0.1 dev eth0 
10.244.0.0/16 dev eth0 proto kernel scope link src 10.244.0.3 
172.16.10.0/24 dev eth1 proto kernel scope link src 172.16.10.2 
172.16.20.0/24 via 172.16.10.1 dev eth1 
pradga@pradga-mbp CN2 % 
```

### vn2-pod

```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn2-pod -- ip a   
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
42: eth1@if43: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:0e:30:fa:58:08 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.20.2/24 brd 172.16.20.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::d4aa:2bff:fe58:d741/64 scope link 
       valid_lft forever preferred_lft forever
44: eth0@if45: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:f4:49:16:03:7b brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.4/16 brd 10.244.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5447:2cff:feed:f795/64 scope link 
       valid_lft forever preferred_lft forever
pradga@pradga-mbp CN2 %               
```



```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn2-pod -- ip route
default via 10.244.0.1 dev eth0 
10.244.0.0/16 dev eth0 proto kernel scope link src 10.244.0.4 
172.16.20.0/24 dev eth1 proto kernel scope link src 172.16.20.2 
pradga@pradga-mbp CN2 % 
```

```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn2-pod -- ip route add 172.16.10.0/24 via 172.16.20.1
pradga@pradga-mbp CN2 %
```

```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn2-pod -- ip route                                   
default via 10.244.0.1 dev eth0 
10.244.0.0/16 dev eth0 proto kernel scope link src 10.244.0.4 
172.16.10.0/24 via 172.16.20.1 dev eth1 
172.16.20.0/24 dev eth1 proto kernel scope link src 172.16.20.2 
pradga@pradga-mbp CN2 % 
```



```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn2-pod -- ping 172.16.10.2
PING 172.16.10.2 (172.16.10.2) 56(84) bytes of data.
64 bytes from 172.16.10.2: icmp_seq=1 ttl=63 time=4.76 ms
64 bytes from 172.16.10.2: icmp_seq=2 ttl=63 time=0.288 ms
64 bytes from 172.16.10.2: icmp_seq=3 ttl=63 time=0.091 ms
^C
--- 172.16.10.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2025ms
rtt min/avg/max/mdev = 0.091/1.715/4.767/2.159 ms
pradga@pradga-mbp CN2 % 
```


Verification from the `vn1-pod`.

```sh
pradga@pradga-mbp CN2 % kubectl exec -it vn1-pod -- ping 172.16.20.2
PING 172.16.20.2 (172.16.20.2) 56(84) bytes of data.
64 bytes from 172.16.20.2: icmp_seq=1 ttl=63 time=0.136 ms
64 bytes from 172.16.20.2: icmp_seq=2 ttl=63 time=0.222 ms
64 bytes from 172.16.20.2: icmp_seq=3 ttl=63 time=0.132 ms
64 bytes from 172.16.20.2: icmp_seq=4 ttl=63 time=0.087 ms
64 bytes from 172.16.20.2: icmp_seq=5 ttl=63 time=0.102 ms
¸64 bytes from 172.16.20.2: icmp_seq=6 ttl=63 time=0.099 ms
^C
--- 172.16.20.2 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5128ms
rtt min/avg/max/mdev = 0.087/0.129/0.222/0.046 ms
pradga@pradga-mbp CN2 % 

```


We can see that because of the common RouteTarget, `target-1234-567`, between the two virtual networks, both Pods are able to communicate with each other, though they are in different virtual networks.

