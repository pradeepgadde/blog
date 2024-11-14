---
layout: single
title:  "GitOps: ArgoCD Custom Healthcheck"
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

# GitOps: ArgoCD Custom Healthcheck

In this post, let us create a custom healthcheck for a configmap.

```sh

myk8scluster ~ ➜  kubectl get cm -n argocd
NAME                        DATA   AGE
argocd-cm                   0      4m38s
argocd-cmd-params-cm        0      4m38s
argocd-gpg-keys-cm          0      4m38s
argocd-notifications-cm     0      4m38s
argocd-rbac-cm              0      4m38s
argocd-ssh-known-hosts-cm   1      4m38s
argocd-tls-certs-cm         0      4m38s
kube-root-ca.crt            1      4m41s

myk8scluster ~ ➜  kubectl get cm -n health-check
NAME                   DATA   AGE
kube-root-ca.crt       1      72s
moving-shapes-colors   5      70s

myk8scluster ~ ➜  kubectl get cm moving-shapes-colors -n health-check -o json
{
    "apiVersion": "v1",
    "data": {
        "CIRCLE_COLOR": "pink",
        "OVAL_COLOR": "lightgreen",
        "RECTANGLE_COLOR": "blue",
        "SQUARE_COLOR": "orange",
        "TRIANGLE_COLOR": "white"
    },
    "kind": "ConfigMap",
    "metadata": {
        "annotations": {
            "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"data\":{\"CIRCLE_COLOR\":\"pink\",\"OVAL_COLOR\":\"lightgreen\",\"RECTANGLE_COLOR\":\"blue\",\"SQUARE_COLOR\":\"orange\",\"TRIANGLE_COLOR\":\"white\"},\"kind\":\"ConfigMap\",\"metadata\":{\"annotations\":{},\"labels\":{\"app.kubernetes.io/instance\":\"health-check-app\"},\"name\":\"moving-shapes-colors\",\"namespace\":\"health-check\"}}\n"
        },
        "creationTimestamp": "2024-11-14T20:23:34Z",
        "labels": {
            "app.kubernetes.io/instance": "health-check-app"
        },
        "name": "moving-shapes-colors",
        "namespace": "health-check",
        "resourceVersion": "4277",
        "uid": "f8b5fb9c-976c-4055-8e0e-dbf492695f33"
    }
}

myk8scluster ~ ➜  kubectl get cm moving-shapes-colors -n health-check -o yaml
apiVersion: v1
data:
  CIRCLE_COLOR: pink
  OVAL_COLOR: lightgreen
  RECTANGLE_COLOR: blue
  SQUARE_COLOR: orange
  TRIANGLE_COLOR: white
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"CIRCLE_COLOR":"pink","OVAL_COLOR":"lightgreen","RECTANGLE_COLOR":"blue","SQUARE_COLOR":"orange","TRIANGLE_COLOR":"white"},"kind":"ConfigMap","metadata":{"annotations":{},"labels":{"app.kubernetes.io/instance":"health-check-app"},"name":"moving-shapes-colors","namespace":"health-check"}}
  creationTimestamp: "2024-11-14T20:23:34Z"
  labels:
    app.kubernetes.io/instance: health-check-app
  name: moving-shapes-colors
  namespace: health-check
  resourceVersion: "4277"
  uid: f8b5fb9c-976c-4055-8e0e-dbf492695f33

myk8scluster ~ ➜  kubectl edit configmap argocd-cm -n argocd
Edit cancelled, no changes made.

myk8scluster ~ ➜  kubectl edit configmap argocd-cm -n argocd
configmap/argocd-cm edited

myk8scluster ~ ➜  kubectl get configmap argocd-cm -n argocd -o yaml
apiVersion: v1
data:
  resource.customizations.health.ConfigMap: |
    hs = {}
    hs.status = "Healthy"
      if obj.data.TRIANGLE_COLOR == "white" then
         hs.status = "Degraded"
        hs.message = "Use any color other than White "
      end
    return hs
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ConfigMap","metadata":{"annotations":{},"labels":{"app.kubernetes.io/name":"argocd-cm","app.kubernetes.io/part-of":"argocd"},"name":"argocd-cm","namespace":"argocd"}}
  creationTimestamp: "2024-11-14T20:20:01Z"
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
  name: argocd-cm
  namespace: argocd
  resourceVersion: "4610"
  uid: 86512525-a527-495e-97cf-4656f27c8ccc

myk8scluster ~ ➜  kubectl get configmap argocd-cm -n argocd -o json
{
    "apiVersion": "v1",
    "data": {
        "resource.customizations.health.ConfigMap": "hs = {}\nhs.status = \"Healthy\"\n  if obj.data.TRIANGLE_COLOR == \"white\" then\n     hs.status = \"Degraded\"\n    hs.message = \"Use any color other than White \"\n  end\nreturn hs\n"
    },
    "kind": "ConfigMap",
    "metadata": {
        "annotations": {
            "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"ConfigMap\",\"metadata\":{\"annotations\":{},\"labels\":{\"app.kubernetes.io/name\":\"argocd-cm\",\"app.kubernetes.io/part-of\":\"argocd\"},\"name\":\"argocd-cm\",\"namespace\":\"argocd\"}}\n"
        },
        "creationTimestamp": "2024-11-14T20:20:01Z",
        "labels": {
            "app.kubernetes.io/name": "argocd-cm",
            "app.kubernetes.io/part-of": "argocd"
        },
        "name": "argocd-cm",
        "namespace": "argocd",
        "resourceVersion": "4610",
        "uid": "86512525-a527-495e-97cf-4656f27c8ccc"
    }
}

myk8scluster ~ ➜  kubectl get all -n health-check
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          9m12s

NAME                        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   172.20.73.201   <none>        80:30335/TCP   9m12s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   1/1     1            1           9m12s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   1         1         1       9m12s

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          9m17s

NAME                        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   172.20.73.201   <none>        80:30335/TCP   9m17s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   1/1     1            1           9m17s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   1         1         1       9m17s

myk8scluster ~ ➜  watch kubectl get all -n health-check 

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          10m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          19s

NAME                        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   172.20.73.201   <none>        80:30335/TCP   10m

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           10m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       10m

myk8scluster ~ ➜  argocd app list
NAME                     CLUSTER                         NAMESPACE     PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                          PATH            TARGET
argocd/health-check-app  https://kubernetes.default.svc  health-check  default  Synced  Healthy  Auto-Prune  <none>      https://3000-port-d5e0b782421d4dcd.labs.kodekloud.com//bob/gitops-argocd.git  ./health-check  HEAD

myk8scluster ~ ➜  argocd app -h
Manage applications

Usage:
  argocd app [flags]
  argocd app [command]

Examples:
  # List all the applications.
  argocd app list

  # Get the details of a application
  argocd app get my-app

  # Set an override parameter
  argocd app set my-app -p image.tag=v1.0.1

Available Commands:
  actions         Manage Resource actions
  add-source      Adds a source to the list of sources in the application
  create          Create an application
  delete          Delete an application
  delete-resource Delete resource in an application
  diff            Perform a diff against the target and live state.
  edit            Edit application
  get             Get application details
  history         Show application deployment history
  list            List applications
  logs            Get logs of application pods
  manifests       Print manifests of an application
  patch           Patch application
  patch-resource  Patch resource in an application
  remove-source   Remove a source from multiple sources application. Counting starts with 1. Default value is -1.
  resources       List resource of application
  rollback        Rollback application to a previous deployed version by History ID, omitted will Rollback to the previous version
  set             Set application parameters
  sync            Sync an application to its target state
  terminate-op    Terminate running operation of an application
  unset           Unset application parameters
  wait            Wait for an application to reach a synced and healthy state

Flags:
      --as string                      Username to impersonate for the operation
      --as-group stringArray           Group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --as-uid string                  UID to impersonate for the operation
      --certificate-authority string   Path to a cert file for the certificate authority
      --client-certificate string      Path to a client certificate file for TLS
      --client-key string              Path to a client key file for TLS
      --cluster string                 The name of the kubeconfig cluster to use
      --context string                 The name of the kubeconfig context to use
      --disable-compression            If true, opt-out of response compression for all requests to the server
  -h, --help                           help for app
      --insecure-skip-tls-verify       If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kubeconfig string              Path to a kube config. Only required if out-of-cluster
  -n, --namespace string               If present, the namespace scope for this CLI request
      --password string                Password for basic authentication to the API server
      --proxy-url string               If provided, this URL will be used to connect via proxy
      --request-timeout string         The length of time to wait before giving up on a single server request. Non-zero values should contain a corresponding time unit (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests. (default "0")
      --tls-server-name string         If provided, this name will be used to validate server certificate. If this is not provided, hostname used to contact the server is used.
      --token string                   Bearer token for authentication to the API server
      --user string                    The name of the kubeconfig user to use
      --username string                Username for basic authentication to the API server

Global Flags:
      --auth-token string               Authentication token
      --client-crt string               Client certificate file
      --client-crt-key string           Client certificate key file
      --config string                   Path to Argo CD config (default "/root/.config/argocd/config")
      --controller-name string          Name of the Argo CD Application controller; set this or the ARGOCD_APPLICATION_CONTROLLER_NAME environment variable when the controller's name label differs from the default, for example when installing via the Helm chart (default "argocd-application-controller")
      --core                            If set to true then CLI talks directly to Kubernetes instead of talking to Argo CD API server
      --grpc-web                        Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2.
      --grpc-web-root-path string       Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2. Set web root.
  -H, --header strings                  Sets additional header to all requests made by Argo CD CLI. (Can be repeated multiple times to add multiple headers, also supports comma separated headers)
      --http-retry-max int              Maximum number of retries to establish http connection to Argo CD server
      --insecure                        Skip server certificate and domain verification
      --kube-context string             Directs the command to the given kube-context
      --logformat string                Set the logging format. One of: text|json (default "text")
      --loglevel string                 Set the logging level. One of: debug|info|warn|error (default "info")
      --plaintext                       Disable TLS
      --port-forward                    Connect to a random argocd-server port using port forwarding
      --port-forward-namespace string   Namespace name which should be used for port forwarding
      --redis-haproxy-name string       Name of the Redis HA Proxy; set this or the ARGOCD_REDIS_HAPROXY_NAME environment variable when the HA Proxy's name label differs from the default, for example when installing via the Helm chart (default "argocd-redis-ha-haproxy")
      --redis-name string               Name of the Redis deployment; set this or the ARGOCD_REDIS_NAME environment variable when the Redis's name label differs from the default, for example when installing via the Helm chart (default "argocd-redis")
      --repo-server-name string         Name of the Argo CD Repo server; set this or the ARGOCD_REPO_SERVER_NAME environment variable when the server's name label differs from the default, for example when installing via the Helm chart (default "argocd-repo-server")
      --server string                   Argo CD server address
      --server-crt string               Server certificate file
      --server-name string              Name of the Argo CD API server; set this or the ARGOCD_SERVER_NAME environment variable when the server's name label differs from the default, for example when installing via the Helm chart (default "argocd-server")

Use "argocd app [command] --help" for more information about a command.

myk8scluster ~ ➜  argocd app list
NAME                     CLUSTER                         NAMESPACE     PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                                          PATH            TARGET
argocd/health-check-app  https://kubernetes.default.svc  health-check  default  Synced  Healthy  Auto-Prune  <none>      https://3000-port-d5e0b782421d4dcd.labs.kodekloud.com//bob/gitops-argocd.git  ./health-check  HEAD

myk8scluster ~ ➜  argocd app get argocd/health-check-app
Name:               argocd/health-check-app
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          health-check
URL:                https://localhost:32766/applications/health-check-app
Source:
- Repo:             https://3000-port-d5e0b782421d4dcd.labs.kodekloud.com//bob/gitops-argocd.git
  Target:           HEAD
  Path:             ./health-check
SyncWindow:         Sync Allowed
Sync Policy:        Automated (Prune)
Sync Status:        Synced to HEAD (194eac2)
Health Status:      Healthy

GROUP  KIND        NAMESPACE     NAME                  STATUS  HEALTH   HOOK  MESSAGE
       Service     health-check  random-shapes-svc     Synced  Healthy        service/random-shapes-svc created
       ConfigMap   health-check  moving-shapes-colors  Synced  Healthy        
apps   Deployment  health-check  random-shapes         Synced  Healthy        

myk8scluster ~ ➜  kubectl -n health-check delete svc --all 
service "random-shapes-svc" deleted

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          12m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          116s

NAME                        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   172.20.199.101   <none>        80:30335/TCP   5s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           12m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       12m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          12m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          2m12s

NAME                        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   172.20.199.101   <none>        80:30335/TCP   21s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           12m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       12m

myk8scluster ~ ➜  argocd app set health-check-app  --sync-policy=auto --self-heal --auto-prune

myk8scluster ~ ➜  argocd app get argocd/health-check-app
Name:               argocd/health-check-app
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          health-check
URL:                https://localhost:32766/applications/health-check-app
Source:
- Repo:             https://3000-port-d5e0b782421d4dcd.labs.kodekloud.com//bob/gitops-argocd.git
  Target:           HEAD
  Path:             ./health-check
SyncWindow:         Sync Allowed
Sync Policy:        Automated (Prune)
Sync Status:        Synced to HEAD (194eac2)
Health Status:      Healthy

GROUP  KIND        NAMESPACE     NAME                  STATUS  HEALTH   HOOK  MESSAGE
       Service     health-check  random-shapes-svc     Synced  Healthy        service/random-shapes-svc created
       ConfigMap   health-check  moving-shapes-colors  Synced  Healthy        
apps   Deployment  health-check  random-shapes         Synced  Healthy        

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m6s

NAME                        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   172.20.199.101   <none>        80:30335/TCP   2m15s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m10s

NAME                        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   172.20.199.101   <none>        80:30335/TCP   2m19s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m23s

NAME                        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   172.20.199.101   <none>        80:30335/TCP   2m32s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m25s

NAME                        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/random-shapes-svc   NodePort   172.20.199.101   <none>        80:30335/TCP   2m34s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m33s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m36s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m37s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m40s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m41s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m42s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m43s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m44s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m45s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          14m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m46s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           14m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       14m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          15m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m47s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           15m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       15m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          15m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m48s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           15m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       15m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          15m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m49s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           15m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       15m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          15m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m57s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           15m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       15m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          15m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          4m58s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           15m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       15m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          15m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          5m1s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           15m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       15m

myk8scluster ~ ➜  kubectl get all -n health-check 
NAME                                 READY   STATUS    RESTARTS   AGE
pod/random-shapes-54b8766c84-dppwh   1/1     Running   0          15m
pod/random-shapes-54b8766c84-f9p9q   1/1     Running   0          5m2s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-shapes   2/2     2            2           15m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/random-shapes-54b8766c84   2         2         2       15m

myk8scluster ~ ➜ 
```

