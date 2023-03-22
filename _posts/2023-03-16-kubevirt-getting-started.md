---
layout: single
title:  "Getting Started with KubeVirt"
date:   2023-03-16 9:55:04 +0530
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

## Using KubeVirt

Create a Virtual Machine

```sh
(base) pradeep:~$wget https://kubevirt.io/labs/manifests/vm.yaml
less vm.yaml

--2023-03-16 22:42:52--  https://kubevirt.io/labs/manifests/vm.yaml
Resolving kubevirt.io (kubevirt.io)... 185.199.111.153
Connecting to kubevirt.io (kubevirt.io)|185.199.111.153|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 845 [text/yaml]
Saving to: 'vm.yaml'

vm.yaml                                            100%[================================================================================================================>]     845  --.-KB/s    in 0s      

2023-03-16 22:42:52 (9.71 MB/s) - 'vm.yaml' saved [845/845]
```

```sh
(base) pradeep:~$cat vm.yaml 
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: testvm
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/size: small
        kubevirt.io/domain: testvm
    spec:
      domain:
        devices:
          disks:
            - name: containerdisk
              disk:
                bus: virtio
            - name: cloudinitdisk
              disk:
                bus: virtio
          interfaces:
          - name: default
            masquerade: {}
        resources:
          requests:
            memory: 64M
      networks:
      - name: default
        pod: {}
      volumes:
        - name: containerdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          cloudInitNoCloud:
            userDataBase64: SGkuXG4=
(base) pradeep:~$
```

Create a VM

```sh
(base) pradeep:~$kubectl apply -f https://kubevirt.io/labs/manifests/vm.yaml

virtualmachine.kubevirt.io/testvm created
(base) pradeep:~$

```

```sh
(base) pradeep:~$kubectl get vms
NAME     AGE   STATUS    READY
testvm   36s   Stopped   False
(base) pradeep:~$
```

```yaml
(base) pradeep:~$kubectl get vms -o yaml testvm
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"kubevirt.io/v1","kind":"VirtualMachine","metadata":{"annotations":{},"name":"testvm","namespace":"default"},"spec":{"running":false,"template":{"metadata":{"labels":{"kubevirt.io/domain":"testvm","kubevirt.io/size":"small"}},"spec":{"domain":{"devices":{"disks":[{"disk":{"bus":"virtio"},"name":"containerdisk"},{"disk":{"bus":"virtio"},"name":"cloudinitdisk"}],"interfaces":[{"masquerade":{},"name":"default"}]},"resources":{"requests":{"memory":"64M"}}},"networks":[{"name":"default","pod":{}}],"volumes":[{"containerDisk":{"image":"quay.io/kubevirt/cirros-container-disk-demo"},"name":"containerdisk"},{"cloudInitNoCloud":{"userDataBase64":"SGkuXG4="},"name":"cloudinitdisk"}]}}}}
    kubevirt.io/latest-observed-api-version: v1
    kubevirt.io/storage-observed-api-version: v1alpha3
  creationTimestamp: "2023-03-16T17:14:13Z"
  finalizers:
  - kubevirt.io/virtualMachineControllerFinalize
  generation: 1
  name: testvm
  namespace: default
  resourceVersion: "3252"
  uid: e139560f-f095-4b0a-a40b-ed1b3276d7fe
spec:
  running: false
  template:
    metadata:
      creationTimestamp: null
      labels:
        kubevirt.io/domain: testvm
        kubevirt.io/size: small
    spec:
      domain:
        devices:
          disks:
          - disk:
              bus: virtio
            name: containerdisk
          - disk:
              bus: virtio
            name: cloudinitdisk
          interfaces:
          - masquerade: {}
            name: default
        machine:
          type: q35
        resources:
          requests:
            memory: 64M
      networks:
      - name: default
        pod: {}
      volumes:
      - containerDisk:
          image: quay.io/kubevirt/cirros-container-disk-demo
        name: containerdisk
      - cloudInitNoCloud:
          userDataBase64: SGkuXG4=
        name: cloudinitdisk
status:
  conditions:
  - lastProbeTime: "2023-03-16T17:14:13Z"
    lastTransitionTime: "2023-03-16T17:14:13Z"
    message: VMI does not exist
    reason: VMINotExists
    status: "False"
    type: Ready
  printableStatus: Stopped
  volumeSnapshotStatuses:
  - enabled: false
    name: containerdisk
    reason: Snapshot is not supported for this volumeSource type [containerdisk]
  - enabled: false
    name: cloudinitdisk
    reason: Snapshot is not supported for this volumeSource type [cloudinitdisk]
(base) pradeep:~$

```

Start the VM now

```sh
base) pradeep:~$kubectl virt start testvm
VM testvm was scheduled to start
(base) pradeep:~$
```

