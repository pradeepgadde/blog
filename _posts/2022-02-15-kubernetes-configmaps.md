---
layout: single
title:  "Kubernetes Config Maps"
date:   2022-02-15 10:55:04 +0530
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

# Kubernetes ConfigMaps


### ConfigMaps

As seen in the explanation, ConfigMaps hold the configuration data for pods to consume. For example, we could store the color of the Webapp from the previous example, in the form of a ConfigMap.

```shell
pradeep@learnk8s$ kubectl explain configmaps
KIND:     ConfigMap
VERSION:  v1

DESCRIPTION:
     ConfigMap holds configuration data for pods to consume.
```
Before we create any configmaps, let us verify if we have any configmaps already in our cluster.

```shell
pradeep@learnk8s$ kubectl get configmaps
NAME               DATA   AGE
kube-root-ca.crt   1      7h16m
```
There is one configmap named `kube-root-ca.crt` already present with one DATA.
To see what is stored in this configmap, we can describe it and look at the Annotations.

```shell
pradeep@learnk8s$ kubectl describe configmaps
Name:         kube-root-ca.crt
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/description:
                Contains a CA bundle that can be used to verify the kube-apiserver when using internal endpoints such as the internal service IP or kubern...

Data
====
ca.crt:
----
-----BEGIN CERTIFICATE-----
MIIDBjCCAe6gAwIBAgIBATANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5p
a3ViZUNBMB4XDTIxMDYyODA0NTM1NloXDTMxMDYyNzA0NTM1NlowFTETMBEGA1UE
AxMKbWluaWt1YmVDQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKZu
VYaX2I0BbQ7le2fCQiYL0UaoadaYm0UtRoYHCkGovRH4MBJlJipYp7IVZ+4oOgqZ
VtBC2o9oRzXnWEM6pJyTsrMrEhtDyTxAeCLY5iVwkLRCxrqxvsCVeGsTaF+0SMCy
PUXFaC20jrDVAxAUXs26Le0Fl2BjOv9k9K2hEScRuC0ogkF/oL+aC3BoGpdFmPVG
S+R18kS1UyBGpUWkktyXhtEAmZVdHn+PGeMW2W3cuHyF4OEqQiXE5xcXt6TQG64F
qgmE5xISVDp1/6VjlJeiVMqlyqZVm7dEKdtwcx6p8UKqnr7nL14STkzv6pDywk5v
egCFa7sP+amk41K6VMECAwEAAaNhMF8wDgYDVR0PAQH/BAQDAgKkMB0GA1UdJQQW
MBQGCCsGAQUFBwMCBggrBgEFBQcDATAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQW
BBSL+jGni3bexvU9wL8lBR0FeLmCQDANBgkqhkiG9w0BAQsFAAOCAQEAd/sJdAwa
6hOYM52vQj0CiYfNqFAs1rK7klxzpPVa07sYIO8n8FjyzGTead+FxzeuxnOBxibu
y/447esB/RPE3d7hv+piQqT9FD7H/lskpHyIvffz4ai15P3DFtKQeY464bxVnynQ
eK4dvzySrqxDs5Q5mMC1PQn9ap7VvIRnz1wEr6hlHMqJ31G58rmnZ5V+eJBBUYvG
a39/4Q1PIMyk98T6QmJDTuLQngD8QJagcRMm4D2mzdieomt9jmIsrd5jINMhsC6d
pL/PMobEfOxGsZxSKW9RC4/mabcuo+Dty+xAN/cYlOrq6zGSatvQXu/60iDSx8un
D9XARLPvbRrYnA==
-----END CERTIFICATE-----


BinaryData
====

Events:  <none>
```

We can create configmaps in multiple different ways. Let us use the help option to see some examples.

