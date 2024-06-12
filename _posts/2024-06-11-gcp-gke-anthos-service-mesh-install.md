---

layout: single
title:  "Installing Anthos Service Mesh on Google Kubernetes Engine"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/k8s-banner.png
  og_image: /assets/images/k8s-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Installing Anthos Service Mesh on Google Kubernetes Engine

[Istio](http://istio.io) is an open source framework for connecting, securing, and managing microservices. It can  be used with any services, including but not limited to services that  are hosted in a Kubernetes cluster. Istio lets you create a network of  deployed services with load balancing, service-to-service  authentication, monitoring, and more, without requiring any changes in  service code.

As one example - in reliable distributed systems, it's common for a  system to want to retry a request after a failure, possibly with an  exponential backoff delay. There are libraries for Java, Golang and  NodeJS that do this. However, employing them within the app means each  different app will need to solve that problem independently. The Istio  sidecar could do this for the app, automatically.

### Anthos Service Mesh

[Anthos Service Mesh (ASM)](https://cloud.google.com/anthos/service-mesh) is powered by Istio. With Anthos Service Mesh, you get an Anthos  tested, fully supported, distribution of Istio, letting you create and  deploy a service mesh with Anthos GKE, whether your cluster is operating in Google Cloud or on-premises.

You can use included configuration profiles with recommended settings customized for either Google Kubernetes Engine or Anthos GKE on-prem.

Finally, Anthos Service Mesh has a suite of additional features and  tools that help you observe and manage secure, reliable services in a  unified way:

- **Service metrics and logs** for HTTP(S) traffic within your mesh's GKE cluster are automatically ingested to Google Cloud.
- **Preconfigured service dashboards** give you the information you need to understand your services.
- In-depth **telemetry** lets you dig deep into your metrics and logs, filtering and slicing your data on a wide variety of attributes.
- **Service-to-service relationships** at a glance help you understand who connects to which service and the services that each service depends on.
- **Service Level Objectives (SLOs)** provide insights into  the health of your services. You can easily define an SLO and alert on  your own standards of service health.

Anthos Service Mesh is the easiest and richest way to implement an Istio-based service mesh on your Anthos clusters.

- Provision a cluster on Google Kubernetes Engine (GKE)
- Install and configure Anthos Service Mesh
- Deploy Bookinfo, an Istio-enabled multi-service application
- Enable external access using an Istio Ingress Gateway
- Use the Bookinfo application
- Monitor service performance with the Anthos Service Mesh Dashboard

## Set up your project

### Verify the SDK configuration

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-a042913bcd3e.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ gcloud config list
[accessibility]
screen_reader = True
[component_manager]
disable_update_check = True
[compute]
gce_metadata_read_timeout_sec = 30
[core]
account = student-01-ea69a93ec367@qwiklabs.net
disable_usage_reporting = True
project = qwiklabs-gcp-01-a042913bcd3e
[metrics]
environment = devshell

Your active configuration is: [cloudshell-23408]
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ gcloud config set project qwiklabs-gcp-01-a042913bcd3e
Updated property [core/project].
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} \
    --format="value(projectNumber)")
export CLUSTER_NAME=central
export CLUSTER_ZONE=us-east1-b
export WORKLOAD_POOL=${PROJECT_ID}.svc.id.goog
export MESH_ID="proj-${PROJECT_NUMBER}"
Your active configuration is: [cloudshell-23408]
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ gcloud projects get-iam-policy $PROJECT_ID \
    --flatten="bindings[].members" \
    --filter="bindings.members:user:$(gcloud config get-value core/account 2>/dev/null)"
---
bindings:
  members: user:student-01-ea69a93ec367@qwiklabs.net
  role: roles/owner
etag: BwYapyLQXjA=
version: 1
---
bindings:
  members: user:student-01-ea69a93ec367@qwiklabs.net
  role: roles/viewer
