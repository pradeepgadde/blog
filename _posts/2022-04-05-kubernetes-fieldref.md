---
layout: single
title:  "Kubernetes Expose Pod Information to Containers using ENV Variables"
date:   2022-04-04 10:58:04 +0530
categories: Kubernetes
tags: kubeadm minikube
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


# Expose Pod Information to Containers Through Environment Variables

We can pass values to environment variables using `fieldRef` and `fieldPath`. The `fieldPath` will be taking the JsonPath as input.

```yaml
pradeep@learnk8s$ cat podinfo-as-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: fieldref-demo
spec:
  containers:
    - name: field-ref-container
      image: k8s.gcr.io/busybox
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_NODE_NAME MY_POD_NAME MY_POD_NAMESPACE;
          printenv MY_POD_IP MY_POD_SERVICE_ACCOUNT;
          sleep 10;
        done;
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: MY_POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
  restartPolicy: Never

```



```sh
pradeep@learnk8s$ kubectl create -f podinfo-as-env.yaml
pod/fieldref-demo created
```

```sh
pradeep@learnk8s$ kubectl get pods
NAME            READY   STATUS    RESTARTS   AGE
fieldref-demo   1/1     Running   0          28s
```

```sh
pradeep@learnk8s$ kubectl describe pods fieldref-demo
Name:         fieldref-demo
Namespace:    default
Priority:     0
Node:         minikube-m02/192.168.177.32
Start Time:   Mon, 04 Apr 2022 22:04:26 +0530
Labels:       <none>
Annotations:  cni.projectcalico.org/containerID: f352909e972ad7f4dc859d602bd0901983d2dbc426ae938f96042c4e8ad5c34d
              cni.projectcalico.org/podIP: 10.244.205.195/32
              cni.projectcalico.org/podIPs: 10.244.205.195/32
Status:       Running
IP:           10.244.205.195
IPs:
  IP:  10.244.205.195
Containers:
  field-ref-container:
    Container ID:  docker://33e3b8f8659bbb967be95a731f5593a9ea4339e49e1fa70a7d977c1f26582e73
    Image:         k8s.gcr.io/busybox
    Image ID:      docker-pullable://k8s.gcr.io/busybox@sha256:d8d3bc2c183ed2f9f10e7258f84971202325ee6011ba137112e01e30f206de67
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
    Args:
      while true; do echo -en '\n'; printenv MY_NODE_NAME MY_POD_NAME MY_POD_NAMESPACE; printenv MY_POD_IP MY_POD_SERVICE_ACCOUNT; sleep 10; done;
    State:          Running
      Started:      Mon, 04 Apr 2022 22:04:52 +0530
    Ready:          True
    Restart Count:  0
    Environment:
      MY_NODE_NAME:             (v1:spec.nodeName)
      MY_POD_NAME:             fieldref-demo (v1:metadata.name)
      MY_POD_NAMESPACE:        default (v1:metadata.namespace)
      MY_POD_IP:                (v1:status.podIP)
      MY_POD_SERVICE_ACCOUNT:   (v1:spec.serviceAccountName)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ff6cw (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-ff6cw:
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
  Normal  Scheduled  34s   default-scheduler  Successfully assigned default/fieldref-demo to minikube-m02
  Normal  Pulling    31s   kubelet            Pulling image "k8s.gcr.io/busybox"
  Normal  Pulled     9s    kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 22.262573897s
  Normal  Created    9s    kubelet            Created container field-ref-container
  Normal  Started    9s    kubelet            Started container field-ref-container
```



```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME            READY   STATUS    RESTARTS   AGE     IP               NODE           NOMINATED NODE   READINESS GATES
fieldref-demo   1/1     Running   0          4m54s   10.244.205.195   minikube-m02   <none>           <none>
```



```sh
pradeep@learnk8s$ kubectl logs fieldref-demo

minikube-m02
fieldref-demo
default
10.244.205.195
default

minikube-m02
fieldref-demo
default
10.244.205.195
default

minikube-m02
fieldref-demo
default
10.244.205.195
default

minikube-m02
fieldref-demo
default
10.244.205.195
default

<output_omitted>
```

