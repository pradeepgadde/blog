---
layout: single
title:  "GitOps: ArgoCD App Reconciliation Timeout"
categories: Kubernetes
tags: ArgoCD
classes: wide
show_date: true
header:
  overlay_image: /assets/images/argo.png
  og_image: /assets/images/argo.png
  teaser: /assets/images/argo.png
  caption: "Photo credit: [**Argo**](https://argo-cd.readthedocs.io/en/stable/)"
  actions:
    - label: "Learn more"
      url: "https://argo-cd.readthedocs.io/en/stable/"

author:
  name     : "ArgoCD"
  avatar   : "/assets/images/argo.png"

sidebar:
  - title: "Blog"

    text: "Checkout other topics"
    nav: my-sidebar
---

# GitOps: ArgoCD App Reconciliation Timeout

In this post, let us modify the application reconciliation timeout of ArgoCD.

```sh
myk8scluster ~ ➜  kubectl get ns
NAME              STATUS   AGE
argocd            Active   6m14s
default           Active   20m
kube-flannel      Active   20m
kube-node-lease   Active   20m
kube-public       Active   20m
kube-system       Active   20m
webhook           Active   79s

myk8scluster ~ ➜  kubectl get all -n argocd
NAME                                                   READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0                    1/1     Running   0          6m17s
pod/argocd-applicationset-controller-d7c857898-mkz7x   1/1     Running   0          6m18s
pod/argocd-dex-server-75d98bff7c-t8dbb                 1/1     Running   0          6m18s
pod/argocd-notifications-controller-684947df85-qtm6x   1/1     Running   0          6m18s
pod/argocd-redis-84c8cd4d8-7nbll                       1/1     Running   0          6m18s
pod/argocd-repo-server-6b5cf8488-qzfqk                 1/1     Running   0          6m17s
pod/argocd-server-5f8984f889-qqt6m                     1/1     Running   0          6m17s

NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP   172.20.161.118   <none>        7000/TCP,8080/TCP            6m18s
service/argocd-dex-server                         ClusterIP   172.20.23.46     <none>        5556/TCP,5557/TCP,5558/TCP   6m18s
service/argocd-metrics                            ClusterIP   172.20.45.80     <none>        8082/TCP                     6m18s
service/argocd-notifications-controller-metrics   ClusterIP   172.20.50.191    <none>        9001/TCP                     6m18s
service/argocd-redis                              ClusterIP   172.20.85.185    <none>        6379/TCP                     6m18s
service/argocd-repo-server                        ClusterIP   172.20.190.10    <none>        8081/TCP,8084/TCP            6m18s
service/argocd-server                             NodePort    172.20.141.136   <none>        80:32765/TCP,443:32766/TCP   6m18s
service/argocd-server-metrics                     ClusterIP   172.20.126.178   <none>        8083/TCP                     6m18s

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           6m18s
deployment.apps/argocd-dex-server                  1/1     1            1           6m18s
deployment.apps/argocd-notifications-controller    1/1     1            1           6m18s
deployment.apps/argocd-redis                       1/1     1            1           6m18s
deployment.apps/argocd-repo-server                 1/1     1            1           6m18s
deployment.apps/argocd-server                      1/1     1            1           6m17s

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-d7c857898   1         1         1       6m18s
replicaset.apps/argocd-dex-server-75d98bff7c                 1         1         1       6m18s
replicaset.apps/argocd-notifications-controller-684947df85   1         1         1       6m18s
replicaset.apps/argocd-redis-84c8cd4d8                       1         1         1       6m18s
replicaset.apps/argocd-repo-server-6b5cf8488                 1         1         1       6m17s
replicaset.apps/argocd-repo-server-7bbc57875d                0         0         0       6m18s
replicaset.apps/argocd-server-5f8984f889                     1         1         1       6m17s

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     6m17s

myk8scluster ~ ➜  kubectl describe deploy argocd-server -n argocd
Name:                   argocd-server
Namespace:              argocd
CreationTimestamp:      Thu, 14 Nov 2024 19:15:33 +0000
Labels:                 app.kubernetes.io/component=server
                        app.kubernetes.io/name=argocd-server
                        app.kubernetes.io/part-of=argocd
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app.kubernetes.io/name=argocd-server
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app.kubernetes.io/name=argocd-server
  Service Account:  argocd-server
  Containers:
   argocd-server:
    Image:       quay.io/argoproj/argocd:v2.4.11
    Ports:       8080/TCP, 8083/TCP
    Host Ports:  0/TCP, 0/TCP
    Command:
      argocd-server
    Liveness:   http-get http://:8080/healthz%3Ffull=true delay=3s timeout=1s period=30s #success=1 #failure=3
    Readiness:  http-get http://:8080/healthz delay=3s timeout=1s period=30s #success=1 #failure=3
    Environment:
      ARGOCD_SERVER_INSECURE:                            <set to the key 'server.insecure' of config map 'argocd-cmd-params-cm'>                            Optional: true
      ARGOCD_SERVER_BASEHREF:                            <set to the key 'server.basehref' of config map 'argocd-cmd-params-cm'>                            Optional: true
      ARGOCD_SERVER_ROOTPATH:                            <set to the key 'server.rootpath' of config map 'argocd-cmd-params-cm'>                            Optional: true
      ARGOCD_SERVER_LOGFORMAT:                           <set to the key 'server.log.format' of config map 'argocd-cmd-params-cm'>                          Optional: true
      ARGOCD_REPO_SERVER_LOGLEVEL:                       <set to the key 'server.log.level' of config map 'argocd-cmd-params-cm'>                           Optional: true
      ARGOCD_SERVER_REPO_SERVER:                         <set to the key 'repo.server' of config map 'argocd-cmd-params-cm'>                                Optional: true
      ARGOCD_SERVER_DEX_SERVER:                          <set to the key 'server.dex.server' of config map 'argocd-cmd-params-cm'>                          Optional: true
      ARGOCD_SERVER_DISABLE_AUTH:                        <set to the key 'server.disable.auth' of config map 'argocd-cmd-params-cm'>                        Optional: true
      ARGOCD_SERVER_ENABLE_GZIP:                         <set to the key 'server.enable.gzip' of config map 'argocd-cmd-params-cm'>                         Optional: true
      ARGOCD_SERVER_REPO_SERVER_TIMEOUT_SECONDS:         <set to the key 'server.repo.server.timeout.seconds' of config map 'argocd-cmd-params-cm'>         Optional: true
      ARGOCD_SERVER_X_FRAME_OPTIONS:                     <set to the key 'server.x.frame.options' of config map 'argocd-cmd-params-cm'>                     Optional: true
      ARGOCD_SERVER_CONTENT_SECURITY_POLICY:             <set to the key 'server.content.security.policy' of config map 'argocd-cmd-params-cm'>             Optional: true
      ARGOCD_SERVER_REPO_SERVER_PLAINTEXT:               <set to the key 'server.repo.server.plaintext' of config map 'argocd-cmd-params-cm'>               Optional: true
      ARGOCD_SERVER_REPO_SERVER_STRICT_TLS:              <set to the key 'server.repo.server.strict.tls' of config map 'argocd-cmd-params-cm'>              Optional: true
      ARGOCD_TLS_MIN_VERSION:                            <set to the key 'server.tls.minversion' of config map 'argocd-cmd-params-cm'>                      Optional: true
      ARGOCD_TLS_MAX_VERSION:                            <set to the key 'server.tls.maxversion' of config map 'argocd-cmd-params-cm'>                      Optional: true
      ARGOCD_TLS_CIPHERS:                                <set to the key 'server.tls.ciphers' of config map 'argocd-cmd-params-cm'>                         Optional: true
      ARGOCD_SERVER_CONNECTION_STATUS_CACHE_EXPIRATION:  <set to the key 'server.connection.status.cache.expiration' of config map 'argocd-cmd-params-cm'>  Optional: true
      ARGOCD_SERVER_OIDC_CACHE_EXPIRATION:               <set to the key 'server.oidc.cache.expiration' of config map 'argocd-cmd-params-cm'>               Optional: true
      ARGOCD_SERVER_LOGIN_ATTEMPTS_EXPIRATION:           <set to the key 'server.login.attempts.expiration' of config map 'argocd-cmd-params-cm'>           Optional: true
      ARGOCD_SERVER_STATIC_ASSETS:                       <set to the key 'server.staticassets' of config map 'argocd-cmd-params-cm'>                        Optional: true
      ARGOCD_APP_STATE_CACHE_EXPIRATION:                 <set to the key 'server.app.state.cache.expiration' of config map 'argocd-cmd-params-cm'>          Optional: true
      REDIS_SERVER:                                      <set to the key 'redis.server' of config map 'argocd-cmd-params-cm'>                               Optional: true
      REDISDB:                                           <set to the key 'redis.db' of config map 'argocd-cmd-params-cm'>                                   Optional: true
      ARGOCD_DEFAULT_CACHE_EXPIRATION:                   <set to the key 'server.default.cache.expiration' of config map 'argocd-cmd-params-cm'>            Optional: true
      ARGOCD_MAX_COOKIE_NUMBER:                          <set to the key 'server.http.cookie.maxnumber' of config map 'argocd-cmd-params-cm'>               Optional: true
      ARGOCD_SERVER_OTLP_ADDRESS:                        <set to the key 'otlp.address' of config map 'argocd-cmd-params-cm'>                               Optional: true
    Mounts:
      /app/config/server/tls from argocd-repo-server-tls (rw)
      /app/config/ssh from ssh-known-hosts (rw)
      /app/config/tls from tls-certs (rw)
      /home/argocd from plugins-home (rw)
      /tmp from tmp (rw)
  Volumes:
   plugins-home:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
   tmp:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
   ssh-known-hosts:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      argocd-ssh-known-hosts-cm
    Optional:  false
   tls-certs:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      argocd-tls-certs-cm
    Optional:  false
   argocd-repo-server-tls:
    Type:          Secret (a volume populated by a Secret)
    SecretName:    argocd-repo-server-tls
    Optional:      true
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   argocd-server-5f8984f889 (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  6m38s  deployment-controller  Scaled up replica set argocd-server-5f8984f889 to 1

myk8scluster ~ ➜  

myk8scluster ~ ➜  kubectl get pods
No resources found in default namespace.

myk8scluster ~ ➜  kubectl get pods -A
NAMESPACE      NAME                                               READY   STATUS    RESTARTS      AGE
argocd         argocd-application-controller-0                    1/1     Running   0             11m
argocd         argocd-applicationset-controller-d7c857898-mkz7x   1/1     Running   0             11m
argocd         argocd-dex-server-75d98bff7c-t8dbb                 1/1     Running   0             11m
argocd         argocd-notifications-controller-684947df85-qtm6x   1/1     Running   0             11m
argocd         argocd-redis-84c8cd4d8-7nbll                       1/1     Running   0             11m
argocd         argocd-repo-server-6b5cf8488-qzfqk                 1/1     Running   0             11m
argocd         argocd-server-5f8984f889-qqt6m                     1/1     Running   0             11m
kube-flannel   kube-flannel-ds-jdcnz                              1/1     Running   1 (24m ago)   25m
kube-system    coredns-768b85b76f-kx44d                           1/1     Running   0             25m
kube-system    coredns-768b85b76f-vt5zm                           1/1     Running   0             25m
kube-system    etcd-myk8scluster                                  1/1     Running   0             25m
kube-system    kube-apiserver-myk8scluster                        1/1     Running   0             25m
kube-system    kube-controller-manager-myk8scluster               1/1     Running   0             25m
kube-system    kube-proxy-t7vp9                                   1/1     Running   0             25m
kube-system    kube-scheduler-myk8scluster                        1/1     Running   0             25m
webhook        nginx-6cfb64b7c5-j5wvw                             1/1     Running   0             6m25s
webhook        nginx-6cfb64b7c5-j6hmj                             1/1     Running   0             105s
webhook        nginx-6cfb64b7c5-k6lzj                             1/1     Running   0             105s

myk8scluster ~ ➜  kubectl get pods -n webhook
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6cfb64b7c5-j5wvw   1/1     Running   0          6m35s
nginx-6cfb64b7c5-j6hmj   1/1     Running   0          115s
nginx-6cfb64b7c5-k6lzj   1/1     Running   0          115s

myk8scluster ~ ➜  kubectl edit configmap argocd-cm -n argocd
error: configmaps "argocd-cm" is invalid
configmap/argocd-cm edited

myk8scluster ~ ➜  kubectl get configmap argocd-cm -n argocd
NAME        DATA   AGE
argocd-cm   1      14m

myk8scluster ~ ➜  kubectl describe configmap argocd-cm -n argocd
Name:         argocd-cm
Namespace:    argocd
Labels:       app.kubernetes.io/name=argocd-cm
              app.kubernetes.io/part-of=argocd
Annotations:  <none>

Data
====
timeout.reconciliation:
----
60s

BinaryData
====

Events:  <none>




```

