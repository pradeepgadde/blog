---
layout: single
title:  "Juniper CN2-Enable Pods with Multiple Network Interfaces "
date:   2022-08-22 04:55:04 +0530
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
  name     : "Junos"
  avatar   : "/assets/images/junos.png"
 
sidebar:
  - title: "Blog"
 
    text: "Checkout other topics"
    nav: my-sidebar
---
# CN2-Enable Pods with Multiple Network Interfaces

## Reference
[https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/topics/task/cn-cloud-native-multiple-interface-pod.html](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-feature-guide/cn-cloud-native-network-feature/topics/task/cn-cloud-native-multiple-interface-pod.html)


## NetworkAttachmentDefinition
To support multiple interfaces we must first define the `NetworkAttachmentDefinition` object and include the `juniper.net/networks` annotation.

```sh
pradeep@CN2 % kubectl api-resources| grep cni
network-attachment-definitions    net-attach-def   k8s.cni.cncf.io/v1                          true         NetworkAttachmentDefinition
```
```sh

pradeep@CN2 % kubectl explain net-attach-def
KIND:     NetworkAttachmentDefinition
VERSION:  k8s.cni.cncf.io/v1

DESCRIPTION:
     <empty>

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>

pradeep@CN2 % 
```

```yaml
pradeep@CN2 % kubectl get net-attach-def
No resources found in default namespace.
pradeep@CN2 % kubectl get net-attach-def -A
No resources found
pradeep@CN2 %
```

Here is a simple definition to create a NetworkAttachmentDefinition called `vn1`.  Later you will see a VirtualNetwork by the same name `vn1` gets created and a subnet called `vn1-v4` (IPv4 subnet of vn1 virtual network).

By including the `juniper.net/networks` annotation, Contrail-related objects will be created in the network, including the `VirtualNetwork` object and the `Subnet` object.

Here is a simple definition to create a NetworkAttachmentDefinition called `vn1`.  Later you will see a VirtualNetwork by the same name `vn1` gets created and a subnet called `vn1-v4` (IPv4 subnet of vn1 virtual network).

```yaml
pradeep@CN2 % cat nad-demo.yaml 
  apiVersion: "k8s.cni.cncf.io/v1"
  kind: NetworkAttachmentDefinition
  metadata:
    name: vn1
    namespace: default
    annotations:
      juniper.net/networks: '{
        "ipamV4Subnet": "172.16.10.0/24",
       
        "routeTargetList": ["target:23:4561"],
        "importRouteTargetList": ["target:10.2.2.2:561"],
        "exportRouteTargetList": ["target:10.1.1.1:561"]
        
      }'
      
  spec:
    config: '{
    "cniVersion": "0.3.1",
    "name": "vn1",
    "type": "contrail-k8s-cni"
  }'

pradeep@CN2 % 

```



```sh
pradeep@CN2 % kubectl apply -f nad-demo.yaml 
networkattachmentdefinition.k8s.cni.cncf.io/vn1 created
pradeep@CN2 %
```



```sh
pradeep@CN2 % kubectl get net-attach-def -A 
NAMESPACE   NAME   AGE
default     vn1    4s
pradeep@CN2 %
```



```sh
pradeep@CN2 % kubectl describe  net-attach-def -A
Name:         vn1
Namespace:    default
Labels:       <none>
Annotations:  juniper.net/networks:
                { "ipamV4Subnet": "172.16.10.0/24",
                "routeTargetList": ["target:23:4561"], "importRouteTargetList": ["target:10.2.2.2:561"], "exportRouteTargetList": ["target:10.1.1.1:561"]
                }
              juniper.net/networks-status: success creating VirtualNetwork vn1 v4Subnet: 172.16.10.0/24 
API Version:  k8s.cni.cncf.io/v1
Kind:         NetworkAttachmentDefinition
Metadata:
  Creation Timestamp:  2022-08-19T20:56:09Z
  Finalizers:
    virtual-network.finalizers.kube-manager.juniper.net
  Generation:  1
  Managed Fields:
    API Version:  k8s.cni.cncf.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:juniper.net/networks:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:config:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2022-08-19T20:56:09Z
    API Version:  k8s.cni.cncf.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          f:juniper.net/networks-status:
        f:finalizers:
          .:
          v:"virtual-network.finalizers.kube-manager.juniper.net":
    Manager:         kubemanager
    Operation:       Update
    Time:            2022-08-19T20:56:09Z
  Resource Version:  118075
  UID:               6adde7dc-553f-47d3-86e3-edbb4de136bf
Spec:
  Config:  { "cniVersion": "0.3.1", "name": "vn1", "type": "contrail-k8s-cni" }
Events:    <none>
pradeep@CN2 %
```

From the above description, we can see that ` juniper.net/networks-status: success creating VirtualNetwork vn1 v4Subnet: 172.16.10.0/24 `.

This confirms that additional resources VirtualNetwork and Subnet got created from the NetworkAttachmentDefinition.

## VirtualNetwork

```sh
pradeep@CN2 % kubectl explain vn 
KIND:     VirtualNetwork
VERSION:  core.contrail.juniper.net/v1alpha1

DESCRIPTION:
     VirtualNetwork is a collection of endpoints (interface or IP(s) or MAC(s))
     that can communicate with each other. It is also a collection of subnets
     whose default gateways are connected by an implicit router.

{trimmed}

pradeep@CN2 % 

```

In the descritpion of the VirtualNetwork, look at the annotation `juniper.net/nad-name: vn1` and the corresponding Subnet, `vn1-v4` in this case.


