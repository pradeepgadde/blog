---

layout: single
title:  "Managing Traffic Flow with Anthos Service Mesh"
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

# Managing Traffic Flow with Anthos Service Mesh

With Anthos Service Mesh, you can manage service discovery, traffic routing, and load balancing for your services without having to update code in your services. Anthos Service Mesh simplifies configuration of service-level properties like timeouts and retries, and makes it straightforward to set up tasks like staged rollouts with percentage-based traffic splits.

Anthos Service Mesh’s traffic management model relies on the following two components:

- Control plane: manages and configures the Envoy proxies to route traffic and enforce policies.
- Data plane: encompasses all network communication between microservices performed at runtime by the Envoy proxies.

These components enable mesh traffic management features including:

- Service discovery
- Load balancing
- Traffic routing and control

Tasks:

- Configure and use Istio Gateways
- Apply default destination rules, for all available versions
- Apply virtual services to route by default to only one version
- Route to a specific version of a service based on user identity
- Shift traffic gradually from one version of a microservice to another
- Use the Anthos Service Mesh dashboard to view routing to multiple versions
- Setup networking best practices such as retries, circuit breakers and timeouts

## Review Traffic Management use cases

Different traffic management capabilities are enabled by using different configuration options.

### traffic splitting

Route traffic to multiple versions of a service.

### timeouts

Set a timeout, the amount of time Istio waits for a response to a request. The timeout for HTTP requests is 15 seconds, but it can be overridden.

### retries

A retry is an attempt to complete an operation multiple times if it fails. Adjust the maximum number of retry attempts, or the number of attempts possible within the default or overridden timeout period.

### fault injection: inserting delays

Fault injection is a testing method that introduces errors into a system to ensure that it can withstand and recover from error conditions.

### fault injection: inserting aborts

For example, returns an HTTP 400 error code for 10% of the requests to the ratings service "v1".

### conditional routing: based on source labels

A rule can indicate that it only applies to calls from workloads (pods) implementing the version v2 of the reviews service.

### conditional routing: based on request headers

For example, a rule only applies to an incoming request if it includes a custom "end-user" header that contains the string “pradeep”.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-0514bbab990a.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ CLUSTER_ZONE=us-central1-f
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ export CLUSTER_NAME=gke
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ export GCLOUD_PROJECT=$(gcloud config get-value project)
gcloud container clusters get-credentials $CLUSTER_NAME \
    --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
