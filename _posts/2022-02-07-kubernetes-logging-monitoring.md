---
layout: single
title:  "Kubernetes Logging and Monitoring"
date:   2022-02-07 10:55:04 +0530
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

# Kubernetes Logging and Monitoring


## Logging and Monitoring

To monitor cluster components, we need to deploy `metrics-server`.
Metrics Server is a cluster-wide aggregator of resource usage data. Resource usage metrics, such as container CPU and memory usage, are available in Kubernetes through the Metrics API. These metrics can be accessed either directly by the user with the kubectl top command, or by a controller in the cluster, for example Horizontal Pod Autoscaler, to make decisions.

Through the Metrics API, you can get the amount of resource currently used by a given node or a given pod.

The minikube tool includes a set of built-in addons that can be enabled, disabled and opened in the local Kubernetes environment.

```shell
pradeep@learnk8s$ minikube addons list -p k8s
|-----------------------------|---------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE |    STATUS    |           MAINTAINER           |
|-----------------------------|---------|--------------|--------------------------------|
| ambassador                  | k8s     | disabled     | third-party (ambassador)       |
| auto-pause                  | k8s     | disabled     | google                         |
| csi-hostpath-driver         | k8s     | disabled     | kubernetes                     |
| dashboard                   | k8s     | disabled     | kubernetes                     |
| default-storageclass        | k8s     | enabled ✅   | kubernetes                     |
| efk                         | k8s     | disabled     | third-party (elastic)          |
| freshpod                    | k8s     | disabled     | google                         |
| gcp-auth                    | k8s     | disabled     | google                         |
| gvisor                      | k8s     | disabled     | google                         |
| helm-tiller                 | k8s     | disabled     | third-party (helm)             |
| ingress                     | k8s     | disabled     | unknown (third-party)          |
| ingress-dns                 | k8s     | disabled     | google                         |
| istio                       | k8s     | disabled     | third-party (istio)            |
| istio-provisioner           | k8s     | disabled     | third-party (istio)            |
| kubevirt                    | k8s     | disabled     | third-party (kubevirt)         |
| logviewer                   | k8s     | disabled     | unknown (third-party)          |
| metallb                     | k8s     | disabled     | third-party (metallb)          |
| metrics-server              | k8s     | disabled     | kubernetes                     |
| nvidia-driver-installer     | k8s     | disabled     | google                         |
| nvidia-gpu-device-plugin    | k8s     | disabled     | third-party (nvidia)           |
| olm                         | k8s     | disabled     | third-party (operator          |
|                             |         |              | framework)                     |
| pod-security-policy         | k8s     | disabled     | unknown (third-party)          |
| portainer                   | k8s     | disabled     | portainer.io                   |
| registry                    | k8s     | disabled     | google                         |
| registry-aliases            | k8s     | disabled     | unknown (third-party)          |
| registry-creds              | k8s     | disabled     | third-party (upmc enterprises) |
| storage-provisioner         | k8s     | enabled ✅   | google                         |
| storage-provisioner-gluster | k8s     | disabled     | unknown (third-party)          |
| volumesnapshots             | k8s     | disabled     | kubernetes                     |
|-----------------------------|---------|--------------|--------------------------------|
💡  To see addons list for other profiles use: `minikube addons -p name list`
```
Let us enable the `metric-server` addon now in our cluster.
```shell
pradeep@learnk8s$ minikube addons enable metrics-server -p k8s
    ▪ Using image k8s.gcr.io/metrics-server/metrics-server:v0.4.2
🌟  The 'metrics-server' addon is enabled
```
#### Verify