```sh
pradeep@CN2 % kubectl get vn                     
NAME   VNI   IP FAMILIES   STATE     AGE
vn1    5     v4            Success   13s
pradeep@CN2 % kubectl describe vn
Name:         vn1
Namespace:    default
Labels:       back-reference.core.juniper.net/c6dbefed51ec7143c19e3d9285d04ea8f7343de4632174839c9549f1=Subnet_default_vn1-v4
Annotations:  juniper.net/nad-cluster-name: contrail-k8s-kubemanager-mk
              juniper.net/nad-name: vn1
              juniper.net/nad-namespace: default
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         VirtualNetwork
Metadata:
  Creation Timestamp:  2022-08-19T20:58:39Z
  Finalizers:
    virtual-network-id-deallocation.finalizers.core.juniper.net
    route-target.finalizers.core.juniper.net
    vn-routinginstance-delete.finalizers.core.juniper.net
  Generation:  1
  Managed Fields:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:juniper.net/nad-cluster-name:
          f:juniper.net/nad-name:
          f:juniper.net/nad-namespace:
      f:spec:
        f:exportRouteTargetList:
        f:fabricSNAT:
        f:importRouteTargetList:
        f:routeTargetList:
        f:v4SubnetReference:
          .:
          f:apiVersion:
          f:kind:
          f:name:
          f:namespace:
          f:resourceVersion:
          f:uid:
        f:virtualNetworkProperties:
          f:forwardingMode:
          f:rpf:
    Manager:      kubemanager
    Operation:    Update
    Time:         2022-08-19T20:58:39Z
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .:
          v:"route-target.finalizers.core.juniper.net":
          v:"virtual-network-id-deallocation.finalizers.core.juniper.net":
          v:"vn-routinginstance-delete.finalizers.core.juniper.net":
      f:status:
        f:observation:
        f:state:
        f:virtualNetworkNetworkId:
    Manager:         manager
    Operation:       Update
    Time:            2022-08-19T20:58:40Z
  Resource Version:  118098
  UID:               e8254ef9-761b-4516-9779-9adaf93e2cec
Spec:
  Export Route Target List:
    target:10.1.1.1:561
  Fabric SNAT:  false
  Fq Name:
    default-domain
    default
    vn1
  Import Route Target List:
    target:10.2.2.2:561
  Route Target List:
    target:23:4561
  v4SubnetReference:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fq Name:
      default-domain
      default
      vn1-v4
    Kind:              Subnet
    Name:              vn1-v4
    Namespace:         default
    Resource Version:  118072
    UID:               abbd4b83-c70a-46dd-8dd4-71f0eb81cd89
  Virtual Network Properties:
    Forwarding Mode:  l2_l3
    Rpf:              enable
Status:
  Observation:                 
  State:                       Success
  Virtual Network Network Id:  5
Events:                        <none>
pradeep@CN2 % 
```

## Create a Pod with an Interface assigned to the Virtual Network

After the NetworkAttachmentDefinition object is created, we can assign virtual networks to the Pods using the annotation `k8s.v1.cni.cncf.io/networks`.

In the following example, we are creating a Pod named `vn1-pod` with an additional interface and associating that with the `vn1` virtual network that we created in the previous steps.


```yaml
pradeep@CN2 % cat vn-pod.yaml 
  apiVersion: v1
  kind: Pod
  metadata:
    name: vn1-pod
    namespace: default
    annotations:
      k8s.v1.cni.cncf.io/networks: vn1
      
  spec:
    containers:
    - name: vn1-container
      image: gcr.io/google-containers/toolbox
      command: ["bash","-c","while true; do sleep 60s; done"]
pradeep@CN2 % 
```



```sh
pradeep@CN2 % kubectl apply -f vn-pod.yaml 
pod/vn1-pod created
pradeep@CN2 %
```

So far we have three pods, including the newly created `vn1-pod`.

```sh
pradeep@CN2 % kubectl get pods -o wide
NAME            READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
another-nginx   1/1     Running   0          9h      10.244.0.4   minikube   <none>           <none>
nginx           1/1     Running   0          26h     10.244.0.3   minikube   <none>           <none>
vn1-pod         1/1     Running   0          3m39s   10.244.0.5   minikube   <none>           <none>
pradeep@CN2 %
```

Describe the Pod to see additional details like the `k8s.v1.cni.cncf.io/network-status` annotation.

