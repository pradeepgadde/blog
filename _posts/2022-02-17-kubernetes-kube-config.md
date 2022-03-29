---
layout: single
title:  "Kubernetes Kube Config"
date:   2022-02-17 10:55:04 +0530
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
# Kubernetes Kube Config


### Kube Config

Kubectl command uses a  configuration file that contains `cluster` and `context` information along with `users`.
This configuration file can be viewed directly using the common commands like `cat`, or `kubectl config view` command. Usually the configuraiton is stored in a filed called `config` in the hidden directory `.kube` inside the user home directory (`/Users/pradeep/` in this case).

Here we can see the same output using both methods.

Using the `kubectl config view` command:

```yaml
pradeep@learnk8s$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/pradeep/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: cluster_info
    server: https://192.168.177.29:8443
  name: k8s
contexts:
- context:
    cluster: k8s
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: context_info
    namespace: default
    user: k8s
  name: k8s
current-context: k8s
kind: Config
preferences: {}
users:
- name: k8s
  user:
    client-certificate: /Users/pradeep/.minikube/profiles/k8s/client.crt
    client-key: /Users/pradeep/.minikube/profiles/k8s/client.key
```
Using the standard `cat` command with full path `Users/pradeep/.kube/config`:

```yaml
pradeep@learnk8s$ cat /Users/pradeep/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/pradeep/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: cluster_info
    server: https://192.168.177.29:8443
  name: k8s
contexts:
- context:
    cluster: k8s
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: context_info
    namespace: default
    user: k8s
  name: k8s
current-context: k8s
kind: Config
preferences: {}
users:
- name: k8s
  user:
    client-certificate: /Users/pradeep/.minikube/profiles/k8s/client.crt
    client-key: /Users/pradeep/.minikube/profiles/k8s/client.key
```

The three important sections from this file are:
Cluster:  `k8s`
Context: `k8s`
User: `k8s`

When we setup Minikube, all of this work is done for us automatically.
We can either `get` or `set` or even `delete` all of these three settings with the `kubectl config` command as seen here.

```shell
pradeep@learnk8s$ kubectl config -h
Modify kubeconfig files using subcommands like "kubectl config set current-context my-context"

 The loading order follows these rules:

  1.  If the --kubeconfig flag is set, then only that file is loaded. The flag may only be set once and no merging takes
place.
  2.  If $KUBECONFIG environment variable is set, then it is used as a list of paths (normal path delimiting rules for
your system). These paths are merged. When a value is modified, it is modified in the file that defines the stanza. When
a value is created, it is created in the first file that exists. If no files in the chain exist, then it creates the
last file in the list.
  3.  Otherwise, ${HOME}/.kube/config is used and no merging takes place.

Available Commands:
  current-context Display the current-context
  delete-cluster  Delete the specified cluster from the kubeconfig
  delete-context  Delete the specified context from the kubeconfig
  delete-user     Delete the specified user from the kubeconfig
  get-clusters    Display clusters defined in the kubeconfig
  get-contexts    Describe one or many contexts
  get-users       Display users defined in the kubeconfig
  rename-context  Rename a context from the kubeconfig file
  set             Set an individual value in a kubeconfig file
  set-cluster     Set a cluster entry in kubeconfig
  set-context     Set a context entry in kubeconfig
  set-credentials Set a user entry in kubeconfig
  unset           Unset an individual value in a kubeconfig file
  use-context     Set the current-context in a kubeconfig file
  view            Display merged kubeconfig settings or a specified kubeconfig file

Usage:
  kubectl config SUBCOMMAND [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

Here is the output for all three `get` commands. While working with multiple clusters, this becomes really useful and we have to keep switching contexts. This is how we work in production environments.

For our little minikube environment also, when we created multiple profiles with the `-p` option (if you forget what it is, refer to the Initial setup section), minikube automatically does this context switching! 

Here is that last line from the `minikube start` command output.

```shell
🏄  Done! kubectl is now configured to use "k8s" cluster and "default" namespace by default 
```

```shell
pradeep@learnk8s$ kubectl config get-users
NAME
k8s
```
```shell
pradeep@learnk8s$ kubectl config get-clusters
NAME
k8s
```
```shell
pradeep@learnk8s$ kubectl config get-contexts
CURRENT   NAME   CLUSTER   AUTHINFO   NAMESPACE
*         k8s    k8s       k8s        default
```


### Kube Config

Kubectl command uses a  configuration file that contains `cluster` and `context` information along with `users`.
This configuration file can be viewed directly using the common commands like `cat`, or `kubectl config view` command. Usually the configuraiton is stored in a filed called `config` in the hidden directory `.kube` inside the user home directory (`/Users/pradeep/` in this case).

Here we can see the same output using both methods.

Using the `kubectl config view` command:

```yaml
pradeep@learnk8s$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/pradeep/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: cluster_info
    server: https://192.168.177.29:8443
  name: k8s
