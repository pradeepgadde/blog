---

layout: single
title:  "Configuring a multi-cluster mesh with Anthos Service Mesh"
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

# Configuring a multi-cluster mesh with Anthos Service Mesh

Many applications will eventually need to be distributed and have services running across multiple Anthos clusters. Anthos Service Mesh enables this with multi-cluster meshes.

A multi-cluster service mesh is a mesh composed of services running on more than one cluster.

A multi-cluster service mesh has the advantage that all the services look the same to clients, regardless of where the workloads are actually running. It’s transparent to the application whether a service is deployed in a single or multi-cluster service mesh.

In this lab, you build a service mesh encompassing two clusters, **west** and **east**. You deploy an application comprised of services, some running on **west** and some on **east**. You test the application to make sure that services can communicate across clusters without problem.



- Install and configure a multi-cluster mesh
- Deploy and use Online Boutique in multiple configurations
- Review the deployments in each cluster
- Understand service-to-service flow between clusters
- Migrate workloads from one cluster to another

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-f7c1f4b09b4a.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ C1_LOCATION=us-west1-b
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ C2_LOCATION=us-east1-b
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ export PROJECT_ID=${PROJECT_ID:-$(gcloud config get-value project)}
export C1_NAME=west
export C2_NAME=east
export FLEET_PROJECT_ID=${FLEET_PROJECT_ID:-$PROJECT_ID}
export DIR_PATH=${DIR_PATH:-$(pwd)}
export GATEWAY_NAMESPACE=asm-gateways
Your active configuration is: [cloudshell-30440]
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.17 > asmcli
chmod +x asmcli
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  184k  100  184k    0     0   219k      0 --:--:-- --:--:-- --:--:--  219k
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ # configure the west cluster context
gcloud container clusters get-credentials $C1_NAME --zone $C1_LOCATION --project $PROJECT_ID
kubectx $C1_NAME=.

# configure the east cluster context
gcloud container clusters get-credentials $C2_NAME --zone $C2_LOCATION --project $PROJECT_ID
kubectx $C2_NAME=.
Fetching cluster endpoint and auth data.
kubeconfig entry generated for west.
Context "gke_qwiklabs-gcp-03-f7c1f4b09b4a_us-west1-b_west" renamed to "west".
Fetching cluster endpoint and auth data.
kubeconfig entry generated for east.
Context "gke_qwiklabs-gcp-03-f7c1f4b09b4a_us-east1-b_east" renamed to "east".
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ kubectx
east
west
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

## Install Anthos Service Mesh on both clusters