```sh
pradeep@CN2 % kubectl describe pods vn1-pod 
Name:         vn1-pod
Namespace:    default
Priority:     0
Node:         minikube/192.168.177.47
Start Time:   Sat, 20 Aug 2022 02:30:16 +0530
Labels:       <none>
Annotations:  k8s.v1.cni.cncf.io/network-status:
                [{
                    "name": "contrail-k8s-kubemanager-mk-contrail/default-podnetwork",
                    "interface": "eth0",
                    "ips": [
                        "10.244.0.5"
                    ],
                    "mac": "02:11:8e:c0:2f:e0",
                    "default": true,
                    "dns": {}
                },{
                    "name": "default/vn1",
                    "interface": "eth1",
                    "ips": [
                        "172.16.10.2"
                    ],
                    "mac": "02:21:0f:f6:47:fc",
                    "dns": {}
                }]
              k8s.v1.cni.cncf.io/networks: vn1
              kube-manager.juniper.net/vm-uuid: 71748541-b2d5-4b1d-bcb1-54f11dacc799
Status:       Running
IP:           10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  vn1-container:
    Container ID:  cri-o://3491a643149a8089de2c47d01f454924df16d0c56fd0d9579ff0f27eb78122a1
    Image:         gcr.io/google-containers/toolbox
    Image ID:      gcr.io/google-containers/toolbox@sha256:4de42fa6aa0b8b85fa9839ca2eba4457335fa3840007cd2b72e5869c2af87d35
    Port:          <none>
    Host Port:     <none>
    Command:
      bash
      -c
      while true; do sleep 60s; done
    State:          Running
      Started:      Sat, 20 Aug 2022 02:33:18 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qpzcp (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-qpzcp:
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
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m49s  default-scheduler  Successfully assigned default/vn1-pod to minikube
  Normal  Pulling    3m18s  kubelet            Pulling image "gcr.io/google-containers/toolbox"
  Normal  Pulled     47s    kubelet            Successfully pulled image "gcr.io/google-containers/toolbox" in 2m30.604935391s
  Normal  Created    47s    kubelet            Created container vn1-container
  Normal  Started    47s    kubelet            Started container vn1-container
pradeep@CN2 %
```

From the Annotations section, w can see that the `vn1-pod` has got two interfaces, `eth0` and `eth1`. The first interface `eth0` obtained the IP address (`10.244.0.5`) from the `default-podnetwork` where as the second interface `eth1` got its IP address (`172.16.10.2`) from the `vn1` virtual network.


## Subnet
```sh
pradeep@CN2 % kubectl explain subnet        
KIND:     Subnet
VERSION:  core.contrail.juniper.net/v1alpha1

DESCRIPTION:
     Subnet represents a block of IP addresses and its configuration. IPAM
     allocates and releases IP address from that block on demand. It can be used
     by different VirtualNetwork in the mean time.

{trimmed}

pradeep@CN2 % 
```

```sh
pradeep@CN2 % kubectl get subnet
NAME     CIDR             USAGE   STATE     AGE
vn1-v4   172.16.10.0/24   1.56%   Success   4m49s
pradeep@CN2 % kubectl describe subnet
Name:         vn1-v4
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         Subnet
Metadata:
  Creation Timestamp:  2022-08-19T20:58:39Z
  Finalizers:
    subnet-pool.finalizers.core.juniper.net
  Generation:  1
  Managed Fields:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        f:cidr:
        f:defaultGateway:
    Manager:      kubemanager
    Operation:    Update
    Time:         2022-08-19T20:58:39Z
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .:
          v:"subnet-pool.finalizers.core.juniper.net":
      f:status:
        f:allocationUsage:
        f:ipCount:
        f:observation:
        f:state:
    Manager:         manager
    Operation:       Update
    Time:            2022-08-19T20:58:40Z
  Resource Version:  118409
  UID:               abbd4b83-c70a-46dd-8dd4-71f0eb81cd89
Spec:
  Cidr:             172.16.10.0/24
  Default Gateway:  172.16.10.1
  Fq Name:
    default-domain
    default
    vn1-v4
Status:
  Allocation Usage:  1.56%
  Ip Count:          4
  Observation:       
  State:             Success
Events:              <none>
pradeep@CN2 %
```

Verify that the `vn1-pod` has got two interfaces indeed and their IP addresses, by logging into the container.


```sh
pradeep@CN2 % kubectl exec -it vn1-pod -- bash
root@vn1-pod:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
235: eth0@if236: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:11:8e:c0:2f:e0 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.0.5/16 brd 10.244.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::34a7:f8ff:fe73:c932/64 scope link 
       valid_lft forever preferred_lft forever
237: eth1@if238: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:21:0f:f6:47:fc brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.10.2/24 brd 172.16.10.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::7c88:7aff:fec6:8425/64 scope link 
       valid_lft forever preferred_lft forever
root@vn1-pod:/# exit
exit

```

This confirms that our Pod has multiple interfaces, eth0 and eth1. Each interface in a different sunet.

## RoutingInstance

```sh
pradeep@CN2 % kubectl explain ri    
KIND:     RoutingInstance
VERSION:  core.contrail.juniper.net/v1alpha1

DESCRIPTION:
     RoutingInstance is a group of customer attachment points with the same
     connectivity policies. Corresponding to the VRF in L3VPN/EVPN.

{trimmed}

pradeep@CN2 % 

```

```sh
pradeep@CN2 % kubectl get ri
NAME   ROUTETARGET     STATE     AGE
vn1    64512:8000009   Success   7m4s
pradeep@CN2 %
```

