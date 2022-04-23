---
layout: single
title:  "Accessing the Kubernetes API from within a Pod"
date:   2022-04-22 11:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/api-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Accessing the Kubernetes API from within a Pod

Create a simple Pod
```sh
pradeep@learnk8s$ kubectl run nginx --image=nginx
pod/nginx created
```

Verify that Pod is running 
```sh
pradeep@learnk8s$ kubectl get pods -o wide -w
NAME    READY   STATUS              RESTARTS   AGE   IP       NODE       NOMINATED NODE   READINESS GATES
nginx   0/1     ContainerCreating   0          12s   <none>   minikube   <none>           <none>
nginx   1/1     Running             0          23s   172.17.0.3   minikube   <none>           <none>

```
Login to the Pod Shell by using the `kubectl exec` command and set some Shell variables which will be used later along with the `curl` command.

While running in a Pod, the Kubernetes apiserver is accessible via a Service named kubernetes in the default namespace. Therefore, Pods can use the `kubernetes.default.svc` hostname to query the API server. Official client libraries do this automatically.

The recommended way to authenticate to the API server is with a service account credential. By default, a Pod is associated with a service account, and a credential (token) for that service account is placed into the filesystem tree of each container in that Pod, at `/var/run/secrets/kubernetes.io/serviceaccount/token`.

If available, a certificate bundle is placed into the filesystem tree of each container at `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`, and should be used to verify the serving certificate of the API server.

Finally, the default namespace to be used for namespaced API operations is placed in a file at `/var/run/secrets/kubernetes.io/serviceaccount/namespace` in each container.


```sh
pradeep@learnk8s$ kubectl exec -it nginx -- bash
root@nginx:/# APISERVER=https://kubernetes.default.svc
root@nginx:/# SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
root@nginx:/# NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)
root@nginx:/# TOKEN=$(cat ${SERVICEACCOUNT}/token)
root@nginx:/# CACERT=${SERVICEACCOUNT}/ca.crt
````
Verify the values of these variables.

```sh
root@nginx:/# echo $APISERVER
https://kubernetes.default.svc
root@nginx:/# echo $NAMESPACE
default
root@nginx:/# echo $TOKEN
eyJhbGciOiJSUzI1NiIsImtpZCI6InZsU2pyQmlJTGNYQ1pDLWJDejZwa0paOTItcERjM0pHVnl3M2taN3pKN2sifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjgyMjA5NTU4LCJpYXQiOjE2NTA2NzM1NTgsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJuZ2lueCIsInVpZCI6Ijg4NjQ0NmM4LTNhMzMtNDVhOS05MWRiLTI4N2VkYWEwMmQzYSJ9LCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVmYXVsdCIsInVpZCI6IjM2MmNhNWE5LTJhZDMtNGE1ZC04YjIyLTliMjc4ZjZkZTI2MyJ9LCJ3YXJuYWZ0ZXIiOjE2NTA2NzcxNjV9LCJuYmYiOjE2NTA2NzM1NTgsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.czypWeofE0JCtYh4P9S1umkdX4Fc9RXyfGJLaIfuTNhA1jlGb56yiQI7k-hf4ChBvG8ZVqlUuMoioXcrUYJWkwmUy67r0PimyPoBlatx_RCTvkg6feeK0UggPg3E0twHZNLoxqRCKdBbMoj1M7wASwpIYIp1Bu9gQyR7X30u8WQPe58Splflv-wT3mrJjLMFvC6KMLES595w-Lj2N0oXqaeaQdaNRpoz2E5TQVBOBxkXQa1LL-OxQn4wPSqaWvApSyrEcNm3_SpgEV2QhTpgv3O88QV78QRuRLHmEvBf8pu5e-nJqqKBHZptPtnjVWcMDdSp7yrmXpbSDLV7JZ9w1A
root@nginx:/# echo $CACERT
/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```
In an earlier post, we discussed the usage of [kubectl proxy](https://pradeepgadde.com/blog/kubernetes/2022/04/10/kubectl-proxy.html).
It is possible to avoid using the `kubectl proxy` by passing the authentication token directly to the API server. The internal certificate secures the connection.


```sh
root@nginx:/# curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "172.16.30.9:8443"
    }
  ]
}
```
This seems to have worked.
Let us try to get the Pods in the default namespace.

```sh
root@nginx:/# curl --cacrt ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/default/pods
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```
Hmm, it is Forbidden!
Try one more, to list all namespaces

```sh
root@nginx:/# curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/            
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "namespaces is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"namespaces\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "namespaces"
  },
  "code": 403
}root@nginx:/# exit
exit
pradeep@learnk8s$ 

```

There is another way using `kubectl proxy`.

If you would like to query the API without an official client library, you can run `kubectl proxy` as the command of a new sidecar container in the Pod. This way, `kubectl proxy` will authenticate to the API and expose it on the localhost interface of the Pod, so that other containers in the Pod can use it directly.

We will try this in another post.