```sh
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ # switch to west context
kubectx west

# install Anthos Service Mesh
./asmcli install \
  --project_id $PROJECT_ID \
  --cluster_name $C1_NAME \
  --cluster_location $C1_LOCATION \
  --fleet_id $FLEET_PROJECT_ID \
  --output_dir $DIR_PATH \
  --enable_all \
  --ca mesh_ca
Switched to context "west".
asmcli: Using PROJECT_ID = qwiklabs-gcp-03-f7c1f4b09b4a from environment variable.
asmcli: Use `unset $VAR` if configuring using environment is unexpected.
asmcli: Setting up necessary files...
asmcli: Using /home/student_03_b88b4332a813/asm_kubeconfig as the kubeconfig...
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
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Enabling required APIs...
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Checking for project qwiklabs-gcp-03-f7c1f4b09b4a...
asmcli: Reading labels for us-west1-b/west...
asmcli: Querying for core/account...
asmcli: Binding student-03-b88b4332a813@qwiklabs.net to cluster admin role...
clusterrolebinding.rbac.authorization.k8s.io/student-03-b88b4332a813-cluster-admin-binding created
asmcli: Creating istio-system namespace...
namespace/istio-system created
asmcli: Confirming node pool requirements for qwiklabs-gcp-03-f7c1f4b09b4a/us-west1-b/west...
asmcli: Checking Istio installations...
asmcli: Initializing meshconfig API...
asmcli: Cluster has Membership ID west-connect in the Hub of project qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Binding user:student-03-b88b4332a813@qwiklabs.net to required IAM roles...
asmcli: Configuring kpt package...
asm/
set 16 field(s) of setter "gcloud.core.project" to value "qwiklabs-gcp-03-f7c1f4b09b4a"
asm/
set 2 field(s) of setter "gcloud.project.projectNumber" to value "186053649688"
asm/
set 15 field(s) of setter "gcloud.container.cluster" to value "west"
asm/
set 15 field(s) of setter "gcloud.compute.location" to value "us-west1-b"
asm/
set 1 field(s) of setter "gcloud.compute.network" to value "qwiklabs-gcp-03-f7c1f4b09b4a-default"
asm/
set 2 field(s) of setter "gcloud.project.environProjectNumber" to value "186053649688"
asm/
set 2 field(s) of setter "anthos.servicemesh.rev" to value "asm-1178-20"
asm/
set 2 field(s) of setter "anthos.servicemesh.tag" to value "1.17.8-asm.20"
asm/
set 3 field(s) of setter "anthos.servicemesh.trustDomain" to value "qwiklabs-gcp-03-f7c1f4b09b4a.svc.id.goog"
asm/
set 1 field(s) of setter "anthos.servicemesh.tokenAudiences" to value "istio-ca,qwiklabs-gcp-03-f7c1f4b09b4a.svc.id.goog"
asm/
set 3 field(s) of setter "anthos.servicemesh.created-by" to value "asmcli-1.17.8-asm.20.config1"
asm/
set 2 field(s) of setter "anthos.servicemesh.idp-url" to value "https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-f7c1f4b09b4a/locations/us-west1-b/clusters/west"
asm/
set 2 field(s) of setter "anthos.servicemesh.trustDomainAliases" to value "qwiklabs-gcp-03-f7c1f4b09b4a.svc.id.goog"
namespace/istio-system labeled
asmcli: Installing ASM control plane...

Thank you for installing Istio 1.17.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/hMHGiwZHPU7UQRWe9
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
client version: 1.17.8-asm.20
control plane version: 1.17.8-asm.20
data plane version: none
asmcli: *****************************
asmcli: The ASM control plane installation is now complete.
asmcli: To enable automatic sidecar injection on a namespace, you can use the following command:
asmcli: kubectl label namespace <NAMESPACE> istio-injection- istio.io/rev=asm-1178-20 --overwrite
asmcli: If you use 'istioctl install' afterwards to modify this installation, you will need
asmcli: to specify the option '--set revision=asm-1178-20' to target this control plane
asmcli: instead of installing a new one.
asmcli: To finish the installation, enable Istio sidecar injection and restart your workloads.
asmcli: For more information, see:
asmcli: https://cloud.google.com/service-mesh/docs/proxy-injection
asmcli: The ASM package used for installation can be found at:
asmcli: /home/student_03_b88b4332a813/asm
asmcli: The version of istioctl that matches the installation can be found at:
asmcli: /home/student_03_b88b4332a813/istio-1.17.8-asm.20/bin/istioctl
asmcli: A symlink to the istioctl binary can be found at:
asmcli: /home/student_03_b88b4332a813/istioctl
asmcli: The combined configuration generated for installation can be found at:
asmcli: /home/student_03_b88b4332a813/asm-1178-20-manifest-raw.yaml
asmcli: The full, expanded set of kubernetes resources can be found at:
asmcli: /home/student_03_b88b4332a813/asm-1178-20-manifest-expanded.yaml
asmcli: *****************************
asmcli: Successfully installed ASM.
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

```sh
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ # create the gateway namespace
kubectl create namespace $GATEWAY_NAMESPACE

# enable sidecar injection on the gateway namespace
kubectl label ns $GATEWAY_NAMESPACE \
  istio.io/rev=$(kubectl -n istio-system get pods -l app=istiod -o json | jq -r '.items[0].metadata.labels["istio.io/rev"]') \
  --overwrite

# Apply the configurations
kubectl apply -n $GATEWAY_NAMESPACE \
  -f $DIR_PATH/samples/gateways/istio-ingressgateway
namespace/asm-gateways created
namespace/asm-gateways labeled
horizontalpodautoscaler.autoscaling/istio-ingressgateway created
deployment.apps/istio-ingressgateway created
poddisruptionbudget.policy/istio-ingressgateway created
role.rbac.authorization.k8s.io/istio-ingressgateway created
rolebinding.rbac.authorization.k8s.io/istio-ingressgateway created
service/istio-ingressgateway created
serviceaccount/istio-ingressgateway created
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