etag: BwYapyLQXjA=
version: 1
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$
```

- **WORKLOAD_POOL** will be used to enable Workload Identity, which        is the recommended way to safely access Google Cloud services from GKE        applications.
- **MESH_ID** will be used to set the        **mesh_id** label on the cluster, which is required for        metrics to get displayed on the Anthos Service Mesh Dashboard in the        Cloud Console.

## Set up your GKE cluster

### Create the cluster

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ gcloud config set compute/zone ${CLUSTER_ZONE}
gcloud container clusters create ${CLUSTER_NAME} \
    --machine-type=e2-standard-4 \
    --num-nodes=4 \
    --subnetwork=default \
    --release-channel=regular \
    --labels mesh_id=${MESH_ID} \
    --workload-pool=${WORKLOAD_POOL} \
    --logging=SYSTEM,WORKLOAD
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster central in us-east1-b... Cluster is being health-checked (master is healthy)...done.                                                                              
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-a042913bcd3e/zones/us-east1-b/clusters/central].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east1-b/central?project=qwiklabs-gcp-01-a042913bcd3e
kubeconfig entry generated for central.
NAME: central
LOCATION: us-east1-b
MASTER_VERSION: 1.29.4-gke.1043002
MASTER_IP: 35.243.196.76
MACHINE_TYPE: e2-standard-4
NODE_VERSION: 1.29.4-gke.1043002
NUM_NODES: 4
STATUS: RUNNING
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```



