---
layout: single
title:  "Kubernetes Secrets"
date:   2022-02-15 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/secret-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Secrets


### Secrets

Secret holds secret data of a certain type.
```shell
pradeep@learnk8s$ kubectl explain secrets
KIND:     Secret
VERSION:  v1

DESCRIPTION:
     Secret holds secret data of a certain type. The total bytes of the values
     in the Data field must be less than MaxSecretSize bytes.
<SNIP>
```

Verify if there are any secrets present in our cluster already!
```shell
pradeep@learnk8s$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-b2xs6   kubernetes.io/service-account-token   3      17h
```
There is one secret of type `service-account-token`.

Describe it to see additional details of the secret.
```shell
pradeep@learnk8s$ kubectl describe secrets
Name:         default-token-b2xs6
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 5500622d-56e8-47c9-9440-7882c1d35512

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1111 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkJGbThhZWVoY01TV3VDZ2hMU1RMenFlY0o2ckdsY2N3ZjB2ZFl5QWEtQm8ifQ.eyJpc3MiOiJrdWJlck5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VddC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tYjJ4czYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMjaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjU1MDA2MjJkLTU2ZTgtNDdjOS05NDQwLTc4ODJjMWQzNTUxMiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.NGKbj5mw1EsZm4E14zDDpS2vNgL_L0WuGgP7Ex6k2TrXqKaO4wxX-ca3US1iSaUeYOIM4DLnXjDbXbP5J6YV-8ke--Yxhkks8iC_3tA9k1Q0YvK0RXS0T4WsL9i12sD44i-9LoJpL4Zpu3qJO-s4V5Plg9ifC69NpKoEu3CoOTMGmOX7DqDmGotogl5BJHflZekweX8GMOGP5WAv1FkUcROWyhj8wVMEFOzZIS8O8ssF67wHukWuUZ3IYDpCHy3QjQSKOWgeGGfFSDUXfW_mEQCE1ryl_c_-3TK7kGtlqN4l8aKrd2iBF-1MP4mdbq9WKDGBJDlZlGbHgPOZe2eBMw
```
How to create secrets?  Secrets can be of three types in Kubernetes: Docker Registry, Generic TLS.

```shell
pradeep@learnk8s$ kubectl create secret -h
Create a secret using specified subcommand.

Available Commands:
  docker-registry Create a secret for use with a Docker registry
  generic         Create a secret from a local file, directory, or literal value
  tls             Create a TLS secret

Usage:
  kubectl create secret [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

```shell
pradeep@learnk8s$ kubectl create secret generic -h
Create a secret based on a file, directory, or specified literal value.

 A single secret may package one or more key/value pairs.

 When creating a secret based on a file, the key will default to the basename of the file, and the value will default to
the file content. If the basename is an invalid key or you wish to chose your own, you may specify an alternate key.

 When creating a secret based on a directory, each file whose basename is a valid key in the directory will be packaged
into the secret. Any directory entries except regular files are ignored (e.g. subdirectories, symlinks, devices, pipes,
etc).

Examples:
  # Create a new secret named my-secret with keys for each file in folder bar
  kubectl create secret generic my-secret --from-file=path/to/bar

  # Create a new secret named my-secret with specified keys instead of names on disk
  kubectl create secret generic my-secret --from-file=ssh-privatekey=path/to/id_rsa
--from-file=ssh-publickey=path/to/id_rsa.pub

  # Create a new secret named my-secret with key1=supersecret and key2=topsecret
  kubectl create secret generic my-secret --from-literal=key1=supersecret --from-literal=key2=topsecret

  # Create a new secret named my-secret using a combination of a file and a literal
  kubectl create secret generic my-secret --from-file=ssh-privatekey=path/to/id_rsa --from-literal=passphrase=topsecret

  # Create a new secret named my-secret from env files
  kubectl create secret generic my-secret --from-env-file=path/to/foo.env --from-env-file=path/to/bar.env
  <SNIP>
```

Create a secret called `webapp-color-secret` and store the webapp color `blue` in it.
```shell
pradeep@learnk8s$ kubectl create secret generic webapp-color-secret --from-literal=APP_COLOR=blue
secret/webapp-color-secret created
```
```shell
pradeep@learnk8s$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-b2xs6   kubernetes.io/service-account-token   3      18h
webapp-color-secret   Opaque                                1      5s
```
```shell
pradeep@learnk8s$ kubectl describe secrets webapp-color-secret
Name:         webapp-color-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
APP_COLOR:  4 bytes
```
It is time to use this secret in a Pod definition. We use the secrets in the same way we used configmaps. Instead of `configMapRef`, it would be `secretRef` now in the `envFrom` section.

```yaml
pradeep@learnk8s$ cat pod-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kodekloud-secret
spec:
  containers:
    - name: kodekloud-secret
      image: kodekloud/webapp-color:v3
      envFrom:
      - secretRef:
          name: webapp-color-secret
  restartPolicy: Never
```
```shell
pradeep@learnk8s$ kubectl create -f pod-secret.yaml
pod/kodekloud-secret created
```

```shell
pradeep@learnk8s$ kubectl get pods -o wide| grep secret
kodekloud-secret            1/1     Running            0              114s   10.244.1.32   k8s-m02   <none>           <none>
```

To verify the color, login to the minikube node.
```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.1.32:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #2980b9;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-secret!</h1>



  <h2>
    Application Version: v3
  </h2>


</div>$ curl 10.244.1.32:8080/color
blue$ exit
logout
```
We can see that the app is using the `blue` color as defined in the secret.

As another final confirmation, we can describe this pod, to see 
*Environment Variables from:
  webapp-color-secret  Secret  Optional: false*

```shell
pradeep@learnk8s$ kubectl describe pods kodekloud-secret
Name:         kodekloud-secret
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Wed, 16 Feb 2022 06:47:47 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.1.32
IPs:
  IP:  10.244.1.32
Containers:
  kodekloud-secret:
    Container ID:   docker://799356d294bd0e1e6d37143ae8b12babcd9d50410d879ed099032dd4ceaec68a
    Image:          kodekloud/webapp-color:v3
    Image ID:       docker-pullable://kodekloud/webapp-color@sha256:3ecd19b1b85db381a0b6f78272458c3c274ac2a38e878d65700393899adb3177
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 16 Feb 2022 06:47:49 +0530
    Ready:          True
    Restart Count:  0
    Environment Variables from:
      webapp-color-secret  Secret  Optional: false
    Environment:           <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xw2ml (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-xw2ml:
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
  Normal  Scheduled  18s   default-scheduler  Successfully assigned default/kodekloud-secret to k8s-m02
  Normal  Pulled     17s   kubelet            Container image "kodekloud/webapp-color:v3" already present on machine
  Normal  Created    16s   kubelet            Created container kodekloud-secret
  Normal  Started    16s   kubelet            Started container kodekloud-secret
```
