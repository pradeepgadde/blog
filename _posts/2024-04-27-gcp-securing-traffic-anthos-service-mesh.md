---

layout: single
title:  "Securing Traffic with Anthos Service Mesh"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Securing Traffic with Anthos Service Mesh

Anthos Service Mesh security helps you mitigate insider threats and  reduce the risk of a data breach by ensuring that all communications  between workloads are encrypted, mutually authenticated, and authorized.

 PERMISSIVE mode mTLS allows services to  receive both plaintext and mTLS traffic from clients, allowing you to  incrementally adopt mTLS.

STRICT mode mTLS across your  service mesh effectively blocking plaintext traffic to all your Istio  injected services and 

scope STRICT mode mTLS down to a single  namespace

- Enforce STRICT mTLS mode across the service mesh
- Enforce STRICT mTLS mode on a single namespace
- Explore the security configurations in the Anthos Service Mesh Dashboard
- Add authorization policies to enforce access based on a JSON Web Token (JWT)
- Add authorization policies for HTTP traffic in an Istio mesh

## Confirm Anthos Service Mesh setup

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-5d22baeeb714.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ CLUSTER_ZONE=us-central1-c
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ export CLUSTER_NAME=gke
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ # get the project id
export GCLOUD_PROJECT=$(gcloud config get-value project)

# configure kubectl
gcloud container clusters get-credentials $CLUSTER_NAME \
    --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