```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ kubectl create clusterrolebinding cluster-admin-binding   --clusterrole=cluster-admin   --user=$(whoami)@qwiklabs.net
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-binding created
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ gcloud container clusters get-credentials ${CLUSTER_NAME} \
     --zone $CLUSTER_ZONE \
     --project $PROJECT_ID
Fetching cluster endpoint and auth data.
kubeconfig entry generated for central.
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

## Prepare to install Anthos Service Mesh

Google provides a tool, `asmcli`, which allows you to install or upgrade Anthos Service Mesh. If you let it, `asmcli` will configure your project and cluster as follows:

- Grant you the required Identity and Access Management (IAM) permissions on your Google Cloud project.
- Enable the required Google APIs on your Cloud project.
- Set a label on the cluster that identifies the mesh.
- Create a service account that lets data plane components, such as  the sidecar proxy, securely access your project's data and resources.
- Register the cluster to the fleet if it isn't already registered.

You will use `asmcli` to install Anthos Service Mesh on your cluster.

Download the version that installs Anthos Service Mesh 1.20.3 to the current working directory:

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.20 > asmcli
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  185k  100  185k    0     0   223k      0 --:--:-- --:--:-- --:--:--  223k
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ chmod +x asmcli
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ gcloud services enable mesh.googleapis.com
Operation "operations/acat.p2-68212809128-e844c359-6cf9-4015-aa49-460df25d2f57" finished successfully.
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

## Validate Anthos Service Mesh

You can run `asmcli validate` to make sure that your project and cluster are set up as required to install Anthos Service Mesh. With this option, `asmcli` doesn't make any changes to your project or cluster, and it doesn't install Anthos Service Mesh.

asmcli validates that:

- Your environment has the required tools.
- The cluster meets the minimum requirements.
- You have the required permissions on the specified project.
- The project has all the required Google APIs enabled.

Run the following command to validate your configuration and download the installation file and `asm` package to the **OUTPUT_DIR** directory

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ ./asmcli validate \
  --project_id $PROJECT_ID \
  --cluster_name $CLUSTER_NAME \
  --cluster_location $CLUSTER_ZONE \
  --fleet_id $PROJECT_ID \
  --output_dir ./asm_output
asmcli: Using PROJECT_ID = qwiklabs-gcp-01-a042913bcd3e from environment variable.
asmcli: Using CLUSTER_NAME = central from environment variable.
asmcli: Use `unset $VAR` if configuring using environment is unexpected.
asmcli: Setting up necessary files...
asmcli: Using /home/student_01_ea69a93ec367/asm_output/asm_kubeconfig as the kubeconfig...
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
asmcli: Checking required APIs...
asmcli: Checking for project qwiklabs-gcp-01-a042913bcd3e...
asmcli: Reading labels for us-east1-b/central...
asmcli: [ERROR]: Current user must have the cluster-admin role on central.
Please add the cluster role binding and retry, or run the script with the
'--enable_cluster_roles' flag to allow the script to enable it on your behalf.
Alternatively, use --enable_all|-e to allow this tool to handle all dependencies.
asmcli: Checking for istio-system namespace...
asmcli: [ERROR]: The istio-system namespace doesn't exist.
Please create the "istio-system" and retry, or run the script with the
'--enable_namespace_creation' flag to allow the script to enable it on your behalf.
Alternatively, use --enable_all|-e to allow this tool to handle all dependencies.
asmcli: Confirming node pool requirements for qwiklabs-gcp-01-a042913bcd3e/us-east1-b/central...
asmcli: Checking Istio installations...
asmcli: [WARNING]: There is no way to validate that the meshconfig API has been initialized.
asmcli: [WARNING]: This needs to happen once per GCP project. If the API has not been initialized
asmcli: [WARNING]: for qwiklabs-gcp-01-a042913bcd3e, please re-run this tool with the --enable_gcp_components
asmcli: [WARNING]: flag. Otherwise, installation will succeed but Anthos Service Mesh
asmcli: [WARNING]: will not function correctly.
asmcli: [WARNING]: Please see the errors above.
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

## Install Anthos Service Mesh

The following command will install Anthos Service Mesh. The  --enable_all flag allows the script to enable the required Google APIs,  set Identity and Access Management permissions, and make the required  updates to your cluster, which includes enabling GKE Workload Identity.

Run the following command to install Anthos Service Mesh:

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ ./asmcli install \
  --project_id $PROJECT_ID \
  --cluster_name $CLUSTER_NAME \
  --cluster_location $CLUSTER_ZONE \
  --fleet_id $PROJECT_ID \
  --output_dir ./asm_output \
  --enable_all \
  --option legacy-default-ingressgateway \
  --ca mesh_ca \
  --enable_gcp_components
asmcli: Using PROJECT_ID = qwiklabs-gcp-01-a042913bcd3e from environment variable.
asmcli: Using CLUSTER_NAME = central from environment variable.
asmcli: Use `unset $VAR` if configuring using environment is unexpected.
asmcli: Setting up necessary files...
asmcli: Using /home/student_01_ea69a93ec367/asm_output/asm_kubeconfig as the kubeconfig...
asmcli: Checking installation tool dependencies...
asmcli: Fetching/writing GCP credentials to kubeconfig file...
asmcli: [WARNING]: nc not found, skipping k8s connection verification
asmcli: [WARNING]: (Installation will continue normally.)
asmcli: Getting account information...
asmcli: Verifying cluster registration.
asmcli: Enabling required APIs...
asmcli: Verifying cluster registration.
asmcli: Binding user:student-01-ea69a93ec367@qwiklabs.net to required IAM roles...
asmcli: Registering the cluster as central...
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-01-a042913bcd3e
asmcli: Checking for project qwiklabs-gcp-01-a042913bcd3e...
asmcli: Reading labels for us-east1-b/central...
asmcli: Querying for core/account...
asmcli: Binding student-01-ea69a93ec367@qwiklabs.net to cluster admin role...
clusterrolebinding.rbac.authorization.k8s.io/student-01-ea69a93ec367-cluster-admin-binding created
asmcli: Creating istio-system namespace...
namespace/istio-system created
asmcli: Confirming node pool requirements for qwiklabs-gcp-01-a042913bcd3e/us-east1-b/central...
asmcli: Checking Istio installations...
asmcli: Initializing meshconfig API...
asmcli: Cluster has Membership ID central in the Hub of project qwiklabs-gcp-01-a042913bcd3e
asmcli: Binding user:student-01-ea69a93ec367@qwiklabs.net to required IAM roles...
asmcli: Configuring kpt package...
asm/
set 16 field(s) of setter "gcloud.core.project" to value "qwiklabs-gcp-01-a042913bcd3e"
asm/
set 2 field(s) of setter "gcloud.project.projectNumber" to value "68212809128"
asm/
set 15 field(s) of setter "gcloud.container.cluster" to value "central"
asm/
set 15 field(s) of setter "gcloud.compute.location" to value "us-east1-b"
asm/
set 1 field(s) of setter "gcloud.compute.network" to value "qwiklabs-gcp-01-a042913bcd3e-default"
asm/
set 2 field(s) of setter "gcloud.project.environProjectNumber" to value "68212809128"
asm/
set 2 field(s) of setter "anthos.servicemesh.rev" to value "asm-1207-2"
asm/
set 2 field(s) of setter "anthos.servicemesh.tag" to value "1.20.7-asm.2"
asm/
set 3 field(s) of setter "anthos.servicemesh.trustDomain" to value "qwiklabs-gcp-01-a042913bcd3e.svc.id.goog"
asm/
set 1 field(s) of setter "anthos.servicemesh.tokenAudiences" to value "istio-ca,qwiklabs-gcp-01-a042913bcd3e.svc.id.goog"
asm/
set 3 field(s) of setter "anthos.servicemesh.created-by" to value "asmcli-1.20.7-asm.2.config1"
asm/
set 2 field(s) of setter "anthos.servicemesh.idp-url" to value "https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-a042913bcd3e/locations/us-east1-b/clusters/central"
asm/
set 2 field(s) of setter "anthos.servicemesh.trustDomainAliases" to value "qwiklabs-gcp-01-a042913bcd3e.svc.id.goog"
namespace/istio-system labeled
asmcli: Installing ASM control plane...
asmcli: ...done!
asmcli: Installing ASM CanonicalService controller in asm-system namespace...
namespace/asm-system created
customresourcedefinition.apiextensions.k8s.io/canonicalservices.anthos.cloud.google.com created
role.rbac.authorization.k8s.io/canonical-service-leader-election-role created
clusterrole.rbac.authorization.k8s.io/canonical-service-manager-role created
clusterrole.rbac.authorization.k8s.io/canonical-service-metrics-reader created
serviceaccount/canonical-service-account created
rolebinding.rbac.authorization.k8s.io/canonical-service-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/canonical-service-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/canonical-service-proxy-rolebinding created
service/canonical-service-controller-manager-metrics-service created
deployment.apps/canonical-service-controller-manager created
asmcli: Waiting for deployment...
deployment.apps/canonical-service-controller-manager condition met
asmcli: ...done!
asmcli: 
asmcli: *****************************
client version: 1.20.7-asm.2
control plane version: 1.20.7-asm.2
data plane version: 1.20.7-asm.2 (2 proxies)
asmcli: *****************************
asmcli: The ASM control plane installation is now complete.
asmcli: To enable automatic sidecar injection on a namespace, you can use the following command:
asmcli: kubectl label namespace <NAMESPACE> istio-injection- istio.io/rev=asm-1207-2 --overwrite
asmcli: If you use 'istioctl install' afterwards to modify this installation, you will need
asmcli: to specify the option '--set revision=asm-1207-2' to target this control plane
asmcli: instead of installing a new one.
asmcli: To finish the installation, enable Istio sidecar injection and restart your workloads.
asmcli: For more information, see:
asmcli: https://cloud.google.com/service-mesh/docs/proxy-injection
asmcli: The ASM package used for installation can be found at:
asmcli: /home/student_01_ea69a93ec367/asm_output/asm
asmcli: The version of istioctl that matches the installation can be found at:
asmcli: /home/student_01_ea69a93ec367/asm_output/istio-1.20.7-asm.2/bin/istioctl
asmcli: A symlink to the istioctl binary can be found at:
asmcli: /home/student_01_ea69a93ec367/asm_output/istioctl
asmcli: The combined configuration generated for installation can be found at:
asmcli: /home/student_01_ea69a93ec367/asm_output/asm-1207-2-manifest-raw.yaml
asmcli: The full, expanded set of kubernetes resources can be found at:
asmcli: /home/student_01_ea69a93ec367/asm_output/asm-1207-2-manifest-expanded.yaml
asmcli: *****************************
asmcli: Successfully installed ASM.
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

### Install an ingress gateway

Anthos Service Mesh gives you the option to deploy and manage  gateways as part of your service mesh. A gateway describes a load  balancer operating at the edge of the mesh receiving incoming or  outgoing HTTP/TCP connections. Gateways are Envoy proxies that provide  you with fine-grained control over traffic entering and leaving the  mesh.

1. Create a namespace for the ingress gateway if you don't already have one. Gateways are user workloads, and as a best practice, they  shouldn't be deployed in the control plane namespace.

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ GATEWAY_NS=istio-gateway
kubectl create namespace $GATEWAY_NS
namespace/istio-gateway created
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

2. Enable auto-injection on the gateway by applying a revision label on  the gateway namespace. The revision label is used by the sidecar  injector webhook to associate injected proxies with a particular control plane revision.

- Use the following command to locate the revision label on `istiod`:

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ kubectl get deploy -n istio-system -l app=istiod -o \
jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}'
asm-1207-2
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ REVISION=$(kubectl get deploy -n istio-system -l app=istiod -o \
jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}')
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ kubectl label namespace $GATEWAY_NS \
istio.io/rev=$REVISION --overwrite
namespace/istio-gateway labeled
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ kubectl label namespace default istio-injection=enabled
kubectl label namespace $GATEWAY_NS  istio-injection=enabled
namespace/default labeled
namespace/istio-gateway labeled
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-a042913bcd3e)$ cd ~/asm_output
student_01_ea69a93ec367@cloudshell:~/asm_output (qwiklabs-gcp-01-a042913bcd3e)$ kubectl apply -n $GATEWAY_NS \
  -f samples/gateways/istio-ingressgateway
horizontalpodautoscaler.autoscaling/istio-ingressgateway created
deployment.apps/istio-ingressgateway created
poddisruptionbudget.policy/istio-ingressgateway created
role.rbac.authorization.k8s.io/istio-ingressgateway created
rolebinding.rbac.authorization.k8s.io/istio-ingressgateway created
service/istio-ingressgateway created
serviceaccount/istio-ingressgateway created
student_01_ea69a93ec367@cloudshell:~/asm_output (qwiklabs-gcp-01-a042913bcd3e)$ 
```

