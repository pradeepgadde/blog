---
layout: single
title:  "Kubectl Describe and Explain"
date:   2022-02-04 10:55:04 +0530
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

# Kubectl describe and explain

### Describe Resources

To see details of a specific resource, we can make use of `kubectl describe` command.

Here is the help output.

```sh
pradeep@learnk8s$ kubectl describe -h
Show details of a specific resource or group of resources.

 Print a detailed description of the selected resources, including related resources such as events
or controllers. You may select a single object by name, all objects of that type, provide a name
prefix, or label selector. For example:

  $ kubectl describe TYPE NAME_PREFIX
  
 will first check for an exact match on TYPE and NAME_PREFIX. If no such resource exists, it will
output details for every resource that has a name prefixed with NAME_PREFIX.

Use "kubectl api-resources" for a complete list of supported resources.

Examples:
  # Describe a node
  kubectl describe nodes kubernetes-node-emt8.c.myproject.internal
  
  # Describe a pod
  kubectl describe pods/nginx
  
  # Describe a pod identified by type and name in "pod.json"
  kubectl describe -f pod.json
  
  # Describe all pods
  kubectl describe pods
  
  # Describe pods by label name=myLabel
  kubectl describe po -l name=myLabel
  
  # Describe all pods managed by the 'frontend' replication controller (rc-created pods
  # get the name of the rc as a prefix in the pod the name)
  kubectl describe pods frontend

Options:
  -A, --all-namespaces=false: If present, list the requested object(s) across all namespaces.
Namespace in current context is ignored even if specified with --namespace.
      --chunk-size=500: Return large lists in chunks rather than all at once. Pass 0 to disable.
This flag is beta and may change in the future.
  -f, --filename=[]: Filename, directory, or URL to files containing the resource to describe
  -k, --kustomize='': Process the kustomization directory. This flag can't be used together with -f
or -R.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you
want to manage related manifests organized within the same directory.
  -l, --selector='': Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l
key1=value1,key2=value2)
      --show-events=true: If true, display events related to the described object.

Usage:
  kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | -l label] | TYPE/NAME) [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

Let us describe a pod named `nginx`

```shell
pradeep@learnk8s$ kubectl describe pods nginx
Name:         nginx
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.28
Start Time:   Fri, 04 Feb 2022 07:13:50 +0530
Labels:       new-label=awesome
              run=nginx
Annotations:  <none>
Status:       Running
IP:           10.244.1.6
IPs:
  IP:  10.244.1.6
Containers:
  nginx:
    Container ID:   docker://32bfdccc6c984f51a4c4092e4027ff7f1e53dd518938196c95d5427a44a90e40
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 04 Feb 2022 07:13:56 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9sstv (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-9sstv:
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
  Normal  Scheduled  18m   default-scheduler  Successfully assigned default/nginx to k8s-m02
  Normal  Pulling    18m   kubelet            Pulling image "nginx"
  Normal  Pulled     18m   kubelet            Successfully pulled image "nginx" in 4.864287771s
  Normal  Created    18m   kubelet            Created container nginx
  Normal  Started    18m   kubelet            Started container nginx
```

### Kubectl Explain!

We can make use of the `kubectl explain` command to find out all the fields associated with an API resource like Pods, Deployments, Services etc.

Here is the help output.

```sh
pradeep@learnk8s$ kubectl explain -h
List the fields for supported resources.

 This command describes the fields associated with each supported API resource. Fields are
identified via a simple JSONPath identifier:

  <type>.<fieldName>[.<fieldName>]
  
 Add the --recursive flag to display all of the fields at once without descriptions. Information
about each field is retrieved from the server in OpenAPI format.

Use "kubectl api-resources" for a complete list of supported resources.

Examples:
  # Get the documentation of the resource and its fields
  kubectl explain pods
  
  # Get the documentation of a specific field of a resource
  kubectl explain pods.spec.containers

Options:
      --api-version='': Get different explanations for particular API version (API group/version)
      --recursive=false: Print the fields of fields (Currently only 1 level deep)

Usage:
  kubectl explain RESOURCE [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

Let us try `kubectl explain` for a Pod Specification

```shell
pradeep@learnk8s$ kubectl explain pod.spec
KIND:     Pod
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

     PodSpec is a description of a pod.

FIELDS:
   <SNIP>
   containers	<[]Object> -required-
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.
   <SNIP>
    nodeName	<string>
     NodeName is a request to schedule this pod onto a specific node. If it is
     non-empty, the scheduler simply schedules this pod onto that node, assuming
     that it fits resource requirements.
   <SNIP>  
```

