---
layout: single
title:  "Observing Anthos Service Mesh"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
## Observing Anthos Service Mesh 

A service mesh gives you a framework for connecting, securing, and managing microservices. It provides a networking layer on top of Kubernetes with features such as advanced load balancing capabilities, service-to-service authentication, and monitoring without requiring any changes in service code.

Anthos Service Mesh has a suite of additional features and tools that help you observe and manage secure, reliable services in a unified way. 

- Install Anthos Service Mesh, with tracing enabled and configured to use Cloud Trace as the backend.

- Deploy Bookinfo, an Istio-enabled multi-service application.

- Enable external access using an Istio Ingress Gateway.

- Use the Bookinfo application.

- Evaluate service performance using Cloud Trace features within Google Cloud.

- Create and monitor service-level objectives (SLOs).

- Leverage the Anthos Service Mesh Dashboard to understand service performance.

  

A Google Kubernetes Engine (GKE) cluster named **gke** has already been created and registered. You will install Anthos Service Mesh onto this cluster and override the standard configuration to enable the optional tracing components.

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-a894eae6dd20.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ CLUSTER_NAME=gke
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ CLUSTER_ZONE=us-central1-b
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ PROJECT_ID=qwiklabs-gcp-00-a894eae6dd20
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} \
  --format="value(projectNumber)")
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ FLEET_PROJECT_ID="${PROJECT_ID}"
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ IDNS="${PROJECT_ID}.svc.id.goog"
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ DIR_PATH=.
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ printf '\nCLUSTER_NAME:'$CLUSTER_NAME'\nCLUSTER_ZONE:'$CLUSTER_ZONE'\nPROJECT_ID:'$PROJECT_ID'\nPROJECT_NUMBER:'$PROJECT_NUMBER'\nFLEET PROJECT_ID:'$FLEET_PROJECT_ID'\nIDNS:'$IDNS'\nDIR_PATH:'$DIR_PATH'\n'

CLUSTER_NAME:gke
CLUSTER_ZONE:us-central1-b
PROJECT_ID:qwiklabs-gcp-00-a894eae6dd20
PROJECT_NUMBER:847981355858
FLEET PROJECT_ID:qwiklabs-gcp-00-a894eae6dd20
IDNS:qwiklabs-gcp-00-a894eae6dd20.svc.id.goog
DIR_PATH:.
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$  gcloud container clusters get-credentials $CLUSTER_NAME \
     --zone $CLUSTER_ZONE --project $PROJECT_ID
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke.
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://34.122.33.209
  name: gke_qwiklabs-gcp-00-a894eae6dd20_us-central1-b_gke
contexts:
- context:
    cluster: gke_qwiklabs-gcp-00-a894eae6dd20_us-central1-b_gke
    user: gke_qwiklabs-gcp-00-a894eae6dd20_us-central1-b_gke
  name: gke_qwiklabs-gcp-00-a894eae6dd20_us-central1-b_gke
current-context: gke_qwiklabs-gcp-00-a894eae6dd20_us-central1-b_gke
kind: Config
preferences: {}
users:
- name: gke_qwiklabs-gcp-00-a894eae6dd20_us-central1-b_gke
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args: null
      command: gke-gcloud-auth-plugin
      env: null
      installHint: Install gke-gcloud-auth-plugin for use with kubectl by following
        https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#install_plugin
      interactiveMode: IfAvailable
      provideClusterInfo: true
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$  gcloud container clusters list
NAME: gke
LOCATION: us-central1-b
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 34.122.33.209
MACHINE_TYPE: e2-standard-2
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 3
STATUS: RUNNING
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ sudo curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.15 -o /usr/bin/asmcli && sudo chmod +x /usr/bin/asmcli
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  182k  100  182k    0     0   294k      0 --:--:-- --:--:-- --:--:--  294k
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$  asmcli --version
1.15.7-asm.23+config1
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