### Enable sidecar injection

Anthos Service Mesh uses sidecar proxies to enhance network security, reliability, and observability. With Anthos Service Mesh, these  functions are abstracted away from an application's primary container  and implemented in a common out-of-process proxy delivered as a separate container in the same Pod.

1. Before you deploy workloads, make sure to configure sidecar proxy  injection so that Anthos Service Mesh can monitor and secure traffic.
2. To enable auto-injection, apply the revision label and remove the istio-injection label if it exists.
3. In the following command, you specify the namespace, **default**, where you want to enable auto-injection, and REVISION is the revision label you noted in the previous step:

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output (qwiklabs-gcp-01-a042913bcd3e)$ kubectl label namespace default istio-injection-istio.io/rev=$REVISION --overwrite
namespace/default labeled
student_01_ea69a93ec367@cloudshell:~/asm_output (qwiklabs-gcp-01-a042913bcd3e)$ 
```

If this cluster had already been running workloads, you would need to restart the pods to re-trigger auto injection.

## Deploy Bookinfo, an Istio-enabled multi-service application

In this task, you will set up the **Bookinfo** sample microservices application and explore the app.

### Bookinfo overview

Now that ASM is configured and verified, you can deploy one of the sample applications provided with the installation — [BookInfo](https://istio.io/docs/guides/bookinfo.html). This is a simple mock bookstore application made up of four  microservices - all managed using Istio. Each microservice is written in a different language, to demonstrate how you can use Istio in a  multi-language environment, without any changes to code.

The microservices are:

- **productpage:** calls the details and reviews microservices to populate the page.
- **details**: contains book information.
- **reviews:** contains book reviews. It also calls the ratings microservice.
- **ratings**: contains book ranking information that accompanies a book review.

There are 3 versions of the **reviews** microservice:

- Reviews **v1** doesn't call the ratings service.
- Reviews **v2** calls the ratings service and displays each rating as 1 - 5 black stars.
- Reviews **v3** calls the ratings service and displays each rating as 1 - 5 red stars.

You can find the source code and all the other files used in this example in the Istio [samples/bookinfo](https://github.com/istio/istio/tree/master/samples/bookinfo) directory.

### Deploy Bookinfo

```yaml
student_01_ea69a93ec367@cloudshell:~/asm_output (qwiklabs-gcp-01-a042913bcd3e)$ istio_dir=$(ls -d istio-* | tail -n 1)
cd $istio_dir
cat samples/bookinfo/platform/kube/bookinfo.yaml
# Copyright Istio Authors
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.