Your active configuration is: [cloudshell-26468]
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke.
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ gcloud container clusters list
NAME: gke
LOCATION: us-central1-c
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 35.224.214.58
MACHINE_TYPE: e2-standard-2
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 3
STATUS: RUNNING
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl get service -n istio-system
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                 AGE
istiod               ClusterIP   10.27.87.143   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   5h59m
istiod-asm-1157-23   ClusterIP   10.27.85.248   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   5h59m
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl get pods -n istio-system
NAME                                  READY   STATUS    RESTARTS   AGE
istiod-asm-1157-23-66d49d9464-5qbjg   1/1     Running   0          5h59m
istiod-asm-1157-23-66d49d9464-z4tb6   1/1     Running   0          5h59m
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```



## Deploy sleep and httpbin services

In this task, you create a set of namespaces to host the [httpbin](https://github.com/istio/istio/tree/release-1.6/samples/httpbin), and [sleep](https://github.com/istio/istio/tree/release-1.6/samples/sleep) services. You will use those services to explore the impact of mTLS on traffic. The **sleep** service acts as the client and will call the **httpbin** service, which acts as a server.

In Cloud Shell, create namespaces for the example clients and services. Traffic in the **legacy-*** namespaces takes place over plain text, while traffic in the **mtls-*** namespaces happens over mTLS:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl create ns mtls-client
kubectl create ns mtls-service
kubectl create ns legacy-client
kubectl create ns legacy-service

kubectl get namespaces
namespace/mtls-client created
namespace/mtls-service created
namespace/legacy-client created
namespace/legacy-service created
NAME                 STATUS   AGE
asm-system           Active   6h
default              Active   6h15m
gke-managed-system   Active   6h15m
gmp-public           Active   6h14m
gmp-system           Active   6h14m
istio-system         Active   6h2m
kube-node-lease      Active   6h15m
kube-public          Active   6h15m
kube-system          Active   6h15m
legacy-client        Active   3s
legacy-service       Active   1s
mtls-client          Active   6s
mtls-service         Active   4s
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Deploy the legacy services in the **legacy-*** namespaces. You call them legacy because they are not part of the mesh:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ #configurations are stored in Github

kubectl apply -f \
https://raw.githubusercontent.com/istio/istio/release-1.6/samples/sleep/sleep.yaml \
-n legacy-client

kubectl apply -f \
https://raw.githubusercontent.com/istio/istio/release-1.6/samples/httpbin/httpbin.yaml \
-n legacy-service
serviceaccount/sleep created
service/sleep created
deployment.apps/sleep created
serviceaccount/httpbin created
service/httpbin created
deployment.apps/httpbin created
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Enable auto-injection of the Istio sidecar proxy on the **mtls-*** namespaces:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ # get the revision label
export DEPLOYMENT=$(kubectl get deployments -n istio-system | grep istiod)
export VERSION=asm-$(echo $DEPLOYMENT | cut -d'-' -f 3)-$(echo $DEPLOYMENT \
    | cut -d'-' -f 4 | cut -d' ' -f 1)

# enable auto-injection on the namespaces
kubectl label namespace mtls-client istio.io/rev=${VERSION} --overwrite
kubectl label namespace mtls-service istio.io/rev=${VERSION} --overwrite
namespace/mtls-client labeled
namespace/mtls-service labeled
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Deploy the services in the **mtls-*** namespaces:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl apply -f \
https://raw.githubusercontent.com/istio/istio/release-1.6/samples/sleep/sleep.yaml \
-n mtls-client

kubectl apply -f \
https://raw.githubusercontent.com/istio/istio/release-1.6/samples/httpbin/httpbin.yaml \
-n mtls-service
serviceaccount/sleep created
service/sleep created
deployment.apps/sleep created
serviceaccount/httpbin created
service/httpbin created
deployment.apps/httpbin created
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Verify that the **sleep** service and the **httpbin** service are each deployed in both the **mtls-service** and **legacy-service** namespaces:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl get services --all-namespaces
NAMESPACE        NAME                                                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                 AGE
asm-system       canonical-service-controller-manager-metrics-service   ClusterIP   10.27.91.90    <none>        8443/TCP                                6h3m
default          kubernetes                                             ClusterIP   10.27.80.1     <none>        443/TCP                                 6h18m
gmp-system       alertmanager                                           ClusterIP   None           <none>        9093/TCP                                6h17m
gmp-system       gmp-operator                                           ClusterIP   10.27.86.11    <none>        8443/TCP,443/TCP                        6h17m
istio-system     istiod                                                 ClusterIP   10.27.87.143   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   6h3m
istio-system     istiod-asm-1157-23                                     ClusterIP   10.27.85.248   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   6h3m
kube-system      default-http-backend                                   NodePort    10.27.81.33    <none>        80:30914/TCP                            6h17m
kube-system      kube-dns                                               ClusterIP   10.27.80.10    <none>        53/UDP,53/TCP                           6h17m
kube-system      metrics-server                                         ClusterIP   10.27.86.196   <none>        443/TCP                                 6h17m
legacy-client    sleep                                                  ClusterIP   10.27.87.87    <none>        80/TCP                                  116s
legacy-service   httpbin                                                ClusterIP   10.27.82.27    <none>        8000/TCP                                111s
mtls-client      sleep                                                  ClusterIP   10.27.82.17    <none>        80/TCP                                  33s
mtls-service     httpbin                                                ClusterIP   10.27.87.129   <none>        8000/TCP                                29s
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Verify that a **sleep** pod is running in the **mtls-client** and **legacy-client** namespaces and that an **httpbin** pod is running in the **mtls-service** and **legacy-service** namespaces:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl get pods --all-namespaces
NAMESPACE        NAME                                                    READY   STATUS    RESTARTS        AGE
asm-system       canonical-service-controller-manager-74c6dc6698-mnbfh   2/2     Running   0               6h4m
gmp-system       alertmanager-0                                          2/2     Running   0               6h16m
gmp-system       collector-kvnjl                                         2/2     Running   0               6h11m
gmp-system       collector-mc7fx                                         2/2     Running   0               6h11m
gmp-system       collector-vnrsm                                         2/2     Running   0               6h11m
gmp-system       gmp-operator-5c469746b-xqhl7                            1/1     Running   0               6h16m
gmp-system       rule-evaluator-868cb9b64b-5l287                         2/2     Running   2 (6h11m ago)   6h11m
istio-system     istiod-asm-1157-23-66d49d9464-5qbjg                     1/1     Running   0               6h4m
istio-system     istiod-asm-1157-23-66d49d9464-z4tb6                     1/1     Running   0               6h4m
kube-system      event-exporter-gke-7d996c57bf-vs54l                     2/2     Running   0               6h16m
kube-system      fluentbit-gke-j2sdr                                     2/2     Running   0               6h11m
kube-system      fluentbit-gke-qk5lm                                     2/2     Running   0               6h11m
kube-system      fluentbit-gke-x98l9                                     2/2     Running   0               6h11m
kube-system      gke-metadata-server-dzrfx                               1/1     Running   0               6h11m
kube-system      gke-metadata-server-jks6p                               1/1     Running   0               6h11m
kube-system      gke-metadata-server-vswwv                               1/1     Running   0               6h11m
kube-system      gke-metrics-agent-gzhgs                                 2/2     Running   0               6h11m
kube-system      gke-metrics-agent-m6zrn                                 2/2     Running   0               6h11m
kube-system      gke-metrics-agent-tkqkt                                 2/2     Running   0               6h11m
kube-system      konnectivity-agent-74d9755866-gwvh5                     2/2     Running   0               6h11m
kube-system      konnectivity-agent-74d9755866-ld6zl                     2/2     Running   0               6h16m
kube-system      konnectivity-agent-74d9755866-s97vk                     2/2     Running   0               6h11m
kube-system      konnectivity-agent-autoscaler-5847cf65c7-bl4vd          1/1     Running   0               6h16m
kube-system      kube-dns-6f955b858b-ghn4z                               4/4     Running   0               6h16m
kube-system      kube-dns-6f955b858b-lktdd                               4/4     Running   0               6h11m
kube-system      kube-dns-autoscaler-755c7dfdf5-l6f7l                    1/1     Running   0               6h16m
kube-system      kube-proxy-gke-gke-pool-12345-83dd774c-3ztq             1/1     Running   0               6h11m
kube-system      kube-proxy-gke-gke-pool-12345-83dd774c-b860             1/1     Running   0               6h11m
kube-system      kube-proxy-gke-gke-pool-12345-83dd774c-t93w             1/1     Running   0               6h11m
kube-system      l7-default-backend-6779bb6c8d-r2v8p                     1/1     Running   0               6h16m
kube-system      metrics-server-v0.6.3-764c8d87d9-lxn6x                  2/2     Running   0               6h16m
kube-system      netd-j22qw                                              2/2     Running   0               6h11m
kube-system      netd-ljtlt                                              2/2     Running   0               6h11m
kube-system      netd-nt87b                                              2/2     Running   0               6h11m
kube-system      pdcsi-node-7ffz7                                        2/2     Running   0               6h11m
kube-system      pdcsi-node-dm4fv                                        2/2     Running   0               6h11m
kube-system      pdcsi-node-lgj8s                                        2/2     Running   0               6h11m
legacy-client    sleep-7dbc8959b8-4vkxz                                  1/1     Running   0               2m38s
legacy-service   httpbin-6fcb98998c-l9rgk                                1/1     Running   0               2m33s
mtls-client      sleep-7dbc8959b8-zn29d                                  2/2     Running   0               76s
mtls-service     httpbin-6fcb98998c-jkn6t                                2/2     Running   0               71s
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

### Verify that the two sleep clients can communicate with the two httpbin services

- Use Cloud Shell to run this nested command loop:

  ```sh
  student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ for from in "mtls-client" "legacy-client"; do
    for to in "mtls-service" "legacy-service"; do
      kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"
    done
  done
  sleep.mtls-client to httpbin.mtls-service: 200
  sleep.mtls-client to httpbin.legacy-service: 200
  sleep.legacy-client to httpbin.mtls-service: 200
  sleep.legacy-client to httpbin.legacy-service: 200
  student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
  ```

  Now you're ready to enforce security policies for this application.

## Understand authentication and enable service to service authentication with mTLS

1. ### Anthos Service Mesh

   1. In the console, go to **Navigation Menu > Anthos > Service Mesh**.

      In the Anthos Service Mesh dashboard, you will see the 2 services that were created in the `mtls-service` and `mtls-client` namespaces. You don't see the legacy services because you did not label their namespace, and therefore they remain outside of the mesh.

   2. Under **Namespace** dropdown select **mtls-service** namespace and then click on the **httpbin** service located below.

   3. In the left side panel, go to **Connected Services**.

      Notice that you have 2 services:

      - The `sleep` service in the `mtls-client` namespace, which is part of the mesh and has a sidecar proxy. Therefore you see the real name and communication goes over mTLS, as it's the default behavior in Istio.
      - An `unknown` service, which represents the `sleep` service in the `legacy-client`, which is not part of the mesh and has no sidecar proxy. Therefore you do not see the real name and the communication goes over plain text.

      Use your mouse to hover over the lock symbol in the **Request port** column, and verify that green means mTLS and red means plain text.

      Now check out the **Security** tab in the left side panel. It shows you that the **httpbin** service has received both plaintext and mTLS traffic.

2. ### Test auto mutual TLS

   By default, Istio configures destination workloads in `PERMISSIVE` mode. When `PERMISSIVE` mode is enabled a service can accept both plaintext and mTLS traffic. mTLS is used when the request contains the **X-Forwarded-Client-Cert** header.

   1. Use the Cloud Shell to send a request from the **sleep** service in the **mtls-client** namespace to the **httpbin** service in the **mtls-service** namespace:

   ```sh
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec $(kubectl get pod -l app=sleep -n mtls-client -o jsonpath={.items..metadata.name}) -c sleep -n mtls-client -- curl http://httpbin.mtls-service:8000/headers -s | grep X-Forwarded-Client-Cert
       "X-Forwarded-Client-Cert": "By=spiffe://qwiklabs-gcp-03-5d22baeeb714.svc.id.goog/ns/mtls-service/sa/httpbin;Hash=a7792218923e8cc1a44fd35c04937402fcb4e75dd36e40a6a39e455378d0884f;Subject=\"OU=istio_v1_cloud_workload,O=Google LLC,L=Mountain View,ST=California,C=US\";URI=spiffe://qwiklabs-gcp-03-5d22baeeb714.svc.id.goog/ns/mtls-client/sa/sleep"
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
   ```

   The traffic included the **X-Forwarded-Client-Cert** header and therefore was mutually authenticated and encrypted

Now send a request from the **sleep** service in the **mtls-client** namespace to the **httpbin** service in the **legacy-service** namespace:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec $(kubectl get pod -l app=sleep -n mtls-client -o jsonpath={.items..metadata.name}) -c sleep -n mtls-client -- curl http://httpbin.legacy-service:8000/headers -s | grep X-Forwarded-Client-Cert
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

The **X-Forwarded-Client-Cert** header isn't present so the traffic was sent and received in plaintext.

Finally, send a request from the **sleep** service in the **legacy-client** namespace to the **httpbin** service in the **mtls-service** namespace:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec $(kubectl get pod -l app=sleep -n legacy-client -o jsonpath={.items..metadata.name}) -c sleep -n legacy-client -- curl http://httpbin.mtls-service:8000/headers -s | grep X-Forwarded-Client-Cert
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$
```