Use `asmcli` to install Anthos Service Mesh:

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ sudo curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.15 -o /usr/bin/asmcli && sudo chmod +x /usr/bin/asmcli
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  182k  100  182k    0     0   294k      0 --:--:-- --:--:-- --:--:--  294k
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$  asmcli --version
1.15.7-asm.23+config1
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$  asmcli install \
 --project_id $PROJECT_ID \
 --cluster_name $CLUSTER_NAME \
 --cluster_location $CLUSTER_ZONE \
 --fleet_id $FLEET_PROJECT_ID \
 --output_dir $DIR_PATH \
 --managed \
 --enable_all \
 --ca mesh_ca
asmcli: Setting up necessary files...
asmcli: Using /home/student_04_887db0b7be5f/asm_kubeconfig as the kubeconfig...
asmcli: Checking installation tool dependencies...
asmcli: Fetching/writing GCP credentials to kubeconfig file...
asmcli: [WARNING]: nc not found, skipping k8s connection verification
asmcli: [WARNING]: (Installation will continue normally.)
asmcli: Getting account information...
asmcli: Downloading kpt..
asmcli: Downloading ASM..
asmcli: Downloading ASM kpt package...
fetching package "/asm" from "https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages" to "asm"
fetching package "/samples" from "https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages" to "samples"
asmcli: Verifying cluster registration.
asmcli: Enabling required APIs...
asmcli: Enabling the service mesh feature...
asmcli: Verifying cluster registration.
asmcli: Binding user:student-04-887db0b7be5f@qwiklabs.net to required IAM roles...
asmcli: Registering the cluster as gke...
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-00-a894eae6dd20
asmcli: Checking for project qwiklabs-gcp-00-a894eae6dd20...
asmcli: Reading labels for us-central1-b/gke...
asmcli: Querying for core/account...
asmcli: Binding student-04-887db0b7be5f@qwiklabs.net to cluster admin role...
clusterrolebinding.rbac.authorization.k8s.io/student-04-887db0b7be5f-cluster-admin-binding created
asmcli: Creating istio-system namespace...
namespace/istio-system created
asmcli: Configuring kpt package...
asm/
set 16 field(s) of setter "gcloud.container.cluster" to value "gke"
asm/
set 19 field(s) of setter "gcloud.core.project" to value "qwiklabs-gcp-00-a894eae6dd20"
asm/
set 2 field(s) of setter "gcloud.project.projectNumber" to value "847981355858"
asm/
set 16 field(s) of setter "gcloud.compute.location" to value "us-central1-b"
asm/
set 1 field(s) of setter "gcloud.compute.network" to value "qwiklabs-gcp-00-a894eae6dd20-default"
asm/
set 3 field(s) of setter "gcloud.project.environProjectNumber" to value "847981355858"
asm/
set 2 field(s) of setter "anthos.servicemesh.rev" to value "asm-managed"
asm/
set 3 field(s) of setter "anthos.servicemesh.tag" to value "1.15.7-asm.23"
asm/
set 3 field(s) of setter "anthos.servicemesh.trustDomain" to value "qwiklabs-gcp-00-a894eae6dd20.svc.id.goog"
asm/
set 1 field(s) of setter "anthos.servicemesh.tokenAudiences" to value "istio-ca,qwiklabs-gcp-00-a894eae6dd20.svc.id.goog"
asm/
set 3 field(s) of setter "anthos.servicemesh.created-by" to value "asmcli-1.15.7-asm.23.config1"
asm/
set 2 field(s) of setter "anthos.servicemesh.idp-url" to value "https://container.googleapis.com/v1/projects/qwiklabs-gcp-00-a894eae6dd20/locations/us-central1-b/clusters/gke"
asm/
set 2 field(s) of setter "anthos.servicemesh.trustDomainAliases" to value "qwiklabs-gcp-00-a894eae6dd20.svc.id.goog"
namespace/istio-system labeled
asmcli: Waiting for the controlplanerevisions CRD to be installed by AFC. This could take a few minutes if cluster is newly registered.
customresourcedefinition.apiextensions.k8s.io/controlplanerevisions.mesh.cloud.google.com condition met
asmcli: Applying mcp_configmap.yaml...
configmap/asm-options created
asmcli: Configuring ASM managed control plane revision CR for channels...
asmcli: Installing ASM Control Plane Revision CR with asm-managed channel in istio-system namespace...
controlplanerevision.mesh.cloud.google.com/asm-managed created
asmcli: Waiting for deployment...