##################################################################################################
# This file defines the services, service accounts, and deployments for the Bookinfo sample.
#
# To apply all 4 Bookinfo services, their corresponding service accounts, and deployments:
#
#   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
#
# Alternatively, you can deploy any resource separately:
#
#   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -l service=reviews # reviews Service
#   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -l account=reviews # reviews ServiceAccount
#   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -l app=reviews,version=v3 # reviews-v3 Deployment
##################################################################################################

##################################################################################################
# Details service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: details
  labels:
    app: details
    service: details
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: details
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-details
  labels:
    account: details
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: details-v1
  labels:
    app: details
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: details
      version: v1
  template:
    metadata:
      labels:
        app: details
        version: v1
    spec:
      serviceAccountName: bookinfo-details
      containers:
      - name: details
        image: docker.io/istio/examples-bookinfo-details-v1:1.18.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
---
##################################################################################################
# Ratings service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: ratings
  labels:
    app: ratings
    service: ratings
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: ratings
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-ratings
  labels:
    account: ratings
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ratings-v1
  labels:
    app: ratings
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ratings
      version: v1
  template:
    metadata:
      labels:
        app: ratings
        version: v1
    spec:
      serviceAccountName: bookinfo-ratings
      containers:
      - name: ratings
        image: docker.io/istio/examples-bookinfo-ratings-v1:1.18.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