```sh
pradeep@CN2 % kubectl describe ri vn1
Name:         vn1
Namespace:    default
Labels:       back-reference.core.juniper.net/35c179a1e4e94f04e84921a626afd044d12e08c31c0c16f2e2b759cd=RouteTarget_target-10.2.2.2-561
              back-reference.core.juniper.net/3acfef9839cb888c5103a243eb04c2b34931b92c32a156257cd0d501=RouteTarget_target-64512-8000009
              back-reference.core.juniper.net/583035cccdfe4b1ce3d00ad2bb7120ab63890eccc91f4509152b7127=RouteTarget_target-10.1.1.1-561
              back-reference.core.juniper.net/b876e04fdbe627b791a246ddf58db3af64535c18af924e1c5229e5e6=RouteTarget_target-23-4561
              core.juniper.net/parent=2b292ddea1bd7937355c5b143bfd86b1aa46500d307ae7655af90db4
Annotations:  <none>
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         RoutingInstance
Metadata:
  Creation Timestamp:  2022-08-19T20:58:40Z
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
          k:{"uid":"e8254ef9-761b-4516-9779-9adaf93e2cec"}:
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
    Time:       2022-08-19T20:58:40Z
  Owner References:
    API Version:           core.contrail.juniper.net/v1alpha1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  VirtualNetwork
    Name:                  vn1
    UID:                   e8254ef9-761b-4516-9779-9adaf93e2cec
  Resource Version:        118097
  UID:                     9a9fb7e0-7c9a-4084-8662-279612d13106
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
    UID:          e8254ef9-761b-4516-9779-9adaf93e2cec
  Route Target References:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
      Import Export:  export
    Fq Name:
      target-10.1.1.1-561
    Kind:              RouteTarget
    Name:              target-10.1.1.1-561
    Resource Version:  118079
    UID:               e0e1ae40-b8fd-4d44-b276-d4894e9e7863
    API Version:       core.contrail.juniper.net/v1alpha1
    Attributes:
      Import Export:  import
    Fq Name:
      target-10.2.2.2-561
    Kind:              RouteTarget
    Name:              target-10.2.2.2-561
    Resource Version:  118081
    UID:               7dde919d-7f2b-43d3-abb6-a2b8bbc63f9b
    API Version:       core.contrail.juniper.net/v1alpha1
    Attributes:
    Fq Name:
      target-23-4561
    Kind:              RouteTarget
    Name:              target-23-4561
    Resource Version:  118086
    UID:               a03285e3-be39-4968-a4a2-66b2b688295c
Status:
  Default Route Target Reference:
    API Version:  core.contrail.juniper.net/v1alpha1
    Attributes:
    Fq Name:
      target-64512-8000009
    Kind:       RouteTarget
    Name:       target-64512-8000009
    UID:        195f860d-2c99-4f57-a574-8970c57d823a
  Is Default:   true
  Observation:  
  State:        Success
Events:         <none>
pradeep@CN2 %
```



## VirtualMachine
```sh
pradeep@CN2 % kubectl explain vm
KIND:     VirtualMachine
VERSION:  core.contrail.juniper.net/v1alpha1

DESCRIPTION:
     VirtualMachine represents a computational resource, for example, a virtual
     machine, bare metal server, or container.

{trimmed}

pradeep@CN2 % 
```
Corresponding to each Pod, there is a VirtualMachine. Here we see four, however we have created only three pods so far (in the `default` namespace). The fourthe one is the `coredns` pod from the `kube-system` namespace.

```sh
pradeep@CN2 % kubectl get vm 
NAME                                                            TYPE        WORKLOAD                                STATE     AGE
contrail-k8s-kubemanager-mk-another-nginx-3e420ca1              container   /default/another-nginx                  Success   9h
contrail-k8s-kubemanager-mk-coredns-558bd4d5db-r6nck-9cb64c64   container   /kube-system/coredns-558bd4d5db-r6nck   Success   26h
contrail-k8s-kubemanager-mk-nginx-438edddb                      container   /default/nginx                          Success   26h
contrail-k8s-kubemanager-mk-vn1-pod-c478b06e                    container   /default/vn1-pod                        Success   6m38s
pradeep@CN2 %
```

## VirtualMachineInterface
```sh
pradeep@CN2 % kubectl explain vmi
KIND:     VirtualMachineInterface
VERSION:  core.contrail.juniper.net/v1alpha1

DESCRIPTION:
     VirtualMachineInterface represents an interface(port) into virtual network.
     It may or may not have corresponding virtual machine. A virtual machine
     interface has at least a MAC address and an IP address.

{trimmed}

pradeep@CN2 % 

```

Here we can see that the `vn1-pod` is part of two networks: `default-podnetwork ` and `vn1`.  The `eth0` interface of the `vn1-pod` corresponds to the `vn1-pod-9789f5cc` VMI and `eth1` corresponds to `vn1-pod-b03c198` VMI.

```sh
pradeep@CN2 % kubectl get vmi
CLUSTERNAME                   NAME                     NETWORK              PODNAME         IFCNAME   STATE     AGE
contrail-k8s-kubemanager-mk   another-nginx-a2db2a41   default-podnetwork   another-nginx   eth0      Success   9h
contrail-k8s-kubemanager-mk   nginx-3fa02bbb           default-podnetwork   nginx           eth0      Success   26h
contrail-k8s-kubemanager-mk   vn1-pod-9789f5cc         default-podnetwork   vn1-pod         eth0      Success   6m41s
contrail-k8s-kubemanager-mk   vn1-pod-b03c1981         vn1                  vn1-pod         eth1      Success   6m41s
pradeep@CN2 % 
```

## InstanceIP
```sh
pradeep@CN2 % kubectl explain iip
KIND:     InstanceIP
VERSION:  core.contrail.juniper.net/v1alpha1

DESCRIPTION:
     InstanceIP represents an IP address and its configuration used for
     interfaces.

{trimmed}

pradeep@CN2 % 

```

We can see that the `vn1-pod` has two IP addresses, `10.244.0.5` and `172.16.10.2`.

