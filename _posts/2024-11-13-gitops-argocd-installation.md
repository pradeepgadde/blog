---
layout: single
title:  "GitOps ArgoCD Installation"
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

# GitOps ArgoCD Installation

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.

In this post, let us install Argo CD on a single node Kubernetes cluster and then install the ArgoCD CLI.

You will see that first stable version of ArgoCD is installed and then a specific version is used.

```sh

myk8scluster ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
myk8scluster   Ready    control-plane   6m43s   v1.30.0

myk8scluster ~ ➜  kubectl get ns
NAME              STATUS   AGE
default           Active   6m50s
kube-flannel      Active   6m45s
kube-node-lease   Active   6m50s
kube-public       Active   6m50s
kube-system       Active   6m50s

myk8scluster ~ ➜  kubectl create ns argocd
namespace/argocd created

myk8scluster ~ ➜  kubectl get ns
NAME              STATUS   AGE
argocd            Active   2s
default           Active   7m4s
kube-flannel      Active   6m59s
kube-node-lease   Active   7m4s
kube-public       Active   7m4s
kube-system       Active   7m4s

myk8scluster ~ ➜  #kubectl apply -n argocd -f https://raw.githubusercontent.com/a
rgoproj/argo-cd/v2.4.11/manifests/install.yaml

myk8scluster ~ ➜  #kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-redis created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created

myk8scluster ~ ➜  kubectl delete ns argocd 
namespace "argocd" deleted

myk8scluster ~ ➜  kubectl get ns
NAME              STATUS   AGE
default           Active   10m
kube-flannel      Active   10m
kube-node-lease   Active   10m
kube-public       Active   10m
kube-system       Active   10m

myk8scluster ~ ➜  kubectl create ns argocd
namespace/argocd created

myk8scluster ~ ➜  kubectl apply -n argocd -f https://raw.githubusercontent.com/ar
goproj/argo-cd/v2.4.11/manifests/install.yaml
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io configured
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io configured
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io configured
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller unchanged
clusterrole.rbac.authorization.k8s.io/argocd-server configured
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller unchanged
clusterrolebinding.rbac.authorization.k8s.io/argocd-server unchanged
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created

myk8scluster ~ ➜  kubectl get all -n argocd
NAME                                                   READY   STATUS              RESTARTS   AGE
pod/argocd-application-controller-0                    0/1     ContainerCreating   0          5s
pod/argocd-applicationset-controller-d7c857898-nvqrb   0/1     ContainerCreating   0          6s
pod/argocd-dex-server-75d98bff7c-pqfmt                 0/1     Init:0/1            0          6s
pod/argocd-notifications-controller-684947df85-qskwv   0/1     ContainerCreating   0          6s
pod/argocd-redis-84c8cd4d8-xsxvj                       0/1     ContainerCreating   0          5s
pod/argocd-repo-server-7bbc57875d-wtwkg                0/1     Init:0/1            0          5s
pod/argocd-server-5f8984f889-ls4nh                     0/1     ContainerCreating   0          5s

NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP   172.20.252.108   <none>        7000/TCP,8080/TCP            6s
service/argocd-dex-server                         ClusterIP   172.20.74.221    <none>        5556/TCP,5557/TCP,5558/TCP   6s
service/argocd-metrics                            ClusterIP   172.20.119.213   <none>        8082/TCP                     6s
service/argocd-notifications-controller-metrics   ClusterIP   172.20.100.227   <none>        9001/TCP                     6s
service/argocd-redis                              ClusterIP   172.20.56.133    <none>        6379/TCP                     6s
service/argocd-repo-server                        ClusterIP   172.20.155.12    <none>        8081/TCP,8084/TCP            6s
service/argocd-server                             ClusterIP   172.20.176.44    <none>        80/TCP,443/TCP               6s
service/argocd-server-metrics                     ClusterIP   172.20.235.86    <none>        8083/TCP                     6s

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   0/1     1            0           6s
deployment.apps/argocd-dex-server                  0/1     1            0           6s
deployment.apps/argocd-notifications-controller    0/1     1            0           6s
deployment.apps/argocd-redis                       0/1     1            0           6s
deployment.apps/argocd-repo-server                 0/1     1            0           6s
deployment.apps/argocd-server                      0/1     1            0           5s

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-d7c857898   1         1         0       6s
replicaset.apps/argocd-dex-server-75d98bff7c                 1         1         0       6s
replicaset.apps/argocd-notifications-controller-684947df85   1         1         0       6s
replicaset.apps/argocd-redis-84c8cd4d8                       1         1         0       6s
replicaset.apps/argocd-repo-server-7bbc57875d                1         1         0       5s
replicaset.apps/argocd-server-5f8984f889                     1         1         0       5s

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   0/1     5s

myk8scluster ~ ➜  kubectl get cm -n argocd
NAME                        DATA   AGE
argocd-cm                   0      23s
argocd-cmd-params-cm        0      23s
argocd-gpg-keys-cm          0      23s
argocd-notifications-cm     0      23s
argocd-rbac-cm              0      23s
argocd-ssh-known-hosts-cm   1      23s
argocd-tls-certs-cm         0      23s
kube-root-ca.crt            1      35s

myk8scluster ~ ➜  kubectl get cm -n argocd
NAME                        DATA   AGE
argocd-cm                   0      26s
argocd-cmd-params-cm        0      26s
argocd-gpg-keys-cm          0      26s
argocd-notifications-cm     0      26s
argocd-rbac-cm              0      26s
argocd-ssh-known-hosts-cm   1      26s
argocd-tls-certs-cm         0      26s
kube-root-ca.crt            1      38s

myk8scluster ~ ➜  kubectl get svc -n argocd
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   172.20.252.108   <none>        7000/TCP,8080/TCP            41s
argocd-dex-server                         ClusterIP   172.20.74.221    <none>        5556/TCP,5557/TCP,5558/TCP   41s
argocd-metrics                            ClusterIP   172.20.119.213   <none>        8082/TCP                     41s
argocd-notifications-controller-metrics   ClusterIP   172.20.100.227   <none>        9001/TCP                     41s
argocd-redis                              ClusterIP   172.20.56.133    <none>        6379/TCP                     41s
argocd-repo-server                        ClusterIP   172.20.155.12    <none>        8081/TCP,8084/TCP            41s
argocd-server                             ClusterIP   172.20.176.44    <none>        80/TCP,443/TCP               41s
argocd-server-metrics                     ClusterIP   172.20.235.86    <none>        8083/TCP                     41s

myk8scluster ~ ➜  kubectl edit svc argocd-server -n argocd
service/argocd-server edited

myk8scluster ~ ➜  kubectl edit svc argocd-server -n argocd
service/argocd-server edited

myk8scluster ~ ➜  kubectl get svc -n argocd
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   172.20.252.108   <none>        7000/TCP,8080/TCP            2m6s
argocd-dex-server                         ClusterIP   172.20.74.221    <none>        5556/TCP,5557/TCP,5558/TCP   2m6s
argocd-metrics                            ClusterIP   172.20.119.213   <none>        8082/TCP                     2m6s
argocd-notifications-controller-metrics   ClusterIP   172.20.100.227   <none>        9001/TCP                     2m6s
argocd-redis                              ClusterIP   172.20.56.133    <none>        6379/TCP                     2m6s
argocd-repo-server                        ClusterIP   172.20.155.12    <none>        8081/TCP,8084/TCP            2m6s
argocd-server                             NodePort    172.20.176.44    <none>        80:31595/TCP,443:32766/TCP   2m6s
argocd-server-metrics                     ClusterIP   172.20.235.86    <none>        8083/TCP                     2m6s

myk8scluster ~ ➜  kubectl edit svc argocd-server -n argocd
Edit cancelled, no changes made.

myk8scluster ~ ➜  kubectl get secrets -n argocd
NAME                          TYPE     DATA   AGE
argocd-initial-admin-secret   Opaque   1      3m
argocd-notifications-secret   Opaque   0      3m23s
argocd-secret                 Opaque   5      3m23s

myk8scluster ~ ➜  kubectl get secrets argocd-initial-admin-secret -n argocd
NAME                          TYPE     DATA   AGE
argocd-initial-admin-secret   Opaque   1      3m14s

myk8scluster ~ ➜  kubectl describe secrets argocd-initial-admin-secret -n argocd
Name:         argocd-initial-admin-secret
Namespace:    argocd
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  16 bytes

myk8scluster ~ ➜  kubectl get secrets argocd-initial-admin-secret -n argocd -o json
{
    "apiVersion": "v1",
    "data": {
        "password": "THN1d0c0UUxWMUo1dVVRbA=="
    },
    "kind": "Secret",
    "metadata": {
        "creationTimestamp": "2024-11-13T17:48:40Z",
        "name": "argocd-initial-admin-secret",
        "namespace": "argocd",
        "resourceVersion": "1999",
        "uid": "35061199-9434-4e26-af70-fba732cb04ba"
    },
    "type": "Opaque"
}

myk8scluster ~ ➜  kubectl get secrets argocd-initial-admin-secret -n argocd -o json | jq .data.password
"THN1d0c0UUxWMUo1dVVRbA=="

myk8scluster ~ ➜  kubectl get secrets argocd-initial-admin-secret -n argocd -o json | jq .data.password -r
THN1d0c0UUxWMUo1dVVRbA==

myk8scluster ~ ➜  kubectl get secrets argocd-initial-admin-secret -n argocd -o json | jq .data.password -r | tr -d '\n'
THN1d0c0UUxWMUo1dVVRbA==
myk8scluster ~ ➜  kubectl get secrets argocd-initial-admin-secret -n argocd -o json | jq .data.password -r | tr -d '\n' | base64 -d
LsuwG4QLV1J5uUQl
myk8scluster ~ ➜  curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.11/argocd-linux-amd64
chmod +x /usr/local/bin/argocd

myk8scluster ~ ➜  argocd
argocd controls a Argo CD server

Usage:
  argocd [flags]
  argocd [command]

Available Commands:
  account     Manage account settings
  admin       Contains a set of commands useful for Argo CD administrators and requires direct Kubernetes access
  app         Manage applications
  cert        Manage repository certificates and SSH known hosts entries
  cluster     Manage cluster credentials
  completion  output shell completion code for the specified shell (bash or zsh)
  context     Switch between contexts
  gpg         Manage GPG keys used for signature verification
  help        Help about any command
  login       Log in to Argo CD
  logout      Log out from Argo CD
  proj        Manage projects
  relogin     Refresh an expired authenticate token
  repo        Manage repository connection parameters
  repocreds   Manage repository connection parameters
  version     Print version information

Flags:
      --auth-token string               Authentication token
      --client-crt string               Client certificate file
      --client-crt-key string           Client certificate key file
      --config string                   Path to Argo CD config (default "/root/.config/argocd/config")
      --core                            If set to true then CLI talks directly to Kubernetes instead of talking to Argo CD API server
      --grpc-web                        Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2.
      --grpc-web-root-path string       Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2. Set web root.
  -H, --header strings                  Sets additional header to all requests made by Argo CD CLI. (Can be repeated multiple times to add multiple headers, also supports comma separated headers)
  -h, --help                            help for argocd
      --http-retry-max int              Maximum number of retries to establish http connection to Argo CD server
      --insecure                        Skip server certificate and domain verification
      --kube-context string             Directs the command to the given kube-context
      --logformat string                Set the logging format. One of: text|json (default "text")
      --loglevel string                 Set the logging level. One of: debug|info|warn|error (default "info")
      --plaintext                       Disable TLS
      --port-forward                    Connect to a random argocd-server port using port forwarding
      --port-forward-namespace string   Namespace name which should be used for port forwarding
      --server string                   Argo CD server address
      --server-crt string               Server certificate file

Use "argocd [command] --help" for more information about a command.

myk8scluster ~ ➜ 
```