Your active configuration is: [cloudshell-21403]
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke.
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ gcloud container clusters list
NAME: gke
LOCATION: us-central1-f
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 35.222.53.61
MACHINE_TYPE: e2-standard-4
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 3
STATUS: RUNNING
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get pods -n istio-system
NAME                                 READY   STATUS    RESTARTS   AGE
istiod-asm-1157-23-79bf57569-528dp   1/1     Running   0          6h43m
istiod-asm-1157-23-79bf57569-nvnp6   1/1     Running   0          6h43m
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get service -n istio-system
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                 AGE
istiod               ClusterIP   10.108.150.172   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   6h43m
istiod-asm-1157-23   ClusterIP   10.108.159.127   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   6h43m
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
details-v1-cf74bb974-fz5g5       2/2     Running   0          6h43m
productpage-v1-87d54dd59-pntxh   2/2     Running   0          6h43m
ratings-v1-7c4bbf97db-kcclm      2/2     Running   0          6h43m
reviews-v1-5fd6d4f8f8-lmpl5      2/2     Running   0          6h43m
reviews-v2-6f9b55c5db-kfzp5      2/2     Running   0          6h43m
reviews-v3-7d99fd7978-gldjg      2/2     Running   0          6h43m
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get services
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.108.148.244   <none>        9080/TCP   6h43m
kubernetes    ClusterIP   10.108.144.1     <none>        443/TCP    7h5m
productpage   ClusterIP   10.108.145.43    <none>        9080/TCP   6h43m
ratings       ClusterIP   10.108.156.53    <none>        9080/TCP   6h43m
reviews       ClusterIP   10.108.148.105   <none>        9080/TCP   6h43m
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```



## Install Gateways to enable ingress

In a Kubernetes environment, the Kubernetes Ingress Resource is used to specify services that should be exposed outside the cluster. In Anthos Service Mesh, a better approach, which also works in Kubernetes and other environments, is to use a Gateway resource. A Gateway allows mesh features such as monitoring, mTLS, and advanced routing capabilities rules to be applied to traffic **entering** the cluster.

![The Istio gateways and the Mesh boundary layout.](https://cdn.qwiklabs.com/mEjOpgz8BLXxqQ0oS1W2iof8bMYV3NZKSX7eLftfATw%3D) 

Gateways overcome Kubernetes Ingress shortcomings by separating the L4-L6 spec from L7. The **Gateway** configures the L4-L6 functions, such as the ports to expose, or the protocol to use. Then service owners bind **VirtualService** to configure L7 traffic routing options, such as routing based on paths, headers, weights, etc.

There are two options for deploying gateways, either shared or dedicated. Shared gateways use a single centralized gateway is that used by many applications, possibly across many namespaces. In the example below, the Gateway in the ingress namespace delegates ownership of routes to application namespaces, but retains control over TLS configuration. This works well when using shared TLS certificates or shared infrastructure. In this lab we will use this option.

Dedicated gateways give full control and ownership to a single namespace, since an application namespace has its own dedicated gateway. This works well for applications that require isolation for security or performance.

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl create namespace ingress
namespace/ingress created
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl label namespace ingress \
  istio.io/rev=$(kubectl -n istio-system get pods -l app=istiod -o json | jq -r '.items[0].metadata.labels["istio.io/rev"]') \
  --overwrite
namespace/ingress labeled
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ git clone https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages
kubectl apply -n ingress -f anthos-service-mesh-packages/samples/gateways/istio-ingressgateway
Cloning into 'anthos-service-mesh-packages'...
remote: Enumerating objects: 11990, done.
remote: Counting objects: 100% (1395/1395), done.
remote: Compressing objects: 100% (482/482), done.
remote: Total 11990 (delta 1044), reused 1237 (delta 911), pack-reused 10595
Receiving objects: 100% (11990/11990), 3.02 MiB | 22.08 MiB/s, done.
Resolving deltas: 100% (8230/8230), done.
horizontalpodautoscaler.autoscaling/istio-ingressgateway created
deployment.apps/istio-ingressgateway created
poddisruptionbudget.policy/istio-ingressgateway created
role.rbac.authorization.k8s.io/istio-ingressgateway created
rolebinding.rbac.authorization.k8s.io/istio-ingressgateway created
service/istio-ingressgateway created
serviceaccount/istio-ingressgateway created
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get pod,service -n ingress
NAME                                        READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-59c87f6556-8gcq4   1/1     Running   0          61s
pod/istio-ingressgateway-59c87f6556-jqp47   1/1     Running   0          61s
pod/istio-ingressgateway-59c87f6556-xxcqb   1/1     Running   0          61s

NAME                           TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                                      AGE
service/istio-ingressgateway   LoadBalancer   10.108.155.2   35.193.148.208   15021:30753/TCP,80:31324/TCP,443:32316/TCP   63s
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
  namespace: ingress
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
EOF - "*":
gateway.networking.istio.io/bookinfo-gateway created
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - ingress/bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
EOF       number: 9080ageroducts
virtualservice.networking.istio.io/bookinfo configured
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get gateway,virtualservice
NAME                                           AGE
gateway.networking.istio.io/bookinfo-gateway   6h50m

NAME                                          GATEWAYS                       HOSTS   AGE
virtualservice.networking.istio.io/bookinfo   ["ingress/bookinfo-gateway"]   ["*"]   6h50m
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ export GATEWAY_URL=$(kubectl get svc -n ingress istio-ingressgateway \
-o=jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo The gateway address is $GATEWAY_URL
The gateway address is 35.193.148.208
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ sudo apt install siege
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  siege
0 upgraded, 1 newly installed, 0 to remove and 9 not upgraded.
Need to get 102 kB of archives.
After this operation, 269 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu jammy/main amd64 siege amd64 4.0.7-1build3 [102 kB]
Fetched 102 kB in 1s (78.4 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package siege.
(Reading database ... 121585 files and directories currently installed.)
Preparing to unpack .../siege_4.0.7-1build3_amd64.deb ...
Unpacking siege (4.0.7-1build3) ...
Setting up siege (4.0.7-1build3) ...
Processing triggers for man-db (2.10.2-1) ...
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 


```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ siege http://${GATEWAY_URL}/productpage
New configuration template added to /home/student_01_7fdd866025d9/.siege
Run siege -C to view the current settings in that file

```

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-0514bbab990a.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ CLUSTER_ZONE=us-central1-f
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ export CLUSTER_NAME=gke
export GCLOUD_PROJECT=$(gcloud config get-value project)
gcloud container clusters get-credentials $CLUSTER_NAME \
    --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
export GATEWAY_URL=$(kubectl get svc istio-ingressgateway \
-o=jsonpath='{.status.loadBalancer.ingress[0].ip}' -n ingress)
Your active configuration is: [cloudshell-25975]
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke.
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl exec -it \
$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') \
-c ratings -- curl productpage:9080/productpage \
| grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
                                   student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ curl -I http://${GATEWAY_URL}/productpage
HTTP/1.1 200 OK
server: istio-envoy
date: Sat, 27 Apr 2024 05:47:06 GMT
content-type: text/html; charset=utf-8
content-length: 4294
vary: Cookie
x-envoy-upstream-service-time: 17

student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ echo http://${GATEWAY_URL}/productpage
http://35.193.148.208/productpage
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```



**Congratulations!** You exposed an HTTP endpoint for the Bookinfo productpage service to external traffic. The Gateway configuration resources allow external traffic to enter the service mesh and make the traffic management and policy features available for edge services.

