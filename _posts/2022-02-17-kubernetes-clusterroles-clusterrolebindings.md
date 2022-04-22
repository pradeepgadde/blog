---
layout: single
title:  "Kubernetes Cluster Roles and RoleBindings"
date:   2022-02-17 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/crb-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Cluster Roles and Cluste RoleBindings


### Cluster Roles and Cluster Role Bindings

If we notice, user `pradeep` still can't get the nodes information.  It is becuase, he still does not have access to resources at the cluster scope.

```shell
pradeep@learnk8s$ kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "pradeep" cannot list resource "nodes" in API group "" at the cluster scope
```

To get this access, we need to create a Cluste role and Cluste role binding.
```shell
pradeep@learnk8s$ kubectl config use-context k8s
Switched to context "k8s".
```
```shell
pradeep@learnk8s$ kubectl create clusterrole pradeep-cluster --verb=get,list,watch, --resource=nodes
Warning: '' is not a standard resource verb
clusterrole.rbac.authorization.k8s.io/pradeep-cluster created
```
Get all available clusterroles. There are many but we can see our newly created clusterrole `pradeep-cluster`.

```shell
pradeep@learnk8s$ kubectl get clusterrole
NAME                                                                   CREATED AT
admin                                                                  2022-02-15T06:57:58Z
cluster-admin                                                          2022-02-15T06:57:58Z
edit                                                                   2022-02-15T06:57:58Z
kindnet                                                                2022-02-15T06:58:03Z
kubeadm:get-nodes                                                      2022-02-15T06:58:01Z
pradeep-cluster                                                        2022-02-17T07:33:43Z
system:aggregate-to-admin                                              2022-02-15T06:57:58Z
system:aggregate-to-edit                                               2022-02-15T06:57:58Z
system:aggregate-to-view                                               2022-02-15T06:57:58Z
system:auth-delegator                                                  2022-02-15T06:57:58Z
system:basic-user                                                      2022-02-15T06:57:58Z
system:certificates.k8s.io:certificatesigningrequests:nodeclient       2022-02-15T06:57:58Z
system:certificates.k8s.io:certificatesigningrequests:selfnodeclient   2022-02-15T06:57:58Z
system:certificates.k8s.io:kube-apiserver-client-approver              2022-02-15T06:57:58Z
system:certificates.k8s.io:kube-apiserver-client-kubelet-approver      2022-02-15T06:57:58Z
system:certificates.k8s.io:kubelet-serving-approver                    2022-02-15T06:57:58Z
system:certificates.k8s.io:legacy-unknown-approver                     2022-02-15T06:57:58Z
system:controller:attachdetach-controller                              2022-02-15T06:57:58Z
system:controller:certificate-controller                               2022-02-15T06:57:58Z
system:controller:clusterrole-aggregation-controller                   2022-02-15T06:57:58Z
system:controller:cronjob-controller                                   2022-02-15T06:57:58Z
system:controller:daemon-set-controller                                2022-02-15T06:57:58Z
system:controller:deployment-controller                                2022-02-15T06:57:58Z
system:controller:disruption-controller                                2022-02-15T06:57:58Z
system:controller:endpoint-controller                                  2022-02-15T06:57:58Z
system:controller:endpointslice-controller                             2022-02-15T06:57:58Z
system:controller:endpointslicemirroring-controller                    2022-02-15T06:57:58Z
system:controller:ephemeral-volume-controller                          2022-02-15T06:57:58Z
system:controller:expand-controller                                    2022-02-15T06:57:58Z
system:controller:generic-garbage-collector                            2022-02-15T06:57:58Z
system:controller:horizontal-pod-autoscaler                            2022-02-15T06:57:58Z
system:controller:job-controller                                       2022-02-15T06:57:58Z
system:controller:namespace-controller                                 2022-02-15T06:57:58Z
system:controller:node-controller                                      2022-02-15T06:57:58Z
system:controller:persistent-volume-binder                             2022-02-15T06:57:58Z
system:controller:pod-garbage-collector                                2022-02-15T06:57:58Z
system:controller:pv-protection-controller                             2022-02-15T06:57:58Z
system:controller:pvc-protection-controller                            2022-02-15T06:57:58Z
system:controller:replicaset-controller                                2022-02-15T06:57:58Z
system:controller:replication-controller                               2022-02-15T06:57:58Z
system:controller:resourcequota-controller                             2022-02-15T06:57:58Z
system:controller:root-ca-cert-publisher                               2022-02-15T06:57:58Z
system:controller:route-controller                                     2022-02-15T06:57:58Z
system:controller:service-account-controller                           2022-02-15T06:57:58Z
system:controller:service-controller                                   2022-02-15T06:57:58Z
system:controller:statefulset-controller                               2022-02-15T06:57:58Z
system:controller:ttl-after-finished-controller                        2022-02-15T06:57:58Z
system:controller:ttl-controller                                       2022-02-15T06:57:58Z
system:coredns                                                         2022-02-15T06:58:01Z
system:discovery                                                       2022-02-15T06:57:58Z
system:heapster                                                        2022-02-15T06:57:58Z
system:kube-aggregator                                                 2022-02-15T06:57:58Z
system:kube-controller-manager                                         2022-02-15T06:57:58Z
system:kube-dns                                                        2022-02-15T06:57:58Z
system:kube-scheduler                                                  2022-02-15T06:57:58Z
system:kubelet-api-admin                                               2022-02-15T06:57:58Z
system:monitoring                                                      2022-02-15T06:57:58Z
system:node                                                            2022-02-15T06:57:58Z
system:node-bootstrapper                                               2022-02-15T06:57:58Z
system:node-problem-detector                                           2022-02-15T06:57:58Z
system:node-proxier                                                    2022-02-15T06:57:58Z
system:persistent-volume-provisioner                                   2022-02-15T06:57:58Z
system:public-info-viewer                                              2022-02-15T06:57:58Z
system:service-account-issuer-discovery                                2022-02-15T06:57:58Z
system:volume-scheduler                                                2022-02-15T06:57:58Z
view                                                                   2022-02-15T06:57:58Z
```