controlplanerevision.mesh.cloud.google.com/asm-managed condition met
asmcli: 
asmcli: *****************************
1.15.7-asm.23
asmcli: *****************************
asmcli: The ASM control plane installation is now complete.
asmcli: To enable automatic sidecar injection on a namespace, you can use the following command:
asmcli: kubectl label namespace <NAMESPACE> istio-injection- istio.io/rev=asm-managed --overwrite
asmcli: If you use 'istioctl install' afterwards to modify this installation, you will need
asmcli: to specify the option '--set revision=asm-managed' to target this control plane
asmcli: instead of installing a new one.
asmcli: To finish the installation, enable Istio sidecar injection and restart your workloads.
asmcli: For more information, see:
asmcli: https://cloud.google.com/service-mesh/docs/proxy-injection
asmcli: The ASM package used for installation can be found at:
asmcli: /home/student_04_887db0b7be5f/asm
asmcli: The version of istioctl that matches the installation can be found at:
asmcli: /home/student_04_887db0b7be5f/istio-1.15.7-asm.23/bin/istioctl
asmcli: A symlink to the istioctl binary can be found at:
asmcli: /home/student_04_887db0b7be5f/istioctl
asmcli: *****************************
asmcli: Successfully installed ASM.
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

Enable Anthos Service Mesh to send telemetry to Cloud Trace:

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ cat <<EOF | kubectl apply -f -
apiVersion: v1
data:
  mesh: |-
    defaultConfig:
      tracing:
        stackdriver: {}
kind: ConfigMap
metadata:
  name: istio-asm-managed
  namespace: istio-system
EOF
Warning: resource configmaps/istio-asm-managed is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
configmap/istio-asm-managed configured
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl get configmap
NAME               DATA   AGE
kube-root-ca.crt   1      162m
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

You now have a GKE cluster with Anthos Service Mesh installed. Kubernetes Metrics are being recorded to Cloud Monitoring, logs are being recorded to Cloud Logging, and distributed trace information is being sent to Cloud Trace.



**Online Boutique** is a cloud-native microservices demo application. Online Boutique consists of a 10-tier microservices application. The application is a web-based ecommerce app where users can browse items, add them to the cart, and purchase them.

Google uses this application to demonstrate use of technologies like Kubernetes/GKE, Istio/ASM, Google Operations Suite, gRPC and OpenCensus. This application works on any Kubernetes cluster (such as a local one) and on Google Kubernetes Engine. It’s easy to deploy with little to no configuration.



Enable Istio sidecar injection:

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl label namespace default istio.io/rev=asm-managed --overwrite
namespace/default labeled
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

To enable Google to manage your data plane so that the sidecar proxies will be automatically updated for you, annotate the namespace:

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl annotate --overwrite namespace default \
  mesh.cloud.google.com/proxy='{"managed":"true"}'
namespace/default annotated
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

### Install the Online Boutique application on the GKE cluster

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml
kubectl patch deployments/productcatalogservice -p '{"spec":{"template":{"metadata":{"labels":{"version":"v1"}}}}}'
deployment.apps/emailservice created
service/emailservice created
deployment.apps/checkoutservice created
service/checkoutservice created
deployment.apps/recommendationservice created
service/recommendationservice created
deployment.apps/frontend created
service/frontend created
service/frontend-external created
deployment.apps/paymentservice created
service/paymentservice created
deployment.apps/productcatalogservice created
service/productcatalogservice created
deployment.apps/cartservice created
service/cartservice created
deployment.apps/loadgenerator created
deployment.apps/currencyservice created
service/currencyservice created
deployment.apps/shippingservice created
service/shippingservice created
deployment.apps/redis-cart created
service/redis-cart created
deployment.apps/adservice created
service/adservice created
deployment.apps/productcatalogservice patched
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