```sh
tudent_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ # switch to the east context
kubectx east

# install Anthos Service Mesh
./asmcli install \
  --project_id $PROJECT_ID \
  --cluster_name $C2_NAME \
  --cluster_location $C2_LOCATION \
  --fleet_id $FLEET_PROJECT_ID \
  --output_dir $DIR_PATH \
  --enable_all \
  --ca mesh_ca

# create the gateway namespace
  -f $DIR_PATH/samples/gateways/istio-ingressgateway app=istiod -o json | jq -r '.items[0].metadata.labels["istio.io/rev"]') \
Switched to context "east".
asmcli: Using PROJECT_ID = qwiklabs-gcp-03-f7c1f4b09b4a from environment variable.
asmcli: Use `unset $VAR` if configuring using environment is unexpected.
asmcli: Setting up necessary files...
asmcli: Using /home/student_03_b88b4332a813/asm_kubeconfig as the kubeconfig...
asmcli: Checking installation tool dependencies...
asmcli: Fetching/writing GCP credentials to kubeconfig file...
asmcli: [WARNING]: nc not found, skipping k8s connection verification
asmcli: [WARNING]: (Installation will continue normally.)
asmcli: Getting account information...
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Enabling required APIs...
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Checking for project qwiklabs-gcp-03-f7c1f4b09b4a...
asmcli: Reading labels for us-east1-b/east...
asmcli: Querying for core/account...
asmcli: Binding student-03-b88b4332a813@qwiklabs.net to cluster admin role...
clusterrolebinding.rbac.authorization.k8s.io/student-03-b88b4332a813-cluster-admin-binding created
asmcli: Creating istio-system namespace...
namespace/istio-system created
asmcli: Confirming node pool requirements for qwiklabs-gcp-03-f7c1f4b09b4a/us-east1-b/east...
asmcli: Checking Istio installations...
asmcli: Initializing meshconfig API...
asmcli: Cluster has Membership ID east-connect in the Hub of project qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Binding user:student-03-b88b4332a813@qwiklabs.net to required IAM roles...
asmcli: Configuring kpt package...
asm/
set 15 field(s) of setter "gcloud.core.project" to value "qwiklabs-gcp-03-f7c1f4b09b4a"
asm/
set 2 field(s) of setter "gcloud.project.projectNumber" to value "186053649688"
asm/
set 15 field(s) of setter "gcloud.container.cluster" to value "east"
asm/
set 15 field(s) of setter "gcloud.compute.location" to value "us-east1-b"
asm/
set 1 field(s) of setter "gcloud.compute.network" to value "qwiklabs-gcp-03-f7c1f4b09b4a-default"
asm/
set 2 field(s) of setter "gcloud.project.environProjectNumber" to value "186053649688"
asm/
set 2 field(s) of setter "anthos.servicemesh.rev" to value "asm-1178-20"
asm/
set 2 field(s) of setter "anthos.servicemesh.tag" to value "1.17.8-asm.20"
asm/
set 2 field(s) of setter "anthos.servicemesh.trustDomain" to value "qwiklabs-gcp-03-f7c1f4b09b4a.svc.id.goog"
asm/
set 1 field(s) of setter "anthos.servicemesh.tokenAudiences" to value "istio-ca,qwiklabs-gcp-03-f7c1f4b09b4a.svc.id.goog"
asm/
set 3 field(s) of setter "anthos.servicemesh.created-by" to value "asmcli-1.17.8-asm.20.config1"
asm/
set 2 field(s) of setter "anthos.servicemesh.idp-url" to value "https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-f7c1f4b09b4a/locations/us-east1-b/clusters/east"
asm/
set 2 field(s) of setter "anthos.servicemesh.trustDomainAliases" to value "qwiklabs-gcp-03-f7c1f4b09b4a.svc.id.goog"
namespace/istio-system labeled
asmcli: Installing ASM control plane...

Thank you for installing Istio 1.17.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/hMHGiwZHPU7UQRWe9
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
client version: 1.17.8-asm.20
control plane version: 1.17.8-asm.20
data plane version: none
asmcli: *****************************
asmcli: The ASM control plane installation is now complete.
asmcli: To enable automatic sidecar injection on a namespace, you can use the following command:
asmcli: kubectl label namespace <NAMESPACE> istio-injection- istio.io/rev=asm-1178-20 --overwrite
asmcli: If you use 'istioctl install' afterwards to modify this installation, you will need
asmcli: to specify the option '--set revision=asm-1178-20' to target this control plane
asmcli: instead of installing a new one.
asmcli: To finish the installation, enable Istio sidecar injection and restart your workloads.
asmcli: For more information, see:
asmcli: https://cloud.google.com/service-mesh/docs/proxy-injection
asmcli: The ASM package used for installation can be found at:
asmcli: /home/student_03_b88b4332a813/asm
asmcli: The version of istioctl that matches the installation can be found at:
asmcli: /home/student_03_b88b4332a813/istio-1.17.8-asm.20/bin/istioctl
asmcli: A symlink to the istioctl binary can be found at:
asmcli: /home/student_03_b88b4332a813/istioctl
asmcli: The combined configuration generated for installation can be found at:
asmcli: /home/student_03_b88b4332a813/asm-1178-20-manifest-raw.yaml
asmcli: The full, expanded set of kubernetes resources can be found at:
asmcli: /home/student_03_b88b4332a813/asm-1178-20-manifest-expanded.yaml
asmcli: *****************************
asmcli: Successfully installed ASM.
namespace/asm-gateways created
namespace/asm-gateways labeled
horizontalpodautoscaler.autoscaling/istio-ingressgateway created
deployment.apps/istio-ingressgateway created
poddisruptionbudget.policy/istio-ingressgateway created
role.rbac.authorization.k8s.io/istio-ingressgateway created
rolebinding.rbac.authorization.k8s.io/istio-ingressgateway created
service/istio-ingressgateway created
serviceaccount/istio-ingressgateway created
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

## Configure the mesh to span both clusters

GKE automatically adds firewall rules to each node to allow traffic within the same subnet. If your mesh contains multiple subnets, you must explicitly set up the firewall rules to allow cross-subnet traffic. You must add a new firewall rule for each subnet to allow the source IP CIDR blocks and targets ports of all the incoming traffic.

1. Create a firewall rule that allows all traffic between the two clusters in your mesh:

```sh
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$  function join_by { local IFS="$1"; shift; echo "$*"; }

 # get the service IP CIDRs for each cluster
 ALL_CLUSTER_CIDRS=$(gcloud container clusters list \
   --filter="name:($C1_NAME,$C2_NAME)" \
   --format='value(clusterIpv4Cidr)' | sort | uniq)
 ALL_CLUSTER_CIDRS=$(join_by , $(echo "${ALL_CLUSTER_CIDRS}"))

 # get the network tags for each cluster
 ALL_CLUSTER_NETTAGS=$(gcloud compute instances list  \
   --filter="name:($C1_NAME,$C2_NAME)" \
   --format='value(tags.items.[0])' | sort | uniq)
 ALL_CLUSTER_NETTAGS=$(join_by , $(echo "${ALL_CLUSTER_NETTAGS}"))

     --target-tags="${ALL_CLUSTER_NETTAGS}" --quietluster-pods \s
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-f7c1f4b09b4a/global/firewalls/istio-multi-cluster-pods].
Creating firewall...done.                                                                                                                   
NAME: istio-multi-cluster-pods
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 900
ALLOW: tcp,udp,icmp,esp,ah,sctp
DENY: 
DISABLED: False
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