Create a clusterrolebinding called `pradeep-cluster-binding` and bind the user `pradeep` and clusterrole `pradeep-cluster`.

```shell
pradeep@learnk8s$ kubectl create clusterrolebinding pradeep-cluster-binding --clusterrole=pradeep-cluster --user=pradeep
clusterrolebinding.rbac.authorization.k8s.io/pradeep-cluster-binding created
```

Get all available clusterrolebindings.
```shell
pradeep@learnk8s$ kubectl get clusterrolebindings.rbac.authorization.k8s.io
NAME                                                   ROLE                                                                               AGE
cluster-admin                                          ClusterRole/cluster-admin                                                          2d
kindnet                                                ClusterRole/kindnet                                                                2d
kubeadm:get-nodes                                      ClusterRole/kubeadm:get-nodes                                                      2d
kubeadm:kubelet-bootstrap                              ClusterRole/system:node-bootstrapper                                               2d
kubeadm:node-autoapprove-bootstrap                     ClusterRole/system:certificates.k8s.io:certificatesigningrequests:nodeclient       2d
kubeadm:node-autoapprove-certificate-rotation          ClusterRole/system:certificates.k8s.io:certificatesigningrequests:selfnodeclient   2d
kubeadm:node-proxier                                   ClusterRole/system:node-proxier                                                    2d
minikube-rbac                                          ClusterRole/cluster-admin                                                          2d
pradeep-cluster-binding                                ClusterRole/pradeep-cluster                                                        5s
storage-provisioner                                    ClusterRole/system:persistent-volume-provisioner                                   2d
system:basic-user                                      ClusterRole/system:basic-user                                                      2d
system:controller:attachdetach-controller              ClusterRole/system:controller:attachdetach-controller                              2d
system:controller:certificate-controller               ClusterRole/system:controller:certificate-controller                               2d
system:controller:clusterrole-aggregation-controller   ClusterRole/system:controller:clusterrole-aggregation-controller                   2d
system:controller:cronjob-controller                   ClusterRole/system:controller:cronjob-controller                                   2d
system:controller:daemon-set-controller                ClusterRole/system:controller:daemon-set-controller                                2d
system:controller:deployment-controller                ClusterRole/system:controller:deployment-controller                                2d
system:controller:disruption-controller                ClusterRole/system:controller:disruption-controller                                2d
system:controller:endpoint-controller                  ClusterRole/system:controller:endpoint-controller                                  2d
system:controller:endpointslice-controller             ClusterRole/system:controller:endpointslice-controller                             2d
system:controller:endpointslicemirroring-controller    ClusterRole/system:controller:endpointslicemirroring-controller                    2d
system:controller:ephemeral-volume-controller          ClusterRole/system:controller:ephemeral-volume-controller                          2d
system:controller:expand-controller                    ClusterRole/system:controller:expand-controller                                    2d
system:controller:generic-garbage-collector            ClusterRole/system:controller:generic-garbage-collector                            2d
system:controller:horizontal-pod-autoscaler            ClusterRole/system:controller:horizontal-pod-autoscaler                            2d
system:controller:job-controller                       ClusterRole/system:controller:job-controller                                       2d
system:controller:namespace-controller                 ClusterRole/system:controller:namespace-controller                                 2d
system:controller:node-controller                      ClusterRole/system:controller:node-controller                                      2d
system:controller:persistent-volume-binder             ClusterRole/system:controller:persistent-volume-binder                             2d
system:controller:pod-garbage-collector                ClusterRole/system:controller:pod-garbage-collector                                2d
system:controller:pv-protection-controller             ClusterRole/system:controller:pv-protection-controller                             2d
system:controller:pvc-protection-controller            ClusterRole/system:controller:pvc-protection-controller                            2d
system:controller:replicaset-controller                ClusterRole/system:controller:replicaset-controller                                2d
system:controller:replication-controller               ClusterRole/system:controller:replication-controller                               2d
system:controller:resourcequota-controller             ClusterRole/system:controller:resourcequota-controller                             2d
system:controller:root-ca-cert-publisher               ClusterRole/system:controller:root-ca-cert-publisher                               2d
system:controller:route-controller                     ClusterRole/system:controller:route-controller                                     2d
system:controller:service-account-controller           ClusterRole/system:controller:service-account-controller                           2d
system:controller:service-controller                   ClusterRole/system:controller:service-controller                                   2d
system:controller:statefulset-controller               ClusterRole/system:controller:statefulset-controller                               2d
system:controller:ttl-after-finished-controller        ClusterRole/system:controller:ttl-after-finished-controller                        2d
system:controller:ttl-controller                       ClusterRole/system:controller:ttl-controller                                       2d
system:coredns                                         ClusterRole/system:coredns                                                         2d
system:discovery                                       ClusterRole/system:discovery                                                       2d
system:kube-controller-manager                         ClusterRole/system:kube-controller-manager                                         2d
system:kube-dns                                        ClusterRole/system:kube-dns                                                        2d
system:kube-scheduler                                  ClusterRole/system:kube-scheduler                                                  2d
system:monitoring                                      ClusterRole/system:monitoring                                                      2d
system:node                                            ClusterRole/system:node                                                            2d
system:node-proxier                                    ClusterRole/system:node-proxier                                                    2d
system:public-info-viewer                              ClusterRole/system:public-info-viewer                                              2d
system:service-account-issuer-discovery                ClusterRole/system:service-account-issuer-discovery                                2d
system:volume-scheduler                                ClusterRole/system:volume-scheduler                                                2d
```
Again, there are many pre-defined but our newly created clusterrolebinding `pradeep-cluster-binding` is shown.