To be able to access the application from outside the cluster, install the ingress Gateway:

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ git clone https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages
kubectl apply -f anthos-service-mesh-packages/samples/gateways/istio-ingressgateway
Cloning into 'anthos-service-mesh-packages'...
remote: Enumerating objects: 11963, done.
remote: Counting objects: 100% (1368/1368), done.
remote: Compressing objects: 100% (489/489), done.
remote: Total 11963 (delta 1023), reused 1203 (delta 877), pack-reused 10595
Receiving objects: 100% (11963/11963), 2.95 MiB | 5.24 MiB/s, done.
Resolving deltas: 100% (8205/8205), done.
horizontalpodautoscaler.autoscaling/istio-ingressgateway created
deployment.apps/istio-ingressgateway created
poddisruptionbudget.policy/istio-ingressgateway created
role.rbac.authorization.k8s.io/istio-ingressgateway created
rolebinding.rbac.authorization.k8s.io/istio-ingressgateway created
service/istio-ingressgateway created
serviceaccount/istio-ingressgateway created
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

Install the required custom resource definitions

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl apply -k "github.com/kubernetes-sigs/gateway-api/config/crd/experimental?ref=v0.6.0"
kubectl kustomize "https://github.com/GoogleCloudPlatform/gke-networking-recipes.git/gateway-api/config/mesh/crd" | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/tcproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/tlsroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/udproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gcpsessionaffinityfilters.networking.gke.io created
customresourcedefinition.apiextensions.k8s.io/gcpsessionaffinitypolicies.networking.gke.io created
customresourcedefinition.apiextensions.k8s.io/tdgrpcroutes.net.gke.io created
customresourcedefinition.apiextensions.k8s.io/tdmeshes.net.gke.io created
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