The **X-Forwarded-Client-Cert** header isn't present so the traffic was sent and received in plaintext

> **Note:** The **httpbin** service in the **mtls-service** namespace accepted    mTLS traffic from the **sleep** service in the **mtls-client** namespace and    plaintext from the **sleep** service in the **legacy-client** namespace.

### Enforce STRICT mTLS mode across the service mesh

In `STRICT` mode, services injected with the Istio proxy will not accept plaintext traffic and will mutually authenticate with their clients.

You can enforce `STRICT` mTLS mode across the whole mesh or on a per-namespace basis by creating PeerAuthentication resources.

Create a Peer Authentication resources for the entire Service Mesh:

```yaml
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl apply -n istio-system -f - <<EOF
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "mesh-wide-mtls"
spec:
    mtls:
        mode: STRICT
EOF
peerauthentication.security.istio.io/mesh-wide-mtls created
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ for from in "mtls-client" "legacy-client"; do
  for to in "mtls-service" "legacy-service"; do
    kubectl exec $(kubectl get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"
  done
done
sleep.mtls-client to httpbin.mtls-service: 200
sleep.mtls-client to httpbin.legacy-service: 200
sleep.legacy-client to httpbin.mtls-service: 000
command terminated with exit code 56
sleep.legacy-client to httpbin.legacy-service: 200
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

**Note:** The **httpbin** service in the **mtls-service** namespace now rejects the    plaintext traffic it receives from the **sleep** client in the    **legacy-client** namespace.  



Remove the mesh wide mTLS PeerAuthentication resource by running this command in Cloud Shell:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl delete pa mesh-wide-mtls -n istio-system
peerauthentication.security.istio.io "mesh-wide-mtls" deleted
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

### Enforce STRICT mTLS mode on a single namespace

1. In Cloud Shell create a namespace for `STRICT` mTLS:

2. Enable auto-injection of the Istio sidecar proxy on the new namespace:

   ```sh
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl create ns strict-mtls-service
   namespace/strict-mtls-service created
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ # get the revision label
   export DEPLOYMENT=$(kubectl get deployments -n istio-system | grep istiod)
   export VERSION=asm-$(echo $DEPLOYMENT | cut -d'-' -f 3)-$(echo $DEPLOYMENT \
       | cut -d'-' -f 4 | cut -d' ' -f 1)
   
   # enable auto-injection on the namespaces
   kubectl label namespace strict-mtls-service istio.io/rev=${VERSION} --overwrite
   namespace/strict-mtls-service labeled
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
   ```

   deploy another instance of the httpbin service in the **strict-mtls-service** namespace:

   ```sh
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl apply -f \
   https://raw.githubusercontent.com/istio/istio/release-1.6/samples/httpbin/httpbin.yaml \
   -n strict-mtls-service
   serviceaccount/httpbin created
   service/httpbin created
   deployment.apps/httpbin created
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
   ```

   Create a PeerAuthentication resource for the **strict-mtls-service** namespace:

   ```yaml
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl apply -n strict-mtls-service -f - <<EOF
   apiVersion: "security.istio.io/v1beta1"
   kind: "PeerAuthentication"
   metadata:
       name: "restricted-mtls"
       namespace: strict-mtls-service
   spec:
       mtls:
           mode: STRICT
   EOF
   peerauthentication.security.istio.io/restricted-mtls created
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
   ```

   Verify that the **httpbin** service in the **mtls-service** namespace still accepts plaintext traffic:

   ```sh
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec $(kubectl get pod -l app=sleep -n legacy-client -o jsonpath={.items..metadata.name}) -c sleep -n legacy-client -- curl "http://httpbin.mtls-service:8000/ip" -s -o /dev/null -w "sleep.legacy-client to httpbin.mtls-service: %{http_code}\n"
   sleep.legacy-client to httpbin.mtls-service: 200
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
   ```

   Now check to see that the **strict-mtls-service** namespace **httpbin** service does not accept plaintext traffic:

   ```sh
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec $(kubectl get pod -l app=sleep -n legacy-client -o jsonpath={.items..metadata.name}) -c sleep -n legacy-client -- curl "http://httpbin.strict-mtls-service:8000/ip" -s -o /dev/null -w "sleep.legacy-client to httpbin.strict-mtls-service: %{http_code}\n"
   sleep.legacy-client to httpbin.strict-mtls-service: 000
   command terminated with exit code 56
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
   ```

   Verify that the **httpbin** service in the **strict-mtls-service** namespace does accept mTLS traffic:

   ```sh
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec $(kubectl get pod -l app=sleep -n mtls-client -o jsonpath={.items..metadata.name}) -c sleep -n mtls-client -- curl "http://httpbin.strict-mtls-service:8000/ip" -s -o /dev/null -w "sleep.mtls-client to httpbin.strict-mtls-service: %{http_code}\n"
   sleep.mtls-client to httpbin.strict-mtls-service: 200
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
   ```

   ```sh
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl delete pa restricted-mtls -n strict-mtls-service
   peerauthentication.security.istio.io "restricted-mtls" deleted
   student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
   ```

   

## Leverage RequestAuthentication and AuthorizationPolicy resources

This task shows you how to set up and use RequestAuthentication and AuthorizationPolicy resources. Ultimately, you will allow requests that have an approved JWT, and deny requests that don't.

### RequestAuthentication

A RequestAuthentication resource defines the request authentication methods that are supported by a workload. Requests with invalid authentication information will be rejected. Requests with no authentication credentials will be accepted but will not have any authenticated identity.

1. Create a RequestAuthentication resource for the `httpbin` workload in the `mtls-service` namespace. This policy allows the workload to accept requests with a JWT issued by `testing@secure.istio.io`.

```yaml
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl apply -f - <<EOF
apiVersion: "security.istio.io/v1beta1"
kind: "RequestAuthentication"
metadata:
  name: "jwt-example"
  namespace: mtls-service
