---
layout: single
title:  "Kubeadm Tokens and Printing Join Command"
date:   2022-04-04 10:55:04 +0530
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

# Kubeadm Token Print Join Command

We have seen that `kubeadm init` command shows a token to be used while joining any other worker nodes with that cluster. If you did not make note of the token shown, we can generate one again and use that for joining new nodes.

Here is related help and procedure to get a new token.

```sh
lab@k8s1:~$ kubeadm token -h

This command manages bootstrap tokens. It is optional and needed only for advanced use cases.

In short, bootstrap tokens are used for establishing bidirectional trust between a client and a server.
A bootstrap token can be used when a client (for example a node that is about to join the cluster) needs
to trust the server it is talking to. Then a bootstrap token with the "signing" usage can be used.
bootstrap tokens can also function as a way to allow short-lived authentication to the API Server
(the token serves as a way for the API Server to trust the client), for example for doing the TLS Bootstrap.

What is a bootstrap token more exactly?
 - It is a Secret in the kube-system namespace of type "bootstrap.kubernetes.io/token".
 - A bootstrap token must be of the form "[a-z0-9]{6}.[a-z0-9]{16}". The former part is the public token ID,
   while the latter is the Token Secret and it must be kept private at all circumstances!
 - The name of the Secret must be named "bootstrap-token-(token-id)".

You can read more about bootstrap tokens here:
  https://kubernetes.io/docs/admin/bootstrap-tokens/

Usage:
  kubeadm token [flags]
  kubeadm token [command]

Available Commands:
  create      Create bootstrap tokens on the server
  delete      Delete bootstrap tokens on the server
  generate    Generate and print a bootstrap token, but do not create it on the server
  list        List bootstrap tokens on the server

Flags:
      --dry-run             Whether to enable dry-run mode or not
  -h, --help                help for token
      --kubeconfig string   The kubeconfig file to use when talking to the cluster. If the flag is not set, a set of standard locations can be searched for an existing kubeconfig file. (default "/etc/kubernetes/admin.conf")

Global Flags:
      --add-dir-header           If true, adds the file directory to the header of the log messages
      --log-file string          If non-empty, use this log file
      --log-file-max-size uint   Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)
      --one-output               If true, only write logs to their native severity level (vs also writing to each lower severity level)
      --rootfs string            [EXPERIMENTAL] The path to the 'real' host root filesystem.
      --skip-headers             If true, avoid header prefixes in the log messages
      --skip-log-headers         If true, avoid headers when opening log files
  -v, --v Level                  number for the log level verbosity

Use "kubeadm token [command] --help" for more information about a command.
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubeadm token create -h

This command will create a bootstrap token for you.
You can specify the usages for this token, the "time to live" and an optional human friendly description.

The [token] is the actual token to write.
This should be a securely generated random token of the form "[a-z0-9]{6}.[a-z0-9]{16}".
If no [token] is given, kubeadm will generate a random token instead.

Usage:
  kubeadm token create [token]

Flags:
      --certificate-key string   When used together with '--print-join-command', print the full 'kubeadm join' flag needed to join the cluster as a control-plane. To create a new certificate key you must use 'kubeadm init phase upload-certs --upload-certs'.
      --config string            Path to a kubeadm configuration file.
      --description string       A human friendly description of how this token is used.
      --groups strings           Extra groups that this token will authenticate as when used for authentication. Must match "\\Asystem:bootstrappers:[a-z0-9:-]{0,255}[a-z0-9]\\z" (default [system:bootstrappers:kubeadm:default-node-token])
  -h, --help                     help for create
      --print-join-command       Instead of printing only the token, print the full 'kubeadm join' flag needed to join the cluster using the token.
      --ttl duration             The duration before the token is automatically deleted (e.g. 1s, 2m, 3h). If set to '0', the token will never expire (default 24h0m0s)
      --usages strings           Describes the ways in which this token can be used. You can pass --usages multiple times or provide a comma separated list of options. Valid options: [signing,authentication] (default [signing,authentication])

Global Flags:
      --add-dir-header           If true, adds the file directory to the header of the log messages
      --dry-run                  Whether to enable dry-run mode or not
      --kubeconfig string        The kubeconfig file to use when talking to the cluster. If the flag is not set, a set of standard locations can be searched for an existing kubeconfig file. (default "/etc/kubernetes/admin.conf")
      --log-file string          If non-empty, use this log file
      --log-file-max-size uint   Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)
      --one-output               If true, only write logs to their native severity level (vs also writing to each lower severity level)
      --rootfs string            [EXPERIMENTAL] The path to the 'real' host root filesystem.
      --skip-headers             If true, avoid header prefixes in the log messages
      --skip-log-headers         If true, avoid headers when opening log files
  -v, --v Level                  number for the log level verbosity
lab@k8s1:~$
```


```sh
lab@k8s1:~$ kubeadm token create --print-join-command
kubeadm join 192.168.100.1:6443 --token zcxly9.hqnls2aws89j1c25 --discovery-token-ca-cert-hash sha256:45ea28506b18239a67e90374fcd186d9de899f33861be7d7b5f1873f1c49bb6f
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubeadm token list
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
zcxly9.hqnls2aws89j1c25   23h         2022-04-06T01:30:50Z   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
lab@k8s1:~$
```