Configure the Gateway:

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/istio-manifests.yaml
gateway.gateway.networking.k8s.io/istio-gateway created
httproute.gateway.networking.k8s.io/frontend-route created
serviceentry.networking.istio.io/allow-egress-googleapis created
serviceentry.networking.istio.io/allow-egress-google-metadata created
virtualservice.networking.istio.io/frontend created
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```



```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl get deployments
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
adservice               1/1     1            1           3m5s
cartservice             1/1     1            1           3m9s
checkoutservice         1/1     1            1           3m15s
currencyservice         1/1     1            1           3m8s
emailservice            1/1     1            1           3m16s
frontend                1/1     1            1           3m13s
istio-gateway-istio     1/1     1            1           47s
istio-ingressgateway    3/3     3            3           2m4s
loadgenerator           1/1     1            1           3m8s
paymentservice          1/1     1            1           3m12s
productcatalogservice   1/1     1            1           3m11s
recommendationservice   1/1     1            1           3m14s
redis-cart              1/1     1            1           3m6s
shippingservice         1/1     1            1           3m7s
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl get services 
NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                                      AGE
adservice               ClusterIP      10.51.180.46    <none>           9555/TCP                                     3m26s
cartservice             ClusterIP      10.51.180.138   <none>           7070/TCP                                     3m30s
checkoutservice         ClusterIP      10.51.185.155   <none>           5050/TCP                                     3m36s
currencyservice         ClusterIP      10.51.177.18    <none>           7000/TCP                                     3m28s
emailservice            ClusterIP      10.51.189.245   <none>           5000/TCP                                     3m37s
frontend                ClusterIP      10.51.181.89    <none>           80/TCP                                       3m34s
frontend-external       LoadBalancer   10.51.182.48    34.122.48.45     80:31243/TCP                                 3m33s
istio-gateway-istio     LoadBalancer   10.51.178.158   35.226.224.37    15021:31930/TCP,80:32658/TCP                 68s
istio-ingressgateway    LoadBalancer   10.51.185.167   35.232.198.132   15021:31647/TCP,80:32260/TCP,443:32044/TCP   2m23s
kubernetes              ClusterIP      10.51.176.1     <none>           443/TCP                                      169m
paymentservice          ClusterIP      10.51.191.184   <none>           50051/TCP                                    3m32s
productcatalogservice   ClusterIP      10.51.186.255   <none>           3550/TCP                                     3m31s
recommendationservice   ClusterIP      10.51.178.185   <none>           8080/TCP                                     3m35s
redis-cart              ClusterIP      10.51.180.100   <none>           6379/TCP                                     3m27s
shippingservice         ClusterIP      10.51.186.105   <none>           50051/TCP                                    3m28s
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```



When you install a GKE or an Anthos cluster, you can enable cluster logs and metrics to be collected and forwarded to Cloud Logging and Cloud Monitoring. That gives you visibility about the cluster, the nodes, the pods and even the containers in that cluster. However, GKE and Anthos don't monitor the communication between microservices.

With Anthos Service Mesh, because every request goes through an Envoy proxy, microservice telemetry information can be collected and inspected. Envoy proxy extensions then send that telemetry to Google Cloud, where you can inspect it. Use Cloud Trace dashboards to investigate requests and their latencies and obtain a breakdown from all services involved in a request.



## Deploy a canary release that has high latency

In this task, you deploy a new version of a service that has an issue which causes high latency. In subsequent tasks, you use the observability tools to diagnose and resolve.

1. In Cloud Shell, clone the repository that has the configuration files you need for this part of the lab:

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ git clone https://github.com/GoogleCloudPlatform/istio-samples.git \
  ~/istio-samples
Cloning into '/home/student_04_887db0b7be5f/istio-samples'...
remote: Enumerating objects: 1767, done.
remote: Counting objects: 100% (240/240), done.
remote: Compressing objects: 100% (164/164), done.
remote: Total 1767 (delta 119), reused 169 (delta 75), pack-reused 1527
Receiving objects: 100% (1767/1767), 22.08 MiB | 9.37 MiB/s, done.
Resolving deltas: 100% (877/877), done.
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl apply -f ~/istio-samples/istio-canary-gke/canary/destinationrule.yaml
destinationrule.networking.istio.io/productcatalogservice created
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

```sh
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl apply -f ~/istio-samples/istio-canary-gke/canary/productcatalog-v2.yaml
deployment.apps/productcatalogservice-v2 created
student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
```

**Note:** You are creating:



- ​      A DestinationRule to set routing of requests between the service versions    
- ​      A new deployment of the product catalog service that has high latency    
- ​      A VirtualService to split product catalog traffic 75% to v1 and 25% to v2    



  You can open each of the configuration files in the Cloud Shell editor to  better understand the definition of each new resource.

## Define your service level objective



When you are not using Anthos Service Mesh, you can define SLOs with the [Service Monitoring](https://cloud.google.com/monitoring/service-monitoring) API. When you are using Anthos Service Mesh, as with the **gke** cluster, you can define and monitor SLOs via the Anthos Service Mesh dashboard.

1. In the Google Cloud  Console, on the **Navigation menu**, click **Anthos** to open the Anthos Dashboard.

   Only the **gke** cluster, however, is running Anthos Service Mesh, so only its services will be shown in the Service mesh section of the dashboard.

2. Click **Service Mesh** to go to the Anthos Service Mesh dashboard.

   A summary of service performance, including SLO information, is displayed. You will define a new SLO for the product catalog service.

3. In the **Services** list, click **productcatalogservice**.

4. In the menu pane, click **Health**.

5. Click **Create SLO**.

6. In the **Set your SLI** slideout, for **metric**, select **Latency**.

7. Select **Request-based** as the method of evaluation.

8. Click **Continue**.

9. Set **Latency Threshold** to **1000**, and click **Continue**.

10. Set **Period type** to **Calendar**.

11. Set **Period length** to **Calendar day**.

    **Note:** Ninety-nine percent availability over a single day is different from 99% availability over a month. The first SLO would not permit more  than 14 minutes of consecutive downtime (24 hrs * 1%), but the second  SLO would allow consecutive downtime up to ~7 hours (30 days * 1%).

12. Set **Performance goal** to **99.5%**.

    The **Preview** graph shows how your goal is reflected against real historical data.

13. Click **Continue**.

14. Review **Display name**: **99.5% - Latency - Calendar day**.

    You can adjust this as needed.

    The autogenerated JSON document is also displayed. You could use the APIs instead to automate the creation of SLOs in the future.

15. To create the SLO, click **Create SLO**.

## Diagnose the problem

### Use service metrics to see where the problem is

1. Click on your SLO entry in the SLO list.

   This displays an expanded view. Your SLO will probably show that you are already out of error budget. If not, wait 3-5 minutes and refresh the page. Eventually, you will exhaust your error budget, because too many of the requests to this service will hit the new backend, which has high latency.

2. In the menu pane, click **Metrics**.

   Scroll down to the **Latency** section of the **Metrics** view and note that the service latency increased a few minutes earlier, around the time you deployed the canary version of the service.

3. From the **Breakdown By** dropdown, select **Source service**.

   Which pods are showing high latency and causing the overall failure to hit your SLO?

4. To return to the Service Mesh page, in the menu pane, click **Anthos Service Mesh**.

   One SLO is flagged as out of error budget, and  a warning indicator is displayed next to the problem service in the **Services** listing.

   **Note:**   You have only defined a single SLO for a single service. In a real  production environment, you would probably have multiple SLOs for  each service.

   

     Also, you have not defined any alerting policy for your SLO.  You would probably have Cloud Monitoring execute an alert if you  are exhausting your error budget faster than expected.

### Use Cloud Trace to better understand where the delay is

1. In the Google Cloud  Console, on the **Navigation menu**, click **Trace > Trace explorer**.

2. Click on a dot that charts at around 3000ms; it should represent one of the requests to the product catalog service.

   Note that all the time seems to be spent within the catalog service itself. Although calls are made to other services, they all appear to return very quickly, and something within the product catalog service is taking a long time.

   **Note:**   Anthos Service Mesh is automatically collecting information about  network calls within the mesh and providing trace data that documents  time spent on these calls. This is useful and required no extra  developer effort.

   

     However, how time is spent within the workload, in this case the  product catalog service pod, isn't instrumented directly by Istio. If  needed, to get this level of detail, the developer would add  instrumentation logic within the service itself.

## Roll back the release and verify an improvement

1. In Cloud Shell, back out the destination rule canary release:

   kubectl delete -f ~/istio-samples/istio-canary-gke/canary/destinationrule.yaml

2. In Cloud Shell, back out the product catalog canary release:

   kubectl delete -f ~/istio-samples/istio-canary-gke/canary/productcatalog-v2.yaml

3. In Cloud Shell, back out the traffic split canary release:

   kubectl delete -f ~/istio-samples/istio-canary-gke/canary/vs-split-traffic.yaml

   ```sh
   student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl delete -f ~/istio-samples/istio-canary-gke/canary/destinationrule.yaml
   destinationrule.networking.istio.io "productcatalogservice" deleted
   student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl delete -f ~/istio-samples/istio-canary-gke/canary/productcatalog-v2.yaml
   deployment.apps "productcatalogservice-v2" deleted
   student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ kubectl delete -f ~/istio-samples/istio-canary-gke/canary/vs-split-traffic.yaml
   virtualservice.networking.istio.io "productcatalogservice" deleted
   student_04_887db0b7be5f@cloudshell:~ (qwiklabs-gcp-00-a894eae6dd20)$ 
   ```

   

4. In the Google Cloud  Console, on the **Navigation menu**, click **Anthos > Service Mesh**.

5. Click on **productcatalogservice**, and then in the menu pane, click **Health**.

   Note the current compliance percentage.

6. Click **Metrics**.

   On the latency chart, all the latency series show a dip that corresponds to when you rolled back the bad version of the workload.

7. Return to the **Health** page.

8. Compare the current compliance metric with the one you saw earlier.  It should be higher now, reflecting the fact that you are no longer  seeing high-latency requests.

## Visualize your mesh with the Anthos Service Mesh dashboard

1. On the **Navigation menu**, click **Anthos > Service Mesh**.

2. Click **Topology**.

   A chart representing your service mesh is displayed.

   

Take a couple of minutes to explore further and better understand the architecture of the application. You can rearrange nodes, drill down into workloads to see constituent deployments and pods, change time spans, etc.

**Congratulations!** You've used Google Cloud's operations suite tooling to evaluate, troubleshoot, and improve service performance on your Anthos managed cluster.