---
##################################################################################################
# Reviews service
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: reviews
  labels:
    app: reviews
    service: reviews
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: reviews
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-reviews
  labels:
    account: reviews
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v1
  labels:
    app: reviews
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v1
  template:
    metadata:
      labels:
        app: reviews
        version: v1
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v1:1.18.0
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v2
  labels:
    app: reviews
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v2
  template:
    metadata:
      labels:
        app: reviews
        version: v2
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v2:1.18.0
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews-v3
  labels:
    app: reviews
    version: v3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
      version: v3
  template:
    metadata:
      labels:
        app: reviews
        version: v3
    spec:
      serviceAccountName: bookinfo-reviews
      containers:
      - name: reviews
        image: docker.io/istio/examples-bookinfo-reviews-v3:1.18.0
        imagePullPolicy: IfNotPresent
        env:
        - name: LOG_DIR
          value: "/tmp/logs"
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: wlp-output
          mountPath: /opt/ibm/wlp/output
      volumes:
      - name: wlp-output
        emptyDir: {}
      - name: tmp
        emptyDir: {}
---
##################################################################################################
# Productpage services
##################################################################################################
apiVersion: v1
kind: Service
metadata:
  name: productpage
  labels:
    app: productpage
    service: productpage
spec:
  ports:
  - port: 9080
    name: http
  selector:
    app: productpage
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo-productpage
  labels:
    account: productpage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productpage-v1
  labels:
    app: productpage
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: productpage
      version: v1
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9080"
        prometheus.io/path: "/metrics"
      labels:
        app: productpage
        version: v1
    spec:
      serviceAccountName: bookinfo-productpage
      containers:
      - name: productpage
        image: docker.io/istio/examples-bookinfo-productpage-v1:1.18.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 9080
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        emptyDir: {}
---
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

In **Cloud Shell**, use the following command to `inject` the proxy sidecar along with each application Pod that is deployed:

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

