---

layout: single
title:  "Google Cloud Fundamentals: Getting Started with GKE"
date:   2023-04-05 12:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Google Cloud Fundamentals: Getting Started with GKE

In this lab, you create a Google Kubernetes Engine cluster containing several containers, each containing a web server. You place a load balancer in front of the cluster and view its contents.

1. In the Google Cloud Console, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **APIs & Services**.
2. Scroll down in the list of enabled APIs, and confirm that both of these APIs are enabled:

- Kubernetes Engine API
- Container Registry API

If either API is missing, click **Enable APIs and Services** at the top. Search for the above APIs by name and enable each for your current project.



```sh
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ export MY_ZONE="us-east5-c"
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ gcloud container clusters create webfrontend --zone $MY_ZONE --num-nodes 2
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flagDefault change: During creation of nodepools or autoscaling configuration changes for cluster versions greater than 1.24.1-gke.800 a default location policy is applied. For Spot and PVM it defaults to ANY, and for all other VM kinds a BALANCED policy is used. To change the default values use the `--location-policy` flag.Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         Creating cluster webfrontend in us-east5-c... Cluster is being health-checked (master is healthy)...worki                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             Creating cluster webfrontend in us-east5-c... Cluster is being health-checked (master is healthy)...done.
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-04-06ba3c9f3e97/zones/us-east5-c/clusters/webfrontend].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east5-c/webfrontend?project=qwiklabs-gcp-04-06ba3c9f3e97
kubeconfig entry generated for webfrontend.
NAME: webfrontend
LOCATION: us-east5-c
MASTER_VERSION: 1.24.9-gke.3200
MASTER_IP: 34.162.124.128
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.24.9-gke.3200
NUM_NODES: 2
STATUS: RUNNING
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```



```sh
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.3", GitCommit:"9e644106593f3f4aa98f8a84b23db5fa378900bd", GitTreeState:"clean", BuildDate:"2023-03-15T13:40:17Z", GoVersion:"go1.19.7", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.9-gke.3200", GitCommit:"92ea556d4e7418d0e7b5db1ee576a73f8fc47e91", GitTreeState:"clean", BuildDate:"2023-01-20T09:29:29Z", GoVersion:"go1.18.9b7", Compiler:"gc", Platform:"linux/amd64"}
WARNING: version difference between client (1.26) and server (1.24) exceeds the supported minor version skew of +/-1
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```



```sh
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.3", GitCommit:"9e644106593f3f4aa98f8a84b23db5fa378900bd", GitTreeState:"clean", BuildDate:"2023-03-15T13:40:17Z", GoVersion:"go1.19.7", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.9-gke.3200", GitCommit:"92ea556d4e7418d0e7b5db1ee576a73f8fc47e91", GitTreeState:"clean", BuildDate:"2023-01-20T09:29:29Z", GoVersion:"go1.18.9b7", Compiler:"gc", Platform:"linux/amd64"}
WARNING: version difference between client (1.26) and server (1.24) exceeds the supported minor version skew of +/-1
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```



```sh
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl create deploy nginx --image=nginx:1.17.10
deployment.apps/nginx created
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-5fc59799db-gwzhm   1/1     Running   0          8s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.24.0.1    <none>        443/TCP   4m4s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           9s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-5fc59799db   1         1         1       9s
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```



```sh
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl expose deployment nginx --port 80 --type LoadBalancer
service/nginx exposed
student_04_1324a3589973@cloudshell:~
```



```sh
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
kubernetes   ClusterIP      10.24.0.1      <none>           443/TCP        5m56s
nginx        LoadBalancer   10.24.12.102   34.162.206.209   80:32245/TCP   41s
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```



```sh
tudent_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ curl 34.162.206.209
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```



```sh
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl get ep
NAME         ENDPOINTS        AGE
kubernetes   10.202.0.2:443   6m57s
nginx        10.20.1.6:80     101s
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```



```sh
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl get all -o wide
NAME                         READY   STATUS    RESTARTS   AGE     IP          NODE                                         NOMINATED NODE   READINESS GATES
pod/nginx-5fc59799db-8vnv8   1/1     Running   0          25s     10.20.0.8   gke-webfrontend-default-pool-30bf06a4-csfq   <none>           <none>
pod/nginx-5fc59799db-gwzhm   1/1     Running   0          3m59s   10.20.1.6   gke-webfrontend-default-pool-30bf06a4-rc7z   <none>           <none>
pod/nginx-5fc59799db-sqrrf   1/1     Running   0          25s     10.20.1.7   gke-webfrontend-default-pool-30bf06a4-rc7z   <none>           <none>

NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE     SELECTOR
service/kubernetes   ClusterIP      10.24.0.1      <none>           443/TCP        7m54s   <none>
service/nginx        LoadBalancer   10.24.12.102   34.162.206.209   80:32245/TCP   2m39s   app=nginx

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES          SELECTOR
deployment.apps/nginx   3/3     3            3           4m    nginx        nginx:1.17.10   app=nginx

NAME                               DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES          SELECTOR
replicaset.apps/nginx-5fc59799db   3         3         3       4m    nginx        nginx:1.17.10   app=nginx,pod-template-hash=5fc59799db
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```

```sh
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl get ep
NAME         ENDPOINTS                                AGE
kubernetes   10.202.0.2:443                           8m24s
nginx        10.20.0.8:80,10.20.1.6:80,10.20.1.7:80   3m8s
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```



```sh
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
etcd-1               Healthy   {"health":"true"}
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$ kubectl describe cs
Warning: v1 ComponentStatus is deprecated in v1.19+
Name:         scheduler
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  v1
Conditions:
  Message:  ok
  Status:   True
  Type:     Healthy
Kind:       ComponentStatus
Metadata:
  Creation Timestamp:  <nil>
Events:                <none>


Name:         controller-manager
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  v1
Conditions:
  Message:  ok
  Status:   True
  Type:     Healthy
Kind:       ComponentStatus
Metadata:
  Creation Timestamp:  <nil>
Events:                <none>


Name:         etcd-0
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  v1
Conditions:
  Message:  {"health":"true"}
  Status:   True
  Type:     Healthy
Kind:       ComponentStatus
Metadata:
  Creation Timestamp:  <nil>
Events:                <none>


Name:         etcd-1
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  v1
Conditions:
  Message:  {"health":"true"}
  Status:   True
  Type:     Healthy
Kind:       ComponentStatus
Metadata:
  Creation Timestamp:  <nil>
Events:                <none>
student_04_1324a3589973@cloudshell:~ (qwiklabs-gcp-04-06ba3c9f3e97)$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-gke-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-gke-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-gke-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-gke-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-gke-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-gke-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-gke-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-gke-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-gke-9.png)

In this lab, we configured a Kubernetes cluster in Kubernetes Engine.  We populated the cluster with several pods containing an application,  exposed the application, and scaled the application.