```sh
pradeep@CN2 % kubectl get iip            
NAME                                                            IPADDRESS       NETWORK                                                       STATE     AGE
contrail-k8s-kubemanager-mk-another-nginx-65f2e48a              10.244.0.4      contrail-k8s-kubemanager-mk-contrail/default-podnetwork       Success   9h
contrail-k8s-kubemanager-mk-contrail-api-5a7e6fd4               10.97.67.172    contrail-k8s-kubemanager-mk-contrail/default-servicenetwork   Success   26h
contrail-k8s-kubemanager-mk-coredns-558bd4d5db-r6nck-81396e95   10.244.0.2      contrail-k8s-kubemanager-mk-contrail/default-podnetwork       Success   26h
contrail-k8s-kubemanager-mk-frontend-55dee962                   10.108.75.218   contrail-k8s-kubemanager-mk-contrail/default-servicenetwork   Success   9h
contrail-k8s-kubemanager-mk-kube-dns-e0c7a4a3                   10.96.0.10      contrail-k8s-kubemanager-mk-contrail/default-servicenetwork   Success   26h
contrail-k8s-kubemanager-mk-nginx-9e2dcf80                      10.244.0.3      contrail-k8s-kubemanager-mk-contrail/default-podnetwork       Success   26h
contrail-k8s-kubemanager-mk-vn1-pod-2d7e67a3                    10.244.0.5      contrail-k8s-kubemanager-mk-contrail/default-podnetwork       Success   6m20s
contrail-k8s-kubemanager-mk-vn1-pod-f7921747                    172.16.10.2     default/vn1                                                   Success   6m20s
pradeep@CN2 %
```

## RouteTarget

```sh
pradeep@CN2 % kubectl explain rt
KIND:     RouteTarget
VERSION:  core.contrail.juniper.net/v1alpha1

DESCRIPTION:
     RouteTarget is a route-target extended community, a type of BGP extended
     community that used to define VPN membership. The route target appears in a
     field in route update.

{trimmed}

pradeep@CN2 % 
```
Look at the new RTs that got added to the list, when we created the NetworkAttachmentDefinition.
```sh
pradeep@CN2 % kubectl get rt
NAME                   STATE     AGE
target-10.1.1.1-561    Success   8h
target-10.2.2.2-561    Success   8h
target-23-4561         Success   8h
target-64512-8000000   Success   34h
target-64512-8000001   Success   34h
target-64512-8000002   Success   34h
target-64512-8000003   Success   34h
target-64512-8000004   Success   34h
target-64512-8000005   Success   34h
target-64512-8000006   Success   34h
target-64512-8000007   Success   34h
target-64512-8000008   Success   34h
target-64512-8000009   Success   8h
pradeep@CN2 %
```

```sh
pradeep@CN2 % kubectl describe rt target-64512-8000009
Name:         target-64512-8000009
Namespace:    
Labels:       core.juniper.net/systemrt=true
Annotations:  <none>
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         RouteTarget
Metadata:
  Creation Timestamp:  2022-08-19T20:58:40Z
  Generation:          1
  Managed Fields:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .:
          f:core.juniper.net/systemrt:
      f:status:
        f:state:
    Manager:         manager
    Operation:       Update
    Time:            2022-08-19T20:58:40Z
  Resource Version:  118095
  UID:               195f860d-2c99-4f57-a574-8970c57d823a
Spec:
  Fq Name:
    target-64512-8000009
Status:
  Observation:  
  State:        Success
Events:         <none>
pradeep@CN2 %
```

## Juniper API Resources

Let us look at the list of  Juniper specific API resources, and explore some of them.

```sh
pradeep@CN2 % kubectl api-resources| grep juniper
apiservers                                         configplane.juniper.net/v1alpha1            true         ApiServer
controllers                                        configplane.juniper.net/v1alpha1            true         Controller
kubemanagers                                       configplane.juniper.net/v1alpha1            true         Kubemanager
controls                                           controlplane.juniper.net/v1alpha1           true         Control
addressgroups                     ag               core.contrail.juniper.net/v1alpha1          true         AddressGroup
applicationpolicysets             aps              core.contrail.juniper.net/v1alpha1          true         ApplicationPolicySet
bgpasaservices                    bgpaas           core.contrail.juniper.net/v1alpha1          true         BGPAsAService
bgprouters                        br               core.contrail.juniper.net/v1alpha1          true         BGPRouter
firewallpolicies                  fp               core.contrail.juniper.net/v1alpha1          true         FirewallPolicy
firewallrules                     fr               core.contrail.juniper.net/v1alpha1          true         FirewallRule
floatingips                       fip              core.contrail.juniper.net/v1alpha1          false        FloatingIP
globalsystemconfigs               gsc              core.contrail.juniper.net/v1alpha1          false        GlobalSystemConfig
globalvrouterconfigs              gvc              core.contrail.juniper.net/v1alpha1          false        GlobalVrouterConfig
instanceips                       iip              core.contrail.juniper.net/v1alpha1          false        InstanceIP
mirrordestinations                md               core.contrail.juniper.net/v1alpha1          false        MirrorDestination
routetargets                      rt               core.contrail.juniper.net/v1alpha1          false        RouteTarget
routinginstances                  ri               core.contrail.juniper.net/v1alpha1          true         RoutingInstance
subnets                           sn               core.contrail.juniper.net/v1alpha1          true         Subnet
tags                              t                core.contrail.juniper.net/v1alpha1          false        Tag
tagtypes                          tt               core.contrail.juniper.net/v1alpha1          false        TagType
virtualmachineinterfaces          vmi              core.contrail.juniper.net/v1alpha1          true         VirtualMachineInterface
virtualmachines                   vm               core.contrail.juniper.net/v1alpha1          false        VirtualMachine
virtualnetworkrouters             vnr              core.contrail.juniper.net/v1alpha1          true         VirtualNetworkRouter
virtualnetworks                   vn               core.contrail.juniper.net/v1alpha1          true         VirtualNetwork
virtualrouters                    vr               core.contrail.juniper.net/v1alpha1          false        VirtualRouter
vrouters                                           dataplane.juniper.net/v1alpha1              true         Vrouter
pools                                              idallocator.contrail.juniper.net/v1alpha1   false        Pool
pradeep@CN2 %
```