In order for each cluster to discover endpoints in the other cluster, the clusters must all be registered in the same fleet, and each cluster must be configured with a secret that can be used to gain access to the other cluster's API server for endpoint enumeration. The `asmcli` utility will set this up for you.



```sh
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ ./asmcli create-mesh \
    $FLEET_PROJECT_ID \
    ${PROJECT_ID}/${C1_LOCATION}/${C1_NAME} \
    ${PROJECT_ID}/${C2_LOCATION}/${C2_NAME}
asmcli: Using PROJECT_ID = qwiklabs-gcp-03-f7c1f4b09b4a from environment variable.
asmcli: Use `unset $VAR` if configuring using environment is unexpected.
asmcli: Setting up necessary files...
asmcli: Creating temp directory...
asmcli: 
asmcli: *****************************
asmcli: No output folder was specified with --output_dir|-D, so configuration and
asmcli: binaries will be stored in the following directory.
asmcli: /tmp/tmp.ape2PCfqGP
asmcli: *****************************
asmcli: 
asmcli: Using  as the kubeconfig...
asmcli: Checking installation tool dependencies...
asmcli: Downloading kpt..
asmcli: Downloading ASM..
asmcli: Downloading ASM kpt package...
fetching package "/asm" from "https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages" to "asm"
fetching package "/samples" from "https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages" to "samples"
asmcli: Checking for project qwiklabs-gcp-03-f7c1f4b09b4a...
asmcli: Fetching/writing GCP credentials to /tmp/tmp.duvK5ecHop...
asmcli: Fetching/writing GCP credentials to /tmp/tmp.7lg8f1P2Nx...
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-03-f7c1f4b09b4a
asmcli: [WARNING]: nc not found, skipping k8s connection verification
asmcli: [WARNING]: (Installation will continue normally.)
asmcli: Check trust domain aliases of cluster gke_qwiklabs-gcp-03-f7c1f4b09b4a_us-west1-b_west
asmcli: [WARNING]: nc not found, skipping k8s connection verification
asmcli: [WARNING]: (Installation will continue normally.)
asmcli: Check trust domain aliases of cluster gke_qwiklabs-gcp-03-f7c1f4b09b4a_us-east1-b_east
asmcli: Installing remote secret gke-qwiklabs-gcp-03-f7c1f4b09b4a-us-west1-b-west on /tmp/tmp.7lg8f1P2Nx...
secret/istio-remote-secret-gke-qwiklabs-gcp-03-f7c1f4b09b4a-us-west1-b-west created
asmcli: Installing remote secret gke-qwiklabs-gcp-03-f7c1f4b09b4a-us-east1-b-east on /tmp/tmp.duvK5ecHop...
secret/istio-remote-secret-gke-qwiklabs-gcp-03-f7c1f4b09b4a-us-east1-b-east created
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

**Note:** Because your two Anthos GKE clusters are VPC-native clusters and they are on the same VPC network, traffic can flow from one to the other without any special configuration. 



 However, if you were creating a multi-cluster mesh using clusters on different networks, as would be the case if you had one GKE cluster and one Anthos on bare metal cluster, there are a couple of additional steps required to enable mesh functionality 



 The first step would be to install east-west gateways on each cluster. Traffic flowing from the **west** cluster to the **east** cluster would exit through a gateway on the **west** cluster, and vice versa. 



 Once the gateways are in place, you need to expose all services on the gateway to both clusters. That is done by applying the expose-services.yaml configuration provided as part of the Anthos Service Mesh distribution



## Review Service Mesh control planes

Let's verify that all Anthos Service Mesh resources have been created correctly on both clusters.

```sh
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ kubectx west
kubectl get namespaces
Switched to context "west".
NAME                 STATUS   AGE
asm-gateways         Active   10m
asm-system           Active   11m
default              Active   12h
gke-managed-system   Active   12h
gmp-public           Active   12h
gmp-system           Active   12h
istio-system         Active   13m
kube-node-lease      Active   12h
kube-public          Active   12h
kube-system          Active   12h
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