spec:
  selector:
    matchLabels:
      app: httpbin
  jwtRules:
  - issuer: "testing@secure.istio.io"
    jwksUri: "https://raw.githubusercontent.com/istio/istio/release-1.8/security/tools/jwt/samples/jwks.json"
EOF
requestauthentication.security.istio.io/jwt-example created
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Verify that a request with an invalid JWT is denied:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec "$(kubectl get pod -l app=sleep -n mtls-client -o jsonpath={.items..metadata.name})" -c sleep -n mtls-client -- curl "http://httpbin.mtls-service:8000/headers" -s -o /dev/null -H "Authorization: Bearer invalidToken" -w "%{http_code}\n"
401
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Verify that a request without any JWT is allowed:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec "$(kubectl get pod -l app=sleep -n mtls-client -o jsonpath={.items..metadata.name})" -c sleep -n mtls-client -- curl "http://httpbin.mtls-service:8000/headers" -s -o /dev/null -w "%{http_code}\n"
200
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

### AuthorizationPolicy

1. Create an AuthorizationPolicy resource for the `httpbin` workload in the `mtls-service` namespace:

```yaml
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: mtls-service
spec:
  selector:
    matchLabels:
      app: httpbin
  action: ALLOW
  rules:
  - from:
    - source:
       requestPrincipals: ["testing@secure.istio.io/testing@secure.istio.io"]
EOF
authorizationpolicy.security.istio.io/require-jwt created
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

The policy requires all requests to the `httpbin` workload to have a valid JWT with `requestPrincipal` set to `testing@secure.istio.io/testing@secure.istio.io`. Istio constructs the `requestPrincipal` by combining the `iss` and `sub` of the JWT token with a / separator as shown:



Download a legitimate JWT that can be used to send accepted requests:

```json
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ TOKEN=$(curl https://raw.githubusercontent.com/istio/istio/release-1.8/security/tools/jwt/samples/demo.jwt -s) && echo "$TOKEN" | cut -d '.' -f2 - | base64 --decode -
{"exp":4685989700,"foo":"bar","iat":1532389700,"iss":"testing@secure.istio.io","sub":"testing@secure.istio.io"}student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$
```

Note that the `iss` and `sub` keys are set to `testing@secure.istio.io`. This causes Istio to generate the attribute `requestPrincipal` with the value `testing@secure.istio.io/testing@secure.istio.io`

Verify that a request with a valid JWT is allowed:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec "$(kubectl get pod -l app=sleep -n mtls-client -o jsonpath={.items..metadata.name})" -c sleep -n mtls-client -- curl "http://httpbin.mtls-service:8000/headers" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"
200
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Verify that a request without a JWT is denied:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec "$(kubectl get pod -l app=sleep -n mtls-client -o jsonpath={.items..metadata.name})" -c sleep -n mtls-client -- curl "http://httpbin.mtls-service:8000/headers" -s -o /dev/null -w "%{http_code}\n"
403
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```



## Authorizing requests based on method and path

This task shows you how to control access to workloads by using an AuthorizationPolicy that evaluates the request type and URL.

Update the `require-jwt` authorization policy for the `httpbin` workload in the `mtls-service` namespace. The new policy will still have the JWT requirement that you set up in the previous task. In addition, you are going to limit the type of HTTP requests, so that clients can only perform GET requests to the `/ip` endpoint:

```yaml
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: mtls-service
spec:
  selector:
    matchLabels:
      app: httpbin
  action: ALLOW
  rules:
  - from:
    - source:
        requestPrincipals: ["testing@secure.istio.io/testing@secure.istio.io"]
    to:
    - operation:
        methods: ["GET"]
        paths: ["/ip"]
EOF
authorizationpolicy.security.istio.io/require-jwt configured
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Verify that a request to the `httpbin`'s `/ip` endpoint works:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec "$(kubectl get pod -l app=sleep -n mtls-client -o jsonpath={.items..metadata.name})" -c sleep -n mtls-client -- curl "http://httpbin.mtls-service:8000/ip" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"
200
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```



Verify that a request to the `httpbin`'s `/headers` endpoint is denied:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl exec "$(kubectl get pod -l app=sleep -n mtls-client -o jsonpath={.items..metadata.name})" -c sleep -n mtls-client -- curl "http://httpbin.mtls-service:8000/headers" -s -o /dev/null -H "Authorization: Bearer $TOKEN" -w "%{http_code}\n"
403
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```

Remove the **require-jwt** authorization policy by running this command:

```sh
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ kubectl delete AuthorizationPolicy require-jwt -n mtls-service
authorizationpolicy.security.istio.io "require-jwt" deleted
student_03_9e708a64b511@cloudshell:~ (qwiklabs-gcp-03-5d22baeeb714)$ 
```



In this lab, you explored **mutual TLS** authentication in Istio. You saw how PERMISSIVE mode mTLS allows services to receive both plaintext and mTLS traffic from clients, allowing you to incrementally adopt mTLS. You also enabled STRICT mode mTLS across your service mesh effectively blocking plaintext traffic to all your Istio injected services and then you scoped STRICT mode mTLS down to a single namespace.

In addition, you explored **RequestAuthentication** and **AuthorizationPolicy** resources in Istio.