```shell
pradeep@learnk8s$ kubectl  create configmap -h
Create a config map based on a file, directory, or specified literal value.

 A single config map may package one or more key/value pairs.

 When creating a config map based on a file, the key will default to the basename of the file, and the value will
default to the file content.  If the basename is an invalid key, you may specify an alternate key.

 When creating a config map based on a directory, each file whose basename is a valid key in the directory will be
packaged into the config map.  Any directory entries except regular files are ignored (e.g. subdirectories, symlinks,
devices, pipes, etc).

Aliases:
configmap, cm

Examples:
  # Create a new config map named my-config based on folder bar
  kubectl create configmap my-config --from-file=path/to/bar

  # Create a new config map named my-config with specified keys instead of file basenames on disk
  kubectl create configmap my-config --from-file=key1=/path/to/bar/file1.txt --from-file=key2=/path/to/bar/file2.txt

  # Create a new config map named my-config with key1=config1 and key2=config2
  kubectl create configmap my-config --from-literal=key1=config1 --from-literal=key2=config2

  # Create a new config map named my-config from the key=value pairs in the file
  kubectl create configmap my-config --from-file=path/to/bar

  # Create a new config map named my-config from an env file
  kubectl create configmap my-config --from-env-file=path/to/foo.env --from-env-file=path/to/bar.env
  <SNIP>
```
Using this examples, let us create a configmap named `webapp-color` using `--from-literal` option to set the color to `green`.
```shell
pradeep@learnk8s$ kubectl create configmap webapp-color --from-literal=APP_COLOR=green
configmap/webapp-color created
```

```shell
pradeep@learnk8s$ kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      7h26m
webapp-color       1      5s
```
```shell
pradeep@learnk8s$ kubectl describe cm webapp-color
Name:         webapp-color
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
APP_COLOR:
----
green

BinaryData
====

Events:  <none>
```
To use this configmap in a Pod definition, we need to make use of `-envFrom` and `-configMapRef` options in the Pod Spec.
```yaml
pradeep@learnk8s$ cat pod-config-map.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kodekloud-cm
spec:
  containers:
    - name: kodekloud-cm
      image: kodekloud/webapp-color:v3
      envFrom:
      - configMapRef:
          name: webapp-color
  restartPolicy: Never
```
```shell
pradeep@learnk8s$ kubectl create -f pod-config-map.yaml
pod/kodekloud-cm created
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep cm
kodekloud-cm                1/1     Running            0                106s    10.244.0.14   k8s       <none>           <none>
```
Verify that app color is changed to `green`.
```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.0.14:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #16a085;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-cm!</h1>



  <h2>
    Application Version: v3
  </h2>


</div>$ curl 10.244.0.14:8080/color
green$
$ exit
logout
```
If we describe this pod, we can see the following section.
*Environment Variables from:
      webapp-color  ConfigMap  Optional: false*

```shell
pradeep@learnk8s$ kubectl describe pod kodekloud-cm
Name:         kodekloud-cm
Namespace:    default
Priority:     0
Node:         k8s/192.168.177.29
Start Time:   Tue, 15 Feb 2022 20:09:51 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.0.14
IPs:
  IP:  10.244.0.14
Containers:
  kodekloud-cm:
    Container ID:   docker://53b0a746b4c95c65bbdce09f416ca6762ec9ce78d2a9618187a86d80465ec349
    Image:          kodekloud/webapp-color:v3
    Image ID:       docker-pullable://kodekloud/webapp-color@sha256:3ecd19b1b85db381a0b6f78272458c3c274ac2a38e878d65700393899adb3177
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 15 Feb 2022 20:09:53 +0530
    Ready:          True
    Restart Count:  0
    Environment Variables from:
      webapp-color  ConfigMap  Optional: false
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-sq9q4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-sq9q4:
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
  Type     Reason        Age               From               Message
  ----     ------        ----              ----               -------
  Normal   Scheduled     10h               default-scheduler  Successfully assigned default/kodekloud-cm to k8s
  Normal   Pulled        10h               kubelet            Container image "kodekloud/webapp-color:v3" already present on machine
  Normal   Created       10h               kubelet            Created container kodekloud-cm
  Normal   Started       10h               kubelet            Started container kodekloud-cm
  Warning  NodeNotReady  9h (x2 over 10h)  node-controller    Node is not ready
```