```shell
pradeep@learnk8s$ minikube addons list -p k8s
|-----------------------------|---------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE |    STATUS    |           MAINTAINER           |
|-----------------------------|---------|--------------|--------------------------------|
| ambassador                  | k8s     | disabled     | third-party (ambassador)       |
| auto-pause                  | k8s     | disabled     | google                         |
| csi-hostpath-driver         | k8s     | disabled     | kubernetes                     |
| dashboard                   | k8s     | disabled     | kubernetes                     |
| default-storageclass        | k8s     | enabled ✅   | kubernetes                     |
| efk                         | k8s     | disabled     | third-party (elastic)          |
| freshpod                    | k8s     | disabled     | google                         |
| gcp-auth                    | k8s     | disabled     | google                         |
| gvisor                      | k8s     | disabled     | google                         |
| helm-tiller                 | k8s     | disabled     | third-party (helm)             |
| ingress                     | k8s     | disabled     | unknown (third-party)          |
| ingress-dns                 | k8s     | disabled     | google                         |
| istio                       | k8s     | disabled     | third-party (istio)            |
| istio-provisioner           | k8s     | disabled     | third-party (istio)            |
| kubevirt                    | k8s     | disabled     | third-party (kubevirt)         |
| logviewer                   | k8s     | disabled     | unknown (third-party)          |
| metallb                     | k8s     | disabled     | third-party (metallb)          |
| metrics-server              | k8s     | enabled ✅   | kubernetes                     |
| nvidia-driver-installer     | k8s     | disabled     | google                         |
| nvidia-gpu-device-plugin    | k8s     | disabled     | third-party (nvidia)           |
| olm                         | k8s     | disabled     | third-party (operator          |
|                             |         |              | framework)                     |
| pod-security-policy         | k8s     | disabled     | unknown (third-party)          |
| portainer                   | k8s     | disabled     | portainer.io                   |
| registry                    | k8s     | disabled     | google                         |
| registry-aliases            | k8s     | disabled     | unknown (third-party)          |
| registry-creds              | k8s     | disabled     | third-party (upmc enterprises) |
| storage-provisioner         | k8s     | enabled ✅   | google                         |
| storage-provisioner-gluster | k8s     | disabled     | unknown (third-party)          |
| volumesnapshots             | k8s     | disabled     | kubernetes                     |
|-----------------------------|---------|--------------|--------------------------------|
💡  To see addons list for other profiles use: `minikube addons -p name list`
```
```shell
pradeep@learnk8s$ kubectl get pod,svc,rs,deploy -n kube-system
NAME                                  READY   STATUS    RESTARTS         AGE
pod/coredns-64897985d-r9tzv           1/1     Running   8 (5h34m ago)    6d6h
pod/etcd-k8s                          1/1     Running   2 (5h34m ago)    6d6h
pod/kindnet-jpxdd                     1/1     Running   5 (11h ago)      6d6h
pod/kindnet-p77mb                     1/1     Running   50 (5h38m ago)   6d6h
pod/kube-apiserver-k8s                1/1     Running   3 (11h ago)      6d6h
pod/kube-controller-manager-k8s       1/1     Running   4 (5h34m ago)    6d6h
pod/kube-proxy-fszkr                  1/1     Running   1 (2d19h ago)    6d6h
pod/kube-proxy-m747v                  1/1     Running   1                6d6h
pod/kube-scheduler-k8s                1/1     Running   0                166m
pod/metrics-server-6b76bd68b6-7nbnx   1/1     Running   0                2m49s
pod/my-scheduler-k8s                  1/1     Running   0                89m
pod/storage-provisioner               1/1     Running   51 (53m ago)     6d6h

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns         ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   6d6h
service/metrics-server   ClusterIP   10.106.249.50   <none>        443/TCP                  2m49s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-64897985d           1         1         1       6d6h
replicaset.apps/metrics-server-6b76bd68b6   1         1         1       2m50s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns          1/1     1            1           6d6h
deployment.apps/metrics-server   1/1     1            1           2m50s
```

As seen above, `metric-server` is deployed as a Deployment, and exposed as ClusterIP service.

Use `kubectl top node` command to see CPU and memory utilization of each node in the cluster.

