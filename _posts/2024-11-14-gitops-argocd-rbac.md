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

# GitOps: ArgoCD User Management

```sh
myk8scluster
myk8scluster ~ ➜  argocd account list
NAME   ENABLED  CAPABILITIES
admin  true     login

myk8scluster ~ ➜  kubectl -n argocd get cm argocd-cm
NAME        DATA   AGE
argocd-cm   0      3m

myk8scluster ~ ➜  kubectl -n argocd patch configmap argocd-cm --patch='{"data":{"accounts.alice": "apiKey,login"}}'
configmap/argocd-cm patched

myk8scluster ~ ➜  kubectl -n argocd get cm argocd-cm
NAME        DATA   AGE
argocd-cm   1      3m39s

myk8scluster ~ ➜  kubectl -n argocd get cm argocd-cm -o yaml
apiVersion: v1
data:
  accounts.alice: apiKey,login
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ConfigMap","metadata":{"annotations":{},"labels":{"app.kubernetes.io/name":"argocd-cm","app.kubernetes.io/part-of":"argocd"},"name":"argocd-cm","namespace":"argocd"}}
  creationTimestamp: "2024-11-15T00:41:54Z"
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
  name: argocd-cm
  namespace: argocd
  resourceVersion: "20866"
  uid: 7a4571d9-866a-4e6d-88d1-eafd073fcee2

myk8scluster ~ ➜  argocd account update-password --account alice
*** Enter password of currently logged in user (admin): 
*** Enter new password for user alice: 
*** Confirm new password for user alice: 
Password updated

myk8scluster ~ ➜  kubectl -n argocd patch configmap argocd-rbac-cm \
--patch='{"data":{"policy.csv":"p, role:create-app, applications, create, *, allow\ng, alice, role:create-app"}}'
configmap/argocd-rbac-cm patched

myk8scluster ~ ➜  argocd app list
NAME  CLUSTER  NAMESPACE  PROJECT  STATUS  HEALTH  SYNCPOLICY  CONDITIONS  REPO  PATH  TARGET

myk8scluster ~ ➜  

myk8scluster ~ ➜  

myk8scluster ~ ➜  argocd app list
NAME  CLUSTER  NAMESPACE  PROJECT  STATUS  HEALTH  SYNCPOLICY  CONDITIONS  REPO  PATH  TARGET

myk8scluster ~ ➜  argocd login $(kubectl get service argocd-server -n argocd --output=jsonpath='{.spec.clusterIP}') --username admin --password admin123 --insecure
argocd app sync demo-app
'admin:login' logged in successfully
Context '172.20.97.232' updated
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2024-11-15T00:50:47+00:00          ConfigMap    demo-app  moving-shapes-colors  OutOfSync  Missing              
2024-11-15T00:50:47+00:00            Service    demo-app     random-shapes-svc  OutOfSync  Missing              
2024-11-15T00:50:47+00:00   apps  Deployment    demo-app         random-shapes  OutOfSync  Missing              
2024-11-15T00:50:49+00:00          Namespace                          demo-app   Running   Synced              namespace/demo-app created
2024-11-15T00:50:49+00:00          ConfigMap    demo-app  moving-shapes-colors    Synced  Missing              
2024-11-15T00:50:49+00:00          Namespace                          demo-app  Succeeded   Synced              namespace/demo-app created
2024-11-15T00:50:49+00:00          ConfigMap    demo-app  moving-shapes-colors    Synced   Missing              configmap/moving-shapes-colors created
2024-11-15T00:50:49+00:00            Service    demo-app     random-shapes-svc  OutOfSync  Missing              service/random-shapes-svc created
2024-11-15T00:50:49+00:00   apps  Deployment    demo-app         random-shapes  OutOfSync  Missing              deployment.apps/random-shapes created
2024-11-15T00:50:49+00:00            Service    demo-app     random-shapes-svc    Synced  Healthy                  service/random-shapes-svc created
2024-11-15T00:50:49+00:00   apps  Deployment    demo-app         random-shapes    Synced  Progressing              deployment.apps/random-shapes created

Name:               argocd/demo-app
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          demo-app
URL:                https://172.20.97.232/applications/demo-app
Source:
- Repo:             https://3000-port-e50156f7de174551.labs.kodekloud.com/bob/gitops-argocd.git
  Target:           HEAD
  Path:             ./health-check
SyncWindow:         Sync Allowed
Sync Policy:        Manual
Sync Status:        Synced to HEAD (2ba6bd1)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      2ba6bd10734152351be692137d4da6b834c22988
Phase:              Succeeded
Start:              2024-11-15 00:50:47 +0000 UTC
Finished:           2024-11-15 00:50:49 +0000 UTC
Duration:           2s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME                  STATUS     HEALTH       HOOK  MESSAGE
       Namespace              demo-app              Succeeded  Synced             namespace/demo-app created
       ConfigMap   demo-app   moving-shapes-colors  Synced                        configmap/moving-shapes-colors created
       Service     demo-app   random-shapes-svc     Synced     Healthy            service/random-shapes-svc created
apps   Deployment  demo-app   random-shapes         Synced     Progressing        deployment.apps/random-shapes created

myk8scluster ~ ➜  argocd account can-i delete applications '*'
no

myk8scluster ~ ➜  
```