## VirtualNetworkRouter

```sh
pradeep@CN2 % kubectl explain vnr
KIND:     VirtualNetworkRouter
VERSION:  core.contrail.juniper.net/v1alpha1

DESCRIPTION:
     VirtualNetworkRouter establishes connectivity between two or more
     VirtualNetworks

{trimmed}

pradeep@CN2 % 
```

```sh
pradeep@CN2 % kubectl get vnr                    
No resources found in default namespace.
pradeep@CN2 % kubectl get vnr -A
NAMESPACE                              NAME                               TYPE    STATE     AGE
contrail-k8s-kubemanager-mk-contrail   DefaultPodServiceIPFabricNetwork   spoke   Success   34h
contrail-k8s-kubemanager-mk-contrail   DefaultPodServiceNetwork           mesh    Success   34h
contrail-k8s-kubemanager-mk-contrail   DefaultServiceNetwork              hub     Success   34h
contrail                               DefaultIPFabricNetwork             hub     Success   34h
pradeep@CN2 % 
```

```sh
pradeep@CN2 % kubectl describe vnr DefaultServiceNetwork -n contrail-k8s-kubemanager-mk-contrail 
Name:         DefaultServiceNetwork
Namespace:    contrail-k8s-kubemanager-mk-contrail
Labels:       core.juniper.net/virtualnetworkrouter=default-service-network
Annotations:  core.juniper.net/description: Enables connectivity between Default Service virtualnetwork and Pod virtualnetwork of isolated-namespace
              core.juniper.net/display-name: DefaultServiceNetwork
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         VirtualNetworkRouter
Metadata:
  Creation Timestamp:  2022-08-18T18:49:51Z
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
          f:core.juniper.net/description:
          f:core.juniper.net/display-name:
        f:finalizers:
          .:
          v:"remove-vnr-rt-from-vn-ri.finalizers.core.juniper.net":
          v:"vnr-routinginstance-delete.finalizers.core.juniper.net":
        f:labels:
          .:
          f:core.juniper.net/virtualnetworkrouter:
      f:spec:
        f:import:
          f:virtualNetworkRouters:
        f:type:
        f:virtualNetworkSelector:
      f:status:
        f:observation:
        f:state:
    Manager:         manager
    Operation:       Update
    Time:            2022-08-18T18:50:09Z
  Resource Version:  32756
  UID:               3f6040dd-180d-4203-b309-d0b0ac2b6f60
Spec:
  Fq Name:
    default-domain
    contrail-k8s-kubemanager-mk-contrail
    DefaultServiceNetwork
  Import:
    Virtual Network Routers:
      Namespace Selector:
        Match Labels:
          core.juniper.net/clusterName:         contrail-k8s-kubemanager-mk
          core.juniper.net/isolated-namespace:  true
      Virtual Network Router Selector:
        Match Labels:
          core.juniper.net/virtualnetworkrouter:  isolated-namespace-pod-to-default-service-network
  Type:                                           hub
  Virtual Network Selector:
    Match Labels:
      core.juniper.net/virtualnetwork:  default-servicenetwork
Status:
  Observation:  
  State:        Success
Events:         <none>
pradeep@CN2 % 
```

## VirtualRouter

```sh
pradeep@CN2 % kubectl explain vr 
KIND:     VirtualRouter
VERSION:  core.contrail.juniper.net/v1alpha1

DESCRIPTION:
     VirtualRouter is packet forwarding system on devices such as compute
     nodes(servers), TOR(s), routers.

{trimmed}

pradeep@CN2 % 
```


```sh
pradeep@CN2 % kubectl get vr -A
NAME       IP               STATE     AGE
minikube   192.168.177.47   Success   34h
```

