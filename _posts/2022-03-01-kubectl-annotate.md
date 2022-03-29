---
layout: single
title:  "Kubectl Annotate"
date:   2022-03-01 10:55:04 +0530
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

# Kubectl annotate


### Kubectl Annotate

Annotations are like labels (key/value pairs), they store arbitrary string values.

```shell
pradeep@learnk8s$ kubectl annotate -h
Update the annotations on one or more resources.

 All Kubernetes objects support the ability to store additional data with the object as annotations. Annotations are
key/value pairs that can be larger than labels and include arbitrary string values such as structured JSON. Tools and
system extensions may use annotations to store their own data.

 Attempting to set an annotation that already exists will fail unless --overwrite is set. If --resource-version is
specified and does not match the current resource version on the server the command will fail.

Use "kubectl api-resources" for a complete list of supported resources.

Examples:
  # Update pod 'foo' with the annotation 'description' and the value 'my frontend'
  # If the same annotation is set multiple times, only the last value will be applied
  kubectl annotate pods foo description='my frontend'

  # Update a pod identified by type and name in "pod.json"
  kubectl annotate -f pod.json description='my frontend'

  # Update pod 'foo' with the annotation 'description' and the value 'my frontend running nginx', overwriting any
existing value
  kubectl annotate --overwrite pods foo description='my frontend running nginx'

  # Update all pods in the namespace
  kubectl annotate pods --all description='my frontend running nginx'

  # Update pod 'foo' only if the resource is unchanged from version 1
  kubectl annotate pods foo description='my frontend running nginx' --resource-version=1

  # Update pod 'foo' by removing an annotation named 'description' if it exists
  # Does not require the --overwrite flag
  kubectl annotate pods foo description-

Options:
      --all=false: Select all resources, in the namespace of the specified resource types.
  -A, --all-namespaces=false: If true, check the specified action in all namespaces.
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or map key is missing in
the template. Only applies to golang and jsonpath output formats.
      --dry-run='none': Must be "none", "server", or "client". If client strategy, only print the object that would be
sent, without sending it. If server strategy, submit server-side request without persisting the resource.
      --field-manager='kubectl-annotate': Name of the manager used to track field ownership.
      --field-selector='': Selector (field query) to filter on, supports '=', '==', and '!='.(e.g. --field-selector
key1=value1,key2=value2). The server only supports a limited number of field queries per type.
  -f, --filename=[]: Filename, directory, or URL to files identifying the resource to update the annotation
  -k, --kustomize='': Process the kustomization directory. This flag can't be used together with -f or -R.
      --list=false: If true, display the annotations for a given resource.
      --local=false: If true, annotation will NOT contact api-server but run locally.
  -o, --output='': Output format. One of:
json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file.
      --overwrite=false: If true, allow annotations to be overwritten, otherwise reject annotation updates that
overwrite existing annotations.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you want to manage
related manifests organized within the same directory.
      --resource-version='': If non-empty, the annotation update will only succeed if this is the current
resource-version for the object. Only valid when specifying a single resource.
  -l, --selector='': Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l
key1=value1,key2=value2).
      --show-managed-fields=false: If true, keep the managedFields when printing objects in JSON or YAML format.
      --template='': Template string or path to template file to use when -o=go-template, -o=go-template-file. The
template format is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

Usage:
  kubectl annotate [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
[options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```
```shell
pradeep@learnk8s$ kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="image updated to 1.21"
deployment.apps/nginx-deployment annotated
```

```shell
pradeep@learnk8s$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         image updated to 1.21
```
Look at the Annotations section of the description.

```shell
pradeep@learnk8s$ kubectl describe deployments.apps nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 01 Mar 2022 06:05:50 +0530
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 2
                        kubernetes.io/change-cause: image updated to 1.21
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.21
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-5778cd94ff (3/3 replicas created)
Events:
  Type     Reason                 Age    From                   Message
  ----     ------                 ----   ----                   -------
  Warning  ReplicaSetCreateError  14m    deployment-controller  Failed to create new replica set "nginx-deployment-7b96fbf5d8": Unauthorized
  Normal   ScalingReplicaSet      14m    deployment-controller  Scaled up replica set nginx-deployment-7b96fbf5d8 to 3
  Normal   ScalingReplicaSet      6m57s  deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 1
  Normal   ScalingReplicaSet      6m51s  deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 2
  Normal   ScalingReplicaSet      6m51s  deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 2
  Normal   ScalingReplicaSet      6m49s  deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 1
  Normal   ScalingReplicaSet      6m49s  deployment-controller  Scaled up replica set nginx-deployment-5778cd94ff to 3
  Normal   ScalingReplicaSet      6m42s  deployment-controller  Scaled down replica set nginx-deployment-7b96fbf5d8 to 0
```

We can see the details of a particular revision using the `--revision`  option.

```shell
pradeep@learnk8s$ kubectl rollout history deployment/nginx-deployment --revision=1
deployment.apps/nginx-deployment with revision #1
Pod Template:
  Labels:	app=nginx
	pod-template-hash=7b96fbf5d8
  Containers:
   nginx:
    Image:	nginx:1.20
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```
```shell
pradeep@learnk8s$ kubectl rollout history deployment/nginx-deployment --revision=2
deployment.apps/nginx-deployment with revision #2
Pod Template:
  Labels:	app=nginx
	pod-template-hash=5778cd94ff
  Annotations:	kubernetes.io/change-cause: image updated to 1.21
  Containers:
   nginx:
    Image:	nginx:1.21
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```