contexts:
- context:
    cluster: k8s
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: context_info
    namespace: default
    user: k8s
  name: k8s
current-context: k8s
kind: Config
preferences: {}
users:
- name: k8s
  user:
    client-certificate: /Users/pradeep/.minikube/profiles/k8s/client.crt
    client-key: /Users/pradeep/.minikube/profiles/k8s/client.key
```
Using the standard `cat` command with full path `Users/pradeep/.kube/config`:

```yaml
pradeep@learnk8s$ cat /Users/pradeep/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/pradeep/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: cluster_info
    server: https://192.168.177.29:8443
  name: k8s
contexts:
- context:
    cluster: k8s
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: context_info
    namespace: default
    user: k8s
  name: k8s
current-context: k8s
kind: Config
preferences: {}
users:
- name: k8s
  user:
    client-certificate: /Users/pradeep/.minikube/profiles/k8s/client.crt
    client-key: /Users/pradeep/.minikube/profiles/k8s/client.key
```

The three important sections from this file are:
Cluster:  `k8s`
Context: `k8s`
User: `k8s`

When we setup Minikube, all of this work is done for us automatically.
We can either `get` or `set` or even `delete` all of these three settings with the `kubectl config` command as seen here.

```shell
pradeep@learnk8s$ kubectl config -h
Modify kubeconfig files using subcommands like "kubectl config set current-context my-context"

 The loading order follows these rules:

  1.  If the --kubeconfig flag is set, then only that file is loaded. The flag may only be set once and no merging takes
place.
  2.  If $KUBECONFIG environment variable is set, then it is used as a list of paths (normal path delimiting rules for
your system). These paths are merged. When a value is modified, it is modified in the file that defines the stanza. When
a value is created, it is created in the first file that exists. If no files in the chain exist, then it creates the
last file in the list.
  3.  Otherwise, ${HOME}/.kube/config is used and no merging takes place.

Available Commands:
  current-context Display the current-context
  delete-cluster  Delete the specified cluster from the kubeconfig
  delete-context  Delete the specified context from the kubeconfig
  delete-user     Delete the specified user from the kubeconfig
  get-clusters    Display clusters defined in the kubeconfig
  get-contexts    Describe one or many contexts
  get-users       Display users defined in the kubeconfig
  rename-context  Rename a context from the kubeconfig file
  set             Set an individual value in a kubeconfig file
  set-cluster     Set a cluster entry in kubeconfig
  set-context     Set a context entry in kubeconfig
  set-credentials Set a user entry in kubeconfig
  unset           Unset an individual value in a kubeconfig file
  use-context     Set the current-context in a kubeconfig file
  view            Display merged kubeconfig settings or a specified kubeconfig file

Usage:
  kubectl config SUBCOMMAND [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

Here is the output for all three `get` commands. While working with multiple clusters, this becomes really useful and we have to keep switching contexts. This is how we work in production environments.

For our little minikube environment also, when we created multiple profiles with the `-p` option (if you forget what it is, refer to the Initial setup section), minikube automatically does this context switching! 

Here is that last line from the `minikube start` command output.

```shell
🏄  Done! kubectl is now configured to use "k8s" cluster and "default" namespace by default 
```

```shell
pradeep@learnk8s$ kubectl config get-users
NAME
k8s
```
```shell
pradeep@learnk8s$ kubectl config get-clusters
NAME
k8s
```
```shell
pradeep@learnk8s$ kubectl config get-contexts
CURRENT   NAME   CLUSTER   AUTHINFO   NAMESPACE
*         k8s    k8s       k8s        default
```
