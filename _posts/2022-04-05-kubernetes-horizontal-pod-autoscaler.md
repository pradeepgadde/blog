---
layout: single
title:  "Kubernetes Horizontal Pod Autoscaler"
date:   2022-04-04 10:56:04 +0530
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


# Kubernetes Horizontal Pod AutoScaler



What is HPA?


```sh

lab@k8s1:~$ kubectl autoscale -h
Creates an autoscaler that automatically chooses and sets the number of pods that run in a Kubernetes cluster.

 Looks up a deployment, replica set, stateful set, or replication controller by name and creates an autoscaler that uses
the given resource as a reference. An autoscaler can automatically increase or decrease number of pods deployed within
the system as needed.

Examples:

  # Auto scale a deployment "foo", with the number of pods between 2 and 10, no target CPU utilization specified so a

default autoscaling policy will be used
  kubectl autoscale deployment foo --min=2 --max=10

  # Auto scale a replication controller "foo", with the number of pods between 1 and 5, target CPU utilization at 80%
  kubectl autoscale rc foo --max=5 --cpu-percent=80

Options:
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or map key is missing in
the template. Only applies to golang and jsonpath output formats.
      --cpu-percent=-1: The target average CPU utilization (represented as a percent of requested CPU) over all the
pods. If it's not specified or negative, a default autoscaling policy will be used.
      --dry-run='none': Must be "none", "server", or "client". If client strategy, only print the object that would be
sent, without sending it. If server strategy, submit server-side request without persisting the resource.
      --field-manager='kubectl-autoscale': Name of the manager used to track field ownership.
  -f, --filename=[]: Filename, directory, or URL to files identifying the resource to autoscale.
  -k, --kustomize='': Process the kustomization directory. This flag can't be used together with -f or -R.
      --max=-1: The upper limit for the number of pods that can be set by the autoscaler. Required.
      --min=-1: The lower limit for the number of pods that can be set by the autoscaler. If it's not specified or
negative, the server will apply a default value.
      --name='': The name for the newly created object. If not specified, the name of the input resource will be used.
  -o, --output='': Output format. One of:
json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you want to manage
related manifests organized within the same directory.
      --save-config=false: If true, the configuration of current object will be saved in its annotation. Otherwise, the
annotation will be unchanged. This flag is useful when you want to perform kubectl apply on this object in the future.
      --show-managed-fields=false: If true, keep the managedFields when printing objects in JSON or YAML format.
      --template='': Template string or path to template file to use when -o=go-template, -o=go-template-file. The
template format is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

Usage:
  kubectl autoscale (-f FILENAME | TYPE NAME | TYPE/NAME) [--min=MINPODS] --max=MAXPODS [--cpu-percent=CPU] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
lab@k8s1:~$

```



Let us apply this to our deployment



```sh
lab@k8s1:~$ kubectl autoscale deployment web --min=2 --max=10
horizontalpodautoscaler.autoscaling/web autoscaled
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl get hpa
NAME   REFERENCE        TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
web    Deployment/web   <unknown>/80%   2         10        0          6s
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl describe hpa
Name:                                                  web
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Mon, 04 Apr 2022 19:14:52 -0700
Reference:                                             Deployment/web
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  <unknown> / 80%
Min replicas:                                          2
Max replicas:                                          10
Deployment pods:                                       9 current / 0 desired
Conditions:
  Type           Status  Reason                   Message
  ----           ------  ------                   -------
  AbleToScale    True    SucceededGetScale        the HPA controller was able to get the target's current scale
  ScalingActive  False   FailedGetResourceMetric  the HPA was unable to compute the replica count: failed to get cpu utilization: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)
Events:
  Type     Reason                        Age   From                       Message
  ----     ------                        ----  ----                       -------
  Warning  FailedGetResourceMetric       8s    horizontal-pod-autoscaler  failed to get cpu utilization: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)
  Warning  FailedComputeMetricsReplicas  8s    horizontal-pod-autoscaler  invalid metrics (1 invalid out of 1), first error is: failed to get cpu utilization: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)
lab@k8
```



In this cluster, we did not have metrics server installed yet. So let us go ahead and install it to get these metrics populated.

```sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```



```sh
lab@k8s1:~$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
lab@k8s1:~$
```

Seems it is not working, we might have to tweak some configuration files.

```sh
lab@k8s1:~$ kubectl top nodes
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get nodes.metrics.k8s.io)
lab@k8s1:~$ kubectl top nodes
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get nodes.metrics.k8s.io)
lab@k8s1:~$ kubectl top pods
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)
lab@k8s1:~$ kubectl top pods
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)
lab@k8s1:~$
```

Let us delete this.

```sh
lab@k8s1:~$ kubectl delete -f  https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
serviceaccount "metrics-server" deleted
clusterrole.rbac.authorization.k8s.io "system:aggregated-metrics-reader" deleted
clusterrole.rbac.authorization.k8s.io "system:metrics-server" deleted
rolebinding.rbac.authorization.k8s.io "metrics-server-auth-reader" deleted
clusterrolebinding.rbac.authorization.k8s.io "metrics-server:system:auth-delegator" deleted
clusterrolebinding.rbac.authorization.k8s.io "system:metrics-server" deleted
service "metrics-server" deleted
deployment.apps "metrics-server" deleted
apiservice.apiregistration.k8s.io "v1beta1.metrics.k8s.io" deleted
lab@k8s1:~$
```





Let us use `git` to get the required YAML files.

```sh
lab@k8s1:~$ git clone https://github.com/kubernetes-sigs/metrics-server.git
Cloning into 'metrics-server'...
remote: Enumerating objects: 14418, done.
remote: Counting objects: 100% (273/273), done.
remote: Compressing objects: 100% (198/198), done.
remote: Total 14418 (delta 122), reused 166 (delta 64), pack-reused 14145
Receiving objects: 100% (14418/14418), 13.22 MiB | 6.79 MiB/s, done.
Resolving deltas: 100% (7613/7613), done.
lab@k8s1:~$
```

Edit the deployment yaml file to work with our kubeadm cluster. Since we are running kubernetes version 1.22.8, we will use the 1.8+ version of the  Metrics-server.