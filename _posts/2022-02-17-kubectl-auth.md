---
layout: single
title:  "Kubectl Auth"
date:   2022-02-17 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/user-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubectl Auth


### Kubectl Auth

To Inspect authorization, we can make use of `kubectl auth` command. It will show an `yes` or `no`.
Here is the related help and examples.

```shell
pradeep@learnk8s$ kubectl auth can-i -h
Check whether an action is allowed.

 VERB is a logical Kubernetes API verb like 'get', 'list', 'watch', 'delete', etc. TYPE is a Kubernetes resource.
Shortcuts and groups will be resolved. NONRESOURCEURL is a partial URL that starts with "/". NAME is the name of a
particular Kubernetes resource. This command pairs nicely with impersonation. See --as global flag.

Examples:
  # Check to see if I can create pods in any namespace
  kubectl auth can-i create pods --all-namespaces

  # Check to see if I can list deployments in my current namespace
  kubectl auth can-i list deployments.apps

  # Check to see if I can do everything in my current namespace ("*" means all)
  kubectl auth can-i '*' '*'

  # Check to see if I can get the job named "bar" in namespace "foo"
  kubectl auth can-i list jobs.batch/bar -n foo

  # Check to see if I can read pod logs
  kubectl auth can-i get pods --subresource=log

  # Check to see if I can access the URL /logs/
  kubectl auth can-i get /logs/

  # List all allowed actions in namespace "foo"
  kubectl auth can-i --list --namespace=foo

Options:
  -A, --all-namespaces=false: If true, check the specified action in all namespaces.
      --list=false: If true, prints all allowed actions.
      --no-headers=false: If true, prints allowed actions without headers
  -q, --quiet=false: If true, suppress output and just return the exit code.
      --subresource='': SubResource such as pod/log or deployment/scale

Usage:
  kubectl auth can-i VERB [TYPE | TYPE/NAME | NONRESOURCEURL] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

For example, from the `pradeep` context,
```shell
pradeep@learnk8s$ kubectl config get-contexts
CURRENT   NAME      CLUSTER   AUTHINFO   NAMESPACE
          k8s       k8s       k8s        default
*         pradeep   k8s       pradeep
```

```shell
pradeep@learnk8s$ kubectl auth can-i '*' '*'
no
```
From the `k8s` context,
```shell 
pradeep@learnk8s$ kubectl config get-contexts
CURRENT   NAME      CLUSTER   AUTHINFO   NAMESPACE
*         k8s       k8s       k8s        default
          pradeep   k8s       pradeep
```
```shell
pradeep@learnk8s$ kubectl auth can-i '*' '*'
yes
```