```sh
pradeep@CN2 % kubectl describe vr minikube                   
Name:         minikube
Namespace:    
Labels:       back-reference.core.juniper.net/2cc848ce973a7ac114af1c01ebc2dd82483d61789f7786459d2f6dec=VirtualMachine_contrail-k8s-kubemanager-mk-coredns-558bd4d5db-r
              back-reference.core.juniper.net/86c78a80e1213ad90936a33375cf4679fe7e1ad0b59d86f6d077008e=VirtualMachine_contrail-k8s-kubemanager-mk-nginx-438edddb
              back-reference.core.juniper.net/d397644e914cda345305dac815aa9c66aed09997d57c40ad94881e16=VirtualMachine_contrail-k8s-kubemanager-mk-vn1-pod-c478b06e
              back-reference.core.juniper.net/f20791429bbba720910cd0837a35c16ea97178da5f61555f01e4d88a=VirtualMachine_contrail-k8s-kubemanager-mk-another-nginx-3e420c
              core.juniper.net/clusterName=
              core.juniper.net/parent=9b83eddfaaa5778ad6b99cb81c803529cf911d492b9e7ec6d63d029d
              core.juniper.net/virtualRouter=deployer-generated
Annotations:  <none>
API Version:  core.contrail.juniper.net/v1alpha1
Kind:         VirtualRouter
Metadata:
  Creation Timestamp:  2022-08-18T18:53:34Z
  Generation:          15
  Managed Fields:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .:
          f:core.juniper.net/clusterName:
          f:core.juniper.net/virtualRouter:
      f:spec:
        f:parent:
          f:apiVersion:
          f:kind:
          f:name:
          f:uid:
        f:virtualRouterIPAddress:
      f:status:
        f:observation:
        f:state:
    Manager:      manager
    Operation:    Update
    Time:         2022-08-18T18:53:34Z
    API Version:  core.contrail.juniper.net/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        f:virtualMachineReferences:
          .:
          k:{"uid":"436da684-aeec-4a5b-a5bc-4fa9fdd9bfb4"}:
            .:
            f:apiVersion:
            f:kind:
            f:name:
            f:resourceVersion:
            f:uid:
          k:{"uid":"6ff657cf-e363-4f0a-bae1-c1c94fa083e3"}:
            .:
            f:apiVersion:
            f:kind:
            f:name:
            f:resourceVersion:
            f:uid:
          k:{"uid":"71748541-b2d5-4b1d-bcb1-54f11dacc799"}:
            .:
            f:apiVersion:
            f:kind:
            f:name:
            f:resourceVersion:
            f:uid:
          k:{"uid":"e6c76e3e-015d-41f1-824e-5f3fea21e7dc"}:
            .:
            f:apiVersion:
            f:kind:
            f:name:
            f:resourceVersion:
            f:uid:
    Manager:         kubemanager
    Operation:       Update
    Time:            2022-08-19T21:00:16Z
  Resource Version:  118391
  UID:               9331a6c4-10bd-45e6-98eb-10e87355f4c6
Spec:
  Fq Name:
    default-global-system-config
    minikube
  Parent:
    API Version:  core.contrail.juniper.net/v1alpha1
    Kind:         GlobalSystemConfig
    Name:         default-global-system-config
    UID:          d70e3729-7205-45f6-94e9-417bb5f5cdff
  Virtual Machine References:
    API Version:  core.contrail.juniper.net/v1alpha1
    Fq Name:
      contrail-k8s-kubemanager-mk-coredns-558bd4d5db-r6nck-9cb64c64
    Kind:              VirtualMachine
    Name:              contrail-k8s-kubemanager-mk-coredns-558bd4d5db-r6nck-9cb64c64
    Resource Version:  2059
    UID:               6ff657cf-e363-4f0a-bae1-c1c94fa083e3
    API Version:       core.contrail.juniper.net/v1alpha1
    Fq Name:
      contrail-k8s-kubemanager-mk-nginx-438edddb
    Kind:              VirtualMachine
    Name:              contrail-k8s-kubemanager-mk-nginx-438edddb
    Resource Version:  3133
    UID:               436da684-aeec-4a5b-a5bc-4fa9fdd9bfb4
    API Version:       core.contrail.juniper.net/v1alpha1
    Fq Name:
      contrail-k8s-kubemanager-mk-another-nginx-3e420ca1
    Kind:              VirtualMachine
    Name:              contrail-k8s-kubemanager-mk-another-nginx-3e420ca1
    Resource Version:  53129
    UID:               e6c76e3e-015d-41f1-824e-5f3fea21e7dc
    API Version:       core.contrail.juniper.net/v1alpha1
    Fq Name:
      contrail-k8s-kubemanager-mk-vn1-pod-c478b06e
    Kind:                     VirtualMachine
    Name:                     contrail-k8s-kubemanager-mk-vn1-pod-c478b06e
    Resource Version:         118371
    UID:                      71748541-b2d5-4b1d-bcb1-54f11dacc799
  Virtual Router IP Address:  192.168.177.47
Status:
  Observation:  
  State:        Success
Events:         <none>
pradeep@CN2 %
```

## KUBEMANAGER_ENABLE_NAD

The network attachment definition controller is part of the `kube-manager` object. 

If we describe the `kubemanager` pod in the `contrail` namespace, we can see the
`KUBEMANAGER_ENABLE_NAD:true` environment variable.