## Use the Anthos Service Mesh dashboard view routing to multiple versions

There are a couple of items to note when it comes to viewing data in the Anthos Service Mesh dashboard.

The first is that, for most pages, it takes 1-2 minutes for the data  to be available for display. That means that if you look at a page, it  might not have the data you expect for 1-2 minutes. If you don't see the data you want, wait for a minute or so and then refresh the page.

The Topology page also has a big initial delay before data is shown.  It can take up to 5+ minutes for the initial set of data to be  available. If you see a message that there is no data, wait a bit and  then refresh the page and return to the Topology view.

In the previous paragraphs, you are instructed to wait AND to refresh the page. As it turns out, not only is the data a bit delayed in  arriving, but many pages won't show the available data without a page  refresh. So if you expect the data to be available and you don't see it, make sure to do a refresh of the page in your browser.

## Apply default destination rules, for all available versions

In this task, you define all the available versions, called subsets, in destination rules.

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/destination-rule-all.yaml
destinationrule.networking.istio.io/productpage created
destinationrule.networking.istio.io/reviews created
destinationrule.networking.istio.io/ratings created
destinationrule.networking.istio.io/details created
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 

```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get destinationrules
NAME          HOST          AGE
details       details       19s
productpage   productpage   20s
ratings       ratings       19s
reviews       reviews       20s
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```yaml
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get destinationrules -o yaml
apiVersion: v1
items:
- apiVersion: networking.istio.io/v1beta1
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"details","namespace":"default"},"spec":{"host":"details","subsets":[{"labels":{"version":"v1"},"name":"v1"},{"labels":{"version":"v2"},"name":"v2"}]}}
    creationTimestamp: "2024-04-27T05:52:02Z"
    generation: 1
    name: details
    namespace: default
    resourceVersion: "276477"
    uid: 83461a5f-a6c6-4c15-9dd1-3f200fe4f1c9
  spec:
    host: details
    subsets:
    - labels:
        version: v1
      name: v1
    - labels:
        version: v2
      name: v2
- apiVersion: networking.istio.io/v1beta1
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"productpage","namespace":"default"},"spec":{"host":"productpage","subsets":[{"labels":{"version":"v1"},"name":"v1"}]}}
    creationTimestamp: "2024-04-27T05:52:01Z"
    generation: 1
    name: productpage
    namespace: default
    resourceVersion: "276457"
    uid: 399b55d0-80ff-4514-8944-204dec7d442a
  spec:
    host: productpage
    subsets:
    - labels:
        version: v1
      name: v1
- apiVersion: networking.istio.io/v1beta1
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"ratings","namespace":"default"},"spec":{"host":"ratings","subsets":[{"labels":{"version":"v1"},"name":"v1"},{"labels":{"version":"v2"},"name":"v2"},{"labels":{"version":"v2-mysql"},"name":"v2-mysql"},{"labels":{"version":"v2-mysql-vm"},"name":"v2-mysql-vm"}]}}
    creationTimestamp: "2024-04-27T05:52:02Z"
    generation: 1
    name: ratings
    namespace: default
    resourceVersion: "276474"
    uid: 9ab4d168-17d4-4d2a-8115-82311105eaff
  spec:
    host: ratings
    subsets:
    - labels:
        version: v1
      name: v1
    - labels:
        version: v2
      name: v2
    - labels:
        version: v2-mysql
      name: v2-mysql
    - labels:
        version: v2-mysql-vm
      name: v2-mysql-vm
- apiVersion: networking.istio.io/v1beta1
  kind: DestinationRule
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"host":"reviews","subsets":[{"labels":{"version":"v1"},"name":"v1"},{"labels":{"version":"v2"},"name":"v2"},{"labels":{"version":"v3"},"name":"v3"}]}}
    creationTimestamp: "2024-04-27T05:52:01Z"
    generation: 1
    name: reviews
    namespace: default
    resourceVersion: "276467"
    uid: a18742b6-7158-4e76-aed4-83547751160e
  spec:
    host: reviews
    subsets:
    - labels:
        version: v1
      name: v1
    - labels:
        version: v2
      name: v2
    - labels:
        version: v3
      name: v3
kind: List
metadata:
  resourceVersion: ""
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```



## Apply virtual services to route by default to only one version

In this task, you apply virtual services for each service that routes all traffic to v1 of the service workload.

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io/productpage created
virtualservice.networking.istio.io/reviews created
virtualservice.networking.istio.io/ratings created
virtualservice.networking.istio.io/details created
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get virtualservices
NAME          GATEWAYS                       HOSTS             AGE
bookinfo      ["ingress/bookinfo-gateway"]   ["*"]             7h
details                                      ["details"]       18s
productpage                                  ["productpage"]   19s
ratings                                      ["ratings"]       18s
reviews                                      ["reviews"]       19s
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ echo $GATEWAY_URL
35.193.148.208
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```