Istio uses an extended version of the open-source [Envoy proxy](https://www.envoyproxy.io/), a high-performance proxy developed in C++, to mediate all inbound and outbound traffic for all services in the service mesh.

 Istio leverages Envoy's many built-in features including dynamic  service discovery, load balancing, TLS termination, HTTP/2 & gRPC  proxying, circuit breakers, health checks, staged rollouts with %-based  traffic split, fault injection, and rich metrics. 

### Enable external access using an Istio Ingress Gateway

Now that the Bookinfo services are up and running, you need to make  the application accessible from outside of your Kubernetes cluster, e.g. from a browser. An **Istio Gateway** is used for this purpose.

```yaml
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ cat samples/bookinfo/networking/bookinfo-gateway.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  # The selector matches the ingress gateway pod labels.
  # If you installed Istio using Helm following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 8080
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
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
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

Look for the `Gateway` and `VirtualService` mesh resources which get deployed. The `Gateway` exposes services to users outside the service mesh, and allows Istio  features such as monitoring and route rules to be applied to traffic  entering the cluster.

Configure the **ingress gateway** for the application, which exposes an *external IP* you will use later

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

### Verify the Bookinfo deployments

1. Confirm that the application has been deployed correctly, review services, pods, and the ingress gateway:

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ kubectl get services
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   34.118.227.218   <none>        9080/TCP   3m21s
kubernetes    ClusterIP   34.118.224.1     <none>        443/TCP    33m
productpage   ClusterIP   34.118.228.99    <none>        9080/TCP   3m15s
ratings       ClusterIP   34.118.233.205   <none>        9080/TCP   3m19s
reviews       ClusterIP   34.118.225.54    <none>        9080/TCP   3m18s
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
details-v1-698d88b-7z6zv         2/2     Running   0          3m48s
productpage-v1-675fc69cf-87mnr   2/2     Running   0          3m43s
ratings-v1-6484c4d9bb-xdgqt      2/2     Running   0          3m47s
reviews-v1-5b5d6494f4-twdk6      2/2     Running   0          3m46s
reviews-v2-5b667bcbf8-vq76p      2/2     Running   0          3m45s
reviews-v3-5b9bd44f4-wpzhs       2/2     Running   0          3m45s
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

Confirm that the Bookinfo application is running by sending a `curl` request to it from some Pod, within the cluster, for example from `ratings`:

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ kubectl exec -it $(kubectl get pod -l app=ratings \
    -o jsonpath='{.items[0].metadata.name}') \
    -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
                                   student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```



```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ kubectl get gateway
NAME               AGE
bookinfo-gateway   2m35s
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   34.118.239.120   34.139.41.96   15021:31214/TCP,80:30126/TCP,443:30691/TCP,15012:31358/TCP,15443:31908/TCP   15m
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ export GATEWAY_URL=34.139.41.96
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$
```



Check that the Bookinfo app is running by sending a `curl` request to it from *outside* the cluster:

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ curl -I http://${GATEWAY_URL}/productpage
HTTP/1.1 200 OK
server: istio-envoy
date: Wed, 12 Jun 2024 02:03:47 GMT
content-type: text/html; charset=utf-8
content-length: 5294
x-envoy-upstream-service-time: 57

student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

## Use the Bookinfo application

### Try the application in your Web browser

Point your browser to `http://[$GATEWAY_URL]/productpage` to see the BookInfo web page. Don't forget to replace `[$GATEWAY_URL]` with your working **external IP** address.

1. Refresh the page several times.

   Notice how you see three different versions of reviews, since we have not yet used Istio to control the version routing.

   There are three different book review services being called in a round-robin style:

   - no stars
   - black stars
   - red stars

   Switching among the three is normal Kubernetes routing/balancing behavior.

### Generate a steady background load

Run the siege utility to simulate traffic to Bookinfo.

1. In Cloud Shell, install **siege**:

   Siege is a utility for generating load against Web sites.

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ sudo apt install siege
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  siege
0 upgraded, 1 newly installed, 0 to remove and 9 not upgraded.
Need to get 102 kB of archives.
After this operation, 269 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu jammy/main amd64 siege amd64 4.0.7-1build3 [102 kB]
Fetched 102 kB in 1s (71.4 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package siege.
(Reading database ... 122349 files and directories currently installed.)
Preparing to unpack .../siege_4.0.7-1build3_amd64.deb ...
Unpacking siege (4.0.7-1build3) ...
Setting up siege (4.0.7-1build3) ...
Processing triggers for man-db (2.10.2-1) ...
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 

```

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ siege http://${GATEWAY_URL}/productpage
New configuration template added to /home/student_01_ea69a93ec367/.siege
Run siege -C to view the current settings in that file
^C
{       "transactions":                        16616,
        "availability":                       100.00,
        "elapsed_time":                       521.08,
        "data_transferred":                   873.79,
        "response_time":                        0.78,
        "transaction_rate":                    31.89,
        "throughput":                           1.68,
        "concurrency":                         24.97,
        "successful_transactions":             16616,
        "failed_transactions":                     0,
        "longest_transaction":                  2.34,
        "shortest_transaction":                 0.45
}
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

## Evaluate service performance using the Anthos Service Mesh dashboard

### Gather data from the Services table view.

1. In the Console, go to **Navigation menu** > **Anthos** > **Service Mesh**.

If prompted, click **Enable** to enable the Anthos API.

1. On the bottom half of the window, you will see a **Service** section.

2. Click on the **productpage** service to drill down and see more details.

3. Note the summary at the top detailing current requests/second, error rates, latencies, and resource usage.

   If you don't see **Requests > 0**, try exiting and re-entering the productpage service after a few minutes.

   On the left side of the window, click on the **Metrics** option. Explore the different graphs and their breakdown options.

   - What is the current request rate, and how has it changed over time?
   - What is the current error rate, and how has it changed over time?
   - What latencies do you see charted?
   - What is the median request size?
   - What is median response size?
   - What is the aggregate cpu usage?

   Click on the **Connected Services** option from the left side.

   This lists other services that make inbound requests of the  productpage, and services the productpage makes outbound requests to.

   - What services make calls to the **productpage** service?
   - What services does the **productpage** service call?
   - Is mTLS enforced on inter-service calls?

Return to the Anthos Service Mesh dashboard by clicking on the **Anthos Service Mesh** logo in the upper left corner.

At this time, you can explore or drill down on other services.

### Use the Topology view to better visualize your mesh

1. On the Anthos Service Mesh dashboard, view the topology on the right side of the window.

   Here you may need to wait a few minutes for the topology graph to appear.

2. Rearrange the nodes in the graph until you can easily visualize the relationships between services and workloads.

   Remember, external requests start at productpage. You can scroll back and study the Bookinfo architecture diagram at the **Bookinfo Overview**.

## History

```sh
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ history 
    1  gcloud config list
    2  gcloud config set project qwiklabs-gcp-01-a042913bcd3e
    3  export PROJECT_ID=$(gcloud config get-value project)
    4  export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} \
    --format="value(projectNumber)")
    5  export CLUSTER_NAME=central
    6  export CLUSTER_ZONE=us-east1-b
    7  export WORKLOAD_POOL=${PROJECT_ID}.svc.id.goog
    8  export MESH_ID="proj-${PROJECT_NUMBER}"
    9  gcloud projects get-iam-policy $PROJECT_ID     --flatten="bindings[].members"     --filter="bindings.members:user:$(gcloud config get-value core/account 2>/dev/null)"
   10  gcloud config set compute/zone ${CLUSTER_ZONE}
   11  gcloud container clusters create ${CLUSTER_NAME}     --machine-type=e2-standard-4     --num-nodes=4     --subnetwork=default     --release-channel=regular     --labels mesh_id=${MESH_ID}     --workload-pool=${WORKLOAD_POOL}     --logging=SYSTEM,WORKLOAD
   12  kubectl create clusterrolebinding cluster-admin-binding   --clusterrole=cluster-admin   --user=$(whoami)@qwiklabs.net
   13  gcloud container clusters get-credentials ${CLUSTER_NAME}      --zone $CLUSTER_ZONE      --project $PROJECT_ID
   14  curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.20 > asmcli
   15  chmod +x asmcli
   16  gcloud services enable mesh.googleapis.com
   17  ./asmcli validate   --project_id $PROJECT_ID   --cluster_name $CLUSTER_NAME   --cluster_location $CLUSTER_ZONE   --fleet_id $PROJECT_ID   --output_dir ./asm_output
   18  ./asmcli install   --project_id $PROJECT_ID   --cluster_name $CLUSTER_NAME   --cluster_location $CLUSTER_ZONE   --fleet_id $PROJECT_ID   --output_dir ./asm_output   --enable_all   --option legacy-default-ingressgateway   --ca mesh_ca   --enable_gcp_components
   19  GATEWAY_NS=istio-gateway
   20  kubectl create namespace $GATEWAY_NS
   21  kubectl get deploy -n istio-system -l app=istiod -o jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}'
   22  REVISION=$(kubectl get deploy -n istio-system -l app=istiod -o \
jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}')
   23  kubectl label namespace $GATEWAY_NS istio.io/rev=$REVISION --overwrite
   24  kubectl label namespace default istio-injection=enabled
   25  kubectl label namespace $GATEWAY_NS  istio-injection=enabled
   26  cd ~/asm_output
   27  kubectl apply -n $GATEWAY_NS   -f samples/gateways/istio-ingressgateway
   28  kubectl label namespace default istio-injection-istio.io/rev=$REVISION --overwrite
   29  istio_dir=$(ls -d istio-* | tail -n 1)
   30  cd $istio_dir
   31  cat samples/bookinfo/platform/kube/bookinfo.yaml
   32  kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
   33  cat samples/bookinfo/networking/bookinfo-gateway.yaml
   34  kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
   35  kubectl get services
   36  kubectl get pods
   37  kubectl exec -it $(kubectl get pod -l app=ratings \
    -o jsonpath='{.items[0].metadata.name}')     -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
   38  kubectl get gateway
   39  kubectl get svc istio-ingressgateway -n istio-system
   40  export GATEWAY_URL=34.139.41.96
   41  curl -I http://${GATEWAY_URL}/productpage
   42  sudo apt install siege
   43  siege http://${GATEWAY_URL}/productpage
   44  history 
student_01_ea69a93ec367@cloudshell:~/asm_output/istio-1.20.7-asm.2 (qwiklabs-gcp-01-a042913bcd3e)$ 
```

