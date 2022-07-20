---
layout: single
title:  "Kubectl Plugins"
date:   2022-07-20 03:55:04 +0530
categories: Kubernetes
tags: minikube 
classes: wide
show_date: true
header:
  overlay_image: /assets/images/kubernetes.png
  og_image: /assets/images/kubernetes.png
  teaser: /assets/images/kubernetes.png
  caption: "Photo credit: [**Kubernetes**](https://kubernetes.io)"
  actions:
    - label: "Learn more"
      url: "https://kubernetes.io"

author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
 
sidebar:
  - title: "Blog"
 
    text: "Checkout other topics"
    nav: my-sidebar
---

Plugins extend `kubectl` with new sub-commands, allowing for new and custom features not included in the main distribution of kubectl.

A plugin is a standalone executable file, whose name begins with `kubectl-`.

kubectl provides a command `kubectl plugin list` that searches your PATH for valid plugin executables. Executing this command causes a traversal of all files in your PATH. Any files that are executable, and begin with `kubectl-` will show up in the order in which they are present in your PATH in this command's output.

A warning will be included for any files beginning with kubectl- that are not executable. A warning will also be included for any valid plugin files that overlap each other's name.

```sh
(base) pradeep:~$kubectl plugin list
error: unable to find any kubectl plugins in your PATH
(base) pradeep:~$
```

## Writing kubectl plugins

You can write a plugin in any programming language or script that allows you to write command-line commands.

There is no plugin installation or pre-loading required. Plugin executables receive the inherited environment from the `kubectl` binary. A plugin determines which command path it wishes to implement based on its name. For example, a plugin named `kubectl-demo` provides a command `kubectl demo`. You must install the plugin executable somewhere in your `PATH`.

```sh
(base) pradeep:~$cat kubectl-demo 
#!/bin/bash

# optional argument handling
if [[ "$1" == "version" ]]
then
    echo "1.0.0"
    exit 0
fi

# optional argument handling
if [[ "$1" == "config" ]]
then
    echo "$KUBECONFIG"
    exit 0
fi

echo "I am a plugin named kubectl-demo"
(base) pradeep:~$
```

```sh
(base) pradeep:~$chmod +x ./kubectl-demo 
```

```sh
(base) pradeep:~$mv ./kubectl-demo /usr/local/bin 
```

```sh
(base) pradeep:~$kubectl demo
I am a plugin named kubectl-demo
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl demo version
1.0.0
(base) pradeep:~$
```

```sh
(base) pradeep:~$export KUBECONFIG=~/.kube/config
(base) pradeep:~$kubectl demo config             
/Users/pradeep/.kube/config
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl plugin list
The following compatible plugins are available:

/usr/local/bin/kubectl-demo
(base) pradeep:~$
```