```shell
pradeep@learnk8s$ kubectl top node
NAME      CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
k8s       334m         16%    1635Mi          76%
k8s-m02   86m          4%     981Mi           45%
```
Use `kubectl top pod -A` command to see CPU and memory utilization of each node in the cluster.
```shell
pradeep@learnk8s$ kubectl top pod -A
NAMESPACE     NAME                              CPU(cores)   MEMORY(bytes)
default       demo-6c54f77c95-mgz7f             0m           3Mi
default       demo-6c54f77c95-q679r             0m           3Mi
default       demo-6c54f77c95-qqzbf             0m           3Mi
default       demo-6c54f77c95-vjgc2             0m           3Mi
default       demo-6c54f77c95-wv78b             0m           3Mi
default       demo-ds-gtcf7                     0m           3Mi
default       demo-ds-kkw4g                     0m           3Mi
default       nginx-k8s-m02                     0m           3Mi
default       nginx-manual                      0m           3Mi
default       nginx-no-tolerate                 0m           7Mi
default       nginx-node-selector               0m           9Mi
default       nginx-taint-demo                  0m           3Mi
default       with-node-affinity                0m           3Mi
kube-system   coredns-64897985d-r9tzv           0m           12Mi
kube-system   etcd-k8s                          0m           44Mi
kube-system   kindnet-jpxdd                     0m           8Mi
kube-system   kindnet-p77mb                     0m           7Mi
kube-system   kube-apiserver-k8s                0m           235Mi
kube-system   kube-controller-manager-k8s       0m           42Mi
kube-system   kube-proxy-fszkr                  0m           16Mi
kube-system   kube-proxy-m747v                  0m           22Mi
kube-system   kube-scheduler-k8s                0m           14Mi
kube-system   metrics-server-6b76bd68b6-7nbnx   0m           4Mi
kube-system   my-scheduler-k8s                  0m           13Mi
kube-system   storage-provisioner               0m           8Mi
prod          quota-mem-cpu-demo                0m           3Mi
```
You can also use the `--sory-by` field to display in descending order.
```shell
pradeep@learnk8s$ kubectl top pod -A --sort-by=memory
NAMESPACE     NAME                              CPU(cores)   MEMORY(bytes)
kube-system   kube-apiserver-k8s                0m           235Mi
kube-system   etcd-k8s                          0m           44Mi
kube-system   kube-controller-manager-k8s       0m           42Mi
kube-system   kube-proxy-m747v                  0m           22Mi
kube-system   kube-proxy-fszkr                  0m           16Mi
kube-system   kube-scheduler-k8s                0m           14Mi
kube-system   my-scheduler-k8s                  0m           13Mi
kube-system   coredns-64897985d-r9tzv           0m           12Mi
default       nginx-node-selector               0m           9Mi
kube-system   kindnet-jpxdd                     0m           8Mi
kube-system   storage-provisioner               0m           8Mi
default       nginx-no-tolerate                 0m           7Mi
kube-system   kindnet-p77mb                     0m           7Mi
kube-system   metrics-server-6b76bd68b6-7nbnx   0m           4Mi
default       demo-ds-kkw4g                     0m           3Mi
default       nginx-taint-demo                  0m           3Mi
default       demo-6c54f77c95-mgz7f             0m           3Mi
default       nginx-manual                      0m           3Mi
default       with-node-affinity                0m           3Mi
default       demo-ds-gtcf7                     0m           3Mi
default       demo-6c54f77c95-wv78b             0m           3Mi
default       demo-6c54f77c95-q679r             0m           3Mi
default       demo-6c54f77c95-qqzbf             0m           3Mi
default       demo-6c54f77c95-vjgc2             0m           3Mi
default       nginx-k8s-m02                     0m           3Mi
prod          quota-mem-cpu-demo                0m           3Mi
```


#### Examine Pod Logs
Use the `kubectl logs <pod>` name to look at the logs of the affected container

```shell
pradeep@learnk8s$ kubectl logs nginx-manual
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/02/07 11:34:31 [notice] 1#1: using the "epoll" event method
2022/02/07 11:34:31 [notice] 1#1: nginx/1.21.6
2022/02/07 11:34:31 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/02/07 11:34:31 [notice] 1#1: OS: Linux 4.19.202
2022/02/07 11:34:31 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/02/07 11:34:31 [notice] 1#1: start worker processes
2022/02/07 11:34:31 [notice] 1#1: start worker process 32
2022/02/07 11:34:31 [notice] 1#1: start worker process 33
```


#### Kubectl exec

If the container image includes debugging utilities, as is the case with images built from Linux and Windows OS base images, you can run commands inside a specific container with `kubectl exec`.

Here is an example to login to the `nginx-manual` container and execute commands from the `shell` mode.

```shell
pradeep@learnk8s$ kubectl exec -it nginx-manual -- sh
# uname -a
Linux nginx-manual 4.19.202 #1 SMP Thu Dec 23 10:44:17 UTC 2021 x86_64 GNU/Linux
# exit 0
```

