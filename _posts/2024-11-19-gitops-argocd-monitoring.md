---
layout: single
title:  "GitOps: ArgoCD Monitoring"
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

# GitOps: ArgoCD Monitoring

```sh

myk8scluster ~ ➜  kubectl get svc -n argocd
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   172.20.222.174   <none>        7000/TCP,8080/TCP            14m
argocd-dex-server                         ClusterIP   172.20.19.28     <none>        5556/TCP,5557/TCP,5558/TCP   14m
argocd-metrics                            NodePort    172.20.98.129    <none>        8082:30010/TCP               14m
argocd-notifications-controller-metrics   ClusterIP   172.20.197.148   <none>        9001/TCP                     14m
argocd-redis                              ClusterIP   172.20.209.156   <none>        6379/TCP                     14m
argocd-repo-server                        ClusterIP   172.20.117.66    <none>        8081/TCP,8084/TCP            14m
argocd-server                             NodePort    172.20.16.76     <none>        80:32765/TCP,443:32766/TCP   14m
argocd-server-metrics                     ClusterIP   172.20.58.35     <none>        8083/TCP                     14m

myk8scluster ~ ➜  vi argocd-server-metrics.yaml

myk8scluster ~ ➜  vi argocd-repo-server-metrics.yaml

myk8scluster ~ ➜  kubectl apply -f argocd-metrics.yaml
kubectl apply -f argocd-server-metrics.yaml
kubectl apply -f argocd-repo-server-metrics.yaml
kubectl apply -f argocd-applicationset-controller-metrics.yaml
error: the path "argocd-metrics.yaml" does not exist
servicemonitor.monitoring.coreos.com/argocd-server-metrics created
servicemonitor.monitoring.coreos.com/argocd-repo-server-metrics created
error: the path "argocd-applicationset-controller-metrics.yaml" does not exist

myk8scluster ~ ✖ vi argocd-repo-server-metrics.yaml

myk8scluster ~ ➜  vi argocd-applicationset-controller-metrics.yaml

myk8scluster ~ ➜  kubectl apply -f argocd-metrics.yaml
kubectl apply -f argocd-server-metrics.yaml
kubectl apply -f argocd-repo-server-metrics.yaml
kubectl apply -f argocd-applicationset-controller-metrics.yaml
error: the path "argocd-metrics.yaml" does not exist
servicemonitor.monitoring.coreos.com/argocd-server-metrics unchanged
servicemonitor.monitoring.coreos.com/argocd-repo-server-metrics unchanged
servicemonitor.monitoring.coreos.com/argocd-applicationset-controller-metrics created

myk8scluster ~ ➜  vi argocd-metrics.yaml

myk8scluster ~ ➜  kubectl apply -f argocd-metrics.yaml
kubectl apply -f argocd-server-metrics.yaml
kubectl apply -f argocd-repo-server-metrics.yaml
kubectl apply -f argocd-applicationset-controller-metrics.yaml
servicemonitor.monitoring.coreos.com/argocd-metrics created
servicemonitor.monitoring.coreos.com/argocd-server-metrics unchanged
servicemonitor.monitoring.coreos.com/argocd-repo-server-metrics unchanged
servicemonitor.monitoring.coreos.com/argocd-applicationset-controller-metrics unchanged

myk8scluster ~ ➜  kubectl -n monitoring get secrets kode-kloud-prometheus-stack-grafana -o json | jq .data'."admin-password"' -r | base64 -d
prom-operator
myk8scluster ~ ➜  kubectl -n monitoring edit prometheusrules kode-kloud-prometheus-stac-alertmanager.rules
prometheusrule.monitoring.coreos.com/kode-kloud-prometheus-stac-alertmanager.rules edited

myk8scluster ~ ➜  argocd app sync alert-manager-demo
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2024-11-20T00:06:58+00:00            Service  alert-demo  solar-system-service  OutOfSync  Missing              
2024-11-20T00:06:58+00:00   apps  Deployment  alert-demo          solar-system  OutOfSync  Missing              
2024-11-20T00:07:01+00:00          Namespace                        alert-demo   Running   Synced              namespace/alert-demo created
2024-11-20T00:07:01+00:00            Service  alert-demo  solar-system-service    Synced  Healthy              
2024-11-20T00:07:01+00:00          Namespace                        alert-demo  Succeeded   Synced              namespace/alert-demo created
2024-11-20T00:07:01+00:00            Service  alert-demo  solar-system-service    Synced   Healthy              service/solar-system-service created
2024-11-20T00:07:01+00:00   apps  Deployment  alert-demo          solar-system  OutOfSync  Missing              deployment.apps/solar-system created
2024-11-20T00:07:01+00:00   apps  Deployment  alert-demo          solar-system    Synced  Progressing              deployment.apps/solar-system created

Name:               argocd/alert-manager-demo
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          alert-demo
URL:                https://localhost:32766/applications/alert-manager-demo
Source:
- Repo:             https://3000-port-87abf9f8fa094706.labs.kodekloud.com/bob/gitops-argocd.git
  Target:           HEAD
  Path:             ./solar-system
SyncWindow:         Sync Allowed
Sync Policy:        Manual
Sync Status:        Synced to HEAD (67ba87b)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      67ba87b6489b41ac59b8d20a0c72c3510a383c24
Phase:              Succeeded
Start:              2024-11-20 00:06:58 +0000 UTC
Finished:           2024-11-20 00:07:01 +0000 UTC
Duration:           3s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE   NAME                  STATUS     HEALTH       HOOK  MESSAGE
       Namespace               alert-demo            Succeeded  Synced             namespace/alert-demo created
       Service     alert-demo  solar-system-service  Synced     Healthy            service/solar-system-service created
apps   Deployment  alert-demo  solar-system          Synced     Progressing        deployment.apps/solar-system created

myk8scluster ~ ➜  kubectl get ns
NAME              STATUS   AGE
alert-demo        Active   60s
argocd            Active   42m
default           Active   132m
kube-flannel      Active   132m
kube-node-lease   Active   132m
kube-public       Active   132m
kube-system       Active   132m
monitoring        Active   41m

myk8scluster ~ ➜  kubectl get all -n monitoring
NAME                                                                 READY   STATUS                 RESTARTS   AGE
pod/alertmanager-kode-kloud-prometheus-stac-alertmanager-0           2/2     Running                0          40m
pod/kode-kloud-prometheus-stac-operator-7588657779-l87c6             1/1     Running                0          41m
pod/kode-kloud-prometheus-stack-grafana-7f857dddbf-dp9lm             3/3     Running                0          41m
pod/kode-kloud-prometheus-stack-kube-state-metrics-ff4bc8f8c-m92jv   1/1     Running                0          41m
pod/kode-kloud-prometheus-stack-prometheus-node-exporter-pkgkr       0/1     CreateContainerError   0          41m
pod/prometheus-kode-kloud-prometheus-stac-prometheus-0               2/2     Running                0          40m

NAME                                                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-operated                                  ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   40m
service/kode-kloud-prometheus-stac-alertmanager                NodePort    172.20.99.216    <none>        9093:30030/TCP               41m
service/kode-kloud-prometheus-stac-operator                    ClusterIP   172.20.203.254   <none>        443/TCP                      41m
service/kode-kloud-prometheus-stac-prometheus                  NodePort    172.20.104.155   <none>        9090:30040/TCP               41m
service/kode-kloud-prometheus-stack-grafana                    NodePort    172.20.157.179   <none>        80:30050/TCP                 41m
service/kode-kloud-prometheus-stack-kube-state-metrics         ClusterIP   172.20.126.86    <none>        8080/TCP                     41m
service/kode-kloud-prometheus-stack-prometheus-node-exporter   ClusterIP   172.20.87.90     <none>        9100/TCP                     41m
service/prometheus-operated                                    ClusterIP   None             <none>        9090/TCP                     40m

NAME                                                                  DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/kode-kloud-prometheus-stack-prometheus-node-exporter   1         1         0       1            0           <none>          41m

NAME                                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kode-kloud-prometheus-stac-operator              1/1     1            1           41m
deployment.apps/kode-kloud-prometheus-stack-grafana              1/1     1            1           41m
deployment.apps/kode-kloud-prometheus-stack-kube-state-metrics   1/1     1            1           41m

NAME                                                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/kode-kloud-prometheus-stac-operator-7588657779             1         1         1       41m
replicaset.apps/kode-kloud-prometheus-stack-grafana-7f857dddbf             1         1         1       41m
replicaset.apps/kode-kloud-prometheus-stack-kube-state-metrics-ff4bc8f8c   1         1         1       41m

NAME                                                                    READY   AGE
statefulset.apps/alertmanager-kode-kloud-prometheus-stac-alertmanager   1/1     40m
statefulset.apps/prometheus-kode-kloud-prometheus-stac-prometheus       1/1     40m

myk8scluster ~ ➜  kubectl get all -A
NAMESPACE      NAME                                                                 READY   STATUS                 RESTARTS   AGE
alert-demo     pod/solar-system-5f67dbcbb8-426dg                                    1/1     Running                0          78s
alert-demo     pod/solar-system-5f67dbcbb8-4mp7g                                    1/1     Running                0          78s
alert-demo     pod/solar-system-5f67dbcbb8-xgfqf                                    1/1     Running                0          78s
argocd         pod/argocd-application-controller-0                                  1/1     Running                0          42m
argocd         pod/argocd-applicationset-controller-d7c857898-fj49v                 1/1     Running                0          42m
argocd         pod/argocd-dex-server-75d98bff7c-d6w6v                               1/1     Running                0          42m
argocd         pod/argocd-notifications-controller-684947df85-bpvtk                 1/1     Running                0          42m
argocd         pod/argocd-redis-84c8cd4d8-62hhh                                     1/1     Running                0          42m
argocd         pod/argocd-repo-server-6b5cf8488-8z6hv                               1/1     Running                0          42m
argocd         pod/argocd-server-5f8984f889-pq8tf                                   1/1     Running                0          42m
default        pod/vault-0                                                          1/1     Running                0          42m
default        pod/vault-agent-injector-7bf4c9d98-dn2dq                             1/1     Running                0          42m
kube-flannel   pod/kube-flannel-ds-jvz4k                                            1/1     Running                0          132m
kube-system    pod/coredns-768b85b76f-gsp6g                                         1/1     Running                0          132m
kube-system    pod/coredns-768b85b76f-r8njq                                         1/1     Running                0          132m
kube-system    pod/etcd-myk8scluster                                                1/1     Running                0          132m
kube-system    pod/kube-apiserver-myk8scluster                                      1/1     Running                0          132m
kube-system    pod/kube-controller-manager-myk8scluster                             1/1     Running                0          132m
kube-system    pod/kube-proxy-x54fg                                                 1/1     Running                0          132m
kube-system    pod/kube-scheduler-myk8scluster                                      1/1     Running                0          132m
monitoring     pod/alertmanager-kode-kloud-prometheus-stac-alertmanager-0           2/2     Running                0          40m
monitoring     pod/kode-kloud-prometheus-stac-operator-7588657779-l87c6             1/1     Running                0          41m
monitoring     pod/kode-kloud-prometheus-stack-grafana-7f857dddbf-dp9lm             3/3     Running                0          41m
monitoring     pod/kode-kloud-prometheus-stack-kube-state-metrics-ff4bc8f8c-m92jv   1/1     Running                0          41m
monitoring     pod/kode-kloud-prometheus-stack-prometheus-node-exporter-pkgkr       0/1     CreateContainerError   0          41m
monitoring     pod/prometheus-kode-kloud-prometheus-stac-prometheus-0               2/2     Running                0          40m

NAMESPACE     NAME                                                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                        AGE
alert-demo    service/solar-system-service                                   NodePort    172.20.187.71    <none>        80:30333/TCP                   78s
argocd        service/argocd-applicationset-controller                       ClusterIP   172.20.222.174   <none>        7000/TCP,8080/TCP              42m
argocd        service/argocd-dex-server                                      ClusterIP   172.20.19.28     <none>        5556/TCP,5557/TCP,5558/TCP     42m
argocd        service/argocd-metrics                                         NodePort    172.20.98.129    <none>        8082:30010/TCP                 42m
argocd        service/argocd-notifications-controller-metrics                ClusterIP   172.20.197.148   <none>        9001/TCP                       42m
argocd        service/argocd-redis                                           ClusterIP   172.20.209.156   <none>        6379/TCP                       42m
argocd        service/argocd-repo-server                                     ClusterIP   172.20.117.66    <none>        8081/TCP,8084/TCP              42m
argocd        service/argocd-server                                          NodePort    172.20.16.76     <none>        80:32765/TCP,443:32766/TCP     42m
argocd        service/argocd-server-metrics                                  ClusterIP   172.20.58.35     <none>        8083/TCP                       42m
default       service/kubernetes                                             ClusterIP   172.20.0.1       <none>        443/TCP                        132m
default       service/vault                                                  ClusterIP   172.20.127.36    <none>        8200/TCP,8201/TCP              42m
default       service/vault-agent-injector-svc                               ClusterIP   172.20.127.201   <none>        443/TCP                        42m
default       service/vault-internal                                         ClusterIP   None             <none>        8200/TCP,8201/TCP              42m
default       service/vault-ui                                               NodePort    172.20.239.25    <none>        8200:30711/TCP                 42m
kube-system   service/kode-kloud-prometheus-stac-coredns                     ClusterIP   None             <none>        9153/TCP                       41m
kube-system   service/kode-kloud-prometheus-stac-kube-controller-manager     ClusterIP   None             <none>        10257/TCP                      41m
kube-system   service/kode-kloud-prometheus-stac-kube-etcd                   ClusterIP   None             <none>        2379/TCP                       41m
kube-system   service/kode-kloud-prometheus-stac-kube-proxy                  ClusterIP   None             <none>        10249/TCP                      41m
kube-system   service/kode-kloud-prometheus-stac-kube-scheduler              ClusterIP   None             <none>        10259/TCP                      41m
kube-system   service/kode-kloud-prometheus-stac-kubelet                     ClusterIP   None             <none>        10250/TCP,10255/TCP,4194/TCP   40m
kube-system   service/kube-dns                                               ClusterIP   172.20.0.10      <none>        53/UDP,53/TCP,9153/TCP         132m
monitoring    service/alertmanager-operated                                  ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP     40m
monitoring    service/kode-kloud-prometheus-stac-alertmanager                NodePort    172.20.99.216    <none>        9093:30030/TCP                 41m
monitoring    service/kode-kloud-prometheus-stac-operator                    ClusterIP   172.20.203.254   <none>        443/TCP                        41m
monitoring    service/kode-kloud-prometheus-stac-prometheus                  NodePort    172.20.104.155   <none>        9090:30040/TCP                 41m
monitoring    service/kode-kloud-prometheus-stack-grafana                    NodePort    172.20.157.179   <none>        80:30050/TCP                   41m
monitoring    service/kode-kloud-prometheus-stack-kube-state-metrics         ClusterIP   172.20.126.86    <none>        8080/TCP                       41m
monitoring    service/kode-kloud-prometheus-stack-prometheus-node-exporter   ClusterIP   172.20.87.90     <none>        9100/TCP                       41m
monitoring    service/prometheus-operated                                    ClusterIP   None             <none>        9090/TCP                       40m

NAMESPACE      NAME                                                                  DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds                                        1         1         1       1            1           <none>                   132m
kube-system    daemonset.apps/kube-proxy                                             1         1         1       1            1           kubernetes.io/os=linux   132m
monitoring     daemonset.apps/kode-kloud-prometheus-stack-prometheus-node-exporter   1         1         0       1            0           <none>                   41m

NAMESPACE     NAME                                                             READY   UP-TO-DATE   AVAILABLE   AGE
alert-demo    deployment.apps/solar-system                                     3/3     3            3           78s
argocd        deployment.apps/argocd-applicationset-controller                 1/1     1            1           42m
argocd        deployment.apps/argocd-dex-server                                1/1     1            1           42m
argocd        deployment.apps/argocd-notifications-controller                  1/1     1            1           42m
argocd        deployment.apps/argocd-redis                                     1/1     1            1           42m
argocd        deployment.apps/argocd-repo-server                               1/1     1            1           42m
argocd        deployment.apps/argocd-server                                    1/1     1            1           42m
default       deployment.apps/vault-agent-injector                             1/1     1            1           42m
kube-system   deployment.apps/coredns                                          2/2     2            2           132m
monitoring    deployment.apps/kode-kloud-prometheus-stac-operator              1/1     1            1           41m
monitoring    deployment.apps/kode-kloud-prometheus-stack-grafana              1/1     1            1           41m
monitoring    deployment.apps/kode-kloud-prometheus-stack-kube-state-metrics   1/1     1            1           41m

NAMESPACE     NAME                                                                       DESIRED   CURRENT   READY   AGE
alert-demo    replicaset.apps/solar-system-5f67dbcbb8                                    3         3         3       78s
argocd        replicaset.apps/argocd-applicationset-controller-d7c857898                 1         1         1       42m
argocd        replicaset.apps/argocd-dex-server-75d98bff7c                               1         1         1       42m
argocd        replicaset.apps/argocd-notifications-controller-684947df85                 1         1         1       42m
argocd        replicaset.apps/argocd-redis-84c8cd4d8                                     1         1         1       42m
argocd        replicaset.apps/argocd-repo-server-6b5cf8488                               1         1         1       42m
argocd        replicaset.apps/argocd-repo-server-7bbc57875d                              0         0         0       42m
argocd        replicaset.apps/argocd-server-5f8984f889                                   1         1         1       42m
default       replicaset.apps/vault-agent-injector-7bf4c9d98                             1         1         1       42m
kube-system   replicaset.apps/coredns-768b85b76f                                         2         2         2       132m
monitoring    replicaset.apps/kode-kloud-prometheus-stac-operator-7588657779             1         1         1       41m
monitoring    replicaset.apps/kode-kloud-prometheus-stack-grafana-7f857dddbf             1         1         1       41m
monitoring    replicaset.apps/kode-kloud-prometheus-stack-kube-state-metrics-ff4bc8f8c   1         1         1       41m

NAMESPACE    NAME                                                                    READY   AGE
argocd       statefulset.apps/argocd-application-controller                          1/1     42m
default      statefulset.apps/vault                                                  1/1     42m
monitoring   statefulset.apps/alertmanager-kode-kloud-prometheus-stac-alertmanager   1/1     40m
monitoring   statefulset.apps/prometheus-kode-kloud-prometheus-stac-prometheus       1/1     40m

myk8scluster ~ ➜  
```