Test the new routing configuration using the Bookinfo UI.

- Open the Bookinfo site in your browser. The URL is `http://[GATEWAY_URL]/productpage`, where `GATEWAY_URL` is the External IP address of the ingress.
- Refresh the page a few times to issue multiple requests.

Notice that the **Book Reviews** part of the page displays with no rating stars, no matter how many times you refresh. This is because you configured the mesh to route all traffic for the reviews service to the version **reviews:v1** and this version of the service does **not** access the star ratings service.

## Route to a specific version of a service based on user identity

In this task, you change the route configuration so that all traffic from a specific user is routed to a specific service version. In this case, all traffic from user `pradeep` will be routed to the service `reviews:v2`, the version that includes the star ratings feature.

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
virtualservice.networking.istio.io/reviews configured
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl get virtualservice reviews
NAME      GATEWAYS   HOSTS         AGE
reviews              ["reviews"]   3m44s
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
curl -c cookies.txt -F "username=jason" -L -X \
    POST http://$GATEWAY_URL/login
cookie_info=$(grep -Eo "session.*" ./cookies.txt)
cookie_name=$(echo $cookie_info | cut -d' ' -f1)
cookie_value=$(echo $cookie_info | cut -d' ' -f2)
siege -c 5 http://$GATEWAY_URL/productpage \
    --header "Cookie: $cookie_name=$cookie_value"
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl delete -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io "productpage" deleted
virtualservice.networking.istio.io "reviews" deleted
virtualservice.networking.istio.io "ratings" deleted
virtualservice.networking.istio.io "details" deleted
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```



## Shift traffic gradually from one version of a microservice to another

In this task, you gradually migrate traffic from one version of a microservice to another. For example, you might use this approach to migrate traffic from an older version to a new version.

You will send 50% of traffic to reviews:v1 and 50% to reviews:v3. Then, you will complete the migration by sending 100% of traffic to reviews:v3.

In Anthos Service Mesh, you accomplish this goal by configuring a sequence of rules that route a percentage of traffic to one service or another.

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io/productpage created
virtualservice.networking.istio.io/reviews created
virtualservice.networking.istio.io/ratings created
virtualservice.networking.istio.io/details created
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
tudent_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f \
    https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
virtualservice.networking.istio.io/reviews configured
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking//virtual-service-reviews-v3.yaml
virtualservice.networking.istio.io/reviews configured
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl delete -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io "productpage" deleted
virtualservice.networking.istio.io "reviews" deleted
virtualservice.networking.istio.io "ratings" deleted
virtualservice.networking.istio.io "details" deleted
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```



## Add timeouts to avoid waiting indefinitely for service replies

A timeout for HTTP requests can be specified using the timeout field of the route rule. By default, the request timeout is disabled, but in this task you override the reviews service timeout to 1 second. To see its effect, however, you also introduce an artificial 2 second delay in calls to the ratings service. We will start by introducing the delay.

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io/productpage created
virtualservice.networking.istio.io/reviews created
virtualservice.networking.istio.io/ratings created
virtualservice.networking.istio.io/details created
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```yaml
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
EOF
virtualservice.networking.istio.io/reviews configured
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```yaml
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        percent: 100
        fixedDelay: 2s
    route:
    - destination:
        host: ratings
        subset: v1
EOF
virtualservice.networking.istio.io/ratings configured
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```yaml
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
    timeout: 0.5s
EOF
virtualservice.networking.istio.io/reviews configured
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```yaml
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - productpage
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
    retries:
      attempts: 1
      perTryTimeout: 2s
