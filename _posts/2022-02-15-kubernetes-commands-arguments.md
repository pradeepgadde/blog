---
layout: single
title:  "Passing Commands and Arguments to Kubernetes Resources"
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
# Passing Commands and Arguments to Kubernetes Resources


### Commands and Arguments
Let us make use of help, to understand the usage of commands and arguments.
As shown here, with the help of `--command` option we can pass a different command and custom arguments to our container.

```shell
pradeep@learnk8s$ kubectl run -h
Create and run a particular image in a pod.

Examples:
<SNIP>

  # Start the nginx pod using the default command, but use custom arguments (arg1 .. argN) for that command
  kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>

  # Start the nginx pod using a different command and custom arguments
  kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>

Options:
<SNIP>
```
As an example, let us try to create a Ubuntu container that just prints date.
```shell
pradeep@learnk8s$ kubectl run ubuntu-date --image=ubuntu --command date
pod/ubuntu-date created
```
Look at the `Command` header under `Containers` section.
```yaml
pradeep@learnk8s$ kubectl describe pods ubuntu-date
Name:         ubuntu-date
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Tue, 15 Feb 2022 13:55:40 +0530
Labels:       run=ubuntu-date
Annotations:  <none>
Status:       Running
IP:           10.244.1.21
IPs:
  IP:  10.244.1.21
Containers:
  ubuntu-date:
    Container ID:  docker://3c6c897fe4ad67e0a90540ec086901eb63a5f8ab1d4afae85b7d0a734e363a9c
    Image:         ubuntu
    Image ID:      docker-pullable://ubuntu@sha256:669e010b58baf5beb2836b253c1fd5768333f0d1dbcb834f7c07a4dc93f474be
    Port:          <none>
    Host Port:     <none>
    Command:
      date
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 15 Feb 2022 13:55:45 +0530
      Finished:     Tue, 15 Feb 2022 13:55:45 +0530
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-87bh2 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  kube-api-access-87bh2:
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
  Type    Reason     Age              From               Message
  ----    ------     ----             ----               -------
  Normal  Scheduled  9s               default-scheduler  Successfully assigned default/ubuntu-date to k8s-m02
  Normal  Pulled     4s               kubelet            Successfully pulled image "ubuntu" in 4.705982332s
  Normal  Created    4s               kubelet            Created container ubuntu-date
  Normal  Started    4s               kubelet            Started container ubuntu-date
  Normal  Pulling    3s (x2 over 8s)  kubelet            Pulling image "ubuntu"
```
The pod is Terminated with status `Completed`.
```shell
pradeep@learnk8s$ kubectl get pods | grep date
ubuntu-date                 0/1     CrashLoopBackOff   3 (55s ago)   111s
```
Let us create another container that uses an argument as well, in addition to the command.
```shell
pradeep@learnk8s$ kubectl run ubuntu-sleep --image=ubuntu --command sleep 5000
pod/ubuntu-sleep created
```
```yaml
pradeep@learnk8s$ kubectl describe pods ubuntu-sleep
Name:         ubuntu-sleep
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Tue, 15 Feb 2022 13:59:46 +0530
Labels:       run=ubuntu-sleep
Annotations:  <none>
Status:       Running
IP:           10.244.1.22
IPs:
  IP:  10.244.1.22
Containers:
  ubuntu-sleep:
    Container ID:  docker://4a12725e6951fd785eadb5bb692b25755ecfac7c664d2eb97305f46369f9d898
    Image:         ubuntu
    Image ID:      docker-pullable://ubuntu@sha256:669e010b58baf5beb2836b253c1fd5768333f0d1dbcb834f7c07a4dc93f474be
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      5000
    State:          Running
      Started:      Tue, 15 Feb 2022 13:59:56 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gpjg5 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-gpjg5:
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
  Normal  Scheduled  26s   default-scheduler  Successfully assigned default/ubuntu-sleep to k8s-m02
  Normal  Pulling    25s   kubelet            Pulling image "ubuntu"
  Normal  Pulled     17s   kubelet            Successfully pulled image "ubuntu" in 8.387291792s
  Normal  Created    17s   kubelet            Created container ubuntu-sleep
  Normal  Started    16s   kubelet            Started container ubuntu-sleep
```

Let us take a look at the definition of this Pod with command and arguments in YAML.
```shell
pradeep@learnk8s$ kubectl get pods ubuntu-sleep -o yaml > pod-command-arg.yaml
```
```yaml
pradeep@learnk8s$ cat pod-command-arg.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-02-15T08:29:46Z"
  labels:
    run: ubuntu-sleep
  name: ubuntu-sleep
  namespace: default
  resourceVersion: "5745"
  uid: 971c6c08-da5e-4360-901b-7b8ca65a693a
spec:
  containers:
  - command:
    - sleep
    - "5000"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu-sleep
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-gpjg5
      readOnly: true
  dnsPolicy: ClusterFirst
  <SNIP>
```

Now let us use the same kodecloud/web-app image and try to pass the color argument while initializing the pod.