```sh
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ kubectl get deploy -A
NAMESPACE      NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
asm-gateways   istio-ingressgateway                   3/3     3            3           12m
asm-system     canonical-service-controller-manager   1/1     1            1           13m
gmp-system     gmp-operator                           1/1     1            1           12h
gmp-system     rule-evaluator                         1/1     1            1           12h
istio-system   istiod-asm-1178-20                     2/2     2            2           13m
kube-system    event-exporter-gke                     1/1     1            1           12h
kube-system    konnectivity-agent                     3/3     3            3           12h
kube-system    konnectivity-agent-autoscaler          1/1     1            1           12h
kube-system    kube-dns                               2/2     2            2           12h
kube-system    kube-dns-autoscaler                    1/1     1            1           12h
kube-system    l7-default-backend                     1/1     1            1           12h
kube-system    metrics-server-v0.6.3                  1/1     1            1           12h
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```



**Note:** The **istio-system** namespace is where Istiod will be running. The **asm-gateways** namespace is where your gateway pods will be running. And the **asm-system** namespace will be used by the Canonical Service controller 

```sh
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ kubectx east
Switched to context "east".
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ kubectl get ns
NAME                 STATUS   AGE
asm-gateways         Active   7m14s
asm-system           Active   7m32s
default              Active   12h
gke-managed-system   Active   12h
gmp-public           Active   12h
gmp-system           Active   12h
istio-system         Active   9m58s
kube-node-lease      Active   12h
kube-public          Active   12h
kube-system          Active   12h
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

```sh
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ kubectl get deploy -A
NAMESPACE      NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
asm-gateways   istio-ingressgateway                   3/3     3            3           7m26s
asm-system     canonical-service-controller-manager   1/1     1            1           7m49s
gmp-system     gmp-operator                           1/1     1            1           12h
gmp-system     rule-evaluator                         1/1     1            1           12h
istio-system   istiod-asm-1178-20                     2/2     2            2           8m20s
kube-system    event-exporter-gke                     1/1     1            1           12h
kube-system    konnectivity-agent                     3/3     3            3           12h
kube-system    konnectivity-agent-autoscaler          1/1     1            1           12h
kube-system    kube-dns                               2/2     2            2           12h
kube-system    kube-dns-autoscaler                    1/1     1            1           12h
kube-system    l7-default-backend                     1/1     1            1           12h
kube-system    metrics-server-v0.6.3                  1/1     1            1           12h
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