EOF
virtualservice.networking.istio.io/productpage configured
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl delete -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io "productpage" deleted
virtualservice.networking.istio.io "reviews" deleted
virtualservice.networking.istio.io "ratings" deleted
virtualservice.networking.istio.io "details" deleted
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```



## Add circuit breakers to enhance your microservices' resiliency

This task shows you how to configure circuit breaking for connections, requests, and outlier detection.

Circuit breaking is an important pattern for creating resilient microservice applications. Circuit breaking allows you to write applications that limit the impact of failures, latency spikes, and other undesirable effects of network peculiarities.

In this task, you will configure circuit breaking rules and then test the configuration by intentionally “tripping” the circuit breaker.

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io/productpage created
virtualservice.networking.istio.io/reviews created
virtualservice.networking.istio.io/ratings created
virtualservice.networking.istio.io/details created
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```yaml
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
      version: v1
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
EOF   maxEjectionPercent: 100
destinationrule.networking.istio.io/productpage configured
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ siege -c 5 http://$GATEWAY_URL/productpage \
>     --header "Cookie: $cookie_name=$cookie_value"~
^C
{       "transactions":                         3231,
        "availability":                        99.60,
        "elapsed_time":                       514.82,
        "data_transferred":                   169.53,
        "response_time":                        0.80,
        "transaction_rate":                     6.28,
        "throughput":                           0.33,
        "concurrency":                          4.99,
        "successful_transactions":              3231,
        "failed_transactions":                    13,
        "longest_transaction":                  2.47,
        "shortest_transaction":                 0.41
}
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```


    Create a client to send traffic to the productpage service.

The client is a simple load-testing client called fortio. Fortio lets you control the number of connections, concurrency, and delays for outgoing HTTP calls. You will use this client to “trip” the circuit breaker policies you set in the DestinationRule:

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$   kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.9/samples/httpbin/sample-client/fortio-deploy.yaml
service/fortio created
deployment.apps/fortio-deploy created
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```
```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ export FORTIO_POD=$(kubectl get pods -lapp=fortio -o 'jsonpath={.items[0].metadata.name}')
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio curl -quiet http://${GATEWAY_URL}/productpage
HTTP/1.1 200 OK
server: envoy
date: Sat, 27 Apr 2024 06:09:12 GMT
content-type: text/html; charset=utf-8
content-length: 4294
vary: Cookie
x-envoy-upstream-service-time: 26

<!DOCTYPE html>
<html>
  <head>
    <title>Simple Bookstore App</title>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Latest compiled and minified CSS -->
<link rel="stylesheet" href="static/bootstrap/css/bootstrap.min.css">

<!-- Optional theme -->
<link rel="stylesheet" href="static/bootstrap/css/bootstrap-theme.min.css">

  </head>
  <body>
    
    

<nav class="navbar navbar-inverse navbar-static-top">
  <div class="container">
    <div class="navbar-header">
      <a class="navbar-brand" href="#">BookInfo Sample</a>
    </div>
    
    <button type="button" class="btn btn-default navbar-btn navbar-right" data-toggle="modal" href="#login-modal">Sign
      in</button>
    
  </div>
</nav>

<!---
<div class="navbar navbar-inverse navbar-fixed-top">
  <div class="container">
    <div class="navbar-header pull-left">
      <a class="navbar-brand" href="#">Microservices Fabric BookInfo Demo</a>
    </div>
    <div class="navbar-header pull-right">
      <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
    </div>
    <div class="navbar-collapse collapse">

      <button type="button" class="btn btn-default navbar-btn pull-right" data-toggle="modal" data-target="#login-modal">Sign in</button>

    </div>
  </div>
</div>
-->

<div id="login-modal" class="modal fade" role="dialog">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal">&times;</button>
        <h4 class="modal-title">Please sign in</h4>
      </div>
      <div class="modal-body">
        <form method="post" action='login' name="login_form">
          <p><input type="text" class="form-control" name="username" id="username" placeholder="User Name"></p>
          <p><input type="password" class="form-control" name="passwd" placeholder="Password"></p>
          <p>
            <button type="submit" class="btn btn-primary">Sign in</button>
            <button type="button" class="btn btn-default" data-dismiss="modal">Cancel</button>
          </p>
        </form>
      </div>
    </div>

  </div>
</div>

<div class="container-fluid">
  <div class="row">
    <div class="col-md-12">
      <h3 class="text-center text-primary">The Comedy of Errors</h3>
      
      <p>Summary: <a href="https://en.wikipedia.org/wiki/The_Comedy_of_Errors">Wikipedia Summary</a>: The Comedy of Errors is one of <b>William Shakespeare's</b> early plays. It is his shortest and one of his most farcical comedies, with a major part of the humour coming from slapstick and mistaken identity, in addition to puns and word play.</p>
      
    </div>
  </div>

  <div class="row">
    <div class="col-md-6">
      
      <h4 class="text-center text-primary">Book Details</h4>
      <dl>
        <dt>Type:</dt>paperback
        <dt>Pages:</dt>200
        <dt>Publisher:</dt>PublisherA
        <dt>Language:</dt>English
        <dt>ISBN-10:</dt>1234567890
        <dt>ISBN-13:</dt>123-1234567890
      </dl>
      
    </div>

    <div class="col-md-6">
      
      <h4 class="text-center text-primary">Book Reviews</h4>
      
      <blockquote>
        <p>An extremely entertaining play by Shakespeare. The slapstick humour is refreshing!</p>
        <small>Reviewer1</small>
        
      </blockquote>
      
      <blockquote>
        <p>Absolutely fun and entertaining. The play lacks thematic depth when compared to other plays by Shakespeare.</p>
        <small>Reviewer2</small>
        
      </blockquote>
      
      <dl>
        <dt>Reviews served by:</dt>
        <u>reviews-v1-5fd6d4f8f8-lmpl5</u>
        
      </dl>
      
    </div>
  </div>
</div>


    
<!-- Latest compiled and minified JavaScript -->
<script src="static/jquery.min.js"></script>

<!-- Latest compiled and minified JavaScript -->
<script src="static/bootstrap/js/bootstrap.min.js"></script>

<script type="text/javascript">
  $('#login-modal').on('shown.bs.modal', function () {
    $('#username').focus();
  });
</script>

  </body>
</html>
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```
```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -n 20 -loglevel Warning http://${GATEWAY_URL}/productpage
{"ts":1714198196.786403,"level":"info","r":1,"file":"logger.go","line":254,"msg":"Log level is now 3 Warning (was 2 Info)"}
Fortio 1.60.3 running at 0 queries per second, 4->4 procs, for 20 calls: http://35.193.148.208/productpage
Starting at max qps with 2 thread(s) [gomax 4] for exactly 20 calls (10 per thread + 0)
{"ts":1714198196.793022,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.795846,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.798241,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.800698,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.802598,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.805004,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.807610,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.810159,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.812100,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.814339,"level":"warn","r":8,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198196.816127,"level":"warn","r":7,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198196.818248,"level":"warn","r":7,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198196.820075,"level":"warn","r":7,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198196.821870,"level":"warn","r":7,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198196.824100,"level":"warn","r":7,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198196.826122,"level":"warn","r":7,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198196.828159,"level":"warn","r":7,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198196.829959,"level":"warn","r":7,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198196.831755,"level":"warn","r":7,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
Ended after 44.034415ms : 20 calls. qps=454.19
Aggregated Function Time : count 20 avg 0.0034918101 +/- 0.005218 min 0.001785869 max 0.026026445 sum 0.069836203
# range, mid point, percentile, count
>= 0.00178587 <= 0.002 , 0.00189293 , 35.00, 7
> 0.002 <= 0.003 , 0.0025 , 90.00, 11
> 0.005 <= 0.006 , 0.0055 , 95.00, 1
> 0.025 <= 0.0260264 , 0.0255132 , 100.00, 1
# target 50% 0.00227273
# target 75% 0.00272727
# target 90% 0.003
# target 99% 0.0258212
# target 99.9% 0.0260059
Error cases : count 19 avg 0.0023057767 +/- 0.0007294 min 0.001785869 max 0.005152399 sum 0.043809758
# range, mid point, percentile, count
>= 0.00178587 <= 0.002 , 0.00189293 , 36.84, 7
> 0.002 <= 0.003 , 0.0025 , 94.74, 11
> 0.005 <= 0.0051524 , 0.0050762 , 100.00, 1
# target 50% 0.00222727
# target 75% 0.00265909
# target 90% 0.00291818
# target 99% 0.00512344
# target 99.9% 0.0051495
# Socket and IP used for each connection:
[0]   9 socket used, resolved to 35.193.148.208:80, connection timing : count 9 avg 0.00025037311 +/- 7.468e-05 min 0.000148124 max 0.00038296 sum 0.002253358
[1]  10 socket used, resolved to 35.193.148.208:80, connection timing : count 10 avg 0.0003256097 +/- 0.0001156 min 0.000153109 max 0.00048716 sum 0.003256097
Connection time (s) : count 19 avg 0.00028997132 +/- 0.0001053 min 0.000148124 max 0.00048716 sum 0.005509455
Sockets used: 19 (for perfect keepalive, would be 2)
Uniform: false, Jitter: false, Catchup allowed: true
IP addresses distribution:
35.193.148.208:80: 19
Code 200 : 1 (5.0 %)
Code 503 : 19 (95.0 %)
Response Header Sizes : count 20 avg 9.1 +/- 39.67 min 0 max 182 sum 182
Response Body/Total Sizes : count 20 avg 401.45 +/- 934.8 min 187 max 4476 sum 8029
All done 20 calls (plus 0 warmup) 3.492 ms avg, 454.2 qps
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

Bring the number of concurrent connections up to 3:

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 3 -qps 0 -n 30 -loglevel Warning http://${GATEWAY_URL}/productpage
{"ts":1714198247.981977,"level":"info","r":1,"file":"logger.go","line":254,"msg":"Log level is now 3 Warning (was 2 Info)"}
Fortio 1.60.3 running at 0 queries per second, 4->4 procs, for 30 calls: http://35.193.148.208/productpage
Starting at max qps with 3 thread(s) [gomax 4] for exactly 30 calls (10 per thread + 0)
{"ts":1714198247.986388,"level":"warn","r":19,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198247.988333,"level":"warn","r":19,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198248.002132,"level":"warn","r":20,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198248.011439,"level":"warn","r":21,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":2,"run":0}
{"ts":1714198248.014434,"level":"warn","r":21,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":2,"run":0}
{"ts":1714198248.016551,"level":"warn","r":21,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":2,"run":0}
{"ts":1714198248.018328,"level":"warn","r":21,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":2,"run":0}
{"ts":1714198248.020151,"level":"warn","r":21,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":2,"run":0}
{"ts":1714198248.022431,"level":"warn","r":21,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":2,"run":0}
{"ts":1714198248.024660,"level":"warn","r":21,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":2,"run":0}
{"ts":1714198248.026743,"level":"warn","r":21,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":2,"run":0}
{"ts":1714198248.028833,"level":"warn","r":21,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":2,"run":0}
{"ts":1714198248.067171,"level":"warn","r":19,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198248.126482,"level":"warn","r":20,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
{"ts":1714198248.186654,"level":"warn","r":19,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198248.189237,"level":"warn","r":19,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":0,"run":0}
{"ts":1714198248.245309,"level":"warn","r":20,"file":"http_client.go","line":1104,"msg":"Non ok http code","code":503,"status":"HTTP/1.1 503","thread":1,"run":0}
Ended after 557.781663ms : 30 calls. qps=53.784
Aggregated Function Time : count 30 avg 0.032055648 +/- 0.0412 min 0.001781338 max 0.122483374 sum 0.961669437
# range, mid point, percentile, count
>= 0.00178134 <= 0.002 , 0.00189067 , 10.00, 3
> 0.002 <= 0.003 , 0.0025 , 53.33, 13
> 0.018 <= 0.02 , 0.019 , 60.00, 2
> 0.02 <= 0.025 , 0.0225 , 63.33, 1
> 0.025 <= 0.03 , 0.0275 , 70.00, 2
> 0.06 <= 0.07 , 0.065 , 80.00, 3
> 0.07 <= 0.08 , 0.075 , 83.33, 1
> 0.08 <= 0.09 , 0.085 , 86.67, 1
> 0.1 <= 0.12 , 0.11 , 96.67, 3
> 0.12 <= 0.122483 , 0.121242 , 100.00, 1
# target 50% 0.00292308
# target 75% 0.065
# target 90% 0.106667
# target 99% 0.121738
# target 99.9% 0.122409
Error cases : count 17 avg 0.0032099762 +/- 0.003861 min 0.001781338 max 0.018593014 sum 0.054569595
# range, mid point, percentile, count
>= 0.00178134 <= 0.002 , 0.00189067 , 17.65, 3
> 0.002 <= 0.003 , 0.0025 , 94.12, 13
> 0.018 <= 0.018593 , 0.0182965 , 100.00, 1
# target 50% 0.00242308
# target 75% 0.00275
# target 90% 0.00294615
# target 99% 0.0184922
# target 99.9% 0.0185829
# Socket and IP used for each connection:
[0]   6 socket used, resolved to 35.193.148.208:80, connection timing : count 6 avg 0.00030060767 +/- 6.854e-05 min 0.00016685 max 0.000387157 sum 0.001803646
[1]   4 socket used, resolved to 35.193.148.208:80, connection timing : count 4 avg 0.00025245275 +/- 7.65e-05 min 0.000178073 max 0.000360274 sum 0.001009811
[2]   9 socket used, resolved to 35.193.148.208:80, connection timing : count 9 avg 0.00025858056 +/- 7.605e-05 min 0.000168617 max 0.000439194 sum 0.002327225
Connection time (s) : count 19 avg 0.00027056221 +/- 7.667e-05 min 0.00016685 max 0.000439194 sum 0.005140682
Sockets used: 19 (for perfect keepalive, would be 3)
Uniform: false, Jitter: false, Catchup allowed: true
IP addresses distribution:
35.193.148.208:80: 19
Code 200 : 13 (43.3 %)
Code 503 : 17 (56.7 %)
Response Header Sizes : count 30 avg 79 +/- 90.34 min 0 max 183 sum 2370
Response Body/Total Sizes : count 30 avg 2075.0667 +/- 2100 min 187 max 4477 sum 62252
All done 30 calls (plus 0 warmup) 32.056 ms avg, 53.8 qps
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

Now you start to see the expected circuit breaking behavior. Only 43.3% of the requests succeeded and the rest were trapped by circuit breaking:



```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ history 
    1  CLUSTER_ZONE=us-central1-f
    2  export CLUSTER_NAME=gke
    3  export GCLOUD_PROJECT=$(gcloud config get-value project)
    4  gcloud container clusters get-credentials $CLUSTER_NAME     --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
    5  gcloud container clusters list
    6  kubectl get pods -n istio-system
    7  kubectl get service -n istio-system
    8  kubectl get pods
    9  kubectl get services
   10  kubectl create namespace ingress
   11  kubectl label namespace ingress   istio.io/rev=$(kubectl -n istio-system get pods -l app=istiod -o json | jq -r '.items[0].metadata.labels["istio.io/rev"]')   --overwrite
   12  git clone https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages
   13  kubectl apply -n ingress -f anthos-service-mesh-packages/samples/gateways/istio-ingressgateway
   14  kubectl get pod,service -n ingress
   15  cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
  namespace: ingress
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
EOF

   16  cat <<EOF | kubectl apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - ingress/bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