```sh
pradeep@CN2 % kubectl describe  pod contrail-k8s-kubemanager-ccc4dcd66-v968l -n contrail
Name:         contrail-k8s-kubemanager-ccc4dcd66-v968l
Namespace:    contrail
Priority:     0
Node:         minikube/192.168.177.47
Start Time:   Fri, 19 Aug 2022 00:19:51 +0530
Labels:       app=contrail-k8s-kubemanager
              pod-template-hash=ccc4dcd66
Annotations:  <none>
Status:       Running
IP:           192.168.177.47
IPs:
  IP:           192.168.177.47
Controlled By:  ReplicaSet/contrail-k8s-kubemanager-ccc4dcd66
Containers:
  contrail-k8s-kubemanager:
    Container ID:  cri-o://f7ee766203c41e93804db782f810c7f1a9a1fa0108930334e7a7f8ef097358cd
    Image:         hub.juniper.net/cn2/contrail-k8s-kubemanager:22.1.0.93
    Image ID:      hub.juniper.net/cn2/contrail-k8s-kubemanager@sha256:c50386f24304077035c4380bff52163561d178534711077802afd30b17665fc1
    Port:          19445/TCP
    Host Port:     19445/TCP
    Command:
      sh
      -c
      /kubemanager --metrics-addr :0 -enable-leader-election
    State:          Running
      Started:      Mon, 22 Aug 2022 09:31:01 +0530
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Mon, 22 Aug 2022 09:30:47 +0530
      Finished:     Mon, 22 Aug 2022 09:30:58 +0530
    Ready:          True
    Restart Count:  8
    Environment:
      KUBEMANAGER_CLUSTER_NAME:                     contrail-k8s-kubemanager-mk
      KUBEMANAGER_CLUSTER_POD_VIRTUAL_NETWORK:      default-podnetwork
      KUBEMANAGER_CLUSTER_SERVICE_VIRTUAL_NETWORK:  default-servicenetwork
      KUBEMANAGER_CLUSTER_PROJECT:                  contrail-k8s-kubemanager-mk-contrail
      KUBEMANAGER_CLUSTER_NAMESPACE:                contrail-k8s-kubemanager-mk-contrail
      KUBEMANAGER_ENABLE_NAD:                       true
      KUBEMANAGER_EXTERNAL_NETWORK_SELECTORS:       {"default-external":{"networkSelector":{"matchLabels":{"service.contrail.juniper.net/externalNetworkSelector":"default-external"}},"namespaceSelector":{}}}
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-sql6q (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-sql6q:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              node-role.kubernetes.io/master=
Tolerations:                 :NoSchedule op=Exists
                             :NoExecute op=Exists
Events:
  Type     Reason            Age                   From             Message
  ----     ------            ----                  ----             -------
  Warning  FailedMount       44h                   kubelet          MountVolume.SetUp failed for volume "kube-api-access-sql6q" : [failed to fetch token: the server was unable to return a response in the time allotted, but may still be processing the request (post serviceaccounts contrail-serviceaccount), failed to sync configmap cache: timed out waiting for the condition]
  Warning  FailedMount       41h                   kubelet          MountVolume.SetUp failed for volume "kube-api-access-sql6q" : [failed to fetch token: Post "https://control-plane.minikube.internal:8443/api/v1/namespaces/contrail/serviceaccounts/contrail-serviceaccount/token": http2: client connection force closed via ClientConn.Close, failed to sync configmap cache: timed out waiting for the condition]
  Warning  FailedMount       41h                   kubelet          MountVolume.SetUp failed for volume "kube-api-access-sql6q" : [failed to fetch token: Post "https://control-plane.minikube.internal:8443/api/v1/namespaces/contrail/serviceaccounts/contrail-serviceaccount/token": read tcp 192.168.177.47:57306->192.168.177.47:8443: use of closed network connection, failed to sync configmap cache: timed out waiting for the condition]
  Warning  FailedMount       37h                   kubelet          MountVolume.SetUp failed for volume "kube-api-access-sql6q" : [failed to fetch token: Post "https://control-plane.minikube.internal:8443/api/v1/namespaces/contrail/serviceaccounts/contrail-serviceaccount/token": read tcp 192.168.177.47:57452->192.168.177.47:8443: use of closed network connection, failed to sync configmap cache: timed out waiting for the condition]
  Warning  NodeNotReady      37h                   node-controller  Node is not ready
  Normal   Pulled            7m8s (x8 over 47h)    kubelet          Container image "hub.juniper.net/cn2/contrail-k8s-kubemanager:22.1.0.93" already present on machine
  Normal   Created           7m6s (x9 over 47h)    kubelet          Created container contrail-k8s-kubemanager
  Normal   Started           7m6s (x9 over 47h)    kubelet          Started container contrail-k8s-kubemanager
  Warning  DNSConfigForming  2m14s (x14 over 44h)  kubelet          Nameserver limits were exceeded, some nameservers have been omitted, the applied nameserver line is: 1.1.1.1 8.8.8.8 1.0.0.1
pradeep@CN2 % 
```
If you delete a virtual network that was created by the network attachment  definition resource, The network attachment definition controller reconciles the issue and recreates the virtual network.

```sh
pradeep@CN2 % kubectl get vn
NAME   VNI   IP FAMILIES   STATE     AGE
vn1    5     v4            Success   2d7h
```

```sh
pradeep@CN2 % kubectl delete vn vn1 
virtualnetwork.core.contrail.juniper.net "vn1" deleted
```
```sh
pradeep@CN2 % kubectl get vn       
NAME   VNI   IP FAMILIES   STATE     AGE
vn1    5     v4            Success   3s
pradeep@CN2 %
```

We can see that the virtual network got re-created automatically after deletion.

If you delete a network attachment definition resource, The associated VirtualNetwork object is also deleted.

```sh
pradeep@CN2 % kubectl get net-attach-def
NAME   AGE
vn1    2d7h
pradeep@CN2 % kubectl get net-attach-def,vn,subnet
NAME                                              AGE
networkattachmentdefinition.k8s.cni.cncf.io/vn1   2d7h

NAME                                           VNI   IP FAMILIES   STATE     AGE
virtualnetwork.core.contrail.juniper.net/vn1   5     v4            Success   2m45s

NAME                                      CIDR             USAGE   STATE     AGE
subnet.core.contrail.juniper.net/vn1-v4   172.16.10.0/24   1.56%   Success   2d7h
pradeep@CN2 % 
```

```sh
pradeep@CN2 % kubectl delete net-attach-def vn1   
networkattachmentdefinition.k8s.cni.cncf.io "vn1" deleted
pradeep@CN2 %
```

```sh
pradeep@CN2 % kubectl get net-attach-def,vn,subnet
No resources found in default namespace.
pradeep@CN2 % 
```


This concludes our discussion on one of the main features of CN2, enabling Pods to have multiple network interfaces.