## Deploy the Online Boutique application across multiple clusters

In this task, you install the Online Boutique app to both clusters. Online Boutique consists of [10 microservices written in different languages](https://github.com/GoogleCloudPlatform/microservices-demo#service-architecture).

In this lab, you split the application across **west** and **east** clusters.

### Install Online Boutique services on the west cluster

1. Begin by creating a namespace for the Online Boutique services on the **west** cluster, then enable sidecar injection on that namespace:

   ```sh
   student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ # switch to the west context
   kubectx west
   
   # create the namespace
   kubectl create ns boutique
   
   # enable sidecar injection
   kubectl label ns boutique \
     istio.io/rev=$(kubectl -n istio-system get pods -l app=istiod -o json | jq -r '.items[0].metadata.labels["istio.io/rev"]') \
     --overwrite
   Switched to context "west".
   namespace/boutique created
   namespace/boutique labeled
   student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
   ```

   

```sh
student_03_b88b4332a813@cloudshell:~ (qwiklabs-gcp-03-f7c1f4b09b4a)$ git clone https://github.com/GoogleCloudPlatform/training-data-analyst
cd training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique/
Cloning into 'training-data-analyst'...
remote: Enumerating objects: 64765, done.
remote: Counting objects: 100% (224/224), done.
remote: Compressing objects: 100% (145/145), done.
remote: Total 64765 (delta 119), reused 158 (delta 77), pack-reused 64541
Receiving objects: 100% (64765/64765), 698.17 MiB | 14.22 MiB/s, done.
Resolving deltas: 100% (41341/41341), done.
Updating files: 100% (12864/12864), done.
student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

```sh
student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ kubectl apply -n boutique -f west
deployment.apps/frontend created
deployment.apps/recommendationservice created
deployment.apps/productcatalogservice created
deployment.apps/cartservice created
deployment.apps/loadgenerator created
deployment.apps/redis-cart created
gateway.networking.istio.io/frontend-gateway created
virtualservice.networking.istio.io/frontend-ingress created
serviceentry.networking.istio.io/allow-egress-googleapis created
serviceentry.networking.istio.io/allow-egress-google-metadata created
virtualservice.networking.istio.io/frontend created
service/emailservice created
service/checkoutservice created
service/recommendationservice created
service/frontend created
service/paymentservice created
service/productcatalogservice created
service/cartservice created
service/currencyservice created
service/shippingservice created
service/redis-cart created
service/adservice created
student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

### Install Online Boutique services on the east cluster

1. Switch to the **east** context, create a namespace for the Online Boutique services on the **east** cluster, then enable sidecar injection on that namespace:

   ```sh
   student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ # switch to the east context
   kubectx east
   
   # create the namespace
   kubectl create ns boutique
   
   # enable sidecar injection
   kubectl label ns boutique \
     istio.io/rev=$(kubectl -n istio-system get pods -l app=istiod -o json | jq -r '.items[0].metadata.labels["istio.io/rev"]') \
     --overwrite
   Switched to context "east".
   namespace/boutique created
   namespace/boutique labeled
   student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
   ```

   

```sh
student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ kubectl apply -n boutique -f east/deployments.yaml
kubectl apply -n boutique -f east/services.yaml
kubectl apply -n boutique -f east/istio-defaults.yaml
deployment.apps/emailservice created
deployment.apps/checkoutservice created
deployment.apps/shippingservice created
deployment.apps/paymentservice created
deployment.apps/adservice created
deployment.apps/currencyservice created
service/emailservice created
service/checkoutservice created
service/recommendationservice created
service/frontend created
service/paymentservice created
service/productcatalogservice created
service/cartservice created
service/currencyservice created
service/shippingservice created
service/redis-cart created
service/adservice created
gateway.networking.istio.io/frontend-gateway created
virtualservice.networking.istio.io/frontend-ingress created
serviceentry.networking.istio.io/allow-egress-googleapis created
serviceentry.networking.istio.io/allow-egress-google-metadata created
virtualservice.networking.istio.io/frontend created
student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
```

## Evaluate your multi-cluster application

1. Get a URL to access the `istio-ingressgateway` service on each cluster:

   ```sh
   student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ # get the
   student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ # get the IP address for the west cluster service
   kubectx west
   export WEST_GATEWAY_URL=http://$(kubectl get svc istio-ingressgateway \
   -o=jsonpath='{.status.loadBalancer.ingress[0].ip}' -n asm-gateways)
   
   # get the IP address for the east cluster service
   kubectx east
   export EAST_GATEWAY_URL=http://$(kubectl get svc istio-ingressgateway \
   -o=jsonpath='{.status.loadBalancer.ingress[0].ip}' -n asm-gateways)
   
   # compose and output the URLs for each service
   echo "The gateway address for west is $WEST_GATEWAY_URL
   The gateway address for east is $EAST_GATEWAY_URL
   "
   Switched to context "west".
   Switched to context "east".
   The gateway address for west is http://34.83.128.21
   The gateway address for east is http://35.196.196.16
   
   student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
   ```

   

The user connects to the **istio-ingressgateway** service, where   the proxies have been configured by both the Gateway and Virtual Service   configurations. The incoming request is parsed by the proxy, which then   connects to the frontend service which presents the store UI.   



 Each cluster has an **istio-ingressgateway** service and an   **istio-ingressgateway** deployment, so the user can connect to   either of the clusters. The **frontend** service and workload,   though, are only running on the **west** cluster. So when you   connect to the **east** cluster gateway, the proxy will receive   your inbound request, but will connect through the mesh to the frontend   service running on the **west** cluster. We'll see this traffic   flow in more detail shortly.



## Distribute one service across both clusters

1. In the console, go to **Navigation menu > Kubernetes Engine > Services & Ingress**.

2. Look at the recommendationservice entries in the table. Each cluster has a recommendationservice entry, but while the **west** cluster has 1 pod behind the service, the **east** cluster has 0 pods.

   The actual workload is deployed on the **west** cluster and not on the **east** cluster. You're going to deploy the workload to the **east** cluster as well.

   ```sh
   student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ kubectl apply -n boutique -f east/rec-east.yaml
   deployment.apps/recommendationservice created
   student_03_b88b4332a813@cloudshell:~/training-data-analyst/courses/ahybrid/v1.0/AHYBRID081/boutique (qwiklabs-gcp-03-f7c1f4b09b4a)$ 
   ```

   

In this lab, you deployed Anthos Service Mesh to 2 clusters, configured  the two clusters to participate in a single mesh, deployed an  application with services split across both clusters, and made one  service a distributed service running on both clusters.
