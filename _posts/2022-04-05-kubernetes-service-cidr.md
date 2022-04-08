---
layout: single
title:  "Kubernetes Service CIDR"
date:   2022-04-05 15:55:04 +0530
categories: Kubernetes
tags: kubeadm
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/kubeadm.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubernetes Service CIDR

Let us create an `nginx` pod an expose it.

```sh
lab@k8s1:~$ kubectl run nginx --image=nginx
pod/nginx created

```

```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          6s    10.244.3.39   k8s3   <none>           <none>
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl expose pod nginx --name=my-nginx-svc --port=4000
service/my-nginx-svc exposed
lab@k8s1:~$
```
Check the service, endpoints and their IPs

```sh
lab@k8s1:~$ kubectl get svc,ep -l run=nginx
NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/my-nginx-svc   ClusterIP   10.99.98.44   <none>        4000/TCP   21s

NAME                     ENDPOINTS          AGE
endpoints/my-nginx-svc   10.244.3.39:4000   21s
lab@k8s1:~$
```

Let us find and change the Service CIDR on the `kube-apiserver`
```yaml
lab@k8s1:~$ sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.100.1:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.100.1
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=172.16.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.22.8
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.100.1
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 192.168.100.1
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 192.168.100.1
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}
lab@k8s1:~$
```
The `--service-cluster-ip-range` is changed from `10.96.0.0/22` to `172.16.0.0/12`.

Wait for the `kube-apiserver` pod to come back
```sh
lab@k8s1:~$ kubectl get pods -n kube-system
NAME                           READY   STATUS    RESTARTS      AGE
coredns-78fcd69978-cvpx2       1/1     Running   0             6d11h
coredns-78fcd69978-hf5sj       1/1     Running   0             6d11h
etcd-k8s1                      1/1     Running   0             5d5h
kube-apiserver-k8s1            1/1     Running   0             39s
kube-controller-manager-k8s1   1/1     Running   3 (70s ago)   6d15h
kube-flannel-ds-lhcwb          1/1     Running   0             6d15h
kube-flannel-ds-ph9gg          1/1     Running   0             6d14h
kube-flannel-ds-xm28z          1/1     Running   0             6d12h
kube-proxy-brrvs               1/1     Running   0             6d15h
kube-proxy-cdl2t               1/1     Running   0             6d14h
kube-proxy-v8r74               1/1     Running   0             6d12h
kube-scheduler-k8s1            1/1     Running   4 (70s ago)   6d15h
lab@k8s1:~$
```
We can see, the `kube-apiserver-k8s1` pod started `39s` ago.

Let us do the same for  the `kube-controller-manager` as well.

```yaml
lab@k8s1:~$ sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.244.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --port=0
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=172.16.0.0/12
    - --use-service-account-credentials=true
    image: k8s.gcr.io/kube-controller-manager:v1.22.8
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10257
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-controller-manager
    resources:
      requests:
        cpu: 200m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10257
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      name: flexvolume-dir
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /etc/kubernetes/controller-manager.conf
      name: kubeconfig
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      type: DirectoryOrCreate
    name: flexvolume-dir
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /etc/kubernetes/controller-manager.conf
      type: FileOrCreate
    name: kubeconfig
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}
lab@k8s1:~$
```
The `--service-cluster-ip-range` is changed from `10.96.0.0/12` to `172.16.0.0/12`.

Wait for the controller-manager pod to come back

```sh
lab@k8s1:~$ kubectl get pods -n kube-system
NAME                           READY   STATUS    RESTARTS        AGE
coredns-78fcd69978-cvpx2       1/1     Running   0               6d11h
coredns-78fcd69978-hf5sj       1/1     Running   0               6d11h
etcd-k8s1                      1/1     Running   0               5d5h
kube-apiserver-k8s1            1/1     Running   0               4m39s
kube-controller-manager-k8s1   1/1     Running   0               41s
kube-flannel-ds-lhcwb          1/1     Running   0               6d15h
kube-flannel-ds-ph9gg          1/1     Running   0               6d14h
kube-flannel-ds-xm28z          1/1     Running   0               6d12h
kube-proxy-brrvs               1/1     Running   0               6d15h
kube-proxy-cdl2t               1/1     Running   0               6d14h
kube-proxy-v8r74               1/1     Running   0               6d12h
kube-scheduler-k8s1            1/1     Running   4 (5m10s ago)   6d15h
lab@k8s1:~$
```
we can see that the `kube-controller-manager-k8s` pod started `41s` ago.

Let us check the existing pod, service, and endpoint IPs.
```sh
lab@k8s1:~$ kubectl get pod,svc,ep -l run=nginx
NAME        READY   STATUS    RESTARTS   AGE
pod/nginx   1/1     Running   0          11m

NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/my-nginx-svc   ClusterIP   10.99.98.44   <none>        4000/TCP   10m

NAME                     ENDPOINTS          AGE
endpoints/my-nginx-svc   10.244.3.39:4000   10m
lab@k8s1:~$
```

Nothing seems to have changed.

Let us create another service, exposing the same pod

```sh
lab@k8s1:~$ kubectl expose pod nginx --name=my-nginx-svc-latest --port=80
service/my-nginx-svc-latest exposed
lab@k8s1:~$
```

```sh
lab@k8s1:~$  kubectl get pod,svc,ep -l run=nginx
NAME        READY   STATUS    RESTARTS   AGE
pod/nginx   1/1     Running   0          15m

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/my-nginx-svc          ClusterIP   10.99.98.44     <none>        4000/TCP   14m
service/my-nginx-svc-latest   ClusterIP   172.24.51.239   <none>        80/TCP     40s

NAME                            ENDPOINTS          AGE
endpoints/my-nginx-svc          10.244.3.39:4000   14m
endpoints/my-nginx-svc-latest   10.244.3.39:80     40s
lab@k8s1:~$
```
From this we can see that the new service (`my-nginx-svc-lates`) got the IP (`172.24.51.239`) address from the modified Service CIDR (`172.16.0.0/12`). And both old and new services are having the same endpoint (pod).



```sh
lab@k8s1:~$ curl 172.24.51.239:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
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
lab@k8s1:~$
```



This concludes our discussion on changing the Service CIDR. In summary, we have to modify the `--service-cluster-ip-range` in both `kube-apiserver` and `kube-controller-manager` pod definitions. Old services continue to use the old CIDR range, and new services will be utilising the new range.



