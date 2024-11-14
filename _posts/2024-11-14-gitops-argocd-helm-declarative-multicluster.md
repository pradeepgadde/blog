---
layout: single
title:  "GitOps: ArgoCD Multi-Cluster, Declarative, and Helm-Chart Deployment"
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

# GitOps: ArgoCD Multi-Cluster, Declarative, and Helm-Chart Deployment

```sh

myk8scluster ~ ➜  #argocd repo add https://3000-port-11d24383312e4a70.labs.kodekl
oud.com/bob/gitops-argocd.git

myk8scluster ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
myk8scluster   Ready    control-plane   11h   v1.30.0

myk8scluster ~ ➜  kubectl get all -A
NAMESPACE      NAME                                                   READY   STATUS    RESTARTS   AGE
argocd         pod/argocd-application-controller-0                    1/1     Running   0          2m31s
argocd         pod/argocd-applicationset-controller-d7c857898-l9m7b   1/1     Running   0          2m32s
argocd         pod/argocd-dex-server-75d98bff7c-m2p4j                 1/1     Running   0          2m32s
argocd         pod/argocd-notifications-controller-684947df85-bhfpr   1/1     Running   0          2m32s
argocd         pod/argocd-redis-84c8cd4d8-9vm8x                       1/1     Running   0          2m32s
argocd         pod/argocd-repo-server-6b5cf8488-5852x                 1/1     Running   0          2m30s
argocd         pod/argocd-server-5f8984f889-2zlvb                     1/1     Running   0          2m32s
kube-flannel   pod/kube-flannel-ds-7zh8x                              1/1     Running   0          11h
kube-system    pod/coredns-768b85b76f-kwf7v                           1/1     Running   0          11h
kube-system    pod/coredns-768b85b76f-t5vh9                           1/1     Running   0          11h
kube-system    pod/etcd-myk8scluster                                  1/1     Running   0          11h
kube-system    pod/kube-apiserver-myk8scluster                        1/1     Running   0          11h
kube-system    pod/kube-controller-manager-myk8scluster               1/1     Running   0          11h
kube-system    pod/kube-proxy-gv94h                                   1/1     Running   0          11h
kube-system    pod/kube-scheduler-myk8scluster                        1/1     Running   0          11h

NAMESPACE     NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd        service/argocd-applicationset-controller          ClusterIP   10.108.149.104   <none>        7000/TCP,8080/TCP            2m32s
argocd        service/argocd-dex-server                         ClusterIP   10.98.206.75     <none>        5556/TCP,5557/TCP,5558/TCP   2m32s
argocd        service/argocd-metrics                            ClusterIP   10.109.1.244     <none>        8082/TCP                     2m32s
argocd        service/argocd-notifications-controller-metrics   ClusterIP   10.108.65.180    <none>        9001/TCP                     2m32s
argocd        service/argocd-redis                              ClusterIP   10.111.43.211    <none>        6379/TCP                     2m32s
argocd        service/argocd-repo-server                        ClusterIP   10.102.180.166   <none>        8081/TCP,8084/TCP            2m32s
argocd        service/argocd-server                             NodePort    10.108.175.133   <none>        80:32765/TCP,443:32766/TCP   2m32s
argocd        service/argocd-server-metrics                     ClusterIP   10.99.177.134    <none>        8083/TCP                     2m32s
default       service/kubernetes                                ClusterIP   10.96.0.1        <none>        443/TCP                      11h
kube-system   service/kube-dns                                  ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       11h

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   11h
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   11h

NAMESPACE     NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
argocd        deployment.apps/argocd-applicationset-controller   1/1     1            1           2m32s
argocd        deployment.apps/argocd-dex-server                  1/1     1            1           2m32s
argocd        deployment.apps/argocd-notifications-controller    1/1     1            1           2m32s
argocd        deployment.apps/argocd-redis                       1/1     1            1           2m32s
argocd        deployment.apps/argocd-repo-server                 1/1     1            1           2m32s
argocd        deployment.apps/argocd-server                      1/1     1            1           2m32s
kube-system   deployment.apps/coredns                            2/2     2            2           11h

NAMESPACE     NAME                                                         DESIRED   CURRENT   READY   AGE
argocd        replicaset.apps/argocd-applicationset-controller-d7c857898   1         1         1       2m32s
argocd        replicaset.apps/argocd-dex-server-75d98bff7c                 1         1         1       2m32s
argocd        replicaset.apps/argocd-notifications-controller-684947df85   1         1         1       2m32s
argocd        replicaset.apps/argocd-redis-84c8cd4d8                       1         1         1       2m32s
argocd        replicaset.apps/argocd-repo-server-6b5cf8488                 1         1         1       2m30s
argocd        replicaset.apps/argocd-repo-server-7bbc57875d                0         0         0       2m32s
argocd        replicaset.apps/argocd-server-5f8984f889                     1         1         1       2m32s
kube-system   replicaset.apps/coredns-768b85b76f                           2         2         2       11h

NAMESPACE   NAME                                             READY   AGE
argocd      statefulset.apps/argocd-application-controller   1/1     2m32s

myk8scluster ~ ➜  argocd app list
NAME  CLUSTER  NAMESPACE  PROJECT  STATUS  HEALTH  SYNCPOLICY  CONDITIONS  REPO  PATH  TARGET

myk8scluster ~ ➜  argocd cluster list
SERVER                          NAME        VERSION  STATUS   MESSAGE                                                  PROJECT
https://kubernetes.default.svc  in-cluster           Unknown  Cluster has no applications and is not being monitored.  

myk8scluster ~ ➜  argocd repo list
TYPE  NAME  REPO  INSECURE  OCI  LFS  CREDS  STATUS  MESSAGE  PROJECT

myk8scluster ~ ➜  argocd repo add https://3000-port-11d24383312e4a70.labs.kodeklo
ud.com/bob/gitops-argocd.git
Repository 'https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git' added

myk8scluster ~ ➜  argocd repo list
TYPE  NAME  REPO                                                                         INSECURE  OCI    LFS    CREDS  STATUS      MESSAGE  PROJECT
git         https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  false     false  false  false  Successful           

myk8scluster ~ ➜  argocd app create helm-random-shapes \
--repo https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.gi
t \                  
--path ./helm-chart \
--helm-set color.circle=pink \
--helm-set color.square=red \
--helm-set service.type=NodePort \
--dest-namespace default \
--dest-server https://kubernetes.default.svc
application 'helm-random-shapes' created

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                         NAMESPACE  PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                         PATH          TARGET
argocd/helm-random-shapes  https://kubernetes.default.svc  default    default  OutOfSync  Missing  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart  

myk8scluster ~ ➜  argocd app sync helm-random-shapes
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME            STATUS    HEALTH        HOOK  MESSAGE
2024-11-14T22:32:12+00:00          ConfigMap     default  helm-random-shapes-configmap  OutOfSync  Missing              
2024-11-14T22:32:12+00:00            Service     default  helm-random-shapes-service    OutOfSync  Missing              
2024-11-14T22:32:12+00:00   apps  Deployment     default  helm-random-shapes-deploy     OutOfSync  Missing              
2024-11-14T22:32:13+00:00          ConfigMap     default  helm-random-shapes-configmap    Synced  Missing              
2024-11-14T22:32:13+00:00            Service     default  helm-random-shapes-service    Synced  Healthy              
2024-11-14T22:32:13+00:00            Service     default  helm-random-shapes-service      Synced   Healthy              service/helm-random-shapes-service created
2024-11-14T22:32:13+00:00   apps  Deployment     default  helm-random-shapes-deploy     OutOfSync  Missing              deployment.apps/helm-random-shapes-deploy created
2024-11-14T22:32:13+00:00          ConfigMap     default  helm-random-shapes-configmap    Synced   Missing              configmap/helm-random-shapes-configmap created
2024-11-14T22:32:13+00:00   apps  Deployment     default  helm-random-shapes-deploy    Synced  Progressing              deployment.apps/helm-random-shapes-deploy created

Name:               argocd/helm-random-shapes
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:32766/applications/helm-random-shapes
Source:
- Repo:             https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git
  Target:           
  Path:             ./helm-chart
SyncWindow:         Sync Allowed
Sync Policy:        Manual
Sync Status:        Synced to  (e30d1c2)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      e30d1c2e469d69ee6827ca19890e19729f7df9a6
Phase:              Succeeded
Start:              2024-11-14 22:32:12 +0000 UTC
Finished:           2024-11-14 22:32:13 +0000 UTC
Duration:           1s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME                          STATUS  HEALTH       HOOK  MESSAGE
       ConfigMap   default    helm-random-shapes-configmap  Synced                     configmap/helm-random-shapes-configmap created
       Service     default    helm-random-shapes-service    Synced  Healthy            service/helm-random-shapes-service created
apps   Deployment  default    helm-random-shapes-deploy     Synced  Progressing        deployment.apps/helm-random-shapes-deploy created

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH       SYNCPOLICY  CONDITIONS  REPO                                                                         PATH          TARGET
argocd/helm-random-shapes  https://kubernetes.default.svc  default    default  Synced  Progressing  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart  

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH       SYNCPOLICY  CONDITIONS  REPO                                                                         PATH          TARGET
argocd/helm-random-shapes  https://kubernetes.default.svc  default    default  Synced  Progressing  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart  

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                         PATH          TARGET
argocd/helm-random-shapes  https://kubernetes.default.svc  default    default  Synced  Healthy  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart  

myk8scluster ~ ➜  helm ls
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION

myk8scluster ~ ➜  argocd app get helm-random-shapes
Name:               argocd/helm-random-shapes
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://localhost:32766/applications/helm-random-shapes
Source:
- Repo:             https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git
  Target:           
  Path:             ./helm-chart
SyncWindow:         Sync Allowed
Sync Policy:        Manual
Sync Status:        Synced to  (e30d1c2)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME                          STATUS  HEALTH   HOOK  MESSAGE
       ConfigMap   default    helm-random-shapes-configmap  Synced                 configmap/helm-random-shapes-configmap created
       Service     default    helm-random-shapes-service    Synced  Healthy        service/helm-random-shapes-service created
apps   Deployment  default    helm-random-shapes-deploy     Synced  Healthy        deployment.apps/helm-random-shapes-deploy created

myk8scluster ~ ➜  

myk8scluster ~ ➜  argocd cluster list
SERVER                          NAME        VERSION  STATUS      MESSAGE  PROJECT
https://kubernetes.default.svc  in-cluster  1.30     Successful           

myk8scluster ~ ➜  argocd cluster add cluster2
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `cluster2` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0014] ServiceAccount "argocd-manager" created in namespace "kube-system" 
INFO[0014] ClusterRole "argocd-manager-role" created    
INFO[0014] ClusterRoleBinding "argocd-manager-role-binding" created 
INFO[0014] Created bearer token secret for ServiceAccount "argocd-manager" 
Cluster 'https://cluster2-myk8scluster:6443' added

myk8scluster ~ ➜  argocd cluster list
SERVER                              NAME        VERSION  STATUS      MESSAGE                                                  PROJECT
https://cluster2-myk8scluster:6443  cluster2             Unknown     Cluster has no applications and is not being monitored.  
https://kubernetes.default.svc      in-cluster  1.30     Successful                                                           

myk8scluster ~ ➜  kubectl get contexts
error: the server doesn't have a resource type "contexts"

myk8scluster ~ ✖ kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
          cluster2                      cluster2     cluster2           
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   

myk8scluster ~ ➜  

myk8scluster ~ ➜  kubectl -n argocd get secrets | grep -i cluster
cluster-cluster2-myk8scluster-2268674445   Opaque   3      86s

myk8scluster ~ ➜  kubectl 0n argocd describe secrets cluster-cluster2-myk8scluster-2268674445
error: unknown command "0n" for "kubectl"

Did you mean this?
        run
        cp

myk8scluster ~ ✖ kubectl -n argocd describe secrets cluster-cluster2-myk8scluster
-2268674445
Name:         cluster-cluster2-myk8scluster-2268674445
Namespace:    argocd
Labels:       argocd.argoproj.io/secret-type=cluster
Annotations:  managed-by: argocd.argoproj.io

Type:  Opaque

Data
====
config:  5329 bytes
name:    8 bytes
server:  34 bytes

myk8scluster ~ ➜  kubectl -n argocd describe secrets cluster-cluster2-myk8scluster-2268674445 -o yaml
error: unknown shorthand flag: 'o' in -o
See 'kubectl describe --help' for usage.

myk8scluster ~ ✖ kubectl -n argocd get secrets cluster-cluster2-myk8scluster-2268
674445 -o yaml
apiVersion: v1
data:
  config: eyJ0bHNDbGllbnRDb25maWciOnsiaW5zZWN1cmUiOmZhbHNlLCJjZXJ0RGF0YSI6IkxTMHRMUzFDUlVkSlRpQkRSVkpVU1VaSlEwRlVSUzB0TFMwdENrMUpTVVJMVkVORFFXaEhaMEYzU1VKQlowbEpSMDVxUW5ZeGFVeHpiWGQzUkZGWlNrdHZXa2xvZG1OT1FWRkZURUpSUVhkR1ZFVlVUVUpGUjBFeFZVVUtRWGhOUzJFelZtbGFXRXAxV2xoU2JHTjZRV1ZHZHpCNVRrUkZlRTFVVVhoTlZFRXpUVlJHWVVaM01IbE9WRVY0VFZSUmVFMVVSWGxOVkZKaFRVUjNlQXBJZWtGa1FtZE9Wa0pCYjFSR2JYUXhXVzFXYUZwSE1EWlpNbmd4WXpOU2JHTnBNV2hhUnpGd1ltNU5lRWRVUVZoQ1owNVdRa0ZOVkVWSGRERlpiVlo1Q21KdFZqQmFXRTEwV1ZkU2RHRlhOSGRuWjBWcFRVRXdSME5UY1VkVFNXSXpSRkZGUWtGUlZVRkJORWxDUkhkQmQyZG5SVXRCYjBsQ1FWRkVXVkU0YW5NS05XaDBNMUIzUnpobFdYQk9kMGhZTTNaYVVEUkJNR3RhTW1SemFVNVRlRXc0Um5saWNYVlBaeXRaZFZOS01uVlZLMW93ZFVWcGFtZEhTVFUyY1dKRFNRcHpRbmMyZDNkVWExZFNWRmhDWTBsVVNVaFVZV1JPYVZRM1JqWjRjRVJhVFM5ME4yMXZUVXBNYnpjeWRqRlRaekJCYmxrM2NuQnJOVFpGVGpKM1ZtdEpDa3hKY1dwdGIyVkZTbTl5YlRWeGIyMU9kbXcwYzNwSGJGZDBWSGN2Y1ZCSmMwOU1RMWxRTWt0bmFHdEVlV3h4VVZoVWJrNVpWMlZXYlVwMFprdE5aM2NLUTNadE1rZFBkMDlVZUM5aGIwdFRTRTlVVkRGT1FYUktWbUpXTUVOVWJFWlRUak4xUm1WbGFsUndjbkZtVlZWVFRqWjVNM2h2YjJ4eVpsRkVTWEJaVVFwa01UUnBjMUJTVGtvNFNtMXBTMWwxVHpnemNXcG9jMmhSZDFodFVqQm1RMGt5ZDJOVlNsSnVlRUk1Wmk4emJXMURSMWRHWjFsT1EzZDRhRXBTYlVaTkNrNXlZWE5aV1ZwYU1sRjNOM1pxUVdKQlowMUNRVUZIYWxacVFsVk5RVFJIUVRGVlpFUjNSVUl2ZDFGRlFYZEpSbTlFUVZSQ1owNVdTRk5WUlVSRVFVc0tRbWRuY2tKblJVWkNVV05FUVdwQlRVSm5UbFpJVWsxQ1FXWTRSVUZxUVVGTlFqaEhRVEZWWkVsM1VWbE5RbUZCUmtWSkwxbzBTWFJZUkN0TlFqZEZVUXBNTDBoUVNDOTVkREZ0YkhGTlFUQkhRMU54UjFOSllqTkVVVVZDUTNkVlFVRTBTVUpCVVVOWVVWVnZZVXB5ZVRGYWFFRlVTMlZ6ZG5kR1kxWnhSakpOQ2tkNVJXNHpPVTlVUXpCS09FOXRkbkJsVEVveE0yOWtNMnB3VG1KT1ZESlFkVkpCVnpGTlNsVkNkMUEzVld4eGNXeElTMkphV1RGVFJqbGhSbk5IVmxjS2VWVkZSRzlFWWtkbVJGaDRVWEZsUm5vd056Uk1lR3hwWXpsNWVFeEVTVWxqU25obVlqZHdVamxCSzFGa2JqTjNXa2d3U1VkM1ltWllSR2xEWkVKdFZncHNRemhCTjNkT1FsUlpaekZXVEdKSVkwaFNTMGQwVFVsQlNUTXpTM1V2TkdRNVQxVm5iVXRRSzBsNWVXbGlSV0ZxVUZKcmNGZDZSMk5UVjBWaE1YSlFDbVJoVnpScWVuRjRjREpQV1V3Mk56UjJRbUphVFhoNk0zQTRTbm96VVZoM2VrRXJZbWRPVWk5MmMzWXZVbTV3V2prMlZHOXNaVnBKWVhwYWVqSlpZMHdLV0dsdWNrdG5kRTE2WkhNeWIzUmtjMWhTTkhBMlNIUlJXSEo2YW0xcmRIaGhWekp6VFV0aVVFNUdMMmd4V25GR05GRnBVa0poZGxsRmNWTm1DaTB0TFMwdFJVNUVJRU5GVWxSSlJrbERRVlJGTFMwdExTMEsiLCJrZXlEYXRhIjoiTFMwdExTMUNSVWRKVGlCU1UwRWdVRkpKVmtGVVJTQkxSVmt0TFMwdExRcE5TVWxGYjNkSlFrRkJTME5CVVVWQk1rVlFTVGRQV1dKa2VqaENka2h0UzFSalFqRTVOekpVSzBGT1NrZGtibUpKYWxWelV5OUNZMjAyY21wdlVHMU1DbXRwWkhKc1VHMWtUR2hKYnpSQ2FVOWxjVzEzYVV4QlkwOXpUVVUxUm10Vk1YZFlRMFY1UWpBeWJsUlpheXQ0WlhOaFVUSlVVRGRsTlhGRVExTTJUemtLY2psVmIwNUJTakpQTmpaYVQyVm9SR1J6UmxwRFEzbExielZ4U0doRFlVczFkV0Z4U21waU5XVk1UWGh3Vm5KVk9GQTJhbmxNUkdsM2JVUTVhVzlKV2dwQk9IQmhhMFl3TlhwWFJtNXNXbWxpV0hscVNVMUJjalYwYUdwelJHczRaakp4UTJ0b2Vtc3dPVlJSVEZOV1Z6RmtRV3MxVWxWcVpEZG9XRzV2TURaaENqWnVNVVpGYW1WemREaGhTMHBoTXpCQmVVdFhSVWhrWlVseVJEQlVVMlpEV205cGJVeHFkazQyYnpSaVNWVk5SalZyWkVoM2FVNXpTRVpEVlZvNFVXWUtXQzg1TlhCbmFHeG9XVWRFVVhOTldWTlZXbWhVUkdFeWNrZEhSMWRrYTAxUE56UjNSM2RKUkVGUlFVSkJiMGxDUVVKSFlUSjNWbXcyWjBJMFJWcFZaQXBOT0ZGSGEyUlFZME1yY1ZwdlpEZElNMFV6V214RlUwOVRXWFZ0YVhGak1VTlZZVnBxVDJ4M2RFaHViVzl2YzNwc1NUSndkVXRSTXpabFVtbHpZMmhCQ2psRVFsa3lTMkZxT0hJNVFqSmpkVmsxTVVOQmRFdGljRGRKTVV4emRWWnFZUzlMWmxsSmJIcFdaSEZtTmxCMFRYcEJhMnA2YWt0aVNtRkpVakV3ZUV3S2MzVjZMMEpCVkdWWWMzRlVhemgzU2tzclNpdGhRMnBVVGxoS2NrOXZRUzlKT0V0V1YwVnJXSEl6VjI5dFRWaFNWV1F3VjJGRFkzQkRLemxKVldsbFpncHNNVkIwTkdnNVZHYzFhaXRvUmpZM1pVZEhUVWRZWkdWMlFrWkxUa2RyYUVRNVpGaFBhSFJEUlRCMVdHdEplVXB0TVVrMlUyUXZVRzlVVTA5bFVsWmxDbEZhTkVjeFdGSmpTbTlYYnpsVWEyRmhVVkVyU0d4NldtOXBWVFpGV1ZCYVMyNDFTbTgyVFZOQmNtRlpTRnBSTjFaeGVHMDBiVXM1ZVVRd1JVOVhjMVFLVTJoS1ZHSlBhME5uV1VWQk5GVnlNWFoyYjJoc0szaDFTMmRJVW1oR05XUktZMDl4UnpkWGEyMWxjM0ZUYms5RlZXaENLMG8yVVVWbGJsUnZWbFZUYkFwRFdFTm9NV1p6UkdOVmVFSlhjMk00Ykd4M01URnVSRTFGYkVoa04wMUlUa3RTUkVWSWFXSk5iVTlYU2xGeVpHNVhORkJMUjFsUEt6Vk5SbTVvYWt0VkNrdFhkVWhKWWtwbFdsZGpjbWhMVFc1Uk9VeHBaRFl2V0doRE1GaG1ZMWMxUm1GS1lraHBabUp1YlhaWGJtMWhNSGhoYTBwUlZUaERaMWxGUVRsaU0xRUtXazlXU1VVMmNVTlhWMDA1WW1KeFJIbGpSRzh2ZW0xalZHZFhMek16V0VOcVdVNUJUVU5hUTB0M1FuQkJlWEZQTVd0dFNUZGpRMWh6Y0daWFIyRkhlUXB5UlhKRGVEZGpiVWREU2k5R2MyMU9Ra2xzYkdoSE5tSXZXVkYzVGtVelFpOUlSQzlwT0hJclNUbHpRMFZvT1ZCUFMzcHFZamhUWjBVdlZrdFhOM1F5Q21kRk9WcHhhRVZ6TmtkelMxbFJZbnB6TmpWbmRsZERNbWRrVVhKc2ExQlNhWFF2VEdWWVZVTm5XVUp2YXpWblVUVjVRM2t5WVN0c1ZtbFVRVTFsUmtJS1FqRnpkRGcxYjA1VVIycGpSMFpWUTJ4ME9VbFlWVEJ4TTNCMEswRlNaM1ZyY0dFNGMyZERLMFpoZURocEswVnRNVzVUZWt4eVlUZG9hamRwYVVjNVNRcE5XakJ1WkdSRlp6UXJUbU5GWWtGeVptSm9VazFoVG5CdmFFVkJOMUJrZWxwd1RtRlpLMkZJV1ZkWFJUaDNjbXcwTjNveE1qWjZTbXRRYWtocVFVWkhDamx2T0daS01XTTVZVlZqT1ZSbFZqVnVkVnBLTlhkTFFtZFJSSFoxZUV4WWRYY3lUR3RsV0VSNWFIRmtUbm94YkhwMmJpdE5ibVZZWW1Wb01qSlBla29LU0U5bE4ydFVZM1oxUTBNMU5VRlRaMkV2UWxjNFNFZE5NbkU0VmtwcVpXd3JVRVF6TkVkWllsZFdkbVZKVWxkbEwxZExOWGhUV1UxT1lUZEpPVFp3Y3dveVRtVlVlWEEwVHpSS1MzVnNlbWw2Y3pSWVZTOVdObXN5WTBOSGNVOVROekkyTUM5VFlUUnFkRzVxVkcxMlMxaHpWaTlUYmpoNWNqZDNNM1ZqTkRaNUNrTm5lalJIVVV0Q1owRmlOM0ZuTVZoNlRGWnhOVVZETjAxalJrUmxTWG92V0hGb1RIWkdXVXBLVHpWWVlXVmhXV2d2UjJwb05XZHBiM0ZRWW5saFprc0tOREJYUlZweWNFVTBlSGN2WW05NVdVTkVTWHBzV2tSTlQzWlNLMmhQVUhkM2IyUTJWM1J1WjJOemJHTk1NazB2VkhaRU15OW1kbWRtYkM4dk4yVkRaQXA2VXpsNGRrMURZa3N4ZFdKRmMyNHJaMGxOVjNkMVpFaHBUbEZoSzJGa2JIQlhiVU13VDFjeFJHUTVUMjUzYUd4NVVETlFDaTB0TFMwdFJVNUVJRkpUUVNCUVVrbFdRVlJGSUV0RldTMHRMUzB0Q2c9PSIsImNhRGF0YSI6IkxTMHRMUzFDUlVkSlRpQkRSVkpVU1VaSlEwRlVSUzB0TFMwdENrMUpTVVJDVkVORFFXVXlaMEYzU1VKQlowbEpUV2RyU0ZwU1FURkdLM2QzUkZGWlNrdHZXa2xvZG1OT1FWRkZURUpSUVhkR1ZFVlVUVUpGUjBFeFZVVUtRWGhOUzJFelZtbGFXRXAxV2xoU2JHTjZRV1ZHZHpCNVRrUkZlRTFVVVhoTlZFRXpUVlJHWVVaM01IcE9SRVY0VFZSSmVFMVVSWGxOVkVaaFRVSlZlQXBGZWtGU1FtZE9Wa0pCVFZSRGJYUXhXVzFXZVdKdFZqQmFXRTEzWjJkRmFVMUJNRWREVTNGSFUwbGlNMFJSUlVKQlVWVkJRVFJKUWtSM1FYZG5aMFZMQ2tGdlNVSkJVVU0yYzNWaVR6STVOa2xRVkZJMEsyMVZVUzl0VUROaEszZ3pkaTg0VFV0ek1qWjRUR1ZIUm1wSFRITnlPRlJXUTFRcmFXcEZhbVV5U0dFS1VuVXZjRVpxZEZSb2MyNVlNbmRuYW1SYVZsWXJWbTFpZDBJeGNFazNia3B6TVM5aFYyaEdNVWxLWkc4dlZWWmhTa3N3TTBsUWFWQkVUbEF5TlhCWVJBbzRORTUxWmxWa0wwVjZZM0JhU0ZSaVNrWnFRVTlrZEhGblUwdGxReXRCUmt0RVFuZHJNM05hVkhKeFVsVjRNVGxIVlVOM2EySlFjRUpYWldsSWFGZzJDa2s1UWxKbloxSjVUbTFSU1hSd1JVTm1kbEZCVEZKRWFITlJOM05hZEROUVZGaHdVRXBzT1hoalVqTllXVGRMUkhkT1FVSkxWRUZ2T1U1cFlYaFZWbm9LUjFSTU5VcDNhM1l3T1RCa2QxaHJLMmhNVTNkNmQwcHBVbEI2YkdKQloxaFRZMWx6ZVZWUmEyTmtRVzFDT1daeU1VWnFhM1pKV2xWSFpVRm5WVkpVWWdvemJubHlSVFJaU1RsNFRWRkdhMWxyUVVGb0wxTlhRMFZ1WlhOc1FXZE5Ra0ZCUjJwWFZFSllUVUUwUjBFeFZXUkVkMFZDTDNkUlJVRjNTVU53UkVGUUNrSm5UbFpJVWsxQ1FXWTRSVUpVUVVSQlVVZ3ZUVUl3UjBFeFZXUkVaMUZYUWtKU1ExQXlaVU5NVm5jdmFrRmxlRVZETDNoNmVDODRjbVJhY0dGcVFWWUtRbWRPVmtoU1JVVkVha0ZOWjJkd2NtUlhTbXhqYlRWc1pFZFdlazFCTUVkRFUzRkhVMGxpTTBSUlJVSkRkMVZCUVRSSlFrRlJRM0I1ZUhBeFVEZEZaZ3BvVDFRMFNTdGhSV2RNUVRKTUszb3dWbHB2YTFCbFZubG5jMVpyY0RkUk1USXhOMVpOUlc1WGJqSnFiVlI1Y3poM1MyNHlkRzF5U1VOMWNWQTFTbU5RQ21SeVJYZEpaVXBYZEVvNU1WZFdaVlZKVjFWT1dqZEpPVkpZU0hCNk5EVXlka3RDTjFKYVlXWXJkM1psVmxWVFRtOTJOV05uY2tvcmRsSkVlbGgyVWtNS2VqYzFUVFZuWTJKQ1dFeHZkRFpyWWtkWmRtNVJZVlZQWVVsSlRuVnJaRFJYYzB4VlExVlBOWFZuYjBaMVpYcHJWa3hXWkU1WFZrZHRSMDA1VVZwWlFncGtUVWhvT0dNcmRtdzBkSEI1UjFRNVJpdEZVMDVGWlVFdmJHRkJhMFJaVlVaR04yOVVXa1JxTHpSWVZqVTFRWFpRTDAxNVVFdDZZMWxIZEZoUk4weFdDa1l2TWxjd1VXSldWRlF2WWpReVdFbHRNRmw2UVZkM1NrdHVkRXRqZFVkU2VFSnpaSGhwUzJOeFNFNUJNR1V5VG1abkwwRnZZblJUUzBwMFUwY3dla2tLZUZwMFdEVkVXblpKU1RKQkNpMHRMUzB0UlU1RUlFTkZVbFJKUmtsRFFWUkZMUzB0TFMwSyJ9fQ==
  name: Y2x1c3RlcjI=
  server: aHR0cHM6Ly9jbHVzdGVyMi1jb250cm9scGxhbmU6NjQ0Mw==
kind: Secret
metadata:
  annotations:
    managed-by: argocd.argoproj.io
  creationTimestamp: "2024-11-14T22:36:34Z"
  labels:
    argocd.argoproj.io/secret-type: cluster
  name: cluster-cluster2-myk8scluster-2268674445
  namespace: argocd
  resourceVersion: "54143"
  uid: 66c10f77-aa0c-4abb-bf90-18ddd379a5aa
type: Opaque

myk8scluster ~ ➜  kubectl -n argocd get secrets cluster-cluster2-myk8scluster-2268674445 -o json
{
    "apiVersion": "v1",
    "data": {
        "config": "eyJ0bHNDbGllbnRDb25maWciOnsiaW5zZWN1cmUiOmZhbHNlLCJjZXJ0RGF0YSI6IkxTMHRMUzFDUlVkSlRpQkRSVkpVU1VaSlEwRlVSUzB0TFMwdENrMUpTVVJMVkVORFFXaEhaMEYzU1VKQlowbEpSMDVxUW5ZeGFVeHpiWGQzUkZGWlNrdHZXa2xvZG1OT1FWRkZURUpSUVhkR1ZFVlVUVUpGUjBFeFZVVUtRWGhOUzJFelZtbGFXRXAxV2xoU2JHTjZRV1ZHZHpCNVRrUkZlRTFVVVhoTlZFRXpUVlJHWVVaM01IbE9WRVY0VFZSUmVFMVVSWGxOVkZKaFRVUjNlQXBJZWtGa1FtZE9Wa0pCYjFSR2JYUXhXVzFXYUZwSE1EWlpNbmd4WXpOU2JHTnBNV2hhUnpGd1ltNU5lRWRVUVZoQ1owNVdRa0ZOVkVWSGRERlpiVlo1Q21KdFZqQmFXRTEwV1ZkU2RHRlhOSGRuWjBWcFRVRXdSME5UY1VkVFNXSXpSRkZGUWtGUlZVRkJORWxDUkhkQmQyZG5SVXRCYjBsQ1FWRkVXVkU0YW5NS05XaDBNMUIzUnpobFdYQk9kMGhZTTNaYVVEUkJNR3RhTW1SemFVNVRlRXc0Um5saWNYVlBaeXRaZFZOS01uVlZLMW93ZFVWcGFtZEhTVFUyY1dKRFNRcHpRbmMyZDNkVWExZFNWRmhDWTBsVVNVaFVZV1JPYVZRM1JqWjRjRVJhVFM5ME4yMXZUVXBNYnpjeWRqRlRaekJCYmxrM2NuQnJOVFpGVGpKM1ZtdEpDa3hKY1dwdGIyVkZTbTl5YlRWeGIyMU9kbXcwYzNwSGJGZDBWSGN2Y1ZCSmMwOU1RMWxRTWt0bmFHdEVlV3h4VVZoVWJrNVpWMlZXYlVwMFprdE5aM2NLUTNadE1rZFBkMDlVZUM5aGIwdFRTRTlVVkRGT1FYUktWbUpXTUVOVWJFWlRUak4xUm1WbGFsUndjbkZtVlZWVFRqWjVNM2h2YjJ4eVpsRkVTWEJaVVFwa01UUnBjMUJTVGtvNFNtMXBTMWwxVHpnemNXcG9jMmhSZDFodFVqQm1RMGt5ZDJOVlNsSnVlRUk1Wmk4emJXMURSMWRHWjFsT1EzZDRhRXBTYlVaTkNrNXlZWE5aV1ZwYU1sRjNOM1pxUVdKQlowMUNRVUZIYWxacVFsVk5RVFJIUVRGVlpFUjNSVUl2ZDFGRlFYZEpSbTlFUVZSQ1owNVdTRk5WUlVSRVFVc0tRbWRuY2tKblJVWkNVV05FUVdwQlRVSm5UbFpJVWsxQ1FXWTRSVUZxUVVGTlFqaEhRVEZWWkVsM1VWbE5RbUZCUmtWSkwxbzBTWFJZUkN0TlFqZEZVUXBNTDBoUVNDOTVkREZ0YkhGTlFUQkhRMU54UjFOSllqTkVVVVZDUTNkVlFVRTBTVUpCVVVOWVVWVnZZVXB5ZVRGYWFFRlVTMlZ6ZG5kR1kxWnhSakpOQ2tkNVJXNHpPVTlVUXpCS09FOXRkbkJsVEVveE0yOWtNMnB3VG1KT1ZESlFkVkpCVnpGTlNsVkNkMUEzVld4eGNXeElTMkphV1RGVFJqbGhSbk5IVmxjS2VWVkZSRzlFWWtkbVJGaDRVWEZsUm5vd056Uk1lR3hwWXpsNWVFeEVTVWxqU25obVlqZHdVamxCSzFGa2JqTjNXa2d3U1VkM1ltWllSR2xEWkVKdFZncHNRemhCTjNkT1FsUlpaekZXVEdKSVkwaFNTMGQwVFVsQlNUTXpTM1V2TkdRNVQxVm5iVXRRSzBsNWVXbGlSV0ZxVUZKcmNGZDZSMk5UVjBWaE1YSlFDbVJoVnpScWVuRjRjREpQV1V3Mk56UjJRbUphVFhoNk0zQTRTbm96VVZoM2VrRXJZbWRPVWk5MmMzWXZVbTV3V2prMlZHOXNaVnBKWVhwYWVqSlpZMHdLV0dsdWNrdG5kRTE2WkhNeWIzUmtjMWhTTkhBMlNIUlJXSEo2YW0xcmRIaGhWekp6VFV0aVVFNUdMMmd4V25GR05GRnBVa0poZGxsRmNWTm1DaTB0TFMwdFJVNUVJRU5GVWxSSlJrbERRVlJGTFMwdExTMEsiLCJrZXlEYXRhIjoiTFMwdExTMUNSVWRKVGlCU1UwRWdVRkpKVmtGVVJTQkxSVmt0TFMwdExRcE5TVWxGYjNkSlFrRkJTME5CVVVWQk1rVlFTVGRQV1dKa2VqaENka2h0UzFSalFqRTVOekpVSzBGT1NrZGtibUpKYWxWelV5OUNZMjAyY21wdlVHMU1DbXRwWkhKc1VHMWtUR2hKYnpSQ2FVOWxjVzEzYVV4QlkwOXpUVVUxUm10Vk1YZFlRMFY1UWpBeWJsUlpheXQ0WlhOaFVUSlVVRGRsTlhGRVExTTJUemtLY2psVmIwNUJTakpQTmpaYVQyVm9SR1J6UmxwRFEzbExielZ4U0doRFlVczFkV0Z4U21waU5XVk1UWGh3Vm5KVk9GQTJhbmxNUkdsM2JVUTVhVzlKV2dwQk9IQmhhMFl3TlhwWFJtNXNXbWxpV0hscVNVMUJjalYwYUdwelJHczRaakp4UTJ0b2Vtc3dPVlJSVEZOV1Z6RmtRV3MxVWxWcVpEZG9XRzV2TURaaENqWnVNVVpGYW1WemREaGhTMHBoTXpCQmVVdFhSVWhrWlVseVJEQlVVMlpEV205cGJVeHFkazQyYnpSaVNWVk5SalZyWkVoM2FVNXpTRVpEVlZvNFVXWUtXQzg1TlhCbmFHeG9XVWRFVVhOTldWTlZXbWhVUkdFeWNrZEhSMWRrYTAxUE56UjNSM2RKUkVGUlFVSkJiMGxDUVVKSFlUSjNWbXcyWjBJMFJWcFZaQXBOT0ZGSGEyUlFZME1yY1ZwdlpEZElNMFV6V214RlUwOVRXWFZ0YVhGak1VTlZZVnBxVDJ4M2RFaHViVzl2YzNwc1NUSndkVXRSTXpabFVtbHpZMmhCQ2psRVFsa3lTMkZxT0hJNVFqSmpkVmsxTVVOQmRFdGljRGRKTVV4emRWWnFZUzlMWmxsSmJIcFdaSEZtTmxCMFRYcEJhMnA2YWt0aVNtRkpVakV3ZUV3S2MzVjZMMEpCVkdWWWMzRlVhemgzU2tzclNpdGhRMnBVVGxoS2NrOXZRUzlKT0V0V1YwVnJXSEl6VjI5dFRWaFNWV1F3VjJGRFkzQkRLemxKVldsbFpncHNNVkIwTkdnNVZHYzFhaXRvUmpZM1pVZEhUVWRZWkdWMlFrWkxUa2RyYUVRNVpGaFBhSFJEUlRCMVdHdEplVXB0TVVrMlUyUXZVRzlVVTA5bFVsWmxDbEZhTkVjeFdGSmpTbTlYYnpsVWEyRmhVVkVyU0d4NldtOXBWVFpGV1ZCYVMyNDFTbTgyVFZOQmNtRlpTRnBSTjFaeGVHMDBiVXM1ZVVRd1JVOVhjMVFLVTJoS1ZHSlBhME5uV1VWQk5GVnlNWFoyYjJoc0szaDFTMmRJVW1oR05XUktZMDl4UnpkWGEyMWxjM0ZUYms5RlZXaENLMG8yVVVWbGJsUnZWbFZUYkFwRFdFTm9NV1p6UkdOVmVFSlhjMk00Ykd4M01URnVSRTFGYkVoa04wMUlUa3RTUkVWSWFXSk5iVTlYU2xGeVpHNVhORkJMUjFsUEt6Vk5SbTVvYWt0VkNrdFhkVWhKWWtwbFdsZGpjbWhMVFc1Uk9VeHBaRFl2V0doRE1GaG1ZMWMxUm1GS1lraHBabUp1YlhaWGJtMWhNSGhoYTBwUlZUaERaMWxGUVRsaU0xRUtXazlXU1VVMmNVTlhWMDA1WW1KeFJIbGpSRzh2ZW0xalZHZFhMek16V0VOcVdVNUJUVU5hUTB0M1FuQkJlWEZQTVd0dFNUZGpRMWh6Y0daWFIyRkhlUXB5UlhKRGVEZGpiVWREU2k5R2MyMU9Ra2xzYkdoSE5tSXZXVkYzVGtVelFpOUlSQzlwT0hJclNUbHpRMFZvT1ZCUFMzcHFZamhUWjBVdlZrdFhOM1F5Q21kRk9WcHhhRVZ6TmtkelMxbFJZbnB6TmpWbmRsZERNbWRrVVhKc2ExQlNhWFF2VEdWWVZVTm5XVUp2YXpWblVUVjVRM2t5WVN0c1ZtbFVRVTFsUmtJS1FqRnpkRGcxYjA1VVIycGpSMFpWUTJ4ME9VbFlWVEJ4TTNCMEswRlNaM1ZyY0dFNGMyZERLMFpoZURocEswVnRNVzVUZWt4eVlUZG9hamRwYVVjNVNRcE5XakJ1WkdSRlp6UXJUbU5GWWtGeVptSm9VazFoVG5CdmFFVkJOMUJrZWxwd1RtRlpLMkZJV1ZkWFJUaDNjbXcwTjNveE1qWjZTbXRRYWtocVFVWkhDamx2T0daS01XTTVZVlZqT1ZSbFZqVnVkVnBLTlhkTFFtZFJSSFoxZUV4WWRYY3lUR3RsV0VSNWFIRmtUbm94YkhwMmJpdE5ibVZZWW1Wb01qSlBla29LU0U5bE4ydFVZM1oxUTBNMU5VRlRaMkV2UWxjNFNFZE5NbkU0VmtwcVpXd3JVRVF6TkVkWllsZFdkbVZKVWxkbEwxZExOWGhUV1UxT1lUZEpPVFp3Y3dveVRtVlVlWEEwVHpSS1MzVnNlbWw2Y3pSWVZTOVdObXN5WTBOSGNVOVROekkyTUM5VFlUUnFkRzVxVkcxMlMxaHpWaTlUYmpoNWNqZDNNM1ZqTkRaNUNrTm5lalJIVVV0Q1owRmlOM0ZuTVZoNlRGWnhOVVZETjAxalJrUmxTWG92V0hGb1RIWkdXVXBLVHpWWVlXVmhXV2d2UjJwb05XZHBiM0ZRWW5saFprc0tOREJYUlZweWNFVTBlSGN2WW05NVdVTkVTWHBzV2tSTlQzWlNLMmhQVUhkM2IyUTJWM1J1WjJOemJHTk1NazB2VkhaRU15OW1kbWRtYkM4dk4yVkRaQXA2VXpsNGRrMURZa3N4ZFdKRmMyNHJaMGxOVjNkMVpFaHBUbEZoSzJGa2JIQlhiVU13VDFjeFJHUTVUMjUzYUd4NVVETlFDaTB0TFMwdFJVNUVJRkpUUVNCUVVrbFdRVlJGSUV0RldTMHRMUzB0Q2c9PSIsImNhRGF0YSI6IkxTMHRMUzFDUlVkSlRpQkRSVkpVU1VaSlEwRlVSUzB0TFMwdENrMUpTVVJDVkVORFFXVXlaMEYzU1VKQlowbEpUV2RyU0ZwU1FURkdLM2QzUkZGWlNrdHZXa2xvZG1OT1FWRkZURUpSUVhkR1ZFVlVUVUpGUjBFeFZVVUtRWGhOUzJFelZtbGFXRXAxV2xoU2JHTjZRV1ZHZHpCNVRrUkZlRTFVVVhoTlZFRXpUVlJHWVVaM01IcE9SRVY0VFZSSmVFMVVSWGxOVkVaaFRVSlZlQXBGZWtGU1FtZE9Wa0pCVFZSRGJYUXhXVzFXZVdKdFZqQmFXRTEzWjJkRmFVMUJNRWREVTNGSFUwbGlNMFJSUlVKQlVWVkJRVFJKUWtSM1FYZG5aMFZMQ2tGdlNVSkJVVU0yYzNWaVR6STVOa2xRVkZJMEsyMVZVUzl0VUROaEszZ3pkaTg0VFV0ek1qWjRUR1ZIUm1wSFRITnlPRlJXUTFRcmFXcEZhbVV5U0dFS1VuVXZjRVpxZEZSb2MyNVlNbmRuYW1SYVZsWXJWbTFpZDBJeGNFazNia3B6TVM5aFYyaEdNVWxLWkc4dlZWWmhTa3N3TTBsUWFWQkVUbEF5TlhCWVJBbzRORTUxWmxWa0wwVjZZM0JhU0ZSaVNrWnFRVTlrZEhGblUwdGxReXRCUmt0RVFuZHJNM05hVkhKeFVsVjRNVGxIVlVOM2EySlFjRUpYWldsSWFGZzJDa2s1UWxKbloxSjVUbTFSU1hSd1JVTm1kbEZCVEZKRWFITlJOM05hZEROUVZGaHdVRXBzT1hoalVqTllXVGRMUkhkT1FVSkxWRUZ2T1U1cFlYaFZWbm9LUjFSTU5VcDNhM1l3T1RCa2QxaHJLMmhNVTNkNmQwcHBVbEI2YkdKQloxaFRZMWx6ZVZWUmEyTmtRVzFDT1daeU1VWnFhM1pKV2xWSFpVRm5WVkpVWWdvemJubHlSVFJaU1RsNFRWRkdhMWxyUVVGb0wxTlhRMFZ1WlhOc1FXZE5Ra0ZCUjJwWFZFSllUVUUwUjBFeFZXUkVkMFZDTDNkUlJVRjNTVU53UkVGUUNrSm5UbFpJVWsxQ1FXWTRSVUpVUVVSQlVVZ3ZUVUl3UjBFeFZXUkVaMUZYUWtKU1ExQXlaVU5NVm5jdmFrRmxlRVZETDNoNmVDODRjbVJhY0dGcVFWWUtRbWRPVmtoU1JVVkVha0ZOWjJkd2NtUlhTbXhqYlRWc1pFZFdlazFCTUVkRFUzRkhVMGxpTTBSUlJVSkRkMVZCUVRSSlFrRlJRM0I1ZUhBeFVEZEZaZ3BvVDFRMFNTdGhSV2RNUVRKTUszb3dWbHB2YTFCbFZubG5jMVpyY0RkUk1USXhOMVpOUlc1WGJqSnFiVlI1Y3poM1MyNHlkRzF5U1VOMWNWQTFTbU5RQ21SeVJYZEpaVXBYZEVvNU1WZFdaVlZKVjFWT1dqZEpPVkpZU0hCNk5EVXlka3RDTjFKYVlXWXJkM1psVmxWVFRtOTJOV05uY2tvcmRsSkVlbGgyVWtNS2VqYzFUVFZuWTJKQ1dFeHZkRFpyWWtkWmRtNVJZVlZQWVVsSlRuVnJaRFJYYzB4VlExVlBOWFZuYjBaMVpYcHJWa3hXWkU1WFZrZHRSMDA1VVZwWlFncGtUVWhvT0dNcmRtdzBkSEI1UjFRNVJpdEZVMDVGWlVFdmJHRkJhMFJaVlVaR04yOVVXa1JxTHpSWVZqVTFRWFpRTDAxNVVFdDZZMWxIZEZoUk4weFdDa1l2TWxjd1VXSldWRlF2WWpReVdFbHRNRmw2UVZkM1NrdHVkRXRqZFVkU2VFSnpaSGhwUzJOeFNFNUJNR1V5VG1abkwwRnZZblJUUzBwMFUwY3dla2tLZUZwMFdEVkVXblpKU1RKQkNpMHRMUzB0UlU1RUlFTkZVbFJKUmtsRFFWUkZMUzB0TFMwSyJ9fQ==",
        "name": "Y2x1c3RlcjI=",
        "server": "aHR0cHM6Ly9jbHVzdGVyMi1jb250cm9scGxhbmU6NjQ0Mw=="
    },
    "kind": "Secret",
    "metadata": {
        "annotations": {
            "managed-by": "argocd.argoproj.io"
        },
        "creationTimestamp": "2024-11-14T22:36:34Z",
        "labels": {
            "argocd.argoproj.io/secret-type": "cluster"
        },
        "name": "cluster-cluster2-myk8scluster-2268674445",
        "namespace": "argocd",
        "resourceVersion": "54143",
        "uid": "66c10f77-aa0c-4abb-bf90-18ddd379a5aa"
    },
    "type": "Opaque"
}

myk8scluster ~ ➜  kubectl -n argocd get secrets cluster-cluster2-myk8scluster-2268674445 -o json | jq .data.server 
"aHR0cHM6Ly9jbHVzdGVyMi1jb250cm9scGxhbmU6NjQ0Mw=="

myk8scluster ~ ➜  kubectl -n argocd get secrets cluster-cluster2-myk8scluster-2268674445 -o json | jq .data.server -r
aHR0cHM6Ly9jbHVzdGVyMi1jb250cm9scGxhbmU6NjQ0Mw==

myk8scluster ~ ➜  kubectl -n argocd get secrets cluster-cluster2-myk8scluster-2268674445 -o json | jq .data.server -r | base64 -d
https://cluster2-myk8scluster:6443
myk8scluster ~ ➜  kubectl -n argocd get secrets
NAME                                       TYPE     DATA   AGE
argocd-notifications-secret                Opaque   0      17m
argocd-secret                              Opaque   5      17m
cluster-cluster2-myk8scluster-2268674445   Opaque   3      3m22s
repo-1779328080                            Opaque   2      14m

myk8scluster ~ ➜  vi health-check-app.yaml

myk8scluster ~ ➜  cat health-check-app.yaml 
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: health-check-app
spec:
  destination:
    name: ''
    namespace: health-check
    server: 'https://cluster2-myk8scluster:6443'
  source:
    path: /health-check
    repoURL: 'https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd'
    targetRevision: HEAD
  project: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=true


myk8scluster ~ ➜  kubectl apply -f health-check-app.yaml 
application.argoproj.io/health-check-app created

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                         PATH          TARGET
argocd/helm-random-shapes  https://kubernetes.default.svc  default    default  Synced  Healthy  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart  

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                         PATH          TARGET
argocd/helm-random-shapes  https://kubernetes.default.svc  default    default  Synced  Healthy  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart  

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                         PATH          TARGET
argocd/helm-random-shapes  https://kubernetes.default.svc  default    default  Synced  Healthy  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart  

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                             NAMESPACE     PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                         PATH            TARGET
argocd/health-check-app    https://cluster2-myk8scluster:6443  health-check  default  OutOfSync  Missing  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd      ./health-check  HEAD
argocd/helm-random-shapes  https://kubernetes.default.svc      default       default  Synced     Healthy  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart    

myk8scluster ~ ➜  argocd app sync health-check-app
TIMESTAMP                  GROUP        KIND   NAMESPACE                    NAME    STATUS    HEALTH        HOOK  MESSAGE
2024-11-14T22:45:01+00:00          ConfigMap  health-check  moving-shapes-colors  OutOfSync  Missing              
2024-11-14T22:45:01+00:00            Service  health-check     random-shapes-svc  OutOfSync  Missing              
2024-11-14T22:45:01+00:00   apps  Deployment  health-check         random-shapes  OutOfSync  Missing              
2024-11-14T22:45:03+00:00          Namespace                      health-check   Running   Synced              namespace/health-check created
2024-11-14T22:45:03+00:00          ConfigMap  health-check  moving-shapes-colors    Synced  Missing              
2024-11-14T22:45:03+00:00            Service  health-check     random-shapes-svc    Synced  Healthy              
2024-11-14T22:45:03+00:00            Service  health-check     random-shapes-svc    Synced   Healthy              service/random-shapes-svc created
2024-11-14T22:45:03+00:00   apps  Deployment  health-check         random-shapes  OutOfSync  Missing              deployment.apps/random-shapes created
2024-11-14T22:45:03+00:00          Namespace                        health-check  Succeeded   Synced              namespace/health-check created
2024-11-14T22:45:03+00:00          ConfigMap  health-check  moving-shapes-colors    Synced   Missing              configmap/moving-shapes-colors created
2024-11-14T22:45:03+00:00   apps  Deployment  health-check         random-shapes    Synced  Progressing              deployment.apps/random-shapes created

Name:               argocd/health-check-app
Project:            default
Server:             https://cluster2-myk8scluster:6443
Namespace:          health-check
URL:                https://localhost:32766/applications/health-check-app
Source:
- Repo:             https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd
  Target:           HEAD
  Path:             ./health-check
SyncWindow:         Sync Allowed
Sync Policy:        Manual
Sync Status:        Synced to HEAD (e30d1c2)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      e30d1c2e469d69ee6827ca19890e19729f7df9a6
Phase:              Succeeded
Start:              2024-11-14 22:45:01 +0000 UTC
Finished:           2024-11-14 22:45:03 +0000 UTC
Duration:           2s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE     NAME                  STATUS     HEALTH       HOOK  MESSAGE
       Namespace                 health-check          Succeeded  Synced             namespace/health-check created
       ConfigMap   health-check  moving-shapes-colors  Synced                        configmap/moving-shapes-colors created
       Service     health-check  random-shapes-svc     Synced     Healthy            service/random-shapes-svc created
apps   Deployment  health-check  random-shapes         Synced     Progressing        deployment.apps/random-shapes created

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                             NAMESPACE     PROJECT  STATUS  HEALTH       SYNCPOLICY  CONDITIONS  REPO                                                                         PATH            TARGET
argocd/health-check-app    https://cluster2-myk8scluster:6443  health-check  default  Synced  Progressing  Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd      ./health-check  HEAD
argocd/helm-random-shapes  https://kubernetes.default.svc      default       default  Synced  Healthy      Manual      <none>      https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart    

myk8scluster ~ ➜  curl http://cluster2-myk8scluster:30335
<!DOCTYPE html>
<html>
   <head>
      <script type="text/javascript" src="https://code.jquery.com/jquery-1.7.1.min.js"></script>
      <script>
                        function setMarker() {
                          var height = $(window).height() - 50;
                          var width = $(window).width() - 50;
                          var newheight = Math.floor(Math.random() * height);
                          var newwidth = Math.floor(Math.random() * width);
                          return [newheight, newwidth];
                        }

                        $(document).ready(function () {
                          shapeSlider(".circle");
                          shapeSlider(".oval");
                          shapeSlider(".square");
                          shapeSlider(".triangle");
                          shapeSlider(".rectangle");
                        });

                        function shapeSlider(shape) {
                          var mark = setMarker();
                          $(shape).animate({ top: mark[0], left: mark[1] }, 5000, function () {
                                shapeSlider(shape);
                          });
                        }
      </script>


      <title>ArgoCD Health Demo</title>


      <style>
         body {
            background-image: url("background.png")
         }
         div.circle {
                         width: 100px;
                         height: 100px;
                         background-color: pink;
                         position: fixed;
                         border-radius: 50%;
         }
         div.oval {
                         height: 100px;
                         width: 150px;
                         background-color: lightgreen;
                         border-radius: 50%;
                         position: fixed;
         }
         div.square {
                         width: 100px;
                         height: 100px;
                         background-color: orange;
                         position: fixed;
         }
         div.triangle {
                         width: 0;
                         height: 0;
                         border-left: 100px solid transparent;
                         border-right: 100px solid transparent;
                         border-bottom: 100px solid red;
                         position: fixed;
         }
         div.rectangle {
                         height: 100px;
                         width: 150px;
                         background-color: blue;
                         position: fixed;
         }
      </style>
   </head>


   <body>
      <div class='circle'></div>
      <div class='oval'></div>
      <div class='square'></div>
      <div class='triangle'></div>
      <div class='rectangle'></div>
   </body>
</html>

myk8scluster ~ ➜  kubectl get all -n health-check
No resources found in health-check namespace.

myk8scluster ~ ➜  kubectl get all -n healthcheck
No resources found in healthcheck namespace.

myk8scluster ~ ➜  kubectl get ns
NAME              STATUS   AGE
argocd            Active   23m
default           Active   11h
kube-flannel      Active   11h
kube-node-lease   Active   11h
kube-public       Active   11h
kube-system       Active   11h

myk8scluster ~ ➜  kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
          cluster2                      cluster2     cluster2           
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   

myk8scluster ~ ➜  kubectl config set-context cluster2
Context "cluster2" modified.

myk8scluster ~ ➜  kubectl get ns
NAME              STATUS   AGE
argocd            Active   23m
default           Active   11h
kube-flannel      Active   11h
kube-node-lease   Active   11h
kube-public       Active   11h
kube-system       Active   11h

myk8scluster ~ ➜  kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
          cluster2                      cluster2     cluster2           
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   

myk8scluster ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

myk8scluster ~ ➜  kubectl get ns
NAME              STATUS   AGE
default           Active   11h
health-check      Active   117s
kube-flannel      Active   11h
kube-node-lease   Active   11h
kube-public       Active   11h
kube-system       Active   11h

myk8scluster ~ ➜  kubectl get all -n health-check
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-749d854f5f-wcktq   1/1     Running   0          2m2s

NAME                        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   10.106.222.177   <none>        80:30335/TCP   2m2s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   1/1     1            1           2m2s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-749d854f5f   1         1         1       2m2s

myk8scluster ~ ➜  kubectl get nodes
NAME                    STATUS   ROLES           AGE   VERSION
cluster2-myk8scluster   Ready    control-plane   11h   v1.29.0

myk8scluster ~ ➜  kubectl config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         cluster2                      cluster2     cluster2           
          kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   

myk8scluster ~ ➜  kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".

myk8scluster ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
myk8scluster   Ready    control-plane   11h   v1.30.0

myk8scluster ~ ➜  kubectl apply -f kubectl apply -f https://3000-port-11d24383312e4a70.labs.kodekloud.com//bob/gitops-argocd/raw/branch/master/declarative/multi-a
pp/app-of-apps.yml -n argocd
error: Unexpected args: [apply]
See 'kubectl apply -h' for help and examples

myk8scluster ~ ✖  kubectl apply -f https://3000-port-11d24383312e4a70.labs.kodekl
oud.com//bob/gitops-argocd/raw/branch/master/declarative/multi-app/app-of-apps.ym
l -n argocd
application.argoproj.io/app-of-apps created

myk8scluster ~ ➜  kubectl get all -n argocd
NAME                                                   READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0                    1/1     Running   0          28m
pod/argocd-applicationset-controller-d7c857898-l9m7b   1/1     Running   0          28m
pod/argocd-dex-server-75d98bff7c-m2p4j                 1/1     Running   0          28m
pod/argocd-notifications-controller-684947df85-bhfpr   1/1     Running   0          28m
pod/argocd-redis-84c8cd4d8-9vm8x                       1/1     Running   0          28m
pod/argocd-repo-server-6b5cf8488-5852x                 1/1     Running   0          28m
pod/argocd-server-5f8984f889-2zlvb                     1/1     Running   0          28m

NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP   10.108.149.104   <none>        7000/TCP,8080/TCP            28m
service/argocd-dex-server                         ClusterIP   10.98.206.75     <none>        5556/TCP,5557/TCP,5558/TCP   28m
service/argocd-metrics                            ClusterIP   10.109.1.244     <none>        8082/TCP                     28m
service/argocd-notifications-controller-metrics   ClusterIP   10.108.65.180    <none>        9001/TCP                     28m
service/argocd-redis                              ClusterIP   10.111.43.211    <none>        6379/TCP                     28m
service/argocd-repo-server                        ClusterIP   10.102.180.166   <none>        8081/TCP,8084/TCP            28m
service/argocd-server                             NodePort    10.108.175.133   <none>        80:32765/TCP,443:32766/TCP   28m
service/argocd-server-metrics                     ClusterIP   10.99.177.134    <none>        8083/TCP                     28m

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           28m
deployment.apps/argocd-dex-server                  1/1     1            1           28m
deployment.apps/argocd-notifications-controller    1/1     1            1           28m
deployment.apps/argocd-redis                       1/1     1            1           28m
deployment.apps/argocd-repo-server                 1/1     1            1           28m
deployment.apps/argocd-server                      1/1     1            1           28m

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-d7c857898   1         1         1       28m
replicaset.apps/argocd-dex-server-75d98bff7c                 1         1         1       28m
replicaset.apps/argocd-notifications-controller-684947df85   1         1         1       28m
replicaset.apps/argocd-redis-84c8cd4d8                       1         1         1       28m
replicaset.apps/argocd-repo-server-6b5cf8488                 1         1         1       28m
replicaset.apps/argocd-repo-server-7bbc57875d                0         0         0       28m
replicaset.apps/argocd-server-5f8984f889                     1         1         1       28m

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     28m

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                             NAMESPACE     PROJECT  STATUS   HEALTH   SYNCPOLICY  CONDITIONS       REPO                                                                         PATH                       TARGET
argocd/app-of-apps         https://kubernetes.default.svc      argocd        default  Unknown  Healthy  Auto-Prune  ComparisonError  http://165.22.209.118:3000/siddharth/gitops-argocd.git                       ./declarative/app-of-apps  HEAD
argocd/health-check-app    https://cluster2-myk8scluster:6443  health-check  default  Synced   Healthy  Manual      <none>           https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd      ./health-check             HEAD
argocd/helm-random-shapes  https://kubernetes.default.svc      default       default  Synced   Healthy  Manual      <none>           https://3000-port-11d24383312e4a70.labs.kodekloud.com/bob/gitops-argocd.git  ./helm-chart               

myk8scluster ~ ➜  
```