EOF

   17  kubectl get gateway,virtualservice
   18  export GATEWAY_URL=$(kubectl get svc -n ingress istio-ingressgateway \
-o=jsonpath='{.status.loadBalancer.ingress[0].ip}')
   19  echo The gateway address is $GATEWAY_URL
   20  sudo apt install siege
   21  siege http://${GATEWAY_URL}/productpage
   22  curl -c cookies.txt -F "username=jason" -L -X     POST http://$GATEWAY_URL/login
   23  cookie_info=$(grep -Eo "session.*" ./cookies.txt)
   24  cookie_name=$(echo $cookie_info | cut -d' ' -f1)
   25  cookie_value=$(echo $cookie_info | cut -d' ' -f2)
   26  siege -c 5 http://$GATEWAY_URL/productpage     --header "Cookie: $cookie_name=$cookie_value"~
   27  export FORTIO_POD=$(kubectl get pods -lapp=fortio -o 'jsonpath={.items[0].metadata.name}')
   28  kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio curl -quiet http://${GATEWAY_URL}/productpage
   29  kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -n 20 -loglevel Warning http://${GATEWAY_URL}/productpage
   30  kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 3 -qps 0 -n 30 -loglevel Warning http://${GATEWAY_URL}/productpage
   31  history 
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

```sh
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ history 
    1  CLUSTER_ZONE=us-central1-f
    2  export CLUSTER_NAME=gke
    3  export GCLOUD_PROJECT=$(gcloud config get-value project)
    4  gcloud container clusters get-credentials $CLUSTER_NAME     --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
    5  gcloud container clusters list
    6  kubectl get pods -n istio-system
    7  kubectl get service -n istio-system
    8  kubectl get pods
    9  kubectl get services
   10  kubectl create namespace ingress
   11  kubectl label namespace ingress   istio.io/rev=$(kubectl -n istio-system get pods -l app=istiod -o json | jq -r '.items[0].metadata.labels["istio.io/rev"]')   --overwrite
   12  git clone https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages
   13  kubectl apply -n ingress -f anthos-service-mesh-packages/samples/gateways/istio-ingressgateway
   14  kubectl get pod,service -n ingress
   15  cat <<EOF | kubectl apply -f -
   16  apiVersion: networking.istio.io/v1alpha3
   17  kind: Gateway
   18  metadata:
   19    name: bookinfo-gateway
   20    namespace: ingress
   21  spec:
   22    selector:
   23      istio: ingressgateway
   24    servers:
   25    - port:
   26        number: 80
   27        name: http
   28        protocol: HTTP
   29      hosts:
   30      - "*"
   31  EOF
   32  cat <<EOF | kubectl apply -f -
   33  apiVersion: networking.istio.io/v1alpha3
   34  kind: VirtualService
   35  metadata:
   36    name: bookinfo
   37  spec:
   38    hosts:
   39    - "*"
   40    gateways:
   41    - ingress/bookinfo-gateway
   42    http:
   43    - match:
   44      - uri:
   45          exact: /productpage
   46      - uri:
   47          prefix: /static
   48      - uri:
   49          exact: /login
   50      - uri:
   51          exact: /logout
   52      - uri:
   53          prefix: /api/v1/products
   54      route:
   55      - destination:
   56          host: productpage
   57          port:
   58            number: 9080
   59  EOF
   60  kubectl get gateway,virtualservice
   61  export GATEWAY_URL=$(kubectl get svc -n ingress istio-ingressgateway \
   62  -o=jsonpath='{.status.loadBalancer.ingress[0].ip}')
   63  echo The gateway address is $GATEWAY_URL
   64  sudo apt install siege
   65  CLUSTER_ZONE=us-central1-f
   66  export CLUSTER_NAME=gke
   67  export GCLOUD_PROJECT=$(gcloud config get-value project)
   68  gcloud container clusters get-credentials $CLUSTER_NAME     --zone $CLUSTER_ZONE --project $GCLOUD_PROJECT
   69  export GATEWAY_URL=$(kubectl get svc istio-ingressgateway \
-o=jsonpath='{.status.loadBalancer.ingress[0].ip}' -n ingress)
   70  kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
   71  curl -I http://${GATEWAY_URL}/productpage
   72  echo http://${GATEWAY_URL}/productpage
   73  kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/destination-rule-all.yaml
   74  kubectl get destinationrules
   75  kubectl get destinationrules -o yaml
   76  kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
   77  kubectl get virtualservices
   78  echo $GATEWAY_URL
   79  kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
   80  kubectl get virtualservice reviews
   81  kubectl delete -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
   82  kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
   83  kubectl apply -f     https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
   84  kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking//virtual-service-reviews-v3.yaml
   85  kubectl delete -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
   86  kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
   87  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
EOF

   88  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        percent: 100
        fixedDelay: 2s
    route:
    - destination:
        host: ratings
        subset: v1
EOF

   89  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
    timeout: 0.5s
EOF

   90  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
spec:
  hosts:
  - productpage
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
    retries:
      attempts: 1
      perTryTimeout: 2s
EOF

   91  kubectl delete -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
   92  kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/virtual-service-all-v1.yaml
   93  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
      version: v1
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutive5xxErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100
EOF

   94  history 
student_01_7fdd866025d9@cloudshell:~ (qwiklabs-gcp-02-0514bbab990a)$ 
```