```yaml
myk8scluster ~ ➜  kubectl get configmap argocd-cm -n argocd -o yaml
apiVersion: v1
data:
  timeout.reconciliation: 60s
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ConfigMap","metadata":{"annotations":{},"labels":{"app.kubernetes.io/name":"argocd-cm","app.kubernetes.io/part-of":"argocd"},"name":"argocd-cm","namespace":"argocd"}}
  creationTimestamp: "2024-11-14T19:15:32Z"
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
  name: argocd-cm
  namespace: argocd
  resourceVersion: "3090"
  uid: 9140c559-b342-4beb-8e47-f6f3772f413b

myk8scluster ~ ➜  
```

```json
myk8scluster ~ ✖ kubectl get configmap argocd-cm -n argocd -o json
{
    "apiVersion": "v1",
    "data": {
        "timeout.reconciliation": "60s"
    },
    "kind": "ConfigMap",
    "metadata": {
        "annotations": {
            "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"ConfigMap\",\"metadata\":{\"annotations\":{},\"labels\":{\"app.kubernetes.io/name\":\"argocd-cm\",\"app.kubernetes.io/part-of\":\"argocd\"},\"name\":\"argocd-cm\",\"namespace\":\"argocd\"}}\n"
        },
        "creationTimestamp": "2024-11-14T19:15:32Z",
        "labels": {
            "app.kubernetes.io/name": "argocd-cm",
            "app.kubernetes.io/part-of": "argocd"
        },
        "name": "argocd-cm",
        "namespace": "argocd",
        "resourceVersion": "3090",
        "uid": "9140c559-b342-4beb-8e47-f6f3772f413b"
    }
}
```