With that confirmation, let us verify if user `pradeep` can get the nodes without context-switching.

```shell
pradeep@learnk8s$ kubectl get nodes --as pradeep
NAME      STATUS   ROLES                  AGE   VERSION
k8s       Ready    control-plane,master   2d    v1.23.1
k8s-m02   Ready    <none>                 2d    v1.23.1
```
Before switching context, let us describe these clusterrole and clusterrolebindings.

ClusterRole:

```shell
pradeep@learnk8s$ kubectl describe clusterrole pradeep-cluster
Name:         pradeep-cluster
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  nodes      []                 []              [get list watch ]
```
ClusterRoleBinding:

```shell
pradeep@learnk8s$ kubectl describe clusterrolebindings pradeep-cluster-binding
Name:         pradeep-cluster-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  pradeep-cluster
Subjects:
  Kind  Name     Namespace
  ----  ----     ---------
  User  pradeep
```
Switch context now.

```shell
pradeep@learnk8s$ kubectl config use-context pradeep
Switched to context "pradeep".
```
Finally, verify user `pradeep` can get the nodes.
```shell
pradeep@learnk8s$ kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
k8s       Ready    control-plane,master   2d    v1.23.1
k8s-m02   Ready    <none>                 2d    v1.23.1
```
Now, we have accomplished our simple goal w.r.t authorization in Kubernetes.