Looking at the source code of this container  [kodekloudhub/webapp-color](https://github.com/kodekloudhub/webapp-color/blob/master/app.py), we can find out that, A color can be specified in two ways: either as a command line argument or as an environment variable.

```python
from flask import Flask
from flask import render_template
import socket
import random
import os
import argparse

app = Flask(__name__)

color_codes = {
    "red": "#e74c3c",
    "green": "#16a085",
    "blue": "#2980b9",
    "blue2": "#30336b",
    "pink": "#be2edd",
    "darkblue": "#130f40"
}

SUPPORTED_COLORS = ",".join(color_codes.keys())

# Get color from Environment variable
COLOR_FROM_ENV = os.environ.get('APP_COLOR')
# Generate a random color
COLOR = random.choice(["red", "green", "blue", "blue2", "darkblue", "pink"])


@app.route("/")
def main():
    # return 'Hello'
    return render_template('hello.html', name=socket.gethostname(), color=color_codes[COLOR])


if __name__ == "__main__":

    print(" This is a sample web application that displays a colored background. \n"
          " A color can be specified in two ways. \n"
          "\n"
          " 1. As a command line argument with --color as the argument. Accepts one of " + SUPPORTED_COLORS + " \n"
          " 2. As an Environment variable APP_COLOR. Accepts one of " + SUPPORTED_COLORS + " \n"
          " 3. If none of the above then a random color is picked from the above list. \n"
          " Note: Command line argument precedes over environment variable.\n"
          "\n"
          "")

    # Check for Command Line Parameters for color
    parser = argparse.ArgumentParser()
    parser.add_argument('--color', required=False)
    args = parser.parse_args()

    if args.color:
        print("Color from command line argument =" + args.color)
        COLOR = args.color
        if COLOR_FROM_ENV:
            print("A color was set through environment variable -" + COLOR_FROM_ENV + ". However, color from command line argument takes precendence.")
    elif COLOR_FROM_ENV:
        print("No Command line argument. Color from environment variable =" + COLOR_FROM_ENV)
        COLOR = COLOR_FROM_ENV
    else:
        print("No command line argument or environment variable. Picking a Random Color =" + COLOR)

    # Check if input color is a supported one
    if COLOR not in color_codes:
        print("Color not supported. Received '" + COLOR + "' expected one of " + SUPPORTED_COLORS)
        exit(1)

    # Run Flask Application
    app.run(host="0.0.0.0", port=8080)
```
Let us try the first method that we just discussed: as a command argument.
Create a pod using `kodekloud/webapp-color:v3` but pass the --color argument to change the default color from `red` to `blue`. Earlier, when we ran this image (with defaults), the color was `red`.

This can also be confirmed by looking at the Dockerfile definition.

![kodekloud-webapp-color]({{ site.url }}{{ site.baseurl }}/assets/images/kodekloud-webapp-color.png)

```shell
pradeep@learnk8s$ kubectl run kodekloud-change-color --image=kodekloud/webapp-color:v3 --dry-run=client -o yaml -- "--color" "blue" > pod-change-color.yaml
```
Now we can see we are passing the `--color` and `blue` as arguements to this container.

```yaml
pradeep@learnk8s$ cat pod-change-color.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: kodekloud-change-color
  name: kodekloud-change-color
spec:
  containers:
  - args:
    - --color
    - blue
    image: kodekloud/webapp-color:v3
    name: kodekloud-change-color
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
Let us create a pod from this definition file.
```shell
pradeep@learnk8s$ kubectl create -f pod-change-color.yaml
pod/kodekloud-change-color created
```
```shell
pradeep@learnk8s$ kubectl describe pods kodekloud-change-color
Name:         kodekloud-change-color
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Tue, 15 Feb 2022 16:31:54 +0530
Labels:       run=kodekloud-change-color
Annotations:  <none>
Status:       Running
IP:           10.244.1.29
IPs:
  IP:  10.244.1.29
Containers:
  kodekloud-change-color:
    Container ID:  docker://3c6bf9642cb90fcc2f969a3043f3879d9a42917d7e15ed26905b2a340461609e
    Image:         kodekloud/webapp-color:v3
    Image ID:      docker-pullable://kodekloud/webapp-color@sha256:3ecd19b1b85db381a0b6f78272458c3c274ac2a38e878d65700393899adb3177
    Port:          <none>
    Host Port:     <none>
    Args:
      --color
      blue
    State:          Running
      Started:      Tue, 15 Feb 2022 16:31:57 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-6l7n4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-6l7n4:
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
  Normal  Scheduled  12s   default-scheduler  Successfully assigned default/kodekloud-change-color to k8s-m02
  Normal  Pulled     10s   kubelet            Container image "kodekloud/webapp-color:v3" already present on machine
  Normal  Created    10s   kubelet            Created container kodekloud-change-color
  Normal  Started    9s    kubelet            Started container kodekloud-change-color
```
```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep color
kodekloud-change-color      1/1     Running            0                40s    10.244.1.29   k8s-m02   <none>           <none>
```
Verify the color of the webapp now, it should say `blue` instead of the default `red`.

```shell
pradeep@learnk8s$ minikube ssh -p k8s

                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.1.29:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #2980b9;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  <h1>Hello from kodekloud-change-color!</h1>



  <h2>
    Application Version: v3
  </h2>


</div>$
$ curl 10.244.1.29:8080/color
blue$ exit
logout
```

It is confirmed the arguments that we passed worked! (`background: #2980b9 indicates blue, as defined in the color_code, "blue": "#2980b9"`